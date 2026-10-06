# key priciples
- isolate interaction with the data from the statistical thinking so as not to burden the context window with the data
- isolate data exploration from data modelling
- have separate validation and critique funtionalities
- separate the codeing from the modelling
- the human statisticain shoud lbe desined into the loop and not ontop of the loop - these are eficneciy and cretivity enhances not replacment for the human

## Design Requirements for "Anchor in the Iterative Bayesian Workflow":

1. Closed-Loop Structural Lifecycle (Box's Loop)
- The agent must not treat statistical modeling as a one-shot prompt or single generation task.
- It must execute an iterative, closed control loop: Generative Model Building -> MCMC Parameter Sampling -> Model Criticism & Predictive Checks -> Diagnostic-Aware Refinement.

2. Decoupled Interactor–Modeler Architecture
- Separate empirical data exploration and model checking from probabilistic program synthesis.
- Interactor Agent: Operates inside a containerized Python/REPL sandbox to query raw data, compute summary statistics, and execute posterior predictive checks.
- Modeler Agent: Consumes only compact, structured JSON/Markdown reports from the Interactor to generate, edit, and repair PPL code (e.g., Stan or PyMC), preventing context window saturation on large datasets.

3. Automated MCMC Diagnostic Guardrails
- Before parameter posteriors or predictive metrics can be interpreted or trusted, the system must automatically parse sampler outputs and enforce rigid health thresholds:
  * Gelman-Rubin Statistic: Potential scale reduction factor R-hat < 1.05 (ideally < 1.01) across all parameters.
  * Effective Sample Size: Bulk ESS > 400 and Tail ESS > 400 (or > 100 for relaxed tail bounds).
  * Divergent Transitions: Exactly 0 divergent Hamiltonian integration steps post-warmup.
  * Energy Diagnostics: Bayesian Fraction of Missing Information (BFMI) checks.
- Automated Remediation: If diagnostics fail, the workflow must trigger targeted code re-parameterizations (e.g., switching centered parameterizations to non-centered funnel geometries or increasing adapt_delta).

4. Out-of-Sample Predictive Validation & Model Criticism
- Evaluate candidate specifications on held-out test data using strictly proper scoring rules, specifically Negative Log Predictive Density (NLPD) or Pareto-Smoothed Importance Sampling Leave-One-Out (PSIS-LOO / ELPD-LOO) cross-validation.
- Pareto k Diagnostic: Verify that the shape parameter k-hat < 0.7 for all observations to ensure importance sampling stability.
- Posterior Predictive Checks (PPCs): Simulate synthetic data from parameter posteriors and run quantitative discrepancy tests against empirical data to detect systematic misfit.

5. Diagnostic-Aware Refinement & Reversion
- Automatically revert structural edits or resample likelihood/prior components if out-of-sample predictive scores worsen or MCMC diagnostics fail.
- Progressive Model Expansion: Gradually introduce robust likelihoods (e.g., Student-t for outliers), heteroscedastic noise functions, or hierarchical partial pooling based on diagnostic feedback.

Primary Source URLs :

- Bayesian Workflow (Gelman et al., 2020):
  https://arxiv.org/abs/2011.01808

- AutoStan: Autonomous Bayesian Model Improvement via Predictive Feedback (Oliver Dürr, 2026):
  https://arxiv.org/abs/2603.27766
  Code repository: https://github.com/tidit-ch/autostan

- Automated Statistical Model Discovery with Language Models (Michael Y. Li et al., 2024):
  https://arxiv.org/abs/2402.17879

- AgentBayes: Open-Ended Scientific Model Discovery (Alex Farhang et al., 2024):
  https://arxiv.org/abs/2409.09359

- REFINESTAT: Efficient Exploration for Probabilistic Program Synthesis (Madhav Kanda et al.):
  https://openreview.net/forum?id=8ExXncFpf6

- Toward Good Practices for Bayesian Data-Rich Fisheries Stock Assessments Using a Modern Statistical Workflow (Cole C. Monnahan, NOAA):
  https://repository.library.noaa.gov/

---

## Design Requirements for "Optimize for Out-of-Sample Predictive Scoring (Avoid p-Hacking)":
Strict Prohibition of In-Sample and Significance-Based Metrics
System prompts and reward structures must explicitly forbid optimizing candidate models based on in-sample fit metrics (such as R-squared, training-set MSE, or raw log-likelihood) or p-value thresholds (such as p < 0.05).
Reason: Unconstrained, iterative LLM optimization loops rewarded on in-sample fit or p-values engage in automated data dredging and p-hacking, picking up spurious correlations and misfitting noise.
Enforce Strictly Proper Scoring Rules
Candidate models must be evaluated and ranked exclusively using strictly proper scoring rules, primarily Negative Log Predictive Density (NLPD) or Expected Log Pointwise Predictive Density (ELPD).
Proper scoring rules reward true, well-calibrated predictive probability distributions rather than point estimates, penalizing both predictive mean errors and miscalibrated uncertainty.
Out-of-Sample Validation & PSIS-LOO Cross-Validation
Held-Out Data Splits: Where possible, evaluate models by computing NLPD on a protected held-out test split.
Within-Sample Surrogate (PSIS-LOO): When re-fitting models across N test folds is computationally prohibitive, evaluate candidates using Pareto-Smoothed Importance Sampling Leave-One-Out (PSIS-LOO / ELPD-LOO) cross-validation computed directly from log-likelihood draws.
Stability & Outlier Diagnostics (Pareto k-hat)
Monitor the generalized Pareto shape parameter (k-hat) for every observation during PSIS-LOO evaluation.
Threshold: Require k-hat < 0.7 across all data points to ensure importance sampling stability.
Action: If k-hat > 0.7 for specific observations, trigger targeted structural revisions (e.g., replacing Gaussian error models with heavy-tailed Student-t distributions or mixture likelihoods).
Predictive Improvement Stopping Criteria
The autonomous search loop must track out-of-sample predictive density trajectory over iterations.
Automatically halt search when out-of-sample predictive scores cease to improve (e.g., after 3 consecutive non-improving iterations) to prevent mild test-set adaptation or overfitting to the evaluation split.
Primary Source URLs for Copy-Pasting:
AutoStan: Autonomous Bayesian Model Improvement via Predictive Feedback (Oliver Dürr, 2026): https://arxiv.org/abs/2603.27766 Code repository: https://github.com/tidit-ch/autostan
Automated Statistical Model Discovery with Language Models (Michael Y. Li et al., 2024): https://arxiv.org/abs/2402.17879
REFINESTAT: Efficient Exploration for Probabilistic Program Synthesis (Madhav Kanda et al., 2024): https://openreview.net/forum?id=8ExXncFpf6
AgentBayes: Open-Ended Scientific Model Discovery (Alex Farhang et al., 2024): https://arxiv.org/abs/2409.09359
Bayesian Workflow (Andrew Gelman, Aki Vehtari et al., 2020): https://arxiv.org/abs/2011.01808
Toward Good Practices for Bayesian Data-Rich Fisheries Stock Assessments Using a Modern Statistical Workflow (Cole C. Monnahan, NOAA): https://repository.library.noaa.gov/

#Agents
- *orchastrator*  - moderator routing and group chat moderator, ensuring the human statisticain is consulted and informed
- *Data Interactor*: connects to the data, queries it and returns summaries and metadata, synthsises the results into a compact jason report
- *Data Explorere* - expert in EDA
- *Hyposieiser* - hyptohesies with variables and features could expalin the targer etc, suggest causal DAGs
- *Statistical modeller*: forming statistical models
- *Model evaluator*: apply tests to the model such how does it change if records or cohorts are dropped and such
- * Statistical critiqu*: evaluator of the robustnes of the model
- *coder*

# Parallesiation
- terminal multiplexer


# memory management
- simpel text files forming an analysis wiki
- png graphs or intractable graphs, animated giffs

#to look at
Stat model DSL [domain spesific languadge]