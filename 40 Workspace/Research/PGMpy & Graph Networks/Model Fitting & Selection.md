

## The Three CPD Estimation Methods

**Your data demands K2.**

### Method 1: Maximum Likelihood Estimation (MLE)

Learn CPDs purely from data, no prior smoothing.

**When good**: >5,000 clean samples, trust data completely
**For your data**: Works (6,366 transitions), but K2 slightly more robust

### Method 2: K2 Prior (RECOMMENDED)

Add 1 pseudo-count per state combination, minimal smoothing.

```python
k2_est = DiscreteBayesianEstimator(prior_type="K2", state_names=STATE_NAMES)
k2_est.fit(GRN_model, transitions)
GRN_model.add_cpds(*k2_est.parameters_)
```

**When good**: 1,000-50,000 samples, clean data, biological signal present
**Your case**: Perfect fit (6,366 clean transitions)

### Method 3: BDeu Prior (Strong Smoothing)

Distribute prior uniformly, tunable strength via equivalent_sample_size.

**When good**: <1,000 samples, need strong priors
**For your data**: Over-smooths, dilutes biological signal

---

## Why K2 Wins

With 6,366 clean transitions and real biology as signal:

| Estimator | Outcome |
|-----------|---------|
| MLE | Works perfectly, slightly noisier |
| **K2** | Sweet spot: minimal smoothing + robustness |
| BDeu (ESS=100) | Smooths too much, dilutes signal |

**Principle**: When you have large, clean data with real biological signal, **trust the data, not the prior.**

---

## Model Scoring: Validating Structure

After fitting CPDs, score against random baselines:

```python
scorers = {
    'K2': K2(transitions, state_names=STATE_NAMES),
    'BIC': BIC(transitions, state_names=STATE_NAMES),
    'BDeu': BDeu(transitions, state_names=STATE_NAMES),
    'AIC': AIC(transitions, state_names=STATE_NAMES),
}

# Score learned model (47 regulatory edges + 11 self-loops)
learned_scores = {name: scorer.score(GRN_model) for name, scorer in scorers.items()}

# Generate 1,000 random networks with same #edges
random_scores = {name: [] for name in scorers.keys()}
for _ in range(1000):
    random_model = create_random_network(num_edges=47)
    for name, scorer in scorers.items():
        random_scores[name].append(scorer.score(random_model))

# Compare: Are your edges better than random?
for name in scorers:
    if learned_scores[name] > np.percentile(random_scores[name], 90):
        print(f"✓ {name}: Your model > 90th percentile of random")
```

**Interpretation**: If learned model beats 90%+ of random models across most scorers → edges are meaningful.

---

## Decision Flow

```
Data size?
├─ < 1,000 samples → Use BDeu (strong prior)
├─ 1,000 - 50,000 samples → Use K2 (light prior) ← YOU
└─ > 50,000 samples → Use MLE (trust data)
```

---

## Next Steps

1. Formalize success criterion: "learned > 90th percentile" across how many scorers?
2. Test alternative networks: 0, 10, 20 additional edges → measure score improvement
3. Cross-validate: Hold out 10% of transitions, validate CPDs on held-out data

---

## Wiki Reference

The reusable methodology from this workspace has been extracted to the Wiki:
- [[30 Wiki/Concepts/Bayesian CPD Estimation]] — General CPD estimation methods and model selection

**Note**: This Workspace file is the project-specific working version. The Wiki page contains the generalizable knowledge.

## See Also
- [[Discretization]] — Preparing data for CPD learning
- [[Model Scoring]] — Validating learned structure
