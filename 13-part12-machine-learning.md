# Part 12: Machine Learning

> *"Machine learning is the science of getting computers to act without being explicitly programmed. It is the engine that powers modern AI systems, from recommendation engines to self-driving cars." — Adapted from Arthur Samuel & Andrew Ng*

---

## Chapter 82: Mathematics for Machine Learning

### Introduction

Mathematics is the language of machine learning. Without a solid grasp of the underlying mathematics, ML practitioners operate purely by intuition and guesswork — tweaking hyperparameters and hoping for the best. This chapter provides the rigorous mathematical foundation required to understand, implement, and debug machine learning algorithms.

### Why It Matters

Every ML algorithm is built on mathematical principles:

| ML Component | Mathematics Required |
|---|---|
| Loss functions | Calculus (derivatives, gradients) |
| Model parameters | Linear algebra (matrix operations, eigenvectors) |
| Uncertainty estimates | Probability (distributions, Bayes theorem) |
| Model evaluation | Statistics (hypothesis testing, confidence intervals) |
| Optimization | Calculus (gradient descent, Lagrange multipliers) |
| Dimensionality reduction | Linear algebra (SVD, eigendecomposition) |

Without this foundation, you cannot diagnose why a model is failing, choose appropriate algorithms, or read research papers.

### Real World Analogy

Think of mathematics as the **engine specifications** of a car. You can drive (use ML libraries) without knowing the engine, but when something breaks, when you need to push performance to the limit, or when you want to design a new engine, you need to understand what's under the hood.

---

### Statistics

#### Descriptive vs Inferential Statistics

| Aspect | Descriptive Statistics | Inferential Statistics |
|---|---|---|
| **Purpose** | Summarize and describe data | Draw conclusions about populations from samples |
| **Example** | Mean, median, standard deviation of a dataset | Confidence interval for population mean |
| **Population vs Sample** | Works with entire dataset | Works with samples to infer about populations |
| **Output** | Charts, summary statistics | Hypothesis tests, confidence intervals, p-values |

#### Measures of Central Tendency

| Measure | Formula | When to Use |
|---|---|---|
| **Mean (μ)** | $\bar{x} = \frac{1}{n}\sum_{i=1}^{n} x_i$ | Symmetric distributions, no outliers |
| **Median** | Middle value when sorted | Skewed distributions, outliers present |
| **Mode** | Most frequent value | Categorical data, discrete distributions |
| **Geometric Mean** | $(\prod_{i=1}^{n} x_i)^{1/n}$ | Growth rates, ratios, percentages |
| **Harmonic Mean** | $\frac{n}{\sum_{i=1}^{n} 1/x_i}$ | Rates, ratios (F1-score uses this) |

```python
import numpy as np
from scipy import stats

data = np.array([2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 100])

mean = np.mean(data)
median = np.median(data)
mode = stats.mode(data, keepdims=True).mode[0]
geom_mean = stats.gmean(data)
harm_mean = stats.hmean(data)

print(f"Mean: {mean:.2f}, Median: {median:.2f}, Mode: {mode}")
print(f"Geometric Mean: {geom_mean:.2f}, Harmonic Mean: {harm_mean:.2f}")
```

#### Measures of Dispersion

| Measure | Formula | Notes |
|---|---|---|
| **Range** | $\max(x) - \min(x)$ | Simple but sensitive to outliers |
| **Variance (σ²)** | $\frac{1}{n}\sum_{i=1}^{n}(x_i - \bar{x})^2$ | Squared units, hard to interpret |
| **Std Dev (σ)** | $\sqrt{\text{Variance}}$ | Same units as data, interpretable |
| **IQR** | $Q_3 - Q_1$ | Robust to outliers (25th-75th percentile) |
| **MAD** | $\text{median}(|x_i - \text{median}(x)|)$ | Robust alternative to std dev |

```python
import numpy as np

data = np.array([2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 100])

variance = np.var(data, ddof=1)
std_dev = np.std(data, ddof=1)
iqr = np.percentile(data, 75) - np.percentile(data, 25)
mad = np.median(np.abs(data - np.median(data)))

print(f"Variance: {variance:.2f}")
print(f"Std Dev: {std_dev:.2f}")
print(f"IQR: {iqr:.2f}")
print(f"MAD: {mad:.2f}")
```

#### Covariance and Correlation

**Covariance** measures the direction of linear relationship between two variables:

$$\text{Cov}(X, Y) = \frac{1}{n}\sum_{i=1}^{n}(x_i - \bar{x})(y_i - \bar{y})$$

**Correlation** measures both direction and strength of linear relationship (scale-invariant):

| Type | Range | Use Case | Formula |
|---|---|---|---|
| **Pearson (r)** | [-1, +1] | Linear relationships, normally distributed | $\frac{\text{Cov}(X,Y)}{\sigma_X \sigma_Y}$ |
| **Spearman (ρ)** | [-1, +1] | Monotonic relationships, ordinal data | Pearson on ranked data |
| **Kendall Tau (τ)** | [-1, +1] | Small samples, many tied ranks | Concordant - discordant pairs |

```python
from scipy.stats import pearsonr, spearmanr, kendalltau
import numpy as np

x = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
y = np.array([2.1, 4.0, 6.2, 8.1, 9.8, 12.3, 14.1, 15.9, 18.2, 19.7])

pearson_r, p_pearson = pearsonr(x, y)
spearman_r, p_spearman = spearmanr(x, y)
kendall_tau, p_kendall = kendalltau(x, y)

print(f"Pearson r = {pearson_r:.4f} (p = {p_pearson:.6f})")
print(f"Spearman ρ = {spearman_r:.4f} (p = {p_spearman:.6f})")
print(f"Kendall τ = {kendall_tau:.4f} (p = {p_kendall:.6f})")
```

#### Distributions

| Distribution | Support | Parameters | Mean | Variance | Use Case |
|---|---|---|---|---|---|
| **Normal/Gaussian** | $(-\infty, \infty)$ | μ, σ² | μ | σ² | Natural phenomena, CLT |
| **Binomial** | {0, 1, ..., n} | n, p | np | np(1-p) | Number of successes in n trials |
| **Bernoulli** | {0, 1} | p | p | p(1-p) | Single coin flip |
| **Poisson** | {0, 1, 2, ...} | λ | λ | λ | Count of rare events |
| **Uniform** | [a, b] | a, b | (a+b)/2 | (b-a)²/12 | Equal probability |
| **Exponential** | [0, ∞) | λ | 1/λ | 1/λ² | Time between events |
| **Beta** | [0, 1] | α, β | α/(α+β) | αβ/((α+β)²(α+β+1)) | Probabilities, prior for binomial |
| **Gamma** | [0, ∞) | k, θ | kθ | kθ² | Waiting times, sum of exponentials |
| **Chi-squared (χ²)** | [0, ∞) | k (df) | k | 2k | Goodness-of-fit tests |
| **t-distribution** | $(-\infty, \infty)$ | ν (df) | 0 | ν/(ν-2) | Unknown variance, small samples |
| **F-distribution** | [0, ∞) | d₁, d₂ | d₂/(d₂-2) | *complex* | ANOVA, ratio of variances |

> **The 68-95-99.7 Rule (Empirical Rule):** For a normal distribution, approximately 68% of data falls within 1σ of μ, 95% within 2σ, and 99.7% within 3σ.

```python
import numpy as np
from scipy import stats

# Generate data from different distributions
normal_data = np.random.normal(loc=0, scale=1, size=1000)
binomial_data = np.random.binomial(n=10, p=0.5, size=1000)
poisson_data = np.random.poisson(lam=3, size=1000)
exponential_data = np.random.exponential(scale=1, size=1000)
uniform_data = np.random.uniform(low=0, high=1, size=1000)

# PDF evaluation at a point
x = 1.5
pdf_normal = stats.norm.pdf(x, loc=0, scale=1)
cdf_normal = stats.norm.cdf(x, loc=0, scale=1)

print(f"Normal PDF at x={x}: {pdf_normal:.4f}")
print(f"Normal CDF at x={x}: {cdf_normal:.4f}")
print(f"68% interval: ({stats.norm.ppf(0.16)}, {stats.norm.ppf(0.84)})")
print(f"95% interval: ({stats.norm.ppf(0.025)}, {stats.norm.ppf(0.975)})")
```

#### Central Limit Theorem (CLT)

**Theorem:** Given a population with mean μ and finite variance σ², the sampling distribution of the sample mean $\bar{X}$ approaches a normal distribution $N(\mu, \sigma²/n)$ as sample size n → ∞, regardless of the population distribution.

**Proof Intuition:**
1. Let $X_1, X_2, ..., X_n$ be i.i.d. random variables with mean μ and variance σ²
2. Consider the sum $S_n = \sum_{i=1}^{n} X_i$
3. By the additive property of moment generating functions, $M_{S_n}(t) = [M_X(t/n)]^n$
4. As n → ∞, this converges to $e^{μt + σ²t²/2}$, which is the MGF of $N(μ, σ²/n)$
5. Therefore, $\sqrt{n}(\bar{X} - μ) \xrightarrow{d} N(0, σ²)$

**Why It Matters:** CLT is why we can use normal-theory inference (t-tests, confidence intervals) even when the underlying data is not normal, provided the sample size is large enough (typically n ≥ 30).

#### Law of Large Numbers (LLN)

| Version | Statement | Convergence |
|---|---|---|
| **Weak LLN** | $\bar{X}_n \xrightarrow{p} μ$ as n → ∞ | Convergence in probability |
| **Strong LLN** | $\bar{X}_n \xrightarrow{a.s.} μ$ as n → ∞ | Almost sure convergence (stronger) |

> **Intuition:** As you collect more data, the sample average converges to the true population mean. This is why more training data generally improves model performance.

#### Hypothesis Testing

**Framework:**

