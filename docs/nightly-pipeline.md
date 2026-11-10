# The Nightly Pipeline

The workflow in `.github/workflows/evm-benchmarks.yml` runs three times a day on a schedule, every run executes the full JMH suite on the dedicated runner and commits its results to this repository.

## Schedule

| Time UTC | What it benchmarks | Purpose |
|---|---|---|
| 00:00 | The latest commit on besu main | Catches regressions between commits, this is the run that builds the trend over time |
| 08:00 | A pinned commit that never changes | Noise measurement, since the code is identical every difference between runs is measurement noise |
| 16:00 | The same pinned commit | Second noise measurement of the day |

The pinned commit is set in the workflow as the fallback for `BESU_REF`. It currently points at the commit that raised JMH forks to 5, so the noise study measures the configuration the pipeline actually runs. When the pinned commit needs to move, for example after a change that affects benchmark behaviour, update the fallback SHA in the workflow and the noise study restarts fresh from there.

Runs take between 5 and 8 hours. A concurrency group in the workflow makes any overlapping run wait in a queue rather than cancel the one in progress, so a slow run delays the next one instead of breaking it. The job timeout is 12 hours.

## What one run does

1. Checks out this repository without leaving credentials on disk
2. Clones besu at the requested ref, if the ref is an old commit outside the shallow clone window it is fetched directly by SHA
3. Builds besu and runs the JMH suite with the gas profiler on
4. Excludes `executeOperationWithEmptyHashCache` always, and `V2` benchmarks by default
5. Writes a metadata file recording the commit, date, trigger type, runner hardware, and Java version
6. Uploads the raw results as a workflow artifact
7. Commits results and metadata to `data/runs/` on main, which redeploys the dashboard

## Results storage

Every run gets a folder named after its GitHub Actions run id:

```
data/runs/
  index.json                 one entry per run, what the dashboard lists
  <run_id>/
    results.json             the full JMH output, scores, errors, raw iteration data, gas metrics
    metadata.json            commit, ref, date, trigger, filter, runner hardware, Java version
```

`index.json` carries one summary entry per run including the commit SHA, date, trigger type, CPU model, and benchmark count. The dashboard reads the index first and fetches individual result files only when a view needs them.

The `trigger` field in the metadata records whether the schedule or a person started the run. The dashboard uses it to keep manual one off runs out of the trend chart by default.

## Failure handling

A red run on the Actions page means no results were stored for that slot, the dashboard simply shows a gap, nothing corrupts. The common causes seen so far:

- The job exceeded the timeout because the benchmarked commit contained more or slower benchmarks than usual
- The runner machine was restarted while a run was in progress, the log then ends with the operation was canceled
- The pinned commit fell outside the clone window before the fetch by SHA fallback existed, this is fixed

A green run means the benchmarks executed and results were stored. It does not by itself mean every number is sound, a run can complete while individual benchmarks were disturbed. The [noise and significance](noise-and-significance.md) page covers how to judge that.
