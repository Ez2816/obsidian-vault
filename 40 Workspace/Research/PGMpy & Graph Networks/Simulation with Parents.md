
## The Challenge: Multi-Step Genealogical Tracing

Simulate cell development across 9 timepoints (t=0 through t=9) as **traced lineages**, not random i.i.d. samples. Each step's outcome cascades into the next step's parent state.

---

## The Solution: Multi-Step Genealogical Simulation

**Method**: `simulate_n_steps_from_parents(parent_samples, steps=9, daughters=1, evidence=True)`

```python
def simulate_n_steps_from_parents(
    parent_samples: DataFrame,  # P parent cells × 11 genes
    steps: int = 9,             # Time steps to propagate
    daughters: int = 1,
    evidence: bool = True       # Use genealogy-aware sampling?
):
    final_df = pd.DataFrame()
    count = 0
    
    for parent_cell_idx in range(parent_samples.shape[0]):
        t0_states = {
            f"{gene}_t0": parent_samples[gene].iloc[parent_cell_idx]
            for gene in parent_samples.columns
        }
        
        for step_t in range(steps):
            daughters_sample = BayesianModelSampling(model).defined_parents_forward_sample(
                size=daughters, parent_state=t0_states
            )
            
            final_df.loc[count] = daughters_sample.loc[0]
            
            # KEY CASCADE: daughter's t1 becomes new t0
            t0_states = {
                f"{gene}_t0": daughters_sample[f"{gene}_t1"].iloc[0]
                for gene in parent_samples.columns
            }
            count += 1
    
    return final_df  # Shape: (parent_samples.shape[0] × steps, 22)
```

---

## Example Usage

```python
parent_cells = pd.DataFrame({
    'Smad2': [0, 1, 2, ...],  # 700 cells
    'Nanog': [1, 2, 0, ...],
    # ... 11 genes
})

simulated = model.simulate_n_steps_from_parents(
    parent_samples=parent_cells,
    steps=9,           # t=0→1, 1→2, ..., 8→9
    daughters=1,       # 1 daughter per parent per step
    evidence=True,     # Genealogy-aware sampling
    seed=42
)

# Result: Shape (6300, 22)
# 6300 = 700 parents × 9 steps
# 22 = 11 genes × (t0 + t1)
```

---

## Visual Lineage Trace

```
Parent Cell p: Smad2=1, Nanog=2, Sox2=0, ...
    │
    ├─ Step t=0: Sample daughters, store, CASCADE to t1
    ├─ Step t=1: Use previous t1 as new parent, sample, CASCADE
    ├─ Step t=2...8: Continue cascading
    │
Next Parent Cell p+1: Reset and repeat
```

Each row is one cell's state at one timepoint. Rows 0-8 = parent 0's lineage. Rows 9-17 = parent 1's lineage. No cross-contamination.

---

## The Critical Cascade Mechanism

```python
t0_states = {
    f"{gene}_t0": daughters_sample[f"{gene}_t1"].iloc[0]
    for gene in parent_samples.columns
}
```

**Why it matters**: Without cascade, each step would sample from priors (breaking genealogy). With cascade, each step's outcome influences the next (biological realism).

---

## Performance

**Typical runtime**: 700 parents × 9 steps × 1 daughter ≈ 6,300 samples in 30-60 seconds on CPU

---

## When to Use

**Use genealogy-aware when**: Starting cells matter, development/lineage matters, validating CPDs under realistic constraints

**Use standard i.i.d. when**: Random samples from distribution sufficient, genealogy irrelevant

---

## Optional: Multiple Daughters

```python
simulated = model.simulate_n_steps_from_parents(
    parent_samples=parent_cells,
    steps=9,
    daughters=2,  # Generate 2, take first for primary lineage
    evidence=True,
    seed=42
)
```

Generates 2 daughters per step but only propagates primary (index 0) to continue lineage.

---

## Experiment Log: Daughter Sampling Strategy

### Original Approach (Observed Error)
- **Sampling**: 100% first daughter + 1% chance for second daughter
- **Sample size**: ~6,366 samples (700 parents × 9 steps)
- **Result**: **PCA did NOT converge**—error propagates across timesteps
- **Problem**: Extreme bias toward first daughter may not capture full CPD variance

### Current Testing: 50/50 Balanced Weighting
- **Strategy**: 50% chance to take 1st daughter, 50% chance to take 2nd daughter at each step
- **Rationale**: 
  - Reduces sampling bias
  - Captures stochastic variation better
  - May reduce error accumulation per step
- **Status**: Single-timestep validation in progress (t=0→t=1 only)
- **Next**: If single-step passes, extend to full 9-timestep cascade

---

## Wiki Reference

The reusable methodology from this workspace has been extracted to the Wiki:
- [[30 Wiki/Concepts/Multi-Step Lineage Simulation]] — General multi-step genealogical simulation techniques

**Note**: This Workspace file is the project-specific working version. The Wiki page contains the generalizable knowledge.

## See Also
- [[Bayesian Sampling]] — Parent-aware forward sampling (building block)
- [[Validation via PCA]] — Testing genealogy-aware simulation