| Component | Definition | Example |
|---|---|---|
| **Null Hypothesis (H₀)** | Default assumption, no effect | Drug has no effect |
| **Alternative (H₁/Hₐ)** | What we want to prove | Drug has an effect |
| **Type I Error (α)** | False positive — rejecting true H₀ | p(false alarm) |
| **Type II Error (β)** | False negative — failing to reject false H₀ | p(miss) |
| **Power (1-β)** | Probability of correctly rejecting false H₀ | 1 - p(miss) |
| **p-value** | Probability of observing data as extreme as ours, assuming H₀ is true | — |
| **Significance Level (α)** | Threshold for rejecting H₀ (commonly 0.05) | — |

```python
from scipy.stats import ttest_ind, ttest_rel, ttest_1samp

# One-sample t-test: is the mean different from 5?
data = np.random.normal(loc=5.5, scale=1.0, size=100)
t_stat, p_value = ttest_1samp(data, popmean=5.0)
print(f"One-sample t-test: t = {t_stat:.4f}, p = {p_value:.4f}")

# Two-sample independent t-test
group_a = np.random.normal(loc=5.0, scale=1.0, size=50)
group_b = np.random.normal(loc=5.5, scale=1.0, size=50)
t_stat, p_value = ttest_ind(group_a, group_b)
print(f"Two-sample t-test: t = {t_stat:.4f}, p = {p_value:.4f}")

# Paired t-test
before = np.random.normal(loc=100, scale=15, size=30)
after = before - np.random.normal(loc=3, scale=5, size=30)
t_stat, p_value = ttest_rel(before, after)
print(f"Paired t-test: t = {t_stat:.4f}, p = {p_value:.4f}")
```

#### Statistical Tests Reference

| Test | Purpose | Assumptions | Python Function |
|---|---|---|---|
| **One-sample t-test** | Compare mean to known value | Normal, independent | `ttest_1samp` |
| **Two-sample t-test** | Compare two group means | Normal, independent, equal variance | `ttest_ind` |
| **Paired t-test** | Compare paired measurements | Normal, paired differences | `ttest_rel` |
| **z-test** | Compare with known σ | Normal, known σ, large n | `statsmodels.stats.proportion.proportions_ztest` |
| **One-way ANOVA** | Compare ≥3 group means | Normal, independent, equal variance | `f_oneway` |
| **Two-way ANOVA** | Two factors + interaction | Normal, independent, equal variance | `statsmodels.stats.anova.anova_lm` |
| **Chi-squared test** | Independence/categorical | Expected counts ≥ 5 | `chi2_contingency` |
| **Mann-Whitney U** | Compare two groups (non-parametric) | Independent, ordinal/continuous | `mannwhitneyu` |
| **Wilcoxon signed-rank** | Paired comparison (non-parametric) | Paired, symmetric differences | `wilcoxon` |

#### Confidence Intervals

A confidence interval (CI) provides a range of plausible values for a population parameter:

$$\text{CI}_{\alpha} = \bar{x} \pm z_{\alpha/2} \cdot \frac{\sigma}{\sqrt{n}}$$

For mean with unknown σ (using t-distribution):

$$\text{CI}_{\alpha} = \bar{x} \pm t_{\alpha/2, n-1} \cdot \frac{s}{\sqrt{n}}$$

**Interpretation:** If we repeated this experiment 100 times, approximately 95 of those confidence intervals would contain the true population parameter.

**Bootstrapping** is a non-parametric method for constructing CIs by resampling with replacement from the data:

```python
from scipy.stats import bootstrap

data = np.array([12, 15, 14, 10, 13, 14, 15, 16, 14, 12, 11, 13])
res = bootstrap((data,), np.mean, n_resamples=9999, confidence_level=0.95)

print(f"Bootstrap 95% CI: ({res.confidence_interval.low:.2f}, {res.confidence_interval.high:.2f})")
```

#### Bayesian vs Frequentist Statistics

| Aspect | Frequentist | Bayesian |
|---|---|---|
| **Probability** | Long-run frequency | Degree of belief |
| **Parameters** | Fixed (unknown) constants | Random variables with distributions |
| **Prior** | Not used | Required — $P(θ)$ |
| **Inference** | Point estimate + CI + p-value | Posterior distribution $P(θ|D)$ |
| **Interpretation** | "95% of CIs contain θ" | "95% probability θ is in this interval" |
| **Sample Size** | Relies on asymptotics | Works with any sample size |
| **Computation** | Closed-form formulas typically available | Often requires MCMC, variational inference |

---

### Probability

#### Basic Rules

| Rule | Formula | Example |
|---|---|---|
| **Addition (mutually exclusive)** | $P(A \cup B) = P(A) + P(B)$ | Rolling 3 or 4 on a die: 1/6 + 1/6 = 1/3 |
| **Addition (general)** | $P(A \cup B) = P(A) + P(B) - P(A \cap B)$ | Drawing heart or king from deck |
| **Multiplication (independent)** | $P(A \cap B) = P(A) \cdot P(B)$ | Two consecutive coin flips: 0.5 × 0.5 = 0.25 |
| **Multiplication (general)** | $P(A \cap B) = P(A|B) \cdot P(B)$ | — |
| **Complement** | $P(A^c) = 1 - P(A)$ | Probability of not rolling a 6: 1 - 1/6 = 5/6 |

#### Conditional Probability

$$P(A|B) = \frac{P(A \cap B)}{P(B)}$$

> **Intuition:** Given that B occurred, what fraction of that probability space also includes A?

#### Bayes Theorem

$$P(A|B) = \frac{P(B|A) \cdot P(A)}{P(B)} = \frac{P(B|A) \cdot P(A)}{P(B|A)P(A) + P(B|\text{not }A)P(\text{not }A)}$$

**Example 1 — Spam Detection:**
- P(spam) = 0.3 (30% of emails are spam)
- P("free" | spam) = 0.8
- P("free" | not spam) = 0.05
- P("free") = P("free"|spam)·P(spam) + P("free"|not spam)·P(not spam) = 0.8·0.3 + 0.05·0.7 = 0.275

$$P(\text{spam}|\text{"free"}) = \frac{0.8 \cdot 0.3}{0.275} = 0.873$$

**Example 2 — Medical Testing:**
- Disease prevalence: 1% (P(D) = 0.01)
- Test sensitivity: 99% (P(+|D) = 0.99)
- Test false positive: 5% (P(+|¬D) = 0.05)

$$P(D|+) = \frac{0.99 \cdot 0.01}{0.99 \cdot 0.01 + 0.05 \cdot 0.99} = \frac{0.0099}{0.0099 + 0.0495} = 0.167$$

> Despite 99% test accuracy, there's only a 16.7% chance you actually have the disease! This counterintuitive result is a classic example of the base rate fallacy.

#### Random Variables

| Type | Definition | Examples | Representation |
|---|---|---|---|
| **Discrete** | Countable set of values | Coin flip, dice roll, count of emails | PMF: $P(X = x)$ |
| **Continuous** | Uncountable range | Height, weight, temperature | PDF: $f(x)$, CDF: $F(x) = \int_{-\infty}^x f(t)dt$ |

**Expected Value:** $E[X] = \sum x \cdot P(X=x)$ (discrete) or $E[X] = \int x \cdot f(x)dx$ (continuous)

**Variance:** $\text{Var}(X) = E[(X - μ)²] = E[X²] - (E[X])²$

**Moments:**
- 1st moment: $E[X]$ (mean)
- 2nd central moment: $E[(X - μ)²]$ (variance)
- 3rd standardized moment: $\frac{E[(X - μ)³]}{σ³}$ (skewness)
- 4th standardized moment: $\frac{E[(X - μ)⁴]}{σ⁴}$ (kurtosis)

#### Joint, Marginal, and Conditional Distributions

| Distribution | Definition | Interpretation |
|---|---|---|
| **Joint** | $P(X=x, Y=y)$ | Probability of both events |
| **Marginal** | $P(X=x) = \sum_y P(X=x, Y=y)$ | Probability of one variable (sum over other) |
| **Conditional** | $P(X=x|Y=y) = \frac{P(X=x, Y=y)}{P(Y=y)}$ | Probability given known value of other variable |

**Independence:** $P(X,Y) = P(X) \cdot P(Y)$ — knowing X tells you nothing about Y (and vice versa).

**Conditional Independence:** $X \perp Y | Z \iff P(X,Y|Z) = P(X|Z) \cdot P(Y|Z)$ — X and Y are independent given Z.

#### Maximum Likelihood Estimation (MLE) vs Maximum A Posteriori (MAP)

| Aspect | MLE | MAP |
|---|---|---|
| **Objective** | $\hat{θ}_{MLE} = \arg\max_θ P(D|θ)$ | $\hat{θ}_{MAP} = \arg\max_θ P(θ|D) = \arg\max_θ P(D|θ)P(θ)$ |
| **Prior** | No prior | Uses prior distribution |
| **Philosophy** | Frequentist | Bayesian |
| **Small Sample** | Can overfit | Prior regularizes |
| **Convergence** | As n → ∞, converges to true θ | As n → ∞, converges to MLE |

#### Information Theory

| Concept | Formula | Intuition |
|---|---|---|
| **Entropy** | $H(X) = -\sum p(x) \log₂ p(x)$ | Average information (uncertainty) in X |
| **Cross-Entropy** | $H(P,Q) = -\sum p(x) \log₂ q(x)$ | Avg # bits to encode P using Q's distribution |
| **KL Divergence** | $D_{KL}(P\|Q) = \sum p(x) \log\frac{p(x)}{q(x)}$ | How much info is lost when Q approximates P |
| **Mutual Information** | $I(X;Y) = \sum\sum p(x,y) \log\frac{p(x,y)}{p(x)p(y)}$ | Reduction in uncertainty of X given Y |

```python
from scipy.stats import entropy
import numpy as np

fair_coin = np.array([0.5, 0.5])
biased_coin = np.array([0.9, 0.1])

print(f"Fair coin entropy: {entropy(fair_coin, base=2):.4f} bits")
print(f"Biased coin entropy: {entropy(biased_coin, base=2):.4f} bits")

# KL Divergence
p = np.array([0.25, 0.25, 0.25, 0.25])
q = np.array([0.4, 0.3, 0.2, 0.1])
kl_pq = entropy(p, q, base=2)
kl_qp = entropy(q, p, base=2)
print(f"D_KL(P||Q): {kl_pq:.4f} bits")
print(f"D_KL(Q||P): {kl_qp:.4f} bits (asymmetric!)")
```

