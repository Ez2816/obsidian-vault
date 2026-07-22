---
type: course-overview
status: developed
study_status: completed
source_status: user-confirmed
confidence: high
courses:
  - STAT 251
last_reviewed: 2026-07-21
---

# STAT 251 – Introductory Probability and Statistics

## Course summary

STAT 251 introduced probability models and used them to estimate population quantities, test hypotheses, compare groups, and model relationships between variables.

## Descriptive statistics

Descriptive statistics summarize the distribution and relationships in observed data before making population-level inferences.

- Graphical summaries of data
- Mean as a measure of centre
- Variance and standard deviation as measures of spread
- Quantiles as locations within an ordered distribution
- Correlation as a standardized measure of linear association

For observations \(x_1,\ldots,x_n\), the sample mean and sample variance are

\[
\bar{x}=\frac{1}{n}\sum_{i=1}^{n}x_i,
\qquad
s^2=\frac{1}{n-1}\sum_{i=1}^{n}(x_i-\bar{x})^2.
\]

Correlation describes the strength and direction of a linear relationship, but does not by itself establish causation.

## Probability rules

- Complements, unions, and intersections of events
- Conditional probability
- Independence
- Bayes' theorem

Conditional probability is

\[
P(A\mid B)=\frac{P(A\cap B)}{P(B)}, \qquad P(B)>0.
\]

Events \(A\) and \(B\) are independent when \(P(A\cap B)=P(A)P(B)\). Bayes' theorem reverses a conditional probability:

\[
P(A\mid B)=\frac{P(B\mid A)P(A)}{P(B)}.
\]

## Random variables and distributions

- Discrete random variables and probability mass functions
- Continuous random variables and probability density functions
- Common probability distributions
- Expected value and variance

The expected value describes the long-run centre of a random variable:

\[
E[X]=\sum_x xP(X=x)
\]

for a discrete variable, or

\[
E[X]=\int_{-\infty}^{\infty}xf(x)\,dx
\]

for a continuous variable. Variance measures spread around the expectation:

\[
\operatorname{Var}(X)=E[(X-E[X])^2]=E[X^2]-E[X]^2.
\]

## Joint probability distributions

Joint distributions describe multiple random variables together. They support reasoning about marginal and conditional distributions, independence, and relationships between variables.

Covariance measures how two variables vary together:

\[
\operatorname{Cov}(X,Y)=E[(X-E[X])(Y-E[Y])].
\]

## Sampling distributions and the Central Limit Theorem

A sampling distribution is the probability distribution of a statistic across repeated samples. It connects variability in samples to uncertainty about population quantities.

The Central Limit Theorem states that, under suitable conditions, the standardized sample mean approaches a normal distribution as the sample size increases. This provides the basis for many confidence intervals and hypothesis tests.

## Estimation and confidence intervals

- Point estimates use a sample statistic to estimate a population parameter.
- Confidence intervals combine an estimate with its sampling uncertainty.

A common interval structure is

\[
\text{estimate}\ \pm\ (\text{critical value})(\text{standard error}).
\]

The confidence level describes the long-run coverage of the interval-generating procedure, not the probability that a fixed parameter changes or lies in one already-calculated interval.

## Hypothesis testing

- Null and alternative hypotheses
- Test statistics
- \(p\)-values
- Type I and Type II errors

A \(p\)-value is the probability, assuming the null hypothesis and model assumptions are true, of obtaining a result at least as incompatible with the null hypothesis as the observed result. It is not the probability that the null hypothesis is true.

- Type I error: rejecting a true null hypothesis
- Type II error: failing to reject a false null hypothesis

## Analysis of variance (ANOVA)

ANOVA tests whether group means differ by comparing between-group variation with within-group variation. A significant overall result indicates evidence that the means are not all equal; it does not identify the differing groups by itself.

## Simple linear regression and residual analysis

Simple linear regression models a response variable using one explanatory variable:

\[
Y_i=\beta_0+\beta_1x_i+\varepsilon_i.
\]

- \(\beta_0\): intercept
- \(\beta_1\): expected change in the mean response for a one-unit increase in \(x\)
- \(\varepsilon_i\): unexplained variation

Residuals are observed values minus fitted values. Residual analysis checks whether the model's assumptions and linear form are reasonable and helps identify unusual observations or systematic patterns left unexplained.

## Connections to the pgmpy network project

These are later applications of STAT 251 foundations, not additional STAT 251 course topics.

- **Conditional probability and Bayes' theorem:** Bayesian-network edges encode conditional dependence, while conditional probability distributions describe a node given its parents. See [[Bayesian CPD Estimation]].
- **Joint distributions and independence:** the network factorizes a joint distribution into local conditional distributions. Conditional-independence assumptions determine which variables need direct connections.
- **Random variables and probability distributions:** discretized gene-expression states are modeled as random variables with learned conditional distributions. See [[Discretization Strategies]].
- **Expected value, variance, and sampling:** simulations generate samples from the fitted model; comparing simulated and observed summaries asks whether the model reproduces important distributional behaviour.
- **Point estimation:** fitting conditional probability tables from observed transition counts is an estimation problem. The project's maximum-likelihood and smoothed estimators extend the estimation principles introduced in STAT 251.
- **Hypothesis-testing mindset:** comparison against baselines follows the same general logic of specifying a reference model and asking whether observed performance is unusually strong, although the project's scoring procedures are not STAT 251 hypothesis tests.
- **Regression and correlation:** these provide familiar ways to describe associations, but a Bayesian-network edge should not automatically be interpreted as a causal relationship.

## Evidence boundary

- **User-confirmed course content:** all topics in the course-content sections above.
- **Vault-supported later applications:** the pgmpy connections are supported by the linked Wiki pages and the project notes in `40 Workspace/Research/PGMpy & Graph Networks/`.
- **Not claimed as STAT 251 content:** PCA, Bayesian-network structure scores, CPD priors, discretization algorithms, and multi-step simulation methods.

## Sources

- User-provided STAT 251 topic list, 2026-07-21
- [UBC Academic Calendar – STAT 251](https://vancouver.calendar.ubc.ca/course-descriptions/courses/statv-251-introductory-probability-and-statistics)
- [[Bayesian CPD Estimation]]
- [[Discretization Strategies]]
- [[Discrete-to-Continuous Validation]]

