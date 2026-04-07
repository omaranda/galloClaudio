---
name: bayesian-network
description: Design and implement a Bayesian network or hierarchical Bayesian model for ecological data — occupancy models, species detection, abundance estimation, or belief propagation networks.
---

You are acting as the data-scientist agent. Design and implement a Bayesian model.

**Step 1 — Define the ecological question:**
- What quantity are we estimating? (occupancy probability, detection probability, abundance, habitat suitability)
- What data are available? (detection histories, counts, covariates, spatial coordinates)
- What is the observation process? (camera trap triggers, point count, audio detections)
- Are there hierarchical groupings? (sites within landscapes, species within guilds)

**Step 2 — Choose model type:**

**Occupancy model** (imperfect detection):
```
True state: z_i ~ Bernoulli(psi_i)   # site i is occupied
Observation: y_ij ~ Bernoulli(z_i * p_ij)  # j-th visit detects species
logit(psi_i) = alpha + X_i @ beta_occ
logit(p_ij) = gamma + W_ij @ beta_det
```

**N-mixture (abundance)**:
```
N_i ~ Poisson(lambda_i)  # true abundance at site i
y_ij ~ Binomial(N_i, p_ij)  # observed count
log(lambda_i) = alpha + X_i @ beta
logit(p_ij) = gamma + W_ij @ delta
```

**Bayesian Belief Network** (discrete variables):
```
Define DAG: nodes = variables, edges = conditional dependencies
P(X1, X2, ..., Xn) = product of P(Xi | parents(Xi))
```

**Step 3 — Implement in PyMC:**
```python
import pymc as pm
import numpy as np

with pm.Model() as occupancy_model:
    # Priors
    alpha_psi = pm.Normal("alpha_psi", mu=0, sigma=1)
    beta_psi = pm.Normal("beta_psi", mu=0, sigma=1, shape=n_occ_covars)
    alpha_p = pm.Normal("alpha_p", mu=0, sigma=1)

    # Occupancy probability
    logit_psi = alpha_psi + pm.math.dot(X_occ, beta_psi)
    psi = pm.Deterministic("psi", pm.math.invlogit(logit_psi))

    # True occupancy state (latent)
    z = pm.Bernoulli("z", p=psi, shape=n_sites)

    # Detection probability
    logit_p = alpha_p
    p = pm.Deterministic("p", pm.math.invlogit(logit_p))

    # Likelihood
    y_obs = pm.Bernoulli("y_obs", p=z[site_idx] * p, observed=y)

    # Sample
    trace = pm.sample(2000, tune=1000, target_accept=0.9, return_inferencedata=True)
```

**Step 4 — Diagnostics:**
```python
import arviz as az
# Convergence
az.summary(trace, var_names=["alpha_psi", "beta_psi", "alpha_p"])
# R-hat must be < 1.01, ESS > 400 for all parameters
az.plot_trace(trace)

# Posterior predictive check
with occupancy_model:
    ppc = pm.sample_posterior_predictive(trace)
az.plot_ppc(az.from_pymc(posterior_predictive=ppc, observed_data={"y_obs": y}))
```

**Step 5 — Prior sensitivity:**
Run model with 2-3 different prior specifications; results should be stable if data is informative.

**Step 6 — Provide:**
- Complete PyMC model code
- Prior justification (ecological or uninformative)
- Diagnostic plots and interpretation guide
- Posterior summary with ecological interpretation
- Predictions on new sites/conditions