> **Key Insight:** KL divergence is **not symmetric** ($D_{KL}(P\|Q) \neq D_{KL}(Q\|P)$). It measures the cost of using Q to approximate P, not a distance metric.

---

### Linear Algebra

#### Vectors

| Operation | Formula | Use in ML |
|---|---|---|
| **Dot Product** | $a \cdot b = \sum_{i=1}^{n} a_i b_i = \|a\|\|b\|\cos θ$ | Linear regression, neural net forward pass |
| **L1 Norm** | $\|x\|_1 = \sum |x_i|$ | Lasso regularization (sparsity) |
| **L2 Norm** | $\|x\|_2 = \sqrt{\sum x_i²}$ | Ridge regularization, Euclidean distance |
| **L∞ Norm** | $\|x\|_\infty = \max |x_i|$ | Chebyshev distance, robust bounds |
| **Cosine Similarity** | $\cos θ = \frac{a \cdot b}{\|a\|\|b\|}$ | Text similarity, recommendation systems |
| **Orthogonality** | $a \perp b \iff a \cdot b = 0$ | Independent features, orthogonal matrices |

```python
import numpy as np

a = np.array([1, 2, 3])
b = np.array([4, 5, 6])

dot_product = np.dot(a, b)
l1_norm = np.linalg.norm(a, ord=1)
l2_norm = np.linalg.norm(a, ord=2)
l_inf_norm = np.linalg.norm(a, ord=np.inf)
cosine_sim = np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b))

print(f"Dot: {dot_product}, L1: {l1_norm:.2f}, L2: {l2_norm:.2f}")
print(f"L∞: {l_inf_norm:.2f}, Cosine Sim: {cosine_sim:.4f}")
```

#### Matrices

| Property | Formula | Notes |
|---|---|---|
| **Addition** | $(A + B)_{ij} = a_{ij} + b_{ij}$ | Same dimensions required |
| **Multiplication** | $(AB)_{ij} = \sum_k a_{ik}b_{kj}$ | A: m×n, B: n×p → AB: m×p |
| **Transpose** | $(A^T)_{ij} = a_{ji}$ | Swap rows and columns |
| **Inverse** | $A^{-1}A = AA^{-1} = I$ | Only for square, non-singular matrices |
| **Determinant** | $\det(A)$ | Volume scaling factor, singularity test |
| **Trace** | $\text{tr}(A) = \sum a_{ii}$ | Sum of diagonal, equals sum of eigenvalues |
| **Rank** | $\text{rank}(A)$ | Number of linearly independent rows/cols |

#### Important Matrix Types

| Type | Definition | Properties | Use Case |
|---|---|---|---|
| **Symmetric** | $A = A^T$ | Real eigenvalues, orthogonal eigenvectors | Covariance matrix, graph Laplacian |
| **Positive Definite** | $x^T A x > 0 \;\forall x \neq 0$ | All eigenvalues > 0 | Hessian at minimum, kernel matrices |
| **Diagonal** | $a_{ij} = 0$ for $i \neq j$ | Easy inversion, eigenvalues on diagonal | Scaling, normalization |
| **Orthogonal** | $Q^T Q = QQ^T = I$ | $Q^{-1} = Q^T$, \|det\| = 1 | Rotation, SVD matrices |
| **Idempotent** | $A^2 = A$ | Eigenvalues 0 or 1 | Projection matrices, hat matrix |

#### Eigenvalues and Eigenvectors

$$A v = λ v$$

Where v is the eigenvector and λ is the corresponding eigenvalue.

**Characteristic equation:** $\det(A - λI) = 0$

**Eigendecomposition:** For a diagonalizable matrix $A = PDP^{-1}$ where D is diagonal of eigenvalues and P is columns of eigenvectors.

```python
import numpy as np

A = np.array([[3, 1], [1, 3]])
eigenvalues, eigenvectors = np.linalg.eig(A)

print(f"Eigenvalues: {eigenvalues}")
print(f"Eigenvectors:\n{eigenvectors}")
print(f"A * v = λ * v check:\n{A @ eigenvectors[:, 0]} = {eigenvalues[0] * eigenvectors[:, 0]}")
```

#### Singular Value Decomposition (SVD)

$$A_{m×n} = U_{m×m} Σ_{m×n} V^T_{n×n}$$

- **U**: Left singular vectors (orthogonal columns) — eigenvectors of $AA^T$
- **Σ**: Diagonal matrix of singular values (σ₁ ≥ σ₂ ≥ ... ≥ σ_r > 0) — square roots of eigenvalues of $AA^T$
- **V^T**: Right singular vectors (orthogonal rows) — eigenvectors of $A^TA$

**Applications:**

| Application | How SVD Helps |
|---|---|
| **PCA** | Center data → SVD → principal components from V |
| **Dimensionality Reduction** | Keep top-k singular values → truncated SVD |
| **Matrix Completion** | Low-rank approximation via truncated SVD |
| **Recommendation Systems** | Collaborative filtering via SVD (Netflix Prize) |
| **Image Compression** | Keep top-k singular values, discard rest |
| **Pseudoinverse** | $A^+ = VΣ^+U^T$ for least squares solution |

```python
from sklearn.decomposition import TruncatedSVD
import numpy as np

A = np.random.randn(10, 5)
U, s, Vt = np.linalg.svd(A, full_matrices=False)
print(f"U: {U.shape}, s: {s.shape}, Vt: {Vt.shape}")
print(f"Singular values: {s}")

# Truncated SVD (for dimensionality reduction)
svd = TruncatedSVD(n_components=3)
A_reduced = svd.fit_transform(A)
print(f"Explained variance ratio: {svd.explained_variance_ratio_}")
print(f"Cumulative: {svd.explained_variance_ratio_.cumsum()}")
```

#### Matrix Factorizations

| Factorization | Form | Requirements | Use Case |
|---|---|---|---|
| **LU** | $A = LU$ | Square | Solving linear systems |
| **QR** | $A = QR$ | Any matrix (Q orthogonal, R upper triangular) | OLS solution, numerically stable |
| **Cholesky** | $A = LL^T$ | Symmetric positive definite | Efficient solving, multivariate normal sampling |

```python
import scipy.linalg

A = np.array([[4, 2, 1], [2, 5, 3], [1, 3, 6]])

L = scipy.linalg.cholesky(A, lower=True)
print(f"Cholesky factor L:\n{L}")

Q, R = scipy.linalg.qr(np.random.randn(10, 4))
print(f"Q shape: {Q.shape}, R shape: {R.shape}")
```

#### Matrix Norms

| Norm | Formula | Use Case |
|---|---|---|
| **Frobenius** | $\|A\|_F = \sqrt{\sum_i\sum_j a_{ij}²}$ | Matrix reconstruction error |
| **Spectral (L2)** | $\|A\|_2 = σ_{\max}(A)$ | Stability, convergence analysis |
| **Nuclear** | $\|A\|_* = \sum_i σ_i$ | Low-rank matrix completion (convex relaxation of rank) |

#### Linear Transformations

| Transformation | Matrix | Effect |
|---|---|---|
| **Rotation** | $\begin{bmatrix}\cos θ & -\sin θ \\ \sin θ & \cos θ\end{bmatrix}$ | Rotates points by angle θ |
| **Scaling** | $\begin{bmatrix}s_x & 0 \\ 0 & s_y\end{bmatrix}$ | Stretches/shrinks along axes |
| **Projection** | $P = X(X^T X)^{-1} X^T$ | Projects onto column space of X |
| **Shear** | $\begin{bmatrix}1 & k \\ 0 & 1\end{bmatrix}$ | Distorts along one axis |

---

### Calculus

#### Limits and Continuity

A function f is continuous at a if $\lim_{x \to a} f(x) = f(a)$.

> **Why It Matters:** Loss functions must be continuous (and preferably differentiable) for gradient-based optimization to work.

#### Derivatives

| Rule | Formula | Example |
|---|---|---|
| **Power** | $\frac{d}{dx} x^n = n x^{n-1}$ | $\frac{d}{dx} x³ = 3x²$ |
| **Product** | $(fg)' = f'g + fg'$ | $(x²\sin x)' = 2x\sin x + x²\cos x$ |
| **Chain** | $(f \circ g)'(x) = f'(g(x)) \cdot g'(x)$ | $(e^{x²})' = 2x e^{x²}$ |
| **Quotient** | $(\frac{f}{g})' = \frac{f'g - fg'}{g²}$ | $(\frac{x}{e^x})' = \frac{e^x - xe^x}{e^{2x}}$ |
| **Partial** | $\frac{\partial}{\partial x_i} f(x_1,...,x_n)$ | Derivative w.r.t. one variable, others fixed |

#### Gradient

$$\nabla f = \begin{bmatrix} \frac{\partial f}{\partial x_1} & \frac{\partial f}{\partial x_2} & \cdots & \frac{\partial f}{\partial x_n} \end{bmatrix}^T$$

> **Geometric meaning:** The gradient points in the direction of **steepest ascent** of f at a point. The negative gradient points to the steepest descent — this is why gradient descent works.

#### Jacobian Matrix

For $f: \mathbb{R}^n \to \mathbb{R}^m$:

$$J = \begin{bmatrix} \frac{\partial f_1}{\partial x_1} & \cdots & \frac{\partial f_1}{\partial x_n} \\ \vdots & \ddots & \vdots \\ \frac{\partial f_m}{\partial x_1} & \cdots & \frac{\partial f_m}{\partial x_n} \end{bmatrix}$$

#### Hessian Matrix

For $f: \mathbb{R}^n \to \mathbb{R}$:

$$H(f) = \begin{bmatrix} \frac{\partial² f}{\partial x_1²} & \cdots & \frac{\partial² f}{\partial x_1 \partial x_n} \\ \vdots & \ddots & \vdots \\ \frac{\partial² f}{\partial x_n \partial x_1} & \cdots & \frac{\partial² f}{\partial x_n²} \end{bmatrix}$$

**Convexity Test:** f is convex iff its Hessian is positive semidefinite ($x^T H x \geq 0$) for all x.

#### Gradient Descent

