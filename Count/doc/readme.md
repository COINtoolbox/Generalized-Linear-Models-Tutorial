# Bayesian Negative Binomial Regression for Globular Cluster Populations

This repository accompanies [*de Souza et al. (2015)*]{https://academic.oup.com/mnras/article/453/2/1928/1154495} and updates the example with
modern Bayesian workflows in **NIMBLE** and **PyMC**.  
It models the relation between globular-cluster counts and galaxy stellar mass
using Negative Binomial regression with **Errors-in-Variables (EIV)**.

---

## 🧭 Repository structure

- [📗 NIMBLE.md](NIMBLE.md) – canonical R/NIMBLE implementation from the paper  
- [🐍 PyMC.md](PyMC.md) – Python/PyMC version with optional JAX acceleration  

---

## 1️⃣ Conceptual background

### 1.1 Poisson GLM
The classical Poisson Generalized Linear Model assumes **equidispersion**:
\[
\mathrm{Var}(Y_i)=\mathbb{E}[Y_i]=\mu_i.
\]
This is often violated in astrophysical count data, where unmodeled
heterogeneity inflates the variance.

### 1.2 Negative Binomial (NB2)
The NB2 model generalizes the Poisson by introducing an **over-dispersion**
parameter \( \phi \):
\[
Y_i \sim \mathrm{NB2}(\mu_i,\phi), \qquad
\mathbb{E}[Y_i]=\mu_i,\quad
\mathrm{Var}(Y_i)=\mu_i+\frac{\mu_i^2}{\phi}.
\]
When \( \phi \to \infty \), the Poisson model is recovered.

An equivalent hierarchical form is a **Gamma–Poisson mixture**:
\[
g_i\sim\Gamma(\phi, \text{rate}=\phi/\mu_i),\qquad
Y_i\sim\mathrm{Poisson}(g_i),
\]
which makes the source of extra-Poisson variability explicit.

---

## 2️⃣ Model extensions

### 2.1 Errors-in-Variables (EIV)
In this application, the predictor  
\( x=\log_{10}(M_\star/M_\odot) \)  
is derived from photometry and suffers observational uncertainty.  
We therefore model a latent true predictor:

\[
x_i^{\text{true}} \sim \mathcal N(x_i^{\text{obs}}, \sigma_{x,i}), \qquad
\eta_i = \beta_0 + \beta_1 x_i^{\text{true}},\qquad
\mu_i = e^{\eta_i}.
\]

This avoids slope attenuation (“regression dilution”).

### 2.2 Two-layer count model
Observed catalog counts may also be uncertain.
We separate the intrinsic population process from the measurement process:

\[
N_i^{\text{true}} \sim \mathrm{NB}(\mu_i,\phi),\qquad
N_i^{\text{obs}} \sim \mathcal N(N_i^{\text{true}}, \sigma_{N,i}).
\]

For high-quality counts, a single-layer NB likelihood is usually sufficient and much faster.

---

## 3️⃣ Parameterization and priors

| Parameter | Distribution | Notes |
|------------|---------------|-------|
| \( \beta_0, \beta_1 \) | Normal(0, 10) | Weakly informative slope/intercept |
| \( \phi \) | Gamma(2, 0.1) | Large φ → Poisson limit |
| \( x_i^{\text{true}} \) | Normal(x_obs, σ_x) | EIV layer |
| Likelihood | NB2(μ, φ) or NB2⊗Normal | Over-dispersed counts |

PyMC uses the form  
\(\mathrm{Var}(Y)=\mu + \alpha\,\mu^2\),  
so we define \( \alpha = 1/\phi \).

---

## 4️⃣ Inference and computation

- **All-continuous model** ⇒ NUTS or JAX-accelerated NUTS.  
- **Discrete latents** (if you keep \(N_{\text{true}}\)) ⇒ mixture of NUTS + Metropolis → slow.  
- **Marginalized NB⊗Normal** → continuous again, NUTS-only.

### Performance tips
| Technique | Gain |
|------------|------|
| Drop measurement error on \(N\) | 5–20× faster |
| Reduce grid (e.g. M=1000) | ≈4× less memory |
| JAX `numpyro_nuts` vectorized chains | 2–5× faster on CPU/GPU |
| `delim_whitespace=True` when reading COIN CSV | avoids parsing errors |

---

## 5️⃣ Visualization: pseudo-log transform

To visualize zero counts and high dynamic range on one axis, we use the
**base-10 pseudo-log** transform:
\[
y' = \operatorname{asinh}\!\left(\frac{(y/5)}{\ln 10}\right),
\]
which is linear near zero and logarithmic at large \(y\).

Tick labels correspond to 0, \(10^1\), \(10^2\), \(10^3\), \(10^4\), \(10^5\).

---

## 6️⃣ Elliptical galaxies subset

Restrict the analysis to ellipticals (e.g., for morphology-specific scaling):

```python
df = df[df["Type"].astype(str).str.startswith("E")].copy()
