# Bayesian CPD Estimation

## Problem

Given discretized data and a Bayesian network structure, how do you fit Conditional Probability Distributions (CPDs) to learn P(child | parents)?

Three common approaches exist. **Which one should you choose?**

---

## Three Estimation Methods

### Method 1: Maximum Likelihood Estimation (MLE)

Learn CPDs purely from data, no prior smoothing.

```python
mle_est = DiscreteMLE(state_names=STATE_NAMES)
mle_est.fit(model, discretized_data)
model.add_cpds(*mle_est.parameters_)
```

**When good**:
- You have >5,000 clean, representative samples
- You trust the data completely
- You want to see raw patterns without smoothing

**Outcome**: Works well with large datasets, but can produce zero probabilities for unobserved state combinations

---

### Method 2: K2 Prior (Minimal Smoothing) — **RECOMMENDED FOR LARGE DATA**

Add 1 pseudo-count per state combination (minimal smoothing).

```python
k2_est = DiscreteBayesianEstimator(
    prior_type="K2",
    state_names=STATE_NAMES
)
k2_est.fit(model, discretized_data)
model.add_cpds(*k2_est.parameters_)
```

**Math**:
```
P(child | parent) = (count(parent, child) + 1) / (count(parent) + |child_states|)
```

**When good** (MOST CASES):
- 1,000 - 50,000 clean samples
- Biological signal present
- You want light smoothing for rare state combinations

**Outcome**: Ensures no zero probabilities; rare transitions still get learned signal; common transitions dominate

---

### Method 3: BDeu Prior (Strong Smoothing)

Distribute prior uniformly across state combinations, tunable strength.

```python
bdeu_est = DiscreteBayesianEstimator(
    prior_type="BDeu",
    equivalent_sample_size=10,  # Adjust this
    state_names=STATE_NAMES
)
bdeu_est.fit(model, discretized_data)
model.add_cpds(*bdeu_est.parameters_)
```

**When good**:
- <1,000 samples
- You need strong priors (domain knowledge available)
- You want to smooth away noise

**Outcome**: With large data + strong prior, can wash out real biological signal (all states pushed toward uniform)

---

## Decision Matrix

| Sample Size | Data Quality | Signal Strength | Recommendation |
|---|---|---|---|
| <1,000 | Clean | Any | BDeu (need smoothing) |
| 1,000 - 50,000 | Clean | Yes (biological) | **K2** (balance) |
| >50,000 | Clean | Yes | MLE (trust data) |
| Any | Noisy | No | BDeu (smooth noise) |

---

## Empirical Finding: Large Data Favors Light Priors

When you have:
- **6,366 clean transitions** (large dataset)
- **Real biological signal** (not random noise)

Then **K2 > MLE > BDeu(ESS>50)**

Because:
- MLE works (enough samples) but slightly noisier
- K2 is the sweet spot: minimal smoothing + robustness
- BDeu with ESS>10 dilutes real regulatory patterns

**Principle**: When you have large, clean data with real signal, **trust the data, not the prior**.

---

## Model Scoring: Validating Structure

After fitting CPDs, validate learned edges against random baselines:

```python
# Score your learned model (with 47 regulatory edges + 11 self-loops)
learned_scores = {
    'K2': K2_scorer.score(model),
    'BIC': BIC_scorer.score(model),
    'BDeu': BDeu_scorer.score(model),
}

# Generate 1,000 random networks with same #edges
random_scores = []
for _ in range(1000):
    random_model = create_random_network(num_edges=47)
    random_scores.append(scorer.score(random_model))

# Compare: Is your model better than random?
if learned_scores['K2'] > np.percentile(random_scores, 90):
    print("✓ Edges are meaningful (beat 90th percentile of random)")
```

**Interpretation**: If learned model beats 90%+ of random models across most scorers → edges capture real structure.

---

## Common Gotchas

### State Cardinality Mismatch
**Problem**: If your CPDs expect states [0,1,2] but training data only has [0,1], learning fails silently.

**Fix**: Pre-define state space before fitting:
```python
STATE_NAMES = {node: [0, 1, 2] for node in model.nodes()}
estimator.fit(model, data, state_names=STATE_NAMES)
```

### Choosing Prior Strength (BDeu)
**Problem**: ESS parameter is unintuitive. How strong should the prior be?

**Fix**: Test sweep: ESS=1, 5, 10, 50, 100. Score each. Select best.

---

## Related

- [[Discretization Strategies]] — Preparing data for CPD learning
- [[Discrete-to-Continuous Validation]] — Validating CPDs via PCA

---

**Source**: [[40 Workspace/Research/PGMpy & Graph Networks/Model Fitting & Selection]]