$$θ_{t+1} = θ_t - \eta \nabla L(θ_t)$$

Where η is the **learning rate** (step size).

| Variant | Update Rule | Benefit |
|---|---|---|
| **Vanilla GD** | $θ_{t+1} = θ_t - η\nabla L(θ_t)$ | Simple, guaranteed convergence for convex |
| **Momentum** | $v_t = γv_{t-1} + η\nabla L(θ_t); θ_{t+1} = θ_t - v_t$ | Accelerates, dampens oscillations |
| **Nesterov** | $v_t = γv_{t-1} + η\nabla L(θ_t - γv_{t-1})$ | Look-ahead, accelerated convergence |
| **AdaGrad** | $θ_{t+1} = θ_t - \frac{η}{\sqrt{G_t + ε}}\nabla L(θ_t)$ | Adaptive per-parameter learning rates |
| **RMSProp** | $E[g²]_t = βE[g²]_{t-1} + (1-β)g²_t$ | Fixes AdaGrad's diminishing learning rate |
| **Adam** | Combines momentum + RMSProp | Default choice for deep learning |

#### Taylor Series

**First-order approximation:** $f(x + Δ) ≈ f(x) + f'(x)Δ$

**Second-order approximation:** $f(x + Δ) ≈ f(x) + f'(x)Δ + \frac{1}{2}f''(x)Δ²$

> Gradient descent uses first-order approximation. Newton's method uses second-order (Hessian).

#### Optimization Concepts

| Concept | Definition | Example |
|---|---|---|
| **Convex function** | $f(λx + (1-λ)y) ≤ λf(x) + (1-λ)f(y)$ | $x²$, $-log(x)$ |
| **Non-convex** | Fails convexity condition | Neural network loss landscape |
| **Global minimum** | $f(x^*) ≤ f(x)$ for all x | The absolute best solution |
| **Local minimum** | $f(x^*) ≤ f(x)$ for all x in neighborhood | A good, but perhaps not best, solution |
| **Saddle point** | Gradient = 0, but neither min nor max | Common in high-dimensional optimization |

#### Lagrange Multipliers

For constrained optimization of $f(x)$ subject to $g(x) = 0$:

$$\mathcal{L}(x, λ) = f(x) - λg(x)$$

Set $\nabla \mathcal{L} = 0$ to find critical points.

**KKT Conditions** extend this to inequality constraints ($g(x) ≤ 0$).

#### Integration

| Type | Definition | Use in ML |
|---|---|---|
| **Definite** | $\int_a^b f(x)dx$ | Computing probabilities (CDF) |
| **Indefinite** | $\int f(x)dx = F(x) + C$ | Finding antiderivatives |
| **Integration by parts** | $\int u dv = uv - \int v du$ | Computing expectations of certain distributions |
| **Expectation** | $E[g(X)] = \int g(x)f(x)dx$ | Risk/loss computation |

---

### Common Mistakes

| Mistake | Why It's Wrong | Correct Approach |
|---|---|---|
| Using Pearson correlation for non-linear relationships | Only captures linear relationships | Use Spearman or mutual information |
| Ignoring the normality assumption for small-sample t-tests | Type I error inflation | Use non-parametric tests or bootstrap |
| P-value hacking without correction | Inflates false discovery rate | Bonferroni, FDR, Holm-Bonferroni |
| Confusing correlation with causation | Spurious relationships possible | Domain expertise, A/B testing |
| Using sample std dev without Bessel's correction | Biased estimate for population | Use ddof=1 in numpy |
| Forgetting KL divergence is asymmetric | Wrong conclusions about distance | Choose direction based on application |
| Assuming gradient descent always finds global minimum | Only guaranteed for convex functions | Use multiple random restarts |

### Best Practices

1. **Always visualize your data** before applying statistical tests
2. **Check assumptions** before using parametric tests
3. **Use effect sizes** in addition to p-values
4. **Log-transform** right-skewed data when appropriate
5. **Center and scale** data before PCA or distance computations
6. **Use double precision** for matrix operations
7. **Check matrix condition number** before inverting
8. **Prefer QR or SVD** over direct matrix inversion

### Interview Questions

1. Explain the Central Limit Theorem and why it matters for ML.
2. What's the difference between MLE and MAP?
3. How does SVD relate to PCA?
4. Derive the gradient of MSE loss for linear regression.
5. Why is the Hessian important for optimization?
6. How would you test whether two features are independent?
7. What's the difference between Type I and Type II errors?
8. Explain the bias-variance tradeoff in terms of model complexity.
9. How does Bayesian updating work?
10. Why is cosine similarity preferred over Euclidean distance for text data?

### Practical Exercises

1. Implement linear regression from scratch using both closed-form and gradient descent
2. Perform PCA on the Iris dataset using SVD
3. Carry out a complete A/B test analysis
4. Implement a naive Bayes classifier from scratch
5. Compute KL divergence between two fitted Gaussian distributions

### Mini Project

**Objective:** Build a statistical analysis toolkit with:
- Descriptive statistics for all measures
- Normality tests (Shapiro-Wilk, D'Agostino's K²)
- Hypothesis test selection function (auto-choose based on assumptions)
- Bootstrapped confidence intervals
- Outlier detection using IQR, Z-score, and MAD

### Revision Notes

| Topic | Key Takeaway | Formula/Code |
|---|---|---|
| Central Limit Theorem | Sample means → Normal regardless of population | $\bar{X} \sim N(μ, σ²/n)$ |
| Bayes Theorem | Update beliefs with evidence | $P(A|B) = P(B|A)P(A)/P(B)$ |
| SVD | Any matrix → UΣV^T | `np.linalg.svd(A)` |
| Gradient Descent | Step opposite to gradient | `θ -= η * ∇L(θ)` |
| MLE vs MAP | MAP = MLE × prior | MAP adds regularization |
| Entropy | Uncertainty measure | `scipy.stats.entropy(p, base=2)` |
| Covariance vs Correlation | Correlation is normalized covariance | Correlation ∈ [-1, +1] |
| Convexity | Hessian positive semidefinite | $x^T H x \geq 0$ |

---

## Chapter 83: Machine Learning Fundamentals

### Introduction

Machine learning is the systematic study of algorithms that improve through experience. This chapter establishes the universal workflow, evaluation principles, and best practices that apply across all ML paradigms.

### Why It Matters

Understanding fundamentals is like understanding the scientific method — it provides the framework within which all ML work is conducted. Skipping this leads to data leakage, improper validation, incorrect metric choice, and models that fail in production.

### Real World Analogy

Building ML models is like cooking: problem definition = choosing a dish, data collection = getting ingredients, data preparation = chopping/marinating, model selection = cooking method, training = cooking, evaluation = tasting, deployment = serving, monitoring = checking if guests enjoy it.

### Types of Machine Learning

| Type | Definition | Labels Required? | Examples |
|---|---|---|---|
| **Supervised** | Learn mapping X → y from labeled examples | Yes | Regression, classification |
| **Unsupervised** | Find hidden patterns in unlabeled data | No | Clustering, dimensionality reduction |
| **Semi-supervised** | Use small labeled + large unlabeled data | Partial | Pseudo-labeling |
| **Self-supervised** | Generate labels from data structure | No (auto-generated) | BERT, GPT, SimCLR |
| **Reinforcement Learning** | Learn actions to maximize cumulative reward | Reward signal | Game playing, robotics |

### ML Workflow

```
1. Define Problem → 2. Collect Data → 3. Explore (EDA) →
4. Prepare Data → 5. Model Selection → 6. Train (with CV) →
7. Evaluate (hold-out) → 8. Deploy → 9. Monitor (→ back to step 2)
```

### Train/Validation/Test Split

| Split | Purpose | Typical Ratio |
|---|---|---|
| **Training** | Model learns parameters | 60-80% |
| **Validation** | Hyperparameter tuning, model selection | 10-20% |
| **Test** | Final, unbiased evaluation | 10-20% |

```python
from sklearn.model_selection import train_test_split, StratifiedShuffleSplit

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)
```

### Cross-Validation

| Strategy | Description | Best For |
|---|---|---|
| **k-Fold** | Data → k equal folds | General purpose |
| **Stratified k-Fold** | Preserves class proportions | Imbalanced classification |
| **Leave-One-Out (LOO)** | Each sample is its own val set | Very small datasets |
| **Shuffled (Repeated)** | Shuffle + split multiple times | High-variance estimators |
| **Grouped** | Groups stay together | Multiple samples per subject |
| **TimeSeriesSplit** | Expanding window | Time series |

```python
from sklearn.model_selection import KFold, StratifiedKFold, cross_val_score

kf = KFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(model, X, y, cv=kf)
print(f"CV scores: {scores.mean():.4f} ± {scores.std():.4f}")
```

### Bias-Variance Tradeoff

$$\text{Total Error} = \text{Bias²} + \text{Variance} + \text{Irreducible Error}$$

| | High Bias | High Variance |
|---|---|---|
| **Meaning** | Underfitting | Overfitting |
| **Train error** | High | Low |
| **Test error** | High (close to train) | High (much > train) |
| **Fix** | More complex model, more features | Regularization, more data |

### Loss Functions

#### Regression

| Loss | Formula | Robust to Outliers? |
|---|---|---|
| **MSE** | $\frac{1}{n}\sum(y_i - \hat{y}_i)²$ | No |
| **MAE** | $\frac{1}{n}\sum|y_i - \hat{y}_i|$ | Yes |
| **Huber** | MSE for small errors, MAE for large | Yes |
| **Quantile** | $\sum ρ_τ(y_i - \hat{y}_i)$ | Yes |

#### Classification

| Metric | Formula | When to Use |
|---|---|---|
| **Accuracy** | (TP+TN)/(TP+TN+FP+FN) | Balanced classes |
| **Precision** | TP/(TP+FP) | Minimize false positives |
| **Recall** | TP/(TP+FN) | Minimize false negatives |
| **F1** | 2·P·R/(P+R) | Balance precision & recall |
| **ROC-AUC** | Area under TPR vs FPR | Overall ranking quality |
| **PR-AUC** | Area under precision-recall | Imbalanced datasets |
| **Log Loss** | $-\frac{1}{n}\sum[y\log\hat{y} + (1-y)\log(1-\hat{y})]$ | Probabilistic predictions |
| **MCC** | Matthews correlation coefficient | Unbiased for imbalanced |

