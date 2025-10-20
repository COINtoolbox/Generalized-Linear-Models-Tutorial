
---

### 🐍 `PyMC.md`
```markdown
# PyMC Implementation

The Python version reproduces the NIMBLE model in **PyMC v5** with optional
**JAX acceleration**.

---

## Fast model (EIV + NB2)
```python
with pm.Model() as model_fast:
    beta0 = pm.Normal("beta0", 0, 10)
    beta1 = pm.Normal("beta1", 0, 10)
    phi   = pm.Gamma("phi", 2, 0.1)
    alpha = pm.Deterministic("alpha", 1/phi)

    x_true = pm.Normal("x_true", mu=x_obs, sigma=x_err, shape=N)
    mu = pm.Deterministic("mu", pm.math.exp(beta0 + beta1 * x_true))

    pm.NegativeBinomial("N_GC", mu=mu, alpha=alpha, observed=N_GC)
    mu_pred = pm.Deterministic("mu_pred", pm.math.exp(beta0 + beta1 * x_pred))

    idata = pm.sample(draws=3000, tune=1500, chains=3, cores=3,
                      target_accept=0.9, random_seed=42)
