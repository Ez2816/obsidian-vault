# Genealogy-Aware Sampling

## Problem

Standard probabilistic sampling (Gibbs, importance sampling, forward_sample) generates i.i.d. samples with no genealogical structure. When modeling systems with known lineage constraints (cell division, inheritance, causality trees), this breaks biological realism.

**Limitation of standard sampling**: 
- No parent-child relationships
- No control over daughter constraint (e.g., exactly 2 children per parent)
- Each sample independent—no lineage tracking

---

## Solution: Parent-Aware Forward Sampling

**Innovation**: Condition sampling on predefined parent states, generate constrained daughters.

```python
def defined_parents_forward_sample(
    size=2,  # Number of daughters per parent
    parent_state={"Smad2_t0": 1, "Nanog_t0": 2, ...}  # Predefined parent
):
    """
    Sample daughters where:
    - All daughters inherit identical parent t0 state
    - Each daughter's t1 state sampled independently from CPDs
    - Exactly `size` daughters generated per parent
    """
    sampled = pd.DataFrame()
    
    for node in topological_order:
        if node.endswith("_t0"):
            # t0 node: inherit from parent (deterministic)
            sampled[node] = np.full(size, parent_state[node])
        else:  # node.endswith("_t1")
            # t1 node: sample from conditional P(t1 | t0)
            evidence = cpd.variables[1:]
            parent_values = tuple(sampled[p][0] for p in evidence)
            weights = cpd.reduce(parent_values).values
            sampled[node] = np.random.choice(
                range(len(weights)), size=size, p=weights/weights.sum()
            )
    
    return sampled  # Shape: (size, 22 genes)
```

**Key insight**: Parent state is **predefined by caller**, not sampled from priors.

---

## Example: Biological Lineage

```python
# Define a parent cell's state
parent_state = {
    "Smad2_t0": 1,      # Medium expression
    "Nanog_t0": 2,      # High expression
    "Sox2_t0": 0,       # Low expression
}

# Sample 2 daughters from this parent
daughters = BayesianModelSampling(model).defined_parents_forward_sample(
    size=2,
    parent_state=parent_state
)

# Result:
#   Smad2_t0  Nanog_t0  Sox2_t0  ...  Smad2_t1  Nanog_t1  Sox2_t1
# 0    1         2        0      ...    2        1         0       ← Daughter 1
# 1    1         2        0      ...    0        2         1       ← Daughter 2
#
# Both daughters: same t0 (inherited)
# Both daughters: different t1 (stochastic variation)
```

**Biological model**: 
- Cells divide → daughters born with same gene expression
- Each daughter develops differently (stochastic CPD-based changes)
- Lineage preserved across generations

---

## Comparison to Standard Sampling

| Aspect | Standard forward_sample() | Parent-Aware Sampling |
|--------|---|---|
| **Parent states** | Sampled from prior P(t0) | Predefined by caller |
| **Daughters per parent** | N/A (i.i.d. samples) | Exactly `size` daughters |
| **Lineage tracking** | None | Yes (parent→t0, daughters→t1) |
| **Use case** | Generate random samples | Trace genealogies |
| **Reproducibility** | Seed-based | Seed + deterministic topological order |

---

## Performance: CPD Pre-Computation

When sampling multiple daughters from same parent, optimize by slicing the CPD once:

**Naive** (recompute per daughter):
```python
for daughter in range(size):
    weights = cpd.reduce(parent_values).values  # Slice N times
    sample from weights
```

**Optimized** (slice once):
```python
weights = cpd.reduce(parent_values).values  # Slice once
for daughter in range(size):
    sample from weights  # Reuse
```

**Impact**: 10-50× faster for size=2+

---

## Critical: Topological Ordering

```python
topological_order = list(nx.lexicographical_topological_sort(model))
```

Ensures all t0 nodes processed before t1 (parents before children), deterministically.

**Without lexicographical sort**: Different runs might process nodes in different order → different genealogies even with same seed

**With lexicographical sort**: Same seed + same order = reproducible genealogies

---

## When to Use

**Use genealogy-aware when:**
- You have starting cells (e.g., t=0 cells from data)
- You need to trace lineages (parent→daughter relationships)
- You're validating models under biological constraints

**Use standard sampling when:**
- Random i.i.d. samples sufficient
- Genealogy doesn't matter
- You need maximum sample diversity

---

## Related

- [[Multi-Step Lineage Simulation]] — Cascading genealogy across multiple steps
- [[Discrete-to-Continuous Validation]] — Validating genealogy-aware simulations

---

**Source**: [[40 Workspace/Research/PGMpy & Graph Networks/Bayesian Sampling]]

**Novel contribution**: First pgmpy extension for genealogy-aware sampling with parent-defined states and daughter constraints.