```python
from sklearn.metrics import accuracy_score, precision_score, recall_score, f1_score, classification_report

y_true = [0, 1, 1, 0, 1, 0, 1, 0, 0, 1]
y_pred = [0, 1, 0, 0, 1, 0, 1, 1, 0, 1]

print(f"Accuracy: {accuracy_score(y_true, y_pred):.4f}")
print(f"Precision: {precision_score(y_true, y_pred):.4f}")
print(f"Recall: {recall_score(y_true, y_pred):.4f}")
print(f"F1: {f1_score(y_true, y_pred):.4f}")
print(classification_report(y_true, y_pred))
```

### Data Leakage

| Type | Description | Prevention |
|---|---|---|
| **Target Leakage** | Features contain info about target unavailable at prediction time | Remove leakage features |
| **Train-Test Contamination** | Test data influences training | Split before any preprocessing |
| **Time Leakage** | Future data used to predict past | Temporal split |

```python
# WRONG: scales whole dataset before split
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)  # LEAKAGE!

# CORRECT: scale within pipeline
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('model', LogisticRegression())
])
pipeline.fit(X_train, y_train)
```

### Imbalanced Data

| Technique | Description | When to Use |
|---|---|---|
| **SMOTE** | Synthetic interpolation between minority samples | Moderate imbalance |
| **ADASYN** | Adaptive SMOTE — focuses on harder samples | Difficult minority samples |
| **Class Weights** | Higher weight to minority class | Works with most sklearn models |
| **Undersampling** | Randomly remove majority samples | Large datasets |
| **Oversampling** | Duplicate minority samples | Small datasets |

### Pipelines

```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder

preprocessor = ColumnTransformer([
    ('num', StandardScaler(), numeric_features),
    ('cat', OneHotEncoder(), categorical_features)
])

pipeline = Pipeline([
    ('preprocessor', preprocessor),
    ('classifier', RandomForestClassifier())
])
pipeline.fit(X_train, y_train)
```

### Experiment Tracking

| Tool | When to Use |
|---|---|
| **Manual (CSV/dict)** | Quick experiments, personal projects |
| **MLflow** | Team projects, reproducible research |
| **Weights & Biases** | Research teams, hyperparameter sweeps |

### Common Mistakes

| Mistake | Correct Approach |
|---|---|
| Splitting after preprocessing | Split first, then transform |
| Using test set for tuning | Use validation set or CV |
| Ignoring class imbalance | Use stratified splits, class weights |
| Not shuffling time series | Use TimeSeriesSplit |
| Using accuracy for imbalanced data | Use precision, recall, F1, PR-AUC |
| Not fixing random seeds | Set `random_state` everywhere |

### Best Practices

1. **Establish a baseline** before complex models
2. **Fix random seeds** for reproducibility
3. **Log everything** — parameters, metrics, environment
4. **Validate assumptions** before fitting
5. **Start simple** — add complexity only when baseline is beaten
6. **Monitor for drift** in production

### Interview Questions

1. Explain the bias-variance tradeoff.
2. What is data leakage? Three examples?
3. How to evaluate on imbalanced data?
4. Compare cross-validation strategies.
5. What's the difference between supervised and self-supervised learning?

### Practical Exercises

1. Build a complete ML pipeline with preprocessing, feature selection, training, evaluation
2. Implement k-fold CV from scratch
3. Compare metrics on a highly imbalanced dataset
4. Create a learning curve plot
5. Set up MLflow for experiment tracking

### Mini Project

Build an ML pipeline for UCI Adult Income with: missing value handling, categorical encoding, scaling, ≥5 algorithms with CV, stratified splitting, MLflow logging, final model selection, full evaluation report.

### Revision Notes

| Concept | Key Idea |
|---|---|
| Train/Val/Test | Separate data for different purposes |
| Cross-validation | Systematic validation |
| Bias-Variance | Underfitting vs overfitting |
| Data Leakage | Test data influencing training |
| Imbalanced Data | Resample or reweight |
| Pipelines | Chained preprocessing + model |
| Experiment Tracking | Log everything |

---

## Chapter 84: Linear Models

### Introduction

Linear models are the foundation of machine learning. Despite simplicity, they remain powerful, interpretable, and often achieve state-of-the-art on high-dimensional sparse data.

### Why It Matters

Linear models are: **interpretable** — coefficients directly tell feature importance; **fast** — both training and inference at massive scale; **well-understood** — decades of statistical theory; **baseline models** — every project should start here.

### Real World Analogy

Linear regression is like following a straight-line trend: every additional year of experience adds $5,000 to salary: $\text{salary} = 50000 + 5000 \cdot \text{years}$.

---

### Linear Regression

#### Theory

$$y = β₀ + β₁x₁ + β₂x₂ + ... + βₙxₙ + ε = Xβ + ε$$

#### Assumptions

| Assumption | How to Check | What If Violated |
|---|---|---|
| **Linearity** | Residuals vs fitted plot | Add polynomial/interaction terms |
| **Independence** | Durbin-Watson test | Time series models |
| **Homoscedasticity** | Scale-location plot | Weighted least squares, robust SE |
| **Normality** | Q-Q plot, Shapiro-Wilk | Robust regression, bootstrapping |
| **No Multicollinearity** | VIF (>10 is problematic) | Ridge regression, remove features |

#### Ordinary Least Squares

$$β = (X^T X)^{-1} X^T y$$

```python
import statsmodels.api as sm
from sklearn.linear_model import LinearRegression

# sklearn
model = LinearRegression()
model.fit(X_train, y_train)

# statsmodels (with inference)
X_sm = sm.add_constant(X_train)
sm_model = sm.OLS(y_train, X_sm).fit()
print(sm_model.summary())
```

#### Regularization

| Method | Penalty | Effect |
|---|---|---|
| **Ridge (L2)** | $λ\sum β_j²$ | Shrinks coefficients, keeps all features |
| **Lasso (L1)** | $λ\sum |β_j|$ | Sparse solutions, feature selection |
| **Elastic Net** | $λ(α\|β\|₁ + (1-α)\|β\|₂²)$ | Both |

```python
from sklearn.linear_model import Ridge, Lasso, ElasticNet
from sklearn.model_selection import GridSearchCV

ridge_cv = GridSearchCV(Ridge(), {'alpha': [0.001, 0.01, 0.1, 1, 10]}, cv=5)
ridge_cv.fit(X_train, y_train)
print(f"Best alpha: {ridge_cv.best_params_['alpha']}")

lasso = Lasso(alpha=0.01)
lasso.fit(X_train, y_train)
n_selected = np.sum(lasso.coef_ != 0)
print(f"Lasso selected {n_selected} of {X_train.shape[1]} features")
```

#### Diagnostics

```python
from statsmodels.stats.outliers_influence import variance_inflation_factor
from statsmodels.stats.stattools import durbin_watson
import scipy.stats as stats

vif_data = pd.DataFrame({
    "feature": feature_names,
    "VIF": [variance_inflation_factor(X_train, i) for i in range(X_train.shape[1])]
})

residuals = y_test - model.predict(X_test)
dw = durbin_watson(residuals)
print(f"VIF:\n{vif_data}\nDurbin-Watson: {dw:.4f}")

stats.probplot(residuals, dist="norm", plot=plt)
```

### Logistic Regression

$$P(y=1|x) = \frac{1}{1 + e^{-(β₀ + β₁x₁ + ...)}}$$

```python
from sklearn.linear_model import LogisticRegression

lr = LogisticRegression(C=1.0, penalty='l2', solver='lbfgs')
lr.fit(X_train, y_train)
y_prob = lr.predict_proba(X_test)[:, 1]

# Multi-class
lr_multi = LogisticRegression(multi_class='multinomial', solver='lbfgs')
```

### Common Mistakes

| Mistake | Correct Approach |
|---|---|
| Not checking linear regression assumptions | Diagnostic plots, robust SE |
| Ignoring multicollinearity | VIF check, Ridge regression |
| Not scaling for Lasso/Ridge | StandardScaler before regularization |
| Setting C too small | Cross-validate C |

### Best Practices

1. Always fit a linear model first as baseline
2. Scale features for regularized models
3. Check VIF — remove if > 10
4. Use `statsmodels` for inference, `sklearn` for prediction
5. Plot residuals after every fit
6. Use Elastic Net when in doubt

### Interview Questions

1. Derive the OLS estimator. When is it BLUE?
2. Explain L1 vs L2 regularization geometrically.
3. Why is logistic regression called "regression" for classification?
4. How does the logit link map probabilities to real line?
5. What happens with perfect separation?

### Practical Exercises

1. Implement linear regression from scratch (closed form + GD)
2. Use polynomial regression with degree CV
3. Compare Ridge, Lasso, Elastic Net on multicollinear data
4. Logistic regression on text classification

### Mini Project

House price prediction: ≥10 features, EDA, multicollinearity check, polynomial features and interactions, compare OLS/Ridge/Lasso/Elastic Net, CV RMSE and R², coefficient interpretation, verify assumptions.

### Revision Notes

| Model | Formula | Key Parameter |
|---|---|---|
| Linear Regression | $y = Xβ + ε$ | — |
| Ridge | + $λ\|β\|₂²$ | alpha |
| Lasso | + $λ\|β\|₁$ | alpha |
| Elastic Net | + $λ(α\|β\|₁ + (1-α)\|β\|₂²)$ | alpha, l1_ratio |
| Logistic | $P(y=1) = σ(Xβ)$ | C |

---

## Chapter 85: Tree-Based Methods

### Introduction

Tree-based methods are the most versatile class of ML algorithms. From simple decision trees to gradient boosting, they handle non-linearity, missing data, mixed features, and dominate tabular data problems.

### Why It Matters

Tree-based models dominate Kaggle and industry tabular data: XGBoost was the dominant algorithm for years, Random Forest is robust and hard to beat out-of-the-box, LightGBM/CatBoost push efficiency further.

