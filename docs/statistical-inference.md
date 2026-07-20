---
layout: default
title: Statistical Inference
description: GUNDAM User Guide Explanatory Documentation
---

# Statistical Inference

## 1. Overall Statistical Inference Workflow

GUNDAM statistical inference connects the fit parameters to the model prediction and evaluates how well that prediction describes the selected data. During a fit, the numerical minimizer or MCMC sampler proposes a parameter point. GUNDAM then updates the Monte Carlo prediction, compares it with the data, adds the parameter constraints, and returns a single objective value.

In the code and output, this quantity is often called the **likelihood** or **LLH**. More precisely, GUNDAM evaluates a test statistic that generally follows a $$-2\log L$$-like convention for the standard likelihood choices. A numerical minimizer searches for the parameter point that gives the smallest value, while MCMC converts the same quantity into a probability for sampling.

A typical evaluation follows this sequence:

1. The active fit parameters receive new values.
2. The corresponding dials and systematic responses are updated.
3. The affected Monte Carlo events are reweighted.
4. The prediction histograms are rebuilt.
5. The prediction is compared with the data.
6. The statistical and parameter-penalty contributions are added.
7. The total objective is returned to the minimizer or sampler.

During ordinary minimization or sampling, the **model prediction** changes at each tested point, while the prepared **data histograms** normally remain fixed.

---

## 2. Objective Function

The total objective contains a statistical contribution and a parameter-penalty contribution:

$$
Q_{\text{total}}
=
Q_{\text{stat}}
+
Q_{\text{penalty}}
$$

The **statistical contribution** measures the agreement between the predicted and observed histogram contents. The **penalty contribution** represents prior constraints on the fit parameters.

For each configured sample, GUNDAM compares the model and data histograms bin by bin. The contributions from all samples and bins are added to form the total statistical term.

The comparison is controlled by the configured **joint-probability model**. The available choices include:

- **PoissonLLH**, which uses the Poisson deviance and is the standard choice for binned event-count data.
- **Chi2**, which uses a Pearson chi-squared approximation.
- **LeastSquares**, which uses a squared-difference statistic and is mainly intended for debugging or specialized studies.
- **BarlowLLH variants**, which include the statistical uncertainty of the Monte Carlo prediction.
- **Plugin**, which allows a user-provided statistical evaluation function.

For the Poisson model, the bin contribution is

$$
Q_{\text{Poisson}}
=
2\left[
N_{\text{pred}}
-
N_{\text{data}}
+
N_{\text{data}}
\ln\left(
\frac{N_{\text{data}}}{N_{\text{pred}}}
\right)
\right]
$$

For the Pearson chi-squared model,

$$
Q_{\chi^2}
=
\frac{
\left(
N_{\text{pred}}-N_{\text{data}}
\right)^2
}{
N_{\text{pred}}
}
$$

GUNDAM provides several **Barlow-Beeston variants**. These implementations account for finite Monte Carlo statistics, but they do not all use identical approximations or numerical treatments. Users should select the implementation required by the validated configuration of their analysis rather than assuming that the variants are interchangeable.

Parameters with Gaussian prior constraints contribute a penalty when they move away from their prior values. For correlated parameters, the penalty has the form

$$
Q_{\text{penalty}}
=
\left(
\mathbf{p}-\mathbf{p}_0
\right)^T
V^{-1}
\left(
\mathbf{p}-\mathbf{p}_0
\right)
$$

Here, $$\mathbf{p}$$ is the current parameter vector, $$\mathbf{p}_0$$ contains the prior values, and $$V$$ is the prior covariance matrix. Flat-prior parameters do not contribute this Gaussian penalty.

Parameter limits and prior uncertainties serve different purposes. A parameter may remain inside its allowed range while still receiving a large penalty because it is far from its prior value.

---

## 3. Likelihood Evaluation Workflow

