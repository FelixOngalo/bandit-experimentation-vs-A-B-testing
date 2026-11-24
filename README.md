# Bandits vs A/B Testing in R: When Does Adaptive Experimentation Pay Off?

A fixed A/B test is clean but wasteful: half of the visitors see the loser for the whole test. A multi-armed bandit shifts traffic toward winners as it learns, but it distorts the data. This project compares them in a replicated simulation (regret, false positives, drifting rates) and then shows what adaptive allocation does to the estimates you get afterwards.

## Question

Four landing-page variants, 2,000 visitors a day, 28 days. How many conversions does Thompson sampling save compared with an even split, does it still find the best variant, what does daily peeking do to false positives, what happens when the best variant changes mid-test, and how far can you trust the lift estimates after an adaptive test?

## Key findings

*(Fill in after running the scripts. Suggested: regret and conversions per policy in the small-gaps scenario; share of traffic that reached the best variant; probability of correctly declaring the winner; the false-positive rate of daily peeking against the fixed test; the drift results for ordinary and discounted Thompson sampling; the size of the post-bandit bias in the arm rates and in the best arm's lift; whether the day-of-week adjustment helps.)*

## What is in it

- **Replicated simulation** (1,000 experiments per policy and scenario) of five policies: fixed A/B, A/B with daily peeking, epsilon-greedy, Thompson sampling and discounted Thompson sampling, with batch (daily) updating.
- **Three scenarios**: small true differences, no difference at all (false positives), and drift (best variant changes mid-test).
- **Metrics**: conversions, regret with standard error, traffic share to the best arm, probability of declaring a winner and of declaring the *right* winner, false-positive rate, traffic share to the best arm in the last 7 days.
- **A realistic traffic log** from one Thompson-sampling experiment, with bot traffic, duplicate rows, a logging outage and inconsistent labels, cleaned and analysed: frequentist intervals, Bayesian posteriors, P(best) and expected loss.
- **Weekday confounding**: a weekend conversion lift interacts with the changing traffic split; a day-adjusted logistic regression is compared with naive pooled rates.
- **Estimation bias after adaptive experiments**: bias in each arm's rate, overstatement of the best arm's lift, and coverage of the naive confidence interval.
- **A Quarto report** with the method, charts, practical guidance and limitations.

## Tools

R, dplyr, tidyr, tibble, ggplot2, knitr, Quarto. Only base R is used for the sampling, bandit and test logic.

## Reproduce

```r
install.packages(c("dplyr", "tidyr", "tibble", "ggplot2", "knitr"))
source("R/00_simulate_data.R")
source("R/01_analyse_one_experiment.R")
source("R/02_policy_simulation.R")     # 14 batches of 1,000 experiments, allow a few minutes
source("R/03_estimation_bias.R")
quarto::quarto_render("reports/report.qmd")
```

Run from the project root (open `bandit-vs-ab-testing.Rproj`). The seeds are fixed, so results are reproducible.

## Project structure

```
bandit-vs-ab-testing/
├── R/
│   ├── bandit_functions.R             # policies, decisions, scenarios, log simulator and cleaning
│   ├── 00_simulate_data.R
│   ├── 01_analyse_one_experiment.R
│   ├── 02_policy_simulation.R
│   └── 03_estimation_bias.R
├── data/
│   ├── raw/                           # experiment_log.csv (generated)
│   └── clean/                         # .rds results (generated)
├── reports/report.qmd
├── bandit-vs-ab-testing.Rproj
└── README.md
```

## Limitations

- Immediate binary conversions; no delayed or revenue outcomes.
- No context or personalisation (no contextual bandits).
- One traffic level and one update frequency.
- A simple best-versus-runner-up test stands in for a full multi-arm procedure.
- Flat Beta(1, 1) priors throughout.