### Real World Analogy

A decision tree is like playing "20 Questions" — each question splits possibilities, narrowing down to a single answer.

### Decision Trees

#### CART Algorithm

Binary recursive partitioning: at each node, find best feature + threshold to split, recurse until stopping criterion.

#### Splitting Criteria

| Criterion | Classification | Regression |
|---|---|---|
| **Gini** | $1 - \sum p_k²$ | — |
| **Entropy** | $-\sum p_k \log₂ p_k$ | — |
| **MSE** | — | $\frac{1}{n}\sum(y_i - \bar{y})²$ |

```python
from sklearn.tree import DecisionTreeClassifier, plot_tree

dt = DecisionTreeClassifier(max_depth=5, min_samples_split=10, random_state=42)
dt.fit(X_train, y_train)

plot_tree(dt, filled=True, feature_names=feature_names, class_names=class_names)
```

#### Pruning

| Type | Parameters |
|---|---|
| **Pre-pruning** | `max_depth`, `min_samples_split`, `min_samples_leaf`, `max_features` |
| **Post-pruning** | `ccp_alpha` (cost-complexity pruning) |

### Random Forest

#### Bagging

1. Create B bootstrap samples
2. Train a tree on each
3. Average/vote predictions

```python
from sklearn.ensemble import RandomForestClassifier

rf = RandomForestClassifier(
    n_estimators=500, max_depth=15, max_features='sqrt',
    bootstrap=True, oob_score=True, n_jobs=-1, random_state=42
)
rf.fit(X_train, y_train)
print(f"OOB score: {rf.oob_score_:.4f}")

from sklearn.inspection import permutation_importance
perm_imp = permutation_importance(rf, X_test, y_test, n_repeats=10, random_state=42)
```

### Gradient Boosting Machines

Sequential learners correcting previous errors — gradient descent in function space:

$$F_{t+1}(x) = F_t(x) + η \cdot h_t(x)$$

| Algorithm | Key Innovation | Best For |
|---|---|---|
| **XGBoost** | Regularization, sparsity awareness | General purpose |
| **LightGBM** | GOSS + EFB, leaf-wise growth | Large datasets |
| **CatBoost** | Ordered boosting, categorical handling | Categorical features |

```python
import xgboost as xgb
import lightgbm as lgb
import catboost as cb

# XGBoost
xgb_model = xgb.XGBClassifier(
    n_estimators=1000, learning_rate=0.01, max_depth=6,
    subsample=0.8, colsample_bytree=0.8, early_stopping_rounds=50
)
xgb_model.fit(X_train, y_train, eval_set=[(X_val, y_val)], verbose=False)

# LightGBM
lgb_model = lgb.LGBMClassifier(
    num_leaves=31, learning_rate=0.01, subsample=0.8
)
lgb_model.fit(X_train, y_train, eval_set=[(X_val, y_val)],
              callbacks=[lgb.early_stopping(50)])

# CatBoost
cb_model = cb.CatBoostClassifier(iterations=1000, learning_rate=0.01)
cb_model.fit(X_train, y_train, eval_set=(X_val, y_val), verbose=False)
```

#### Feature Importance Types

| Type | Description |
|---|---|
| **Gain** | Total loss reduction from feature |
| **Cover** | Samples affected by splits on feature |
| **Frequency** | How often feature is used for splitting |
| **Permutation** | Drop in performance when shuffled |
| **SHAP** | Shapley values — consistent, model-agnostic |

### Common Mistakes

| Mistake | Correct Approach |
|---|---|
| `max_depth=None` without pruning | Set max_depth, min_samples_leaf |
| Not tuning learning_rate × n_estimators | Set high n_estimators, tune LR |
| Not setting early_stopping_rounds | Always use early stopping |
| Ignoring categorical encoding for XGBoost | Use one-hot, target encoding, or CatBoost |

### Best Practices

1. Random Forest as baseline — robust, parallel, hard to overfit
2. Gradient boosting as finisher — higher potential, needs tuning
3. Always use early stopping for boosting
4. Tune `learning_rate` before `n_estimators`
5. Use `colsample_bytree` < 1.0 for diversity
6. Monitor training vs validation loss

### Interview Questions

1. How does a tree choose the best split?
2. Gini vs entropy — does it matter?
3. How does RF reduce variance?
4. Explain gradient boosting step by step.
5. Compare XGBoost, LightGBM, CatBoost.
6. How do tree models handle missing values?
7. Explain SHAP values.

### Practical Exercises

1. Implement CART from scratch
2. Compare RF with different n_estimators and depths
3. Tune XGBoost with Optuna
4. Compare training time of XGBoost, LightGBM, CatBoost
5. Use SHAP to explain boosting predictions

### Mini Project

Kaggle-style competition: Decision Tree → RF → XGBoost → LightGBM → CatBoost, hyperparameter tuning for ≥3 models, compare CV scores, training/inference time, feature importance overlap, SHAP explanations.

### Revision Notes

| Algorithm | Type | Variance | Bias |
|---|---|---|---|
| Decision Tree | Single | High | Low |
| Random Forest | Bagging | Low | Low |
| XGBoost | Boosting | Medium | Very Low |
| LightGBM | Boosting | Medium | Very Low |
| CatBoost | Boosting | Medium | Very Low |

---

## Chapter 86: Support Vector Machines

### Introduction

SVMs find the maximum margin hyperplane separating classes. Theoretically elegant, powerful for high-dimensional data, and the kernel trick enables non-linear classification.

### Why It Matters

SVMs excel when: features >> samples (text, genomics), data is moderate size (< 100K), or you need strong theoretical guarantees.

### Real World Analogy

Imagine separating two types of balls on a table with the widest possible corridor. Balls closest to the corridor are "support vectors" — they alone determine the boundary.

### Maximum Margin Classifier

$$\text{Margin} = \frac{2}{\|w\|}$$

### Soft Margin SVM

$$\min \frac{1}{2}\|w\|² + C \sum ξ_i$$

**C parameter:** Large C → narrow margin, fewer violations; Small C → wide margin, more violations.

### Kernel Trick

| Kernel | Formula | When to Use |
|---|---|---|
| **Linear** | $x_i^T x_j$ | Text, high-dim sparse |
| **Polynomial** | $(γ x_i^T x_j + r)^d$ | Polynomial relationships |
| **RBF (Gaussian)** | $\exp(-γ\|x_i - x_j\|²)$ | Default, general purpose |
| **Sigmoid** | $\tanh(γ x_i^T x_j + r)$ | Rarely used |

```python
from sklearn.svm import SVC, NuSVC, SVR, LinearSVC
from sklearn.model_selection import GridSearchCV

svm = SVC(kernel='rbf', C=1.0, gamma='scale', probability=True)
svm.fit(X_train, y_train)

# Grid search
param_grid = {'C': [0.1, 1, 10, 100], 'gamma': ['scale', 'auto', 0.01, 0.1]}
grid = GridSearchCV(SVC(probability=True), param_grid, cv=5)
grid.fit(X_train, y_train)

# SVR
svr = SVR(kernel='rbf', C=1.0, epsilon=0.1)
svr.fit(X_train, y_train)
```

### Common Mistakes

| Mistake | Correct Approach |
|---|---|
| Not scaling features | StandardScaler |
| RBF without tuning C and γ | GridSearchCV |
| SVM on n > 100K | LinearSVC, SGDClassifier |
| Default C on noisy data | Cross-validate C |

### Best Practices

1. **Always scale features**
2. **Start with RBF kernel**
3. **Tune C and γ together** on log scale
4. **Use LinearSVC** for linear SVM on large data
5. **Use SGDClassifier(loss='hinge')** for very large data

### Interview Questions

1. Explain the maximum margin principle.
2. How does the kernel trick work?
3. What is the role of C?
4. Why are only support vectors important?
5. Explain RBF γ: what happens when very large or small?

### Practical Exercises

1. Implement linear SVM from scratch (hinge loss + GD)
2. Visualize decision boundaries for linear, polynomial, RBF
3. Tune SVM on MNIST subset
4. Compare SVR with linear regression

### Mini Project

SVM on non-linear dataset (moons, circles): compare linear, polynomial, RBF kernels, tune C and γ with grid search, visualize decision boundaries and support vectors, compare with RF and Logistic Regression.

### Revision Notes

| Concept | Key Parameter |
|---|---|
| Hard Margin | C → ∞ |
| Soft Margin | C |
| Kernel Trick | kernel='rbf' |
| RBF γ | gamma |
| Support Vectors | (determined by model) |
| Nu-SVM | nu |

---

## Chapter 87: Instance-Based Learning

### Introduction

Instance-based learning (lazy learning) memorizes training data and predicts based on similarity to stored instances. kNN is the canonical example.

### Why It Matters

kNN is non-parametric, simple (only k), theoretically grounded (error ≤ 2× Bayes optimal), and often competitive on small datasets.

### Real World Analogy

To classify a new point, ask its k nearest neighbors. If 2 friends are programmers and 1 is a designer, the new person is a programmer.

### k-Nearest Neighbors

```python
from sklearn.neighbors import KNeighborsClassifier, KNeighborsRegressor

knn = KNeighborsClassifier(
    n_neighbors=5, weights='distance',
    metric='minkowski', p=2, algorithm='auto'
)
knn.fit(X_train, y_train)
```

### Distance Metrics

| Metric | Formula | Use Case |
|---|---|---|
| **Euclidean (L2)** | $\sqrt{\sum(x_i - y_i)²}$ | Continuous, default |
| **Manhattan (L1)** | $\sum\|x_i - y_i\|$ | High D, robust |
| **Minkowski** | $(\sum\|x_i - y_i\|^p)^{1/p}$ | Generalization |
| **Hamming** | $\sum \textbf{1}(x_i \neq y_i)$ | Binary/categorical |
| **Cosine** | $1 - \frac{x \cdot y}{\|x\|\|y\|}$ | Text |

### Choosing k

| Small k | Large k |
|---|---|
| Low bias, high variance | High bias, low variance |
| Noisy decision boundary | Smooth decision boundary |

### Efficient Search