The main components involved in one likelihood evaluation are the **FitterEngine**, the **Propagator**, and the **Likelihood Interface**.

The **FitterEngine** organizes the overall inference procedure. It initializes the fit, runs the selected minimization or sampling method, manages optional scans and diagnostics, and coordinates the writing of results.

The **Propagator** applies the current parameter values to the Monte Carlo prediction. It updates the relevant dials, recalculates event weights, and rebuilds the predicted histograms. Its role is to produce the updated model prediction; it does not calculate the total likelihood by itself.

The **Likelihood Interface** compares the updated prediction with the data, evaluates the parameter penalty, combines the two contributions, and returns the total objective to the inference method.

The model and data are handled separately. Depending on the analysis configuration, the data side may contain **real data**, an **Asimov dataset**, or a **toy dataset**. Once the data histograms have been prepared, they normally remain unchanged while the model prediction is repeatedly updated during the fit.

---

## 4. Inference Methods

GUNDAM supports two main inference strategies: **numerical minimization** and **MCMC sampling**. They use the same objective-evaluation workflow, but they use the result differently.

### Numerical Minimization

`RootMinimizer` uses the ROOT minimization framework to search for the parameter point that minimizes the total objective $$Q_{\text{total}}$$. A typical configuration uses **Minuit2** with the **Migrad** algorithm.

The minimization workflow may also include:

- an optional **Simplex** step before the main minimization;
- an optional **Hesse** error calculation;
- an optional **Minos** error calculation.

Simplex is a pre-minimization option rather than a required part of every fit. Hesse and Minos are alternative uncertainty-estimation methods and should not be described as two mandatory stages that always run one after another.

The ROOT-based workflow can provide best-fit parameter values, fit-status information, parameter uncertainties, covariance matrices, and correlation matrices.

### MCMC Sampling

`SimpleMcmc` samples the parameter distribution instead of only locating one minimum. It converts the GUNDAM objective into a log probability according to

$$
\log P
=
-\frac{1}{2}
Q_{\text{total}}
$$

Because the parameter penalty is already included in the total objective, the prior constraints automatically contribute to the sampled probability.

The MCMC configuration controls the burn-in, the number of sampling cycles and steps, the proposal strategy, and the proposal covariance. The main result is a Markov chain of sampled parameter points. Posterior intervals, correlations, and other summaries are obtained by analyzing the saved chain.

**Hesse**, **Minos**, ROOT EDM, and ROOT covariance extraction belong to the numerical-minimization workflow and do not apply directly to `SimpleMcmc`.

---

## 5. Diagnostics and Outputs

GUNDAM can produce additional diagnostics and output objects to help users evaluate and interpret the fit result.

The **Parameter Scanner** evaluates the objective while varying a selected parameter across a configured range. At each point, the model is propagated again and the resulting objective is recorded.

In a standard one-parameter scan, the other parameters remain fixed. These curves are therefore **fixed-parameter scans**, not profile-likelihood scans. A profile-likelihood scan would require the remaining nuisance parameters to be re-minimized at every scan point.

Parameter scans may be performed before or after the fit and can record the total objective, the statistical contribution, the parameter penalty, and optional sample-level or bin-level quantities.

For ROOT minimization, **Hesse** or **Minos** may be used after a successful fit to estimate parameter uncertainties. For MCMC, uncertainty information is instead obtained from the sampled parameter distribution.

Depending on the configuration, a GUNDAM inference run may save:

- pre-fit and post-fit parameter values;
- total, statistical, and penalty objective values;
- pre-fit and post-fit model histograms;
- covariance and correlation matrices;
- parameter-scan graphs;
- MCMC chains;
- likelihood-history information;
- optional diagnostic plots; and
- optional event-level outputs.

These outputs allow users to validate the fit, compare pre-fit and post-fit predictions, inspect parameter constraints, and perform later statistical analysis.

---

[Back to Documentation Overview]({{ '/' | relative_url }})
