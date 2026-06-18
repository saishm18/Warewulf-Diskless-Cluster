# Implementation Notes

This page explains how I approached the build as an engineering project, not just as a software install.

## Build Strategy

From the beginning, the cluster had to satisfy two different goals at the same time:

- it had to function like a real HPC environment
- it had to stay easy to rebuild, debug, and extend

That drove most of the technical decisions.

I did not want a setup where every compute node had its own snowflake operating system state. I wanted a system where the cluster could be reasoned about centrally and where failures could be fixed through the image, overlays, and service design rather than by repairing nodes one at a time.

## Why I Chose a Diskless Model

The diskless model made sense for this hardware and for the goals of the project.

It gave me:

- a single worker-image baseline
- consistent software state across the fleet
- easy rebuild and rollback behavior
- less per-node drift
- a cleaner way to support lab-style or frequently reset environments

That design also forced better discipline. Once nodes are stateless, the image and overlay model becomes the real source of truth, which is exactly what I wanted for a centrally managed cluster.

## Why Warewulf

Warewulf fit the project because it handles the parts that matter in a diskless cluster:

- image management
- node definitions
- overlays
- provisioning logic
- boot integration

It gave me a structured way to define nodes, rebuild images, propagate runtime content, and keep the worker-node environment under central control.

## Why I Separated the Roles

One of the strongest design decisions in the final system was role separation.

I intentionally kept:

- provisioning on a central host
- scheduling on a dedicated controller
- user access on login nodes
- execution on stateless compute nodes

This was important for stability and for clarity.

If provisioning, scheduling, and user access all collapse onto one machine, it becomes much harder to reason about failures. By separating them, I could validate and debug each layer independently.

## Why Shared `/home` and `/opt`

The compute nodes are stateless, so user data and shared software had to live somewhere persistent.

I used shared storage for:

- `/home` so users have a consistent working environment
- `/opt` so software can be provided centrally

This let me keep the compute runtime disposable while still giving users a normal HPC-style workflow.

## Why the Network Design Matters

The cluster uses separate roles for control-plane traffic and workload-side traffic. That was not just a performance choice. It was also a reliability choice.

Multi-NIC clusters can look healthy while still behaving incorrectly if services bind to the wrong interface or prefer the wrong address. I had to treat network identity, routing, and host resolution as part of the architecture itself, not as simple background configuration.

## What Took Real Debugging

This project involved real systems debugging in several areas:

- firmware-sensitive UEFI / GRUB boot behavior
- alignment between host-side boot artifacts and worker-side OS expectations
- runtime overlay rebuild behavior
- login-node reverse connectivity for `srun`
- Slurm node registration edge cases
- persistent runtime state for services in stateless environments

Those problems are why the project is interesting to me. It was not just "install Warewulf and Slurm." It required understanding how the whole system behaved end to end.

## A Few Concrete Examples

Some of the most useful lessons came from problems that only showed up in the live environment:

- a node can succeed at firmware-level PXE and still fail later in the operating-system-controlled network stage
- rebuilding a single overlay fragment is not always enough if the live runtime depends on a combined runtime overlay
- `sbatch` and `srun` do not exercise the same path, so one can work while the other fails
- a cluster can look reachable and still fail if control-plane identity is wrong

Those are the kinds of issues that turned this from a simple configuration task into a real HPC systems build.

## Final Outcome

The result is a cluster that is:

- diskless
- centrally managed
- scheduler-driven
- usable from a login-node-based workflow
- validated with real multi-node benchmarking

That combination was the actual goal of the project.
