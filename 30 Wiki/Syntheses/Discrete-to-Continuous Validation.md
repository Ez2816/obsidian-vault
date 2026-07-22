# Discrete-to-Continuous Validation

## The Bridge Problem

You discretize continuous data to 0/1/2 for Bayesian network learning (CPDs require discrete states). Then you simulate and get back discrete values. But biology happens in continuous space. **How do you validate fairly?**

**Naive approach** (wrong):
```python
# WRONG: Compare PCA on discrete 0/1/2
pca.fit(raw_binned)  # Bins are categorical, PCA assumes continuous
pca.transform(simulated_binned)
# Problem: Spurious results because discretization breaks geometry
```

**Correct approach**:
> Close the discretization loop. Recover continuous values, then validate in continuous space.

---

## The Solution: Bin Recovery

### Step 1: Store Original Values Per Bin

During initial discretization, preserve which raw values went into each bin:

```python
BIN_VALUES = {}
for gene in GENES:
    BIN_VALUES[gene] = {
        0: raw_data[gene][discretized[gene] == 0].tolist(),
        1: raw_data[gene][discretized[gene] == 1].tolist(),
        2: raw_data[gene][discretized[gene] == 2].tolist(),
    }

# Example:
# BIN_VALUES['Smad2'] = {
#     0: [12.3, 45.1, 23.4, ...],  # ~100k values (bottom 33%)
#     1: [2345.0, 2567.2, ...],    # ~100k values (middle 33%)
#     2: [8901.2, 9234.5, ...],    # ~100k values (top 33%)
# }
```

### Step 2: Recover After Simulation

```python
sim_continuous = simulated_binned.copy()
for gene in GENES:
    for bin_idx in range(3):
        mask = simulated_binned[gene] == bin_idx
        if mask.sum() > 0:
            sim_continuous.loc[mask, gene] = np.random.choice(
                BIN_VALUES[gene][bin_idx],
                size=mask.sum()
            )
```

Now simulated data is "pseudo-continuous" — values come from learned bin distributions.

### Step 3: Fit PCA on Raw Data

```python
# Extract and recover continuous values
raw_t0_continuous = bin_recovery(raw_t0, BIN_VALUES)
pca_t0 = PCA(n_components=11)
pca_t0.fit(raw_t0_continuous)

raw_ev_t0 = pca_t0.explained_variance_ratio_
# Result: [0.151, 0.142, 0.111, 0.107, 0.096, ...]
```

### Step 4: Transform Simulated Data

```python
# CRITICAL: Do NOT refit PCA
sim_t0_continuous = bin_recovery(sim_t0, BIN_VALUES)
sim_t0_transformed = pca_t0.transform(sim_t0_continuous)  # Use fitted PCA

sim_ev_t0 = np.var(sim_t0_transformed, axis=0) / np.var(sim_t0_transformed).sum()
```

### Step 5: Compare Variance Profiles

```python
pc_diffs = np.abs(raw_ev_t0 - sim_ev_t0)
print("Per-PC differences:", pc_diffs)
print("Max difference:", pc_diffs.max())

# Success: All PCs within 5% variance difference
```

---

## Why This Works

**Problem solved**:
1. **Discretization is necessary** for CPD learning
2. **Validation must happen in continuous space** (where biology occurs)
3. **Bin recovery bridges both worlds**: Learn in discrete, validate in continuous

**Data flow**:
```
Continuous raw → Discretize (necessary for learning) → Learn CPDs
                                                    ↓
                                        Simulate discrete → Recover continuous
                                                            ↓
                                            Validate via PCA (fair comparison)
```

---

## Success Criteria

### Per-PC Variance Difference

```
✓ Good fit:
  - Max variance difference per PC < 5%
  - Early PCs (1-3) match within 3%
  - Cumulative variance similar by PC 5

✗ Problem signals:
  - Large differences in PC 1-2 (missing major regulatory edges)
  - Large differences in PC 3-5 (missing secondary relationships)
  - Large differences in PC 6+ (often less critical, could be noise)
```

### Example Result

```
Raw explained variance:      [0.151, 0.142, 0.111, 0.107, 0.096, ...]
Simulated explained variance: [0.148, 0.139, 0.108, 0.105, 0.094, ...]
Difference:                   [0.003, 0.003, 0.003, 0.002, 0.002, ...]

→ Excellent fit! CPDs captured variance structure.
```

---

## What This Validates

1. **CPD Accuracy**: Do learned CPDs capture the variance structure of real data?
2. **Edge Quality**: Missing edges show up as variance mismatches
3. **Discretization Impact**: If bins are wrong, variance won't recover
4. **Sampling Quality**: Genealogy-aware vs. i.i.d. sampling (genealogy should perform better)

---

## Critical Data Leakage Prevention

```python
# WRONG: Fit PCA on simulated data
pca_wrong = PCA()
pca_wrong.fit(sim_data)  # Biased!

# RIGHT: Fit on raw data, transform simulated
pca_right = PCA()
pca_right.fit(raw_data)
sim_transformed = pca_right.transform(sim_data)  # Fair!
```

---

## Experimental Variations

### Test 1: Genealogy-Aware vs. I.I.D. Sampling

Does genealogy-aware simulation improve validation?

```python
# I.I.D.: standard forward sampling
iid_samples = model.simulate(n_samples=6300, seed=42)
iid_continuous = bin_recovery(iid_samples, BIN_VALUES)
iid_ev = compute_variance_ratio(pca.transform(iid_continuous))

# Genealogy-aware: parent-constrained simulation
genealogy_samples = model.simulate_n_steps_from_parents(...)
genealogy_continuous = bin_recovery(genealogy_samples, BIN_VALUES)
genealogy_ev = compute_variance_ratio(pca.transform(genealogy_continuous))

# Compare: which better matches raw?
```

**Important**: Genealogy-aware multi-step simulation can exhibit **error propagation** if transitions are not well-modeled. See [[Error Propagation in Multi-Step Simulation]] for mitigation strategies.

### Test 2: Bin Count Impact

Does 2-state, 3-state, or 4-state binning work best?

```python
for n_bins in [2, 3, 4]:
    # Discretize with n_bins
    binned = digitize_with_n_bins(raw_data, n_bins)
    
    # Fit model, simulate, recover, validate
    model = fit_model(binned)
    sim = model.simulate_n_steps_from_parents(...)
    sim_continuous = bin_recovery(sim, BIN_VALUES_nbins)
    
    pca_diff = compute_max_variance_diff(raw_pca, sim_continuous)
    print(f"{n_bins} bins: max diff = {pca_diff:.4f}")

# Select n_bins with best fit
```

### Test 3: Edge Refinement

Does adding edges improve validation?

```python
for num_extra_edges in [0, 10, 20]:
    model_variant = add_random_edges(model, num_extra_edges)
    model_variant.fit(...)
    sim = model_variant.simulate_n_steps_from_parents(...)
    pca_diff = compute_pca_diff(raw_pca, sim)
    
    print(f"+{num_extra_edges} edges: PCA diff = {pca_diff:.4f}")

# Plot: does PCA improve with more edges? (should plateau)
```

---

## Related

- [[Discretization Strategies]] — Computing bins and storing original values
- [[Bayesian CPD Estimation]] — Learning CPDs from discretized data
- [[Genealogy-Aware Sampling]] — Parent-constrained sampling
- [[Multi-Step Lineage Simulation]] — Genealogical simulation

---

**Source**: [[40 Workspace/Research/PGMpy & Graph Networks/Validation via PCA]]

**Key insight**: Discretization is not a loss if you preserve it reversibly. This enables learning in discrete space and validating in continuous space.
