
---

### 🧩 `NIMBLE.md`
```markdown
# NIMBLE Implementation

## Overview
This is the canonical Negative-Binomial regression with 
Errors-in-Variables (EIV) from *de Souza et al.* (2015).
It models the intrinsic relation between globular-cluster counts 
and galaxy stellar mass using R/NIMBLE.

## Model Summary
- Predictor: \(x = \log_{10}(M_\star/M_\odot)\)  
- Likelihood: two-layer  
  \(N_\mathrm{true} \sim \mathrm{NB2}(\mu,\phi)\)  
  \(N_\mathrm{obs} \sim \mathcal{N}(N_\mathrm{true},\,\sigma_N)\)
- EIV: \(x_\mathrm{true} \sim \mathcal{N}(x_\mathrm{obs},\,\sigma_x)\)

```r
# run_nb_eiv_mstar_nimble.R
code <- nimbleCode({
  beta0 ~ dnorm(0, sd=10)
  beta1 ~ dnorm(0, sd=10)
  phi   ~ dgamma(shape=2, rate=0.1)

  for(i in 1:N){
    x_true[i] ~ dnorm(x_obs[i], sd=x_err[i])
    eta[i] <- beta0 + beta1 * x_true[i]
    mu[i]  <- exp(eta[i])
    p[i]   <- phi / (phi + mu[i])
    N_true[i] ~ dnbinom(size=phi, prob=p[i])
    N_GC[i]   ~ dnorm(mean=N_true[i], sd=N_GC_err[i])
  }

  for(j in 1:M){
    eta_pred[j] <- beta0 + beta1 * x_pred[j]
    mu_pred[j]  <- exp(eta_pred[j])
    p_pred[j]   <- phi / (phi + mu_pred[j])
    N_pred[j]   ~ dnbinom(size=phi, prob=p_pred[j])
  }
})
