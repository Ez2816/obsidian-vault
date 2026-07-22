# Error Propagation in Multi-Step Simulation

## The Problem

When simulating across multiple timesteps (t=0 → t=1 → t=2 → ... → t=9), small errors at each transition compound into large errors by the final timestep.

**Specific failure mode**: Even if each individual transition t→t+1 is learned accurately, the cascade mechanism (where one transition's output becomes the next transition's input) can amplify CPD modeling errors.

---

## Why This Happens

### The Cascade Mechanism

```python
# Step t=0: Start with parent state
t0_state = parent_sample

# Step t=1: Sample daughter from t0_state
t1_state = sample_from_cpd(t0_state)

# Step t=2: Use t1_state as NEW t0_state
t0_state = t1_state  # ← This chains errors forward
t2_state = sample_from_cpd(t0_state)  # If t1_state is slightly wrong, error grows
```

### Error Accumulation Path

```
True trajectory: Parent → t1_true → t2_true → t3_true → ...
Simulated:       Parent → t1_sim ─────────────────────→ ...
                            ↑
                         Error at t=1
                            ↓
                      Propagates as new parent state
                            ↓
                      t2 error = f(t1 error) + new sampling noise
                            ↓
                      By t=9, variance structure is corrupted
```

---

## Detection Signal: PCA Divergence

### What Convergence Looks Like

✓ **Good**: Simulated PCA variance profile matches raw data
```
Raw PC variance:  [0.151, 0.142, 0.111, 0.107, 0.096, ...]
Sim PC variance:  [0.148, 0.139, 0.108, 0.105, 0.094, ...]
Max difference:   0.003 (< 5%) ← SUCCESS
```

### What Error Propagation Looks Like

✗ **Bad**: Simulated PCA diverges significantly
```
Raw PC variance:  [0.151, 0.142, 0.111, 0.107, 0.096, ...]
Sim PC variance:  [0.089, 0.078, 0.064, 0.058, 0.041, ...]
Max difference:   0.062 (> 5%) ← ERROR PROPAGATED
```

---

## Root Causes

1. **Biased daughter sampling**: Overweighting first daughter (e.g., 100% first + 1% second) creates unrepresentative cascade starting points
2. **Weak t→t+1 transitions**: If CPDs don't capture true state change distributions, errors compound
3. **Missing regulatory edges**: Incomplete model may miss major drivers of state change
4. **Discrete state granularity**: Coarse binning (e.g., 2 states instead of 3) may lose information needed to track state paths accurately

---

## Mitigation Strategies

### Strategy 1: Single-Step Validation ✓

**Approach**: Learn and simulate at **each timestep independently**

```python
# Learn CPDs only from t → t+1 transition
cpd = fit_cpds(data_t0, data_t1, method='K2')

# Simulate single step
sim_t1 = simulate_one_step_from_parents(parent_state=data_t0, cpd=cpd)

# Validate immediately
pca_diff = compare_pca(raw_t0_to_t1, sim_t0_to_t1)

if pca_diff < threshold:
    print("✓ Single-step validation passed")
else:
    print("✗ CPD modeling issue detected—fix before cascading")
```

**Benefit**: Isolates error to its source. If single-step passes but full-cascade fails, error is cascade mechanism. If single-step fails, error is CPD learning.

### Strategy 2: Balanced Daughter Weighting ✓

**Avoid**: 100% first daughter + 1% second (extreme bias)

**Use**: 50% first daughter, 50% second daughter
- Captures stochastic variation across both daughters
- More representative of true cell division biology
- Reduces cascade starting-point bias

### Strategy 3: Per-Timestep Convergency Check

**Method**: Independently validate at each timestep

```python
# After simulating t=0→t=1, check PCA
pca_t1 = validate_timestep(sim_t1, raw_t1)

# After simulating t=1→t=2, check PCA (independent of t=0→t=1 result)
pca_t2 = validate_timestep(sim_t2, raw_t2)

# Plot: Do PCA differences grow over time?
pca_diffs = [pca_t1, pca_t2, pca_t3, ...]
if monotonic_increase(pca_diffs):
    print("⚠ Error propagating. Switch to per-step learning.")
else:
    print("✓ Errors not accumulating. Cascade mechanism OK.")
```

### Strategy 4: Edge Quality Assessment

**Before cascading**, ensure edges are biologically sound:
```python
# Score learned edges against random baseline
learned_score = scorer.score(model)
random_scores = [scorer.score(random_model) for _ in range(1000)]

if learned_score > np.percentile(random_scores, 90):
    print("✓ Edges beat random baseline. Safe to cascade.")
else:
    print("✗ Weak edge signal. Add more edges or improve data.")
```

---

## When Each Strategy Applies

| Scenario | Best Strategy |
|----------|---|
| Multi-timestep cascade diverges but single-step works | **Per-timestep learning** (abandon cascade for this model) |
| Single-step works inconsistently | **Improve edge detection** or add more regulatory edges |
| Cascading works but daughter sampling is biased | **Rebalance daughter weighting** (50/50 instead of 100%/1%) |
| Single-step fails | **Improve discretization or CPD estimator** before cascading |

---

## Key Insight

> Error propagation is a **validation signal**, not a flaw. If multi-step simulation diverges but single-step validation passes, your model is correct *locally* but doesn't hold *globally*. This guides you toward better edge detection or per-timestep modeling.

---

## Related

- [[Discrete-to-Continuous Validation]] — PCA comparison methodology
- [[Multi-Step Lineage Simulation]] — Cascade mechanism details
- [[Bayesian CPD Estimation]] — Model scoring and edge validation
- [[Genealogy-Aware Sampling]] — Daughter sampling strategies

---

**Source**: [[40 Workspace/Research/PGMpy & Graph Networks/Validation via PCA]] — Friday experiment findings on multi-timestep error propagation

