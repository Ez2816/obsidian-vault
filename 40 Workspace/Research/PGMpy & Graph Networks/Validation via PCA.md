
## The Challenge: Closing the Discretization Loop

Discretize continuous expression to 0/1/2 for CPD learning. Simulate and get back discrete values. But biology happens in continuous space. How do you validate fairly?

**Naive approach**: Compare PCA on discrete bins (wrong—PCA assumes continuous)
**Your approach**: Recover continuous values, then compare

---

## The Solution: Bin Recovery + PCA Comparison

### Step 1: Store Original Values During Discretization

```python
GENE_BIN_VALUES = {}
for gene in GENES:
    GENE_BIN_VALUES[gene] = {
        0: raw_data[gene][discretized[gene] == 0].tolist(),
        1: raw_data[gene][discretized[gene] == 1].tolist(),
        2: raw_data[gene][discretized[gene] == 2].tolist(),
    }
```

Now you can reverse discretization: bin 0 → random sample from learned_bin_0.

### Step 2: Post-Simulation Bin Recovery

After simulation, replace bin labels with random samples from learned values:

```python
sim_continuous = simulated_binned.copy()
for gene in GENES:
    for bin_idx in range(3):
        mask = simulated_binned[gene] == bin_idx
        if mask.sum() > 0:
            sim_continuous.loc[mask, gene] = np.random.choice(
                GENE_BIN_VALUES[gene][bin_idx], size=mask.sum()
            )
```

Result: simulated data in continuous space (same space as raw data).

### Step 3: Fit PCA on Raw Data

```python
raw_t0_continuous = bin_recovery(raw_t0, GENE_BIN_VALUES)
raw_t1_continuous = bin_recovery(raw_t1, GENE_BIN_VALUES)

pca_t0 = PCA(n_components=11)
pca_t0.fit(raw_t0_continuous)

raw_ev_t0 = pca_t0.explained_variance_ratio_
# Example: [0.151, 0.142, 0.111, 0.107, 0.096, ...]
```

### Step 4: Transform Simulated Data

```python
sim_t0_continuous = bin_recovery(sim_t0, GENE_BIN_VALUES)

# CRITICAL: Do NOT refit PCA, use fitted pca_t0
sim_t0_transformed = pca_t0.transform(sim_t0_continuous)

sim_ev_t0 = np.var(sim_t0_transformed, axis=0) / np.var(sim_t0_transformed).sum()
```

### Step 5: Compare Variance Profiles

```python
pc_diffs = np.abs(raw_ev_t0 - sim_ev_t0)
print("Max difference:", pc_diffs.max())
print("Mean difference:", pc_diffs.mean())
```

---

## Interpretation

### Success Indicators

```
✓ Good fit:
  - Max variance difference per PC < 5%
  - Early PCs (1-3) match within 3%
  
✗ Problem signals:
  - Large differences in PC 1-2 (missing edges)
  - Large differences in PC 3-5 (missing secondary relationships)
```

### Example Result

```
Raw:       [0.151, 0.142, 0.111, 0.107, 0.096, ...]
Simulated: [0.148, 0.139, 0.108, 0.105, 0.094, ...]
Diff:      [0.003, 0.003, 0.003, 0.002, 0.002, ...]

→ Excellent fit! CPDs captured variance structure well.
```

---

## Why This Validation Matters

**Closes the discretization loop**: Discretize → Learn → Simulate → Recover → Validate in continuous space

**Tests**:
- CPD accuracy (do learned CPDs capture variance?)
- Edge quality (missing edges show as variance mismatches)
- Discretization impact (wrong bins won't recover)
- Sampling quality (genealogy-aware vs. i.i.d.)

---

## Key Decisions

### Data Leakage Prevention
```python
# WRONG: Fit PCA on simulated data
pca_wrong.fit(sim_data)  # Biased!

# RIGHT: Fit on raw data, transform simulated
pca_right.fit(raw_data)
sim_transformed = pca_right.transform(sim_data)  # Fair!
```

### Continuous Recovery vs. Discrete Comparison
```python
# WRONG: Compare PCA on discrete 0/1/2
pca_discrete.fit(raw_binned)  # 0/1/2 are categorical!

# RIGHT: Recover continuous first
raw_continuous = bin_recovery(raw_binned, GENE_BIN_VALUES)
pca_continuous.fit(raw_continuous)
```

### Success Criterion (TBD)

What counts as "good fit"?
- 5% variance difference per PC?
- Early PCs within 3%?
- Cumulative variance within 2% by PC 5?

Empirically, decide after observing distributions.

---

## Experimental Variations

### Test 1: Genealogy-Aware vs. I.I.D. Sampling

Compare PCA validation for genealogy-aware vs. standard sampling. Hypothesis: genealogy-aware should better match raw variance.

### Test 2: Bin Count Impact

Sweep 2, 3, 4 bins per gene. Measure PCA difference for each. Select optimal granularity.

### Test 3: Edge Refinement

Add 0, 10, 20 extra edges. Refit and validate. Plot: does PCA improve with more edges?

---

## Experiment Log: Friday Multi-Timestep Validation

### Setup (Observed Problem)
- **Date**: Friday (2026-07-19)
- **Approach**: Multi-step genealogy-aware simulation with 700 parent cells, 9 timepoints
- **Sampling strategy**: Primary daughter (100%) + secondary daughter (1% chance) = ~6,366 samples
- **Method**: `simulate_n_steps_from_parents()` with cascade mechanism
- **Outcome**: **PCA results did NOT converge** ❌

### Problem Identified
**Error Propagation Through Timesteps**:
- Error accumulates across the 9 timepoint cascade
- Suspected culprit: transition from t=0 to t=1 introduces systematic drift
- By final timepoints, divergence becomes significant
- PCA validation fails: simulated variance structure no longer matches raw

### Convergency Check (Positive Finding) ✓
**Validates dataset/PCA pipeline is sound**:
- Sampled original raw data independently on 2 different seeds
- Ran PCA separately on both samples
- Results **CONVERGED** → Dataset itself is sound
- Implication: The problem is NOT the PCA methodology or the data quality—it's the simulation cascade

### Current Testing Approach (In Progress)

**Strategy**: Single-timestep learning and validation
1. Sample daughters with 50/50 weighting (1st daughter vs. 2nd daughter)
2. Learn CPDs from **only t=0 → t=1 transition** (single step)
3. Simulate and validate PCA on single transition
4. If single-step validation passes → cascade across full 9 timepoints with separate learning/simulation per step

**Rationale**: 
- Isolates error source: does error come from cascade mechanism or CPD learning?
- Single-timestep learning avoids error propagation altogether
- If each step validates independently, can we use step-wise approach instead of full cascade?

### Next Steps

1. Complete single-timestep validation (PCA comparison at t=0→t=1)
2. If passes: implement per-timestep learning and simulation for all 9 steps
3. Compare results: full-cascade vs. per-step approach
4. Decision point: Is cascade mechanism viable, or must we use independent timesteps?

---

## Wiki Reference

The reusable methodology from this workspace has been extracted to the Wiki:
- [[30 Wiki/Syntheses/Discrete-to-Continuous Validation]] — General validation methodology bridging discrete and continuous spaces

**Note**: This Workspace file is the project-specific working version. The Wiki page contains the generalizable knowledge.

## See Also
- [[Discretization]] — Computing bin thresholds and bin recovery
- [[Model Fitting & Selection]] — CPD estimation methods
- [[Simulation with Parents]] — Genealogy-aware vs. i.i.d. sampling
