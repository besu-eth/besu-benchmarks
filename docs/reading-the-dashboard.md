# Reading the Dashboard

The dashboard lives at [besu-eth.github.io/besu-benchmarks](https://besu-eth.github.io/besu-benchmarks/). It has four tabs, each answers a different question.

## Latest Run

Answers: how does the newest run look, and did anything change since the run before it?

![Latest Run tab](images/latest-run.png)

The meta bar at the top names the run being shown and, next to Compared against, the baseline it is measured against. The baseline is always the most recent earlier run with the same ref, a latest main run compares against the previous latest main run, a pinned run against the previous pinned run. Because of this the tab alternates through the day, after the midnight run it shows the main suite, after the 8:00 and 16:00 runs it shows the pinned suite, which contains fewer benchmarks since the pinned commit is older.

Columns:

| Column | Meaning |
|---|---|
| Score (ns/op) | Average execution time, lower is better |
| MGas/s | Gas throughput derived from the score, higher is better, dash when the benchmark is not gas wired |
| ± Error | JMH's confidence half width for this score, large relative to the score means an untrustworthy measurement |
| vs Previous | Percentage change against the baseline run |
| Status | ok, regression above 5%, faster, or noisy when the error ranges of the two runs overlap and the difference cannot be trusted |

Clicking any row opens the trend panel for that benchmark below the table.

## Trend Chart

Answers: how has one benchmark behaved over time?

![Trend chart](images/trend-chart.png)

Pick a benchmark, or arrive here by clicking a row in Latest Run. Three views:

- **Line** plots the score of the last 20 runs. A lasting step up is a regression, a single spike that returns to normal the next run is an anomaly.
- **Box plot** groups the raw measurements by day, one box per day pooling all of that day's runs. Clicking a day zooms into its individual runs, clicking an individual run opens a bell curve fitted to that run's raw measurements, a tall narrow curve means the run was quiet, a low wide one means it was disturbed. Red dots on any box are single measurements that landed unusually far from the rest of that run, the standard Tukey rule.
- **MGas/s** shows the same box plot view in throughput terms for gas wired benchmarks.

The Nightly runs only checkbox is on by default and keeps manual one off runs out of the chart, untick it to see them.

## Compare Runs

Answers: how do two specific runs compare, benchmark by benchmark?

![Compare runs](images/compare-runs.png)

Pick run A as the baseline and run B as the candidate, press Compare. Each benchmark present in both runs shows its two scores and the speedup or slowdown between them. Benchmarks present in only one run are marked new or removed. The Reverse button swaps A and B in place. This is the view for before and after questions, a release against the previous one, or a fork with an optimization against main.

## Noise Study

Answers: how repeatable are the measurements when the code does not change at all?

![Noise study](images/noise-study.png)

Pick a commit that has two or more stored runs, every run of it appears grouped by the CPU the run landed on, with checkboxes to include or exclude runs or whole groups. For every benchmark the table shows the mean, the average JMH error, the standard deviation, the min and max, each run's individual score with its own error underneath, and the CV badge:

| Badge | CV | Reading |
|---|---|---|
| stable | below 5% | repeated runs agree, differences of a few percent on the other tabs are meaningful |
| medium | 5% to 10% | treat single run differences with care |
| noisy | 10% or above | single run differences of this size can be pure noise |

Since all runs in this view are the same code on the same machine, every spread you see here is measurement noise, which makes this tab the calibration for how much to trust the others.
