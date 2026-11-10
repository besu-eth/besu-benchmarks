# Running Benchmarks

Benchmarks can be run two ways, locally on your machine for quick checks during development, or through the CI workflow for results on the dedicated runner that land on the dashboard.

## Quick start, locally

From a besu checkout, run a single benchmark:

```bash
./gradlew :ethereum:core:jmh -Pincludes=AddOperationBenchmark
```

This compiles besu and runs every benchmark whose name matches the pattern. One benchmark takes a few minutes, most of it compilation. The full suite takes several hours, run it locally only when you really need to.

Results are written to `ethereum/core/build/results/jmh/results.json`.

## Filtering

The jmh task accepts these properties, all optional:

| Property | Meaning | Example |
|---|---|---|
| `-Pincludes` | Regular expression, only matching benchmarks run | `-Pincludes=Add.*\|SStore` |
| `-Pexcludes` | Comma separated regular expressions, matching benchmarks are skipped | `-Pexcludes=V2,executeOperationWithEmptyHashCache` |
| `-Pcases` | Comma separated list of @Param case names to run | `-Pcases=MOD_32_32,MOD_256_256` |
| `-PgasProfiler` | Set to true to emit gas cost as a secondary metric, which the dashboard turns into MGas/s | `-PgasProfiler=true` |
| `-PgasProfilerFork` | EVM fork used for gas cost calculation, defaults to OSAKA | `-PgasProfilerFork=CANCUN` |

Run `./gradlew help --task :ethereum:core:jmh` in besu for the full built in description of these flags.

## Running the CI workflow manually

Open the [Actions tab](https://github.com/besu-eth/besu-benchmarks/actions/workflows/evm-benchmarks.yml) and press Run workflow. The form takes:

| Input | Meaning | Default |
|---|---|---|
| Besu ref | Branch, tag, or commit SHA to benchmark | `main` |
| Besu repository | Repository to clone, can be a personal fork | `besu-eth/besu` |
| JMH include pattern | Same as `-Pincludes`, empty runs everything | empty |
| JMH exclude pattern | Same as `-Pexcludes`, added on top of the always on exclusions | `V2` |
| Enable GasProfiler | Emit gas cost so the dashboard can show MGas/s | on |

Because the repository and ref are both inputs, a one off comparison against a personal fork or an unmerged branch needs no code change, point the form at it and run. The run appears on the dashboard when it finishes, manual runs are excluded from the trend chart by default so they do not distort the nightly history.

A full suite run takes between 5 and 8 hours on the dedicated runner depending on how many benchmarks the targeted commit contains. A filtered run with a handful of benchmarks takes minutes.

## Comparing two runs

To judge a before and after, for example a fork with an optimization against main, run both through the workflow and open the Compare Runs tab on the dashboard. Pick the baseline run as A and the candidate as B, the table shows each benchmark side by side with the change. The Reverse button swaps the two if they were picked in the wrong order.