| Algorithm | Complexity | Best For |
|---|---|---|
| **Brute Force** | O(n) | n < 1000 |
| **KD Tree** | O(log n) | d < 20 |
| **Ball Tree** | O(log n) | d < 100 |

### Curse of Dimensionality

In high dimensions, all points become equidistant. kNN fails when d is large. Rule of thumb: n >> 2^d.

### Common Mistakes

| Mistake | Correct Approach |
|---|---|
| Not scaling features | StandardScaler |
| Using k=1 on noisy data | Use larger k or distance weighting |
| Euclidean for categorical features | Hamming distance |
| kNN on high-D data | PCA first |

### Best Practices

1. Always scale features
2. Use odd k for binary classification
3. Weight by distance
4. Reduce dimensionality if d > 20
5. Use Ball Tree for high D

### Interview Questions

1. What is the curse of dimensionality and kNN?
2. How to choose optimal k?
3. Why is kNN called "lazy"?
4. How does distance weighting improve kNN?

### Practical Exercises

1. Implement kNN from scratch
2. Find optimal k using elbow plot and CV
3. Compare KD Tree vs Ball Tree
4. Visualize feature scaling effects on decision boundaries

### Mini Project

Recommendation system with kNN collaborative filtering: user-item matrix, find k nearest users/items based on rating similarity, predict ratings, evaluate with RMSE, compare cosine vs Euclidean.

### Revision Notes

| Concept | Parameter |
|---|---|
| k (neighbors) | n_neighbors |
| Distance weighting | weights |
| Distance metric | metric |
| Curse of Dimensionality | Reduce D first |

---

## Chapter 88: Unsupervised Learning

### Introduction

Unsupervised learning finds hidden structure in unlabeled data — patterns, groups, and latent representations without target variables.

### Why It Matters

Most data is unlabeled. Unsupervised learning is essential for EDA, dimensionality reduction, anomaly detection, and recommendation systems.

### Clustering

#### k-Means

```python
from sklearn.cluster import KMeans
from sklearn.metrics import silhouette_score

kmeans = KMeans(n_clusters=5, init='k-means++', n_init=10, random_state=42)
kmeans.fit(X)
labels = kmeans.labels_

silhouette = silhouette_score(X, labels)
print(f"Silhouette: {silhouette:.4f}")
```

**Limitations:** Spherical clusters, sensitive to initialization, requires k, sensitive to outliers.

#### Evaluation Metrics

| Metric | Range | Best |
|---|---|---|
| **Inertia (WCSS)** | [0, ∞) | Lower |
| **Silhouette** | [-1, 1] | Higher (close to 1) |
| **Davies-Bouldin** | [0, ∞) | Lower |
| **Calinski-Harabasz** | [0, ∞) | Higher |

#### Hierarchical Clustering

```python
from sklearn.cluster import AgglomerativeClustering
from scipy.cluster.hierarchy import dendrogram, linkage

agg = AgglomerativeClustering(n_clusters=5, linkage='ward')
labels = agg.fit_predict(X)

Z = linkage(X, method='ward')
dendrogram(Z)
```

#### DBSCAN

```python
from sklearn.cluster import DBSCAN

dbscan = DBSCAN(eps=0.5, min_samples=5)
labels = dbscan.fit_predict(X)
n_clusters = len(set(labels)) - (1 if -1 in labels else 0)
n_noise = list(labels).count(-1)
```

**Advantages:** No k needed, arbitrary shapes, identifies outliers.

#### Gaussian Mixture Models

```python
from sklearn.mixture import GaussianMixture

gmm = GaussianMixture(n_components=3, covariance_type='full', random_state=42)
gmm.fit(X)
labels = gmm.predict(X)
probs = gmm.predict_proba(X)
bic = gmm.bic(X)
aic = gmm.aic(X)
```

### Dimensionality Reduction

#### PCA

```python
from sklearn.decomposition import PCA

pca = PCA(n_components=0.95)
X_pca = pca.fit_transform(X)
print(f"Explained variance: {pca.explained_variance_ratio_.cumsum()}")
```

#### t-SNE

```python
from sklearn.manifold import TSNE

tsne = TSNE(n_components=2, perplexity=30, learning_rate=200, random_state=42)
X_tsne = tsne.fit_transform(X)
```

> **Important:** t-SNE is for **visualization only** — not for feature extraction or inference.

#### UMAP

```python
import umap

reducer = umap.UMAP(n_neighbors=15, min_dist=0.1, n_components=2)
X_umap = reducer.fit_transform(X)
```

#### LDA

```python
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis

lda = LinearDiscriminantAnalysis(n_components=2)
X_lda = lda.fit_transform(X, y)
```

### Anomaly Detection

```python
from sklearn.ensemble import IsolationForest
from sklearn.neighbors import LocalOutlierFactor
from sklearn.covariance import EllipticEnvelope

# Isolation Forest
iso_forest = IsolationForest(contamination=0.1, random_state=42)
anomaly_labels = iso_forest.fit_predict(X)

# LOF
lof = LocalOutlierFactor(n_neighbors=20, contamination=0.1)
anomaly_labels = lof.fit_predict(X)

# Elliptic Envelope
envelope = EllipticEnvelope(contamination=0.1, random_state=42)
anomaly_labels = envelope.fit_predict(X)
```

### Common Mistakes

| Mistake | Correct Approach |
|---|---|
| Choosing k without domain knowledge | Elbow + silhouette + domain |
| Euclidean for high-D | PCA first, cosine distance |
| Interpreting t-SNE distances | t-SNE distorts global structure |
| Scaling after PCA | Scale before PCA |
| Expecting k-Means to find non-spherical clusters | DBSCAN, spectral clustering |

### Best Practices

1. Always scale features before clustering
2. Use elbow + silhouette + domain for k
3. Try multiple algorithms
4. Visualize with PCA or t-SNE
5. Use GMM for soft assignments
6. Never use t-SNE except for visualization
7. Use Isolation Forest for high-D anomaly detection

### Interview Questions

1. How does k-Means initialization affect results?
2. Explain EM algorithm for GMM.
3. When to use DBSCAN over k-Means?
4. How does PCA relate to SVD?
5. t-SNE vs PCA?
6. How to detect anomalies?
7. Explain silhouette score.

### Practical Exercises

1. Implement k-Means from scratch
2. Compare k-Means, DBSCAN, hierarchical on synthetic data
3. PCA on MNIST, visualize in 2D
4. Isolation Forest for fraud detection
5. Compare t-SNE and UMAP

### Mini Project

Customer segmentation: RFM features, k-Means, DBSCAN, Hierarchical, GMM. Evaluate with silhouette, Davies-Bouldin. Visualize with PCA + t-SNE. Profile each cluster. Build function to assign new customers.

### Revision Notes

| Algorithm | # Clusters | Shape | Outliers |
|---|---|---|---|
| k-Means | Required | Spherical | Sensitive |
| DBSCAN | Automatic | Arbitrary | Robust |
| Hierarchical | Required (cut) | Any | Semi-robust |
| GMM | Required (BIC/AIC) | Elliptical | Soft |
| Spectral | Required | Arbitrary | Sensitive |

---

## Chapter 89: Model Evaluation and Selection

### Introduction

Choosing and evaluating models properly is the most critical step. Without rigorous evaluation, you cannot know if your model generalizes or memorizes.

### Why It Matters

Overfitting is #1 in applied ML. Model selection with improper evaluation gives false confidence. Business decisions depend on it.

### Cross-Validation Strategies

| Strategy | sklearn Class | Use Case |
|---|---|---|
| **k-Fold** | `KFold` | General |
| **Stratified k-Fold** | `StratifiedKFold` | Imbalanced |
| **Group k-Fold** | `GroupKFold` | Multi-per-subject |
| **Time Series Split** | `TimeSeriesSplit` | Temporal |
| **Repeated k-Fold** | `RepeatedKFold` | Low variance |

```python
from sklearn.model_selection import cross_validate, cross_val_score

scoring = ['accuracy', 'precision', 'recall', 'f1']
scores = cross_validate(model, X_train, y_train, cv=5, scoring=scoring)

for metric in scoring:
    print(f"{metric}: {scores['test_'+metric].mean():.4f} ± {scores['test_'+metric].std():.4f}")
```

### Hyperparameter Tuning

| Method | Pro | Con |
|---|---|---|
| **GridSearchCV** | Guarantees best in grid | Curse of dimensionality |
| **RandomizedSearchCV** | Faster, often better | May miss optimum |
| **Bayesian Optimization** | Efficient | Complex setup |
| **HalvingGridSearchCV** | Efficient for large grids | Needs careful setup |

```python
from sklearn.model_selection import GridSearchCV, RandomizedSearchCV
from scipy.stats import randint

param_grid = {'n_estimators': [100, 200, 500], 'max_depth': [5, 10, 15, None]}
grid = GridSearchCV(RandomForestClassifier(), param_grid, cv=5, n_jobs=-1)
grid.fit(X_train, y_train)

param_dist = {'n_estimators': randint(100, 1000), 'max_depth': [3, 5, 10, None]}
random = RandomizedSearchCV(RandomForestClassifier(), param_dist, n_iter=50, cv=5)
random.fit(X_train, y_train)
```

### Learning Curves

```python
from sklearn.model_selection import learning_curve

train_sizes, train_scores, val_scores = learning_curve(
    model, X, y, cv=5, train_sizes=np.linspace(0.1, 1.0, 10)
)
```

- High bias: both curves converge low, small gap
- High variance: large gap between train and validation

### Calibration

```python
from sklearn.calibration import CalibratedClassifierCV, calibration_curve

calibrated = CalibratedClassifierCV(base_estimator=svm, method='sigmoid', cv=5)
calibrated.fit(X_train, y_train)

fraction, mean_pred = calibration_curve(y_test, prob_pos, n_bins=10)
```

### Model Interpretability

#### Global

```python
from sklearn.inspection import permutation_importance, PartialDependenceDisplay

perm_imp = permutation_importance(model, X_test, y_test, n_repeats=10)
PartialDependenceDisplay.from_estimator(model, X_train, ['feature_0'])
```

#### Local

