

## Standard pgmpy Sampling vs. Your Genealogy-Aware Extension

### The Problem with Standard Sampling

pgmpy's `forward_sample()` generates i.i.d. samples—each sample is independent, parent states sampled from priors P(t0), with no genealogical structure or connection to cell division biology.

**Limitation**: You can't trace lineages. Samples are all equally random draws from the distribution.

---

## Your Solution: Parent-Aware Forward Sampling

**Innovation**: `defined_parents_forward_sample(parent_state={...}, size=N)`

```python
def defined_parents_forward_sample(
    size=2,  # Number of daughters per parent
    parent_state={"Smad2_t0": 1, "Nanog_t0": 2, ...}  # Predefined parent
):
    """
    Sample daughters where:
    - All daughters inherit identical parent t0 state
    - Each daughter's t1 state sampled independently from CPDs
    """
    sampled = pd.DataFrame()
    
    for node in self.topological_order:
        if node.endswith("_t0"):
            sampled[node] = np.full(size, parent_state[node])
        else:  # node.endswith("_t1")
            evidence = cpd.variables[1:]
            parent_values = tuple(sampled[p][0] for p in evidence)
            weights = cpd.reduce(parent_values).values
            sampled[node] = np.random.choice(
                range(len(weights)), size=size, p=weights/weights.sum()
            )
    
    return sampled  # Shape: (size, 22)
```

**Key idea**: Parent state is **predefined by the caller**, not sampled from priors.

---

## Example: What This Enables

```python
parent_state = {
    "Smad2_t0": 1, "Nanog_t0": 2, "Sox2_t0": 0, # ... 11 genes
}

daughters = BayesianModelSampling(model).defined_parents_forward_sample(
    size=2, parent_state=parent_state
)

# Result:
#   Smad2_t0  Nanog_t0  Sox2_t0  ...  Smad2_t1  Nanog_t1  Sox2_t1
# 0    1         2        0      ...    2        1         0
# 1    1         2        0      ...    0        2         1
```

Both daughters: same t0 (inherited). Different t1 (stochastic variation per daughter).

---

## How It Differs from Standard `forward_sample()`

| Aspect | Standard | Your Extension |
|--------|----------|---|
| **Parent t0 state** | Sampled from prior P(t0) | Predefined by caller |
| **Daughters per parent** | N/A | Exactly `size` daughters |
| **Lineage tracking** | None | Yes |
| **Use case** | Random samples | Trace genealogies |

---

## Performance Optimization: CPD Pre-Computation

When sampling multiple daughters from same parent, slice CPD once and reuse (10-50× faster).

---

## Why Topological Order Matters

Lexicographical topological sort ensures all `_t0` nodes processed before `_t1`, deterministically:
```
Acvr1_t0, Acvr1_t1, Cdx2_t0, Cdx2_t1, ..., Wnt3_t0, Wnt3_t1
```

Same seed + deterministic order = reproducible genealogies across runs.

---

## When to Use

**Use when**: Starting with specific cells, tracing lineages matters, validating CPDs under biological constraints

**Don't use when**: Random i.i.d. samples suffice, no prior knowledge of parent states

---

## Experiment Log: Convergency Check & Daughter Weighting

### Convergency Validation (Positive Result) ✓
**Test**: Original dataset quality and PCA pipeline soundness
- Sampled raw data twice independently on 2 different seeds
- Ran PCA separately on each sample
- **Result**: PCA profiles **converged** across both samples
- **Implication**: Dataset is sound; problem is not data quality or PCA methodology

### Daughter Cell Weighting Strategy

**Original approach** (biased, failed):
- 100% first daughter + 1% second daughter
- Results in ~6,366 samples but extreme bias toward primogeniture
- May miss stochastic variation captured by second daughters

**Current testing** (balanced):
- 50% first daughter, 50% second daughter at each step
- More representative of stochastic variation
- Under validation with single-timestep learning/simulation

---

## Wiki Reference

The reusable methodology from this workspace has been extracted to the Wiki:
- [[30 Wiki/Concepts/Genealogy-Aware Sampling]] — General genealogy-aware sampling techniques

**Note**: This Workspace file is the project-specific working version. The Wiki page contains the generalizable knowledge.

## See Also
- [[Simulation with Parents]] — Multi-step genealogical tracing
- [[Validation via PCA]] — Testing genealogy-aware vs. standard sampling
