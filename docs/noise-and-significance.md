# Noise and Significance

A benchmark score is a measurement, and measurements vary even when nothing changed. This page records how much they vary on this pipeline, how that was established, and how to use it when deciding whether a change is real.

## Why this matters

One slow run looks exactly like a regression. If the pipeline flagged every run that came in slower than the previous one, someone would be chasing false alarms every week and would soon stop trusting the dashboard. The goal is the opposite, an alert should be rare enough that when it fires, it is worth looking at.

## The noise study

The method: run the exact same commit repeatedly on the same machine and look at how much the scores spread. Any spread observed this way is measurement noise by definition, the code never changed. The spread is summarized per benchmark as the CV, standard deviation divided by mean.

What it found, in the order it happened:

| Setup | Median CV across the suite | Benchmarks at or above 10% CV |
|---|---|---|
| GitHub free runners, mixed hardware every run | about 13% | hundreds |
| Dedicated runner, 3 JMH forks | 0.7% | 3 |
| Dedicated runner, 5 JMH forks | 0.73% with zero above 7.3% | 0 |

Two conclusions came out of this. Hardware consistency matters most, moving from random free runners to one dedicated machine took the typical benchmark from 13% noise to under 1%. And more forks tame the stragglers, the handful of benchmarks that still wobbled on 3 forks, where one unlucky JVM process skewed the run, settled once each score averaged 5 independent JVM processes.

The 08:00 and 16:00 pinned runs keep this measurement going permanently. If the noise level ever shifts, for example after a runner change, it shows up there first.

## How noise appears inside a single run

JMH's error value is the within run noise signal. The pinned data shows that an anomalous run usually carries a visibly larger error than its neighbours, the forks inside it disagreed. The error is only meaningful relative to the score, an error of 14 on a score of 113 is 12% and high, an error of 149 on a score of 2610 is 6%. On the dashboard this is the ± Error column, and on the noise study tab each individual run shows its own error underneath its score.

## Judging a change today

The current practice is manual, with these rules:

1. Check the CV badge for the benchmark on the noise study tab first. A 6% change on a stable benchmark means something, the same change on a noisy one means nothing.
2. Treat score ± error as a range, not a point. If the ranges of the two runs being compared overlap, the difference is inside measurement uncertainty and the dashboard marks it noisy.
3. Distrust single runs. A real regression stays slow on the next run, an anomaly returns to normal. Waiting one extra night before believing an alarm costs a day and removes most false positives.
4. New benchmarks need settling time. A benchmark added to besu recently has no history, its first comparisons swing more than its steady state.

## Where detection is heading

Automated alerting is the goal and is deliberately not switched on yet. The options were surveyed and discussed with the team, the direction is to combine a threshold with the error ranges, a change counts only when it exceeds the threshold and the two error ranges do not overlap. The repository carries `scripts/benchmark_regression_check.py`, a standalone comparator implementing exactly that rule, it compares two result files, writes a Markdown report, and exits nonzero when a regression survives both tests, so CI can gate on it once the rule is trusted. The plan before enabling any alert is to plant a deliberate regression and confirm it is caught.
