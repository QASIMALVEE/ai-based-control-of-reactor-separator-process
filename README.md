# AI-Based Modeling, Optimization and Control of a Reactor-Separator Process

B.Tech Chemical Engineering, AI in Chemical Engineering course project (Roll No. B23CH1027).

The project builds data-driven models of a reactor-separator process and then uses them for optimization and control. The process has two continuous stirred-tank reactors (CSTRs) in series and a flash tank with recycle. Reactant A converts to the desired product B, which further converts to by-product C (A -> B -> C). The process model follows the Koopman modeling paper arXiv:2408.02315.

## Project stages

| Stage | Work done |
|---|---|
| 1. Modeling | A first-principles model (9 ODEs and flash equations) generates steady-state and dynamic datasets. Linear regression, an MLP and a TSK fuzzy model are trained on both. |
| 2. Optimization | The steady-state MLP is used as a surrogate inside a genetic algorithm to maximize the product B rate in the purge stream and to minimize heating duty. Optima are checked against the first-principles model. |
| 3. Control | A model predictive controller uses the dynamic NARX model as its predictor and a genetic algorithm as its solver. It is compared with a decentralized PI controller. |

## Main results

- **Steady-state modeling:** the MLP gave the best test R2 (0.988), against 0.671 for linear regression and 0.720 for the TSK fuzzy model.
- **Dynamic modeling:** one-step predictions are very accurate, but multi-step open-loop rollout drifts (RMSE about 12 on a held-out trajectory).
- **Optimization, Case A (maximize B rate):** the surrogate predicts 354 kg/h, but the first-principles model gives 325.5 kg/h at that point, slightly below the 327.9 kg/h at the nominal operating point. Surrogate error is larger than the predicted gain.
- **Optimization, Case B (minimize heating duty):** total duty is 11.6 % lower than in Case A, with the B rate within 1.9 % of its target on the first-principles model.
- **Control:** the MPC tracked a setpoint step with lower error than a tuned PI controller (12 to 47 % lower IAE), but was worse at rejecting a feed-temperature disturbance.

The reports in `reports/` give the full formulation, figures and discussion.

## Repository structure

```
.
├── notebooks/   Jupyter notebook with all three stages
├── data/        steady-state and dynamic datasets (CSV)
├── models/      trained models saved with joblib
├── reports/     Stage 2 and Stage 3 reports (PDF and Word)
├── latex/       LaTeX sources and figures for the reports
├── requirements.txt
└── README.md
```

## Running the notebook

1. Install the dependencies: `pip install -r requirements.txt`
2. Open `notebooks/reactor_separator_modeling_optimization_control.ipynb` in Jupyter or Google Colab.
3. Run all cells from top to bottom. The optimization and control sections reuse the models fitted in the modeling section, so the order matters.

The notebook regenerates the datasets and models in its working directory. The saved `.joblib` files contain a custom TSK fuzzy class, so they can only be loaded where that class is defined (as it is in the notebook).

The dynamic dataset uses 12 randomly generated input schedules within the input bounds. Set `INTERACTIVE = True` in the dynamic-input cell to enter your own schedules.

To build the reports from the LaTeX sources: `cd latex && pdflatex Stage2_Optimization_Report.tex` (run twice), and likewise for the Stage 3 file.

## References

- arXiv:2408.02315, Koopman modeling of the reactor-separator process (main reference).
- arXiv:2408.03653 and arXiv:2404.06706, companion papers used for the process equations and parameters.
- Course lecture material on AI/ML in process modeling and evolutionary optimization.
