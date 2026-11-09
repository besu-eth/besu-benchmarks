# Adding a New Benchmark

New benchmarks are added in the besu repository, not here. Once merged into besu main, the midnight run picks them up automatically, nothing in this repository needs changing.

## Where benchmarks live

JMH benchmark classes sit in besu under:

```
ethereum/core/src/jmh/java/org/hyperledger/besu/ethereum/vm/operations/
```

The convention is one class per opcode or operation named `<Operation>OperationBenchmark`, with `@Param` cases covering the input shapes worth measuring. Shared setup helpers like `BenchmarkHelper` and `OperationBenchmarkHelper` live alongside them.

## Gas wiring, making MGas/s appear

A benchmark that only measures time shows a dash in the dashboard's MGas/s column. To get throughput numbers, the class implements the `GasCostBenchmark` interface:

```java
public class MyOperationBenchmark implements GasCostBenchmark {

  @Override
  public long getGasCost(final BenchmarkParams params, final GasCalculator calc) {
    return calc.getBaseTierGasCost();
  }

  @Override
  @Benchmark
  public void executeOperation(final Blackhole blackhole) {
    ...
  }
}
```

`getGasCost` returns the gas the operation costs, taken from the `GasCalculator` so the number tracks the fork schedule rather than being hardcoded. When the suite runs with the gas profiler on, which CI always does, the profiler calls this method and emits the gas as a secondary metric, and the dashboard computes MGas/s as `gas × 1000 / ns_per_op`.

This works for operations whose gas cost is static. Operations with dynamic gas, where the cost depends on the inputs, cannot be expressed this way yet and show a dash.

## Checklist

1. Write the class in the jmh folder, implement `GasCostBenchmark` if the gas cost is static
2. Run it locally and confirm it produces sensible numbers: `./gradlew :ethereum:core:jmh -Pincludes=MyOperationBenchmark`
3. If it is gas wired, confirm the gas metric appears: add `-PgasProfiler=true` and check `results.json` contains a `gas` entry under `secondaryMetrics`
4. Open the PR against besu
5. After the merge, check the next midnight run on the dashboard, the benchmark appears in the Latest Run table, shows MGas/s if wired, and its vs Previous column stays empty until a second run gives it a baseline

## Pitfalls

**State that accumulates across invocations.** JMH calls the benchmark method millions of times. Anything that grows per call, a list that is appended to, frames pushed onto a shared stack, will eventually exhaust memory and kill the fork. This happened with the BlockHash cold cache benchmark, which built a new message frame per invocation and pushed each one onto a shared deque, see [besu issue #11246](https://github.com/besu-eth/besu/issues/11246). Build expensive state once in `@Setup`, and if per invocation state is unavoidable, clean it up, for example with a rollback in `@TearDown`.

**Heavy work inside the measured method.** Only the operation under test belongs in the `@Benchmark` method. Input preparation belongs in `@Setup`, or in pre built pools the method indexes into, otherwise the setup cost is measured as if it were the operation.

**Case count multiplies run time.** Every `@Param` case runs the full 5 forks with all warmup and measurement iterations. A class with 100 cases adds hours to the nightly suite. Add the cases that cover genuinely different code paths, not every input size imaginable.

**Ultra fast operations have percentage noise.** An operation measured at 7 ns swings several percent on sub nanosecond jitter. For these, judge changes in absolute nanoseconds as well as percent.
