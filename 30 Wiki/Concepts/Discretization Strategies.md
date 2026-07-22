# Discretization Strategies

## Problem Statement

When transforming continuous data into discrete states for statistical learning (e.g., Bayesian networks), how do you ensure consistent representation across time, samples, or datasets?

**Common Mistake**: Computing bin thresholds per timepoint/subset separately
- Result: State 0 means different things at t=0 vs. t=5
- Impact: Inconsistent CPD learning, biased model

---

## Solution: Global Percentile Binning

Compute percentile thresholds **once, on all pooled data**, then apply consistently:

```python
# Pool all data
all_data = pd.concat([data_t0, data_t1, ..., data_tn])

# Compute percentiles once
thresholds = {
    feature: np.percentile(all_data[feature], [33, 66])
    for feature in features
}

# Apply consistently everywhere
discretized = all_data.copy()
for feature in features:
    discretized[feature] = np.digitize(
        all_data[feature], 
        bins=thresholds[feature]
    )
```

**Why this works:**
- State 0 = bottom 33% (consistent definition)
- State 1 = middle 33% (consistent definition)
- State 2 = top 33% (consistent definition)
- Same thresholds everywhere → learning algorithms see consistent state transitions

---

## Bin Recovery: Reversing Discretization

**Key insight**: Discretization is not a loss if you preserve original values.

Store original values per bin during discretization:

```python
BIN_VALUES = {}
for feature in features:
    BIN_VALUES[feature] = {
        0: all_data[feature][discretized[feature] == 0].tolist(),
        1: all_data[feature][discretized[feature] == 1].tolist(),
        2: all_data[feature][discretized[feature] == 2].tolist(),
    }
```

Later, reverse discretization by sampling from learned bins:

```python
recovered = discretized.copy()
for feature in features:
    for bin_idx in range(3):
        mask = discretized[feature] == bin_idx
        if mask.sum() > 0:
            recovered.loc[mask, feature] = np.random.choice(
                BIN_VALUES[feature][bin_idx],
                size=mask.sum()
            )
```

**Impact**: Enables validation in continuous space after learning in discrete space.

---

## Bin Count Selection

**Current practice**: 3-state binning (33rd, 66th percentiles)

**Why 3?**
- Captures intuitive levels: low/medium/high
- Balances granularity vs. sample efficiency
- Works well with large datasets (1M+ values)

**To explore**: Adaptive binning per feature (2, 3, or 4 states)
- Sweep bin counts across features
- Score via BIC/AIC
- Select optimal per feature

---

## Common Gotchas

### 1. Empty Bins at Data Boundaries
**Problem**: If percentile thresholds computed on filtered data, first/last bin gets zero counts.
**Fix**: Compute percentiles on ALL values, then digitize everything.

### 2. State Cardinality Mismatch
**Problem**: If CPDs expect states [0,1,2] but training data only has [0,1], learning fails silently.
**Fix**: Pre-define state space before fitting.

### 3. Inconsistent Bin Counts
**Problem**: Mixing 2-state and 3-state binning within same model.
**Fix**: Commit to strategy per model run; sweep systematically.

---

## Success Criterion

The test for whether your discretization is fair:
> If I analyze discretized data at t=0 and again at t=5, do my results align?

If state distributions differ dramatically between times (despite consistent thresholds), the thresholds may need adjustment, but the methodology is sound.

---

## Related

- [[Discrete-to-Continuous Validation]] — How to validate discretized models in continuous space
- [[Bayesian CPD Estimation]] — Using discretized data for CPD learning

---

**Source**: [[40 Workspace/Research/PGMpy & Graph Networks/Discretization]]
