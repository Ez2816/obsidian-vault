

## The Challenge

Raw expression data is continuous (~1.1M values across 10 timepoints, 70-100k cells per timepoint). Bayesian networks require discrete states. How do you bin continuous values fairly and consistently?

**The naive approach**: Compute percentile thresholds per timepoint separately
- Problem: State 0/1/2 means different things at t=0 vs. t=5
- Result: Inconsistent representation, harder to learn CPDs

---

## The Solution: Global Percentile Binning

Compute percentile thresholds **once, on all pooled data**:

```python
# Pool all timepoints together
all_data = pd.concat([DATA_SETS[t] for t in range(10)])

# Compute 33rd and 66th percentiles
thresholds = {
    gene: np.percentile(all_data[gene], [33, 66])
    for gene in GENES
}
# Result: GENE_BINS = {'Smad2': [2583.0, 5291.0], 'Nanog': [...], ...}

# Apply consistently to all data
discretized = all_data.copy()
for gene in GENES:
    discretized[gene] = np.digitize(
        all_data[gene], 
        bins=GENE_BINS[gene]
    )
# Result: Values 0, 1, 2 (low, medium, high)
```

**Why this works:**
- State 0 = low expression (bottom 33%)
- State 1 = medium expression (33-66%)
- State 2 = high expression (top 33%)
- Same definition everywhere → CPDs learn consistent patterns

---

## Critical Insight: Store Original Values Per Bin

This is the key to later validation:

```python
# During discretization, save which original values went into each bin
GENE_BIN_VALUES = {}

for gene in GENES:
    GENE_BIN_VALUES[gene] = {
        0: all_data[gene][discretized[gene] == 0].tolist(),
        1: all_data[gene][discretized[gene] == 1].tolist(),
        2: all_data[gene][discretized[gene] == 2].tolist(),
    }
```

**Later, when you simulate and get back discrete 0/1/2:**
```python
# Replace bin labels with random samples from learned values
sim_continuous = simulated_binned.copy()
for gene in GENES:
    for bin_idx in range(3):
        mask = simulated_binned[gene] == bin_idx
        if mask.sum() > 0:
            sim_continuous.loc[mask, gene] = np.random.choice(
                GENE_BIN_VALUES[gene][bin_idx],
                size=mask.sum()
            )
```

**Impact**: You can now validate discrete CPDs in continuous space (PCA comparison becomes fair).

---

## Bin Count Selection: 3 States vs. Alternatives

Current approach: 3 bins (33rd, 66th percentiles)

**Why 3?**
- Captures low/medium/high expression
- Matches biological intuition (gene "off", "low", "high")
- Sufficient granularity for 1.1M training values

**To explore**: Adaptive binning per gene (2, 3, or 4 states)

```python
# Sweep different bin counts
for n_bins in [2, 3, 4]:
    thresholds = np.percentile(data, np.linspace(0, 100, n_bins+1))
    binned = np.digitize(data, bins=thresholds[1:-1])
    
    model = fit_model_with_binning(binned, n_bins)
    score = bic_scorer.score(model)
    
    print(f"{n_bins} bins: BIC={score}")
    
# Select n_bins with best BIC (balance fit vs. complexity)
```

---

## Gotchas Discovered

### 1. Empty Bins at Data Boundaries
If percentile thresholds are computed on nonzero values only, the first and last bins get zero counts.

**Fix**: Compute thresholds on all values, then digitize
```python
thresholds = np.percentile(data, [33, 66])  # Don't filter first
binned = np.digitize(data, bins=thresholds)  # Then digitize everything
```

### 2. State Cardinality Mismatch
If your CPDs expect states [0, 1, 2] but training data only has [0, 1], pgmpy fails silently.

**Fix**: Pre-define STATE_NAMES before fitting
```python
STATE_NAMES = {node: [0, 1, 2] for node in model.nodes()}
estimator.fit(model, data, state_names=STATE_NAMES)
```

### 3. Bin Count Inconsistency
Don't mix 2-state and 3-state binning within the same model.

**Fix**: Commit to a strategy per model run, sweep systematically.

---

## Why This Matters for CPD Learning

Discretization is **not** a loss—it's a **lens**. 

- Without discretization: Can't fit Bayesian networks (need categorical)
- With global discretization: CPDs learn consistent state transitions
- With bin recovery: Can validate in continuous space (PCA)

The three-step flow (discretize → learn → recover → validate) bridges two spaces:
- **Learning space**: Discrete (necessary for Bayesian networks)
- **Biological space**: Continuous (where real biology happens)

---

## Next Steps

1. Current sweep: Compare BIC for 2, 3, 4 bins across all genes
2. Adaptive binning: Select optimal # states per gene independently
3. Validation: Compare PCA variance profiles for each binning strategy
4. Publish threshold tables for reproducibility

---

## Wiki Reference

The reusable methodology from this workspace has been extracted to the Wiki:
- [[30 Wiki/Concepts/Discretization Strategies]] — General discretization techniques and bin recovery

**Note**: This Workspace file is the project-specific working version. The Wiki page contains the generalizable knowledge.
