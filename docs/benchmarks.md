# Benchmarks

This page summarizes the public-safe benchmark story for the cluster.

## Why I Benchmarked It

The cluster had to prove more than:

- nodes can PXE boot
- images can be rebuilt
- Slurm can show nodes as available

Those checks are necessary, but they are not enough to show that the system can actually do useful HPC work.

I used HPL to answer the practical question: once the cluster boots and schedules jobs correctly, can it execute real distributed numerical workloads at scale?

## Benchmarking Goals

The benchmarking work was intended to validate:

- multi-node execution
- MPI-based workload behavior
- scheduler stability under real workload launch
- the ability to compare achieved performance with theoretical peak
- the normal user-facing execution path, not just privileged admin runs

## Test Context

HPL was run across several node groupings during cluster validation and tuning, including:

- smaller node groups for early stability checks
- mid-sized groups for tuning and scaling behavior
- full-fleet execution across 40 worker nodes

The benchmark parameters were tuned across runs by adjusting:

- matrix size `N`
- block size `NB`
- process grid `P x Q`

That tuning mattered because memory headroom, process-grid shape, and node grouping all affected the final results.

## Best Public-Safe Results

| Cluster Size | Achieved Performance | Theoretical Peak | Percent of Peak |
|---|---:|---:|---:|
| 4 nodes | 1.5044 TFLOPS | 3.379 TFLOPS | 44.52% |
| 17 nodes | 1.9371 TFLOPS | 5.657 TFLOPS | 34.24% |
| 19 nodes | 5.442 TFLOPS | 6.323 TFLOPS | 86.07% |
| 40 nodes | 11.107 TFLOPS | 13.312 TFLOPS | 83.44% |

The 40-node result is the strongest large-scale outcome in the public record because it shows that the cluster scaled across the full worker fleet while maintaining strong efficiency.

## Example HPL Output

Public-safe excerpt from a 40-node HPL run:

```text
T/V                N      NB    P    Q        Time                Gflops
WR00R2R4      353024    224   16   40      2640.77             1.1107e+04

Finished      1 tests with the following results:
              1 tests completed and passed residual checks
```

This is important because it is concrete evidence of successful multi-node execution, not just a design claim.

## Non-Root Benchmark Validation

One of the more meaningful final checks was successful HPL execution through the normal user-facing workflow. That matters because it proves the cluster was not only benchmarkable by the administrator, but usable through the same path an actual user would take.

Public-safe summary of that run:

- 40-node HPL execution through the login-node and Slurm workflow
- non-root execution validated
- residual check passed
- performance result recorded successfully

That result closes the loop between provisioning, scheduling, user workflow, and workload execution.

## What the Benchmark Results Mean

From a systems perspective, the benchmark results demonstrate that the project reached all of the following:

- stable diskless boot
- usable runtime environment
- functional scheduler path
- working user access layer
- real distributed computation

That is what makes the cluster more than a configuration exercise.

## Limits of the Results

These results should still be interpreted in the right context.

- the cluster uses repurposed and mixed hardware
- some node groups differ in memory profile and NIC behavior
- not every run is directly comparable as a perfect like-for-like benchmark
- tuning choices affected both stability and performance

Even so, the results are strong evidence that the environment is computationally real and operationally usable.
