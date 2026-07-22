# Multi-Step Lineage Simulation

## Problem

Simulating biological systems across multiple developmental timepoints requires maintaining genealogical continuity: parent cell at t=n → daughters at t=n+1 → each daughter becomes parent for t=n+2, etc. Standard sampling breaks this connection.

---

## Solution: Cascade Mechanism

Propagate forward through time, automatically using each step's offspring as the next step's parents:

```python
def simulate_n_steps_from_parents(
    parent_samples: DataFrame,  # P parent cells × G genes
    steps: int = 9,             # Time steps to propagate
    daughters: int = 1,         # Daughters per parent per step
    evidence: bool = True       # Use genealogy-aware sampling?
):
    final_df = pd.DataFrame()
    count = 0
    
    for parent_cell_idx in range(parent_samples.shape[0]):
        # Extract parent's current state
        t0_states = {
            f"{gene}_t0": parent_samples[gene].iloc[parent_cell_idx]
            for gene in parent_samples.columns
        }
        
        # Propagate through time steps
        for step_t in range(steps):
            # Sample daughters with this parent state
            daughters_sample = BayesianModelSampling(model).defined_parents_forward_sample(
                size=daughters, parent_state=t0_states
            )
            
            # Store primary daughter
            final_df.loc[count] = daughters_sample.loc[0]
            
            # **CASCADE**: Promote daughter to next parent
            # daughter's t1 becomes new t0
            t0_states = {
                f"{gene}_t0": daughters_sample[f"{gene}_t1"].iloc[0]
                for gene in parent_samples.columns
            }
            count += 1
    
    return final_df  # Shape: (num_parents × steps, genes)
```

**The cascade is the whole point**: Without it, each step is independent. With it, outcomes cascade forward.

---

## Visual Lineage Trace

```
Parent Cell p: Smad2=1, Nanog=2, Sox2=0
    │
    ├─ Step t=0
    │   ├─ t0_states = {Smad2_t0: 1, Nanog_t0: 2, Sox2_t0: 0, ...}
    │   ├─ Sample daughters
    │   ├─ Primary daughter: [Smad2_t1: 2, Nanog_t1: ?, Sox2_t1: ?, ...]
    │   ├─ Store in final_df[0]
    │   └─ CASCADE: t0_states = daughter's t1 values
    │
    ├─ Step t=1
    │   ├─ t0_states = {Smad2_t0: 2, ...}  ← From previous step's t1
    │   ├─ Sample new daughters
    │   └─ Store in final_df[1]
    │
    ├─ Steps t=2...8 (continue cascading)
    │
    └─ Step t=8 (final generation)

Next Parent Cell p+1: Reset t0_states and repeat
```

---

## Output Structure

For 700 parent cells × 9 steps = 6,300 rows:

```
   Smad2_t0  Nanog_t0  Sox2_t0  ...  Smad2_t1  Nanog_t1  Sox2_t1
0    1         2        0      ...    2        1         1       ← Parent 0, t=0
1    2         1        2      ...    0        2         1       ← Parent 0, t=1 (cascaded)
2    0         2        1      ...    1        1         2       ← Parent 0, t=2
...
9    0         1        2      ...    1        2         1       ← Parent 1, t=0 (new parent)
```

**Key**: Rows 0-8 are one traced lineage. Rows 9-17 are a separate lineage. No cross-contamination.

---

## Multiple Daughters Per Step (Optional)

```python
simulated = model.simulate_n_steps_from_parents(
    parent_samples=parent_cells,
    steps=9,
    daughters=2,  # Generate 2 daughters per step
    evidence=True,
    seed=42
)
```

**Behavior**:
- Generate 2 daughters per parent per step
- Take primary daughter (index 0) to continue lineage
- Secondary daughter (index 1) discarded (or could trace separately)

**Use case**: Explore stochastic variation without branch explosion.

---

## Performance

| Operation | Complexity |
|-----------|-----------|
| One step per parent | O(daughters × #edges) |
| All parents, all steps | O(#parents × steps × daughters × #edges) |
| Topological sort (one-time) | O(#nodes + #edges), cached |

**Typical**: 700 parents × 9 steps × 1 daughter ≈ 6,300 samples in 30-60s on CPU

---

## When to Use

**Use multi-step lineage when:**
- You have starting cells (t=0 population)
- Development/growth matters
- You're validating models under genealogical constraints
- You need reproducible cell lineages

**Use standard simulation when:**
- Random i.i.d. samples sufficient
- Genealogy doesn't matter

---

## Related

- [[Genealogy-Aware Sampling]] — Single-step parent-constrained sampling (building block)
- [[Discrete-to-Continuous Validation]] — Validating lineage simulations via PCA

---

**Source**: [[40 Workspace/Research/PGMpy & Graph Networks/Simulation with Parents]]

**Novel contribution**: Multi-step genealogical simulation with automatic cascade mechanism (not standard pgmpy).
