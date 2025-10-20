# Bayesian Negative Binomial Regression for Globular Cluster Populations

This repository accompanies [*de Souza et al. (2015)*](https://academic.oup.com/mnras/article/453/2/1928/1154495) and updates the example with
modern Bayesian workflows in **NIMBLE** and **PyMC**.  
It models the relation between globular-cluster counts and galaxy stellar mass
using Negative Binomial regression with **Errors-in-Variables (EIV)**.

---

## 🧭 Repository structure

- [📗NB nimble](https://github.com/COINtoolbox/Generalized-Linear-Models-Tutorial/blob/master/Count/scripts/NegBin_nimble.R) – canonical R/NIMBLE implementation from the paper  
- [🐍 NB PyMC](https://github.com/COINtoolbox/Generalized-Linear-Models-Tutorial/blob/master/Count/scripts/negbin_pymc.py) – Python/PyMC version

---

##  Conceptual background

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

## Model extensions

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

## Parameterization and priors

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
## Bibtex entry

If you use this tutorial in your research, we kindly as you to cite the original paper:

[de Souza, R. S.,  *et al.*,  The overlooked potential of generalized linear models in astronomy - III. Bayesian negative binomial regression and globular cluster populations, MNRAS, vol. 453, p.1928-1940](http://adsabs.harvard.edu/abs/2015MNRAS.453.1928D)

The corresponding bibitex entry is:

```

@ARTICLE{2015MNRAS.453.1928D,
   author = {{de Souza}, R.~S. and {Hilbe}, J.~M. and {Buelens}, B. and {Riggs}, J.~D. and 
	{Cameron}, E. and {Ishida}, E.~E.~O. and {Chies-Santos}, A.~L. and 
	{Killedar}, M.},
    title = "{The overlooked potential of generalized linear models in astronomy - III. Bayesian negative binomial regression and globular cluster populations}",
  journal = {\mnras},
archivePrefix = "arXiv",
   eprint = {1506.04792},
 primaryClass = "astro-ph.IM",
 keywords = {methods: data analysis, methods: statistical, globular clusters: general},
     year = 2015,
    month = oct,
   volume = 453,
    pages = {1928-1940},
      doi = {10.1093/mnras/stv1825},
   adsurl = {http://adsabs.harvard.edu/abs/2015MNRAS.453.1928D},
  adsnote = {Provided by the SAO/NASA Astrophysics Data System}
}
```
