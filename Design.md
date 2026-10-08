# Meta-Statistician Agents: Design

## Purpose

Build an agentic statistical workflow that helps statisticians explore data, formulate and evaluate models, and produce reliable code. The system should improve the statistician's efficiency and support creative investigation; it must not replace the statistician or move them outside the decision loop.

The initial modelling focus is prediction. Causal diagrams may be used to express hypotheses, but the system must not present causal conclusions without a separately designed causal-inference workflow.

## Design Principles

- Keep raw data interaction separate from statistical reasoning. Pass compact, structured summaries between components instead of placing large datasets in an agent's context.
- Separate data exploration from model construction.
- Separate model validation from statistical critique: validation checks defined criteria, while critique examines assumptions, robustness, and limitations.
- Separate statistical modelling from implementation and coding.
- Keep the human statistician informed and involved in consequential choices, including evaluation strategy, diagnostic criteria, and model changes.
- Do not optimise models against in-sample fit or significance thresholds. Use predictive evaluation appropriate to the analysis and agreed with the statistician.

## Workflow

Model development is iterative, not a one-shot generation task. The workflow follows a closed loop inspired by the Bayesian workflow:

1. Explore the data and record its structure, quality, and relevant summaries.
2. Formulate candidate hypotheses and model specifications.
3. Generate or update probabilistic-programming code and fit the model.
4. Check sampler health and validate predictions.
5. Critique fit, assumptions, sensitivity, and limitations.
6. Present findings and proposed refinements to the statistician. Apply changes only after approval, then repeat the relevant checks.

Candidate refinements might include a robust likelihood (for example, Student-t errors), a model for heteroscedasticity, or hierarchical partial pooling. These are suggestions to assess in context, not automatic defaults.

## Specialised Agents

- **Orchestrator:** Routes work, moderates collaboration, tracks decisions, and ensures the statistician is consulted and kept informed.
- **Data Interactor:** Connects to data through an appropriately isolated environment, queries it, and returns metadata and compact structured reports. It can also run approved data checks and posterior predictive checks.
- **Data Explorer:** Performs exploratory data analysis and reports patterns, data-quality concerns, and useful visualisations.
- **Hypothesis Generator:** Suggests candidate relationships and features. It may sketch DAGs as hypotheses, but does not infer causality from them.
- **Statistical Modeller:** Proposes statistical model structures and priors, and generates or revises probabilistic-programming code (for example, Stan or PyMC).
- **Model Validator:** Runs agreed validation procedures, including predictive checks and sensitivity analyses such as comparing results across records or cohorts.
- **Statistical Critic:** Assesses assumptions, robustness, interpretation, and limitations; it should remain distinct from mechanical validation.
- **Model Explainer:** Explains model behaviour and predictions using suitable interpretation methods, such as SHAP values, partial dependence, or feature effects. It reports method assumptions and limitations, distinguishes association from causation, and does not treat feature attribution as evidence of causal influence.
- **Responsible AI (RAI) Reviewer:** Applies appropriate tools and agreed criteria to assess the model for responsible AI risks, including fairness, privacy, transparency, safety, and potential impacts on affected groups. It documents evidence, gaps, and mitigations for human review; an assessment is not a claim of legal or regulatory certification.
- **Code Safety Reviewer:** Reviews Python package choices and implementation practices for security and maintainability risks, including known vulnerabilities, package provenance and licensing, unsafe APIs, and handling of data and secrets. It recommends safer alternatives and reports unresolved risks before code is run or shipped.
- **Coder:** Implements approved analysis and supporting software, keeping implementation concerns separate from statistical decisions.

## Evaluation and Diagnostics

Evaluation strategy and diagnostic criteria are selected case by case in consultation with the statistician. The system should explain the implications of the selected approach and report the criteria used; the values below are candidate reference points, not universal hard gates.

### Predictive evaluation

- Prefer out-of-sample predictive evaluation using strictly proper scoring rules, such as negative log predictive density (NLPD) or expected log pointwise predictive density (ELPD).
- Choose an appropriate validation strategy with the statistician. Options include a protected held-out set and Pareto-smoothed importance sampling leave-one-out cross-validation (PSIS-LOO), depending on the data, modelling goal, and computational cost.
- Avoid repeatedly adapting a model to a final test set. Where a held-out set is used during iteration, distinguish it from a final protected test set.
- For PSIS-LOO, report Pareto $k$ diagnostics and investigate influential observations or unstable importance sampling. A value below 0.7 is a common reference point, not a universal pass condition.
- Use posterior predictive checks to compare simulated data with observed data using discrepancies relevant to the model and analysis question.
- Track predictive performance across iterations. A stopping rule, such as stopping after several iterations without improvement, should be agreed with the statistician and should account for uncertainty in score differences.

### Sampling diagnostics

Parse and report sampler diagnostics before interpreting posterior summaries. Criteria should be chosen with the statistician for the model and use case. Reference diagnostics include:

- Rank-normalised R-hat: often expected to be below 1.05, with below 1.01 a stricter target.
- Bulk and tail effective sample sizes (ESS): values such as 400 are useful reference points, but required ESS depends on the quantity being estimated.
- Divergent transitions after warm-up: report their count and investigate any divergences.
- Energy diagnostics, including Bayesian fraction of missing information (BFMI).

When diagnostics or predictive checks raise concerns, the agents may explain likely causes and propose targeted remedies, such as a non-centred parameterisation or sampler tuning. They must obtain the statistician's approval before changing model code or adopting a revised model. Preserve the ability to compare with the previous version and report what changed.

## Execution, Memory, and Outputs

- **Data execution:** A containerised Python or REPL environment is a candidate boundary for querying data and running approved checks. The sandbox and permission model remain to be decided.
- **Analysis memory:** Keep a lightweight, human-readable analysis wiki using simple text files. Record decisions, assumptions, model versions, diagnostics, and results so that work can be resumed and reviewed.
- **Visual outputs:** Save plots as image files and consider interactive visualisations where they aid inspection. Choose formats according to the user's workflow; animated GIFs are not a default requirement.
- **Parallel execution:** The execution and coordination approach is undecided. A terminal multiplexer is one option to investigate.

## Decisions to Resolve

- Which probabilistic programming language or languages should be supported first (for example, Stan or PyMC)?
- Should model specifications use a dedicated statistical-model DSL, and if so, which concepts should it express?
- What sandbox, data-access permissions, and execution limits should apply to the Data Interactor?
- What parallel execution and agent coordination mechanism should be used?
- What report schema and visualisation formats should agents use when handing off results?

## Research Sources

- Gelman et al. (2020), [Bayesian Workflow](https://arxiv.org/abs/2011.01808).
- Dürr (2026), [AutoStan: Autonomous Bayesian Model Improvement via Predictive Feedback](https://arxiv.org/abs/2603.27766); [code repository](https://github.com/tidit-ch/autostan).
- Li et al. (2024), [Automated Statistical Model Discovery with Language Models](https://arxiv.org/abs/2402.17879).
- Farhang et al. (2024), [AgentBayes: Open-Ended Scientific Model Discovery](https://arxiv.org/abs/2409.09359).
- Kanda et al., [REFINESTAT: Efficient Exploration for Probabilistic Program Synthesis](https://openreview.net/forum?id=8ExXncFpf6).
- Monnahan, NOAA, [Toward Good Practices for Bayesian Data-Rich Fisheries Stock Assessments Using a Modern Statistical Workflow](https://repository.library.noaa.gov/).