```python
import shap

explainer = shap.TreeExplainer(model)
shap_values = explainer.shap_values(X_test)
shap.summary_plot(shap_values, X_test, feature_names=feature_names)
shap.force_plot(explainer.expected_value, shap_values[0], X_test[0])
```

### Common Mistakes

| Mistake | Correct Approach |
|---|---|
| Tuning on test set | Use validation set or CV |
| Using only accuracy | Use multiple metrics |
| Not fixing random seed | Set random_state |
| Looking only at training | Compare train vs validation |
| Not calibrating probabilities | Use CalibratedClassifierCV |

### Best Practices

1. Hold out test set at beginning, never touch until final
2. Use nested CV with feature selection
3. Report multiple metrics
4. Always look at learning curves
5. Use calibration for accurate probabilities
6. Explain predictions with SHAP or LIME

### Interview Questions

1. Explain nested cross-validation.
2. How to diagnose over/underfitting?
3. Grid vs randomized search?
4. How to compare two models statistically?
5. What is calibration and why important?
6. SHAP vs LIME?

### Practical Exercises

1. Compare GridSearchCV vs RandomizedSearchCV
2. Plot learning curves for high-variance model
3. Use SHAP to explain XGBoost predictions
4. Perform nested CV with feature selection
5. Calibrate SVM probabilities

### Mini Project

Model selection bake-off: ≥6 algorithms, proper CV, tune each with randomized search (100 iterations), compare with learning curves, calibration curves, ROC curves, create comparison table, use SHAP to identify key features.

### Revision Notes

| Step | What | Why |
|---|---|---|
| CV | Cross-validation | Stable performance estimate |
| Grid/Random | Hyperparameter tuning | Find best configuration |
| Learning Curves | Bias-variance diagnosis | Understand fit |
| Calibration | Adjust probabilities | Accurate uncertainty |
| SHAP/LIME | Explain predictions | Interpretability |

---

## Chapter 90: Feature Selection

### Introduction

Feature selection identifies the most relevant features. It reduces overfitting, improves interpretability, and speeds up training.

### Why It Matters

Irrelevant features add noise, correlated features cause multicollinearity, more features require more data (curse of dimensionality), simpler models are easier to maintain.

### Real World Analogy

Packing for a trip: only essentials. Packing everything makes luggage heavy and you won't use most of it.

### Filter Methods

| Method | Measures | Use For |
|---|---|---|
| **Correlation** | Linear relationship | Regression |
| **Chi-Square** | Categorical dependence | Classification |
| **Mutual Information** | Any relationship | Both |
| **ANOVA F-test** | Variance ratio | Classification |
| **Variance Threshold** | Feature variance | Both |

```python
from sklearn.feature_selection import SelectKBest, chi2, f_classif, mutual_info_classif, VarianceThreshold

selector = VarianceThreshold(threshold=0.01)
X_high_var = selector.fit_transform(X)

selector = SelectKBest(score_func=mutual_info_classif, k=20)
X_selected = selector.fit_transform(X, y)
```

### Wrapper Methods

| Method | Description | Complexity |
|---|---|---|
| **RFE** | Recursive elimination | O(n × m) |
| **Forward Selection** | Add best iteratively | O(p²) |
| **Backward Elimination** | Remove worst iteratively | O(p²) |

```python
from sklearn.feature_selection import RFE, RFECV

selector = RFECV(RandomForestClassifier(), step=1, cv=5, scoring='accuracy')
selector.fit(X, y)
print(f"Optimal features: {selector.n_features_}")
```

### Embedded Methods

| Method | Mechanism |
|---|---|
| **Lasso (L1)** | Shrinks coefficients to 0 |
| **RF Importance** | Impurity-based ranking |
| **XGBoost Gain** | Loss reduction from splits |

### Boruta

```python
from boruta import BorutaPy

boruta = BorutaPy(RandomForestClassifier(), n_estimators='auto', random_state=42)
boruta.fit(X.values, y.values)
confirmed = np.where(boruta.support_)[0]
```

### Common Mistakes

| Mistake | Correct Approach |
|---|---|
| Selecting before train-test split | Select within CV loop |
| Single model importance | Use multiple methods |
| Ignoring interactions | Use wrapper or tree methods |
| Arbitrary correlation thresholds | Domain + statistical tests |

### Best Practices

1. Filter methods first — quickly remove irrelevant features
2. Compare multiple selection methods
3. Use RFECV — robust, built into sklearn
4. Validate with domain experts
5. Consider interactions
6. Feature + model selection in nested CV

### Interview Questions

1. Compare filter, wrapper, embedded methods
2. How does Lasso perform feature selection?
3. Feature selection vs dimensionality reduction?
4. Explain Boruta algorithm.
5. How to handle feature selection with data leakage?

### Practical Exercises

1. Compare filter, wrapper, embedded on 100+ feature dataset
2. Implement forward selection from scratch
3. RFECV for logistic regression
4. Apply Boruta
5. Feature selection pipeline with nested CV

### Mini Project

Feature selection pipeline: start with 50+ features, apply filter (correlation, MI, variance), wrapper (RFECV), embedded (Lasso, RF importance). Venn diagram of overlap. Train final model on selected features. Show fewer features = better performance.

### Revision Notes

| Method | Type | Speed | Quality |
|---|---|---|---|
| Variance Threshold | Filter | Very Fast | Low |
| Mutual Information | Filter | Fast | Medium |
| RFE | Wrapper | Slow | High |
| Forward Selection | Wrapper | Slow | High |
| Lasso | Embedded | Fast | Medium-High |
| RF Importance | Embedded | Fast | High |
| Boruta | Hybrid | Medium | Very High |

---

## Chapter 91: Ensemble Methods

### Introduction

Ensemble methods combine multiple models for better predictions than any single model. Foundation of state-of-the-art performance across ML competitions.

### Why It Matters

Ensembles reduce variance (bagging), reduce bias (boosting), improve accuracy (stacking), and are more robust than individual models.

### Real World Analogy

A single doctor may misdiagnose, but a panel of specialists consulted together is likely more accurate.

### Voting

| Type | Description |
|---|---|
| **Hard Voting** | Majority vote |
| **Soft Voting** | Weighted average of probabilities |

```python
from sklearn.ensemble import VotingClassifier

voting_clf = VotingClassifier(estimators=[
    ('lr', LogisticRegression()),
    ('rf', RandomForestClassifier()),
    ('svm', SVC(probability=True))
], voting='soft', weights=[1, 2, 1])
voting_clf.fit(X_train, y_train)
```

### Bagging Variants

| Algorithm | Data | Features |
|---|---|---|
| **Bagging** | Bootstrap | All |
| **Pasting** | No replacement | All |
| **Random Patches** | Bootstrap | Random subsets |
| **Random Subspaces** | All | Random subsets |

```python
from sklearn.ensemble import BaggingClassifier

bagging = BaggingClassifier(
    estimator=DecisionTreeClassifier(),
    n_estimators=500, max_samples=0.8, bootstrap=True, oob_score=True
)
bagging.fit(X_train, y_train)
```

### Stacking

```python
from sklearn.ensemble import StackingClassifier

stack = StackingClassifier(
    estimators=[
        ('knn', KNeighborsClassifier(n_neighbors=5)),
        ('dt', DecisionTreeClassifier(max_depth=5)),
        ('svm', SVC(probability=True))
    ],
    final_estimator=LogisticRegression(),
    cv=5, stack_method='predict_proba'
)
stack.fit(X_train, y_train)
```

### Blending

Manual stacking using hold-out set for meta-features.

### Boosting Comparison

| Algorithm | Innovation |
|---|---|
| **AdaBoost** | Sample weighting |
| **Gradient Boosting** | Gradient descent in function space |
| **XGBoost** | Regularization, sparsity |
| **LightGBM** | GOSS + EFB, leaf-wise |
| **CatBoost** | Ordered boosting, categorical |

### Comparison: Bagging vs Boosting vs Stacking

| Aspect | Bagging | Boosting | Stacking |
|---|---|---|---|
| **Goal** | Reduce variance | Reduce bias | Max accuracy |
| **Base Models** | High variance | Weak learners | Diverse |
| **Training** | Parallel | Sequential | Two-stage |
| **Overfitting** | Low | High (tune carefully) | Medium |
| **Example** | Random Forest | XGBoost | Meta-Learner |

### Common Mistakes

| Mistake | Correct Approach |
|---|---|
| Identical base models in stacking | Diverse models |
| Stacking without CV for level-1 | Use CV for meta-features |
| Too many base models | Simple meta-model (LogReg) |
| Boosting without early stopping | Use early_stopping_rounds |

### Best Practices

1. Diversity is key — diverse accurate models = better ensemble
2. Simple meta-models — Logistic Regression is often best
3. Parallelize bagging — `n_jobs=-1`
4. Early stopping for boosting
5. Check correlation of base model errors

### Interview Questions

1. Bagging vs boosting vs stacking?
2. Why does bagging reduce variance?
3. How does AdaBoost weight samples?
4. What makes a good model for stacking?
5. Why is ensemble diversity important?
6. How does gradient boosting fit trees to residuals?

### Practical Exercises

1. Implement AdaBoost from scratch
2. Build stacking ensemble with 5 diverse base models
3. Compare bagging, boosting, stacking on same dataset
4. Analyze ensemble size vs performance
5. Hard vs soft voting comparison

### Mini Project

Production-grade ensemble: base models (LR, kNN, DT, RF, XGBoost), ensembles (hard voting, soft voting, stacking, blending), compare all on test set, analyze error diversity (correlation matrix), optimize stacking meta-model, deploy best ensemble as single pipeline.

### Revision Notes

| Method | How It Works | Pros | Cons |
|---|---|---|---|
| Voting | Aggregate predictions | Simple | No learning |
| Bagging | Parallel + avg | Reduces variance | No bias reduction |
| Boosting | Sequential + correct | Reduces bias | Can overfit |
| Stacking | Meta on base outputs | Can beat all | Complex, leakage risk |
| Blending | Hold-out meta features | Simple stacking | Less data |

---

> *"The most important thing in machine learning is not the algorithm — it's having clean, relevant data and a rigorous evaluation framework. The algorithm is just the last 10%." — End of Part 12*