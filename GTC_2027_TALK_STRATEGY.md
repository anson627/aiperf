<!--
SPDX-FileCopyrightText: Copyright (c) 2026 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
SPDX-License-Identifier: Apache-2.0
-->

# GTC 2027 Talk Strategy: Model Arrival to Serving SLO

## Recommended Story

The strongest GTC story is not simply that work landed in both ModelExpress and
AIPerf. It is a single, measurable production story:

> We turned model onboarding into a reproducible lifecycle, from model arrival,
> through ModelExpress transfer and Dynamo deployment, to verified performance
> and accuracy with AIPerf.

This connects the projects around a real operational problem and creates a more
compelling proposal than a collection of unrelated pull requests.

## Proposed Open Source Project

Build an open, reproducible **model arrival to serving SLO** benchmark:

1. Deploy or scale a model through NVIDIA Dynamo.
2. Have ModelExpress select and execute the weight-loading path.
3. Wait for every intended replica, rather than merely one endpoint, to become
   ready.
4. Use AIPerf to determine when the deployment reaches its required latency,
   throughput, goodput, and accuracy thresholds.
5. Publish comparable results for:

   - Cold download with the inference engine's native loader
   - Warm filesystem or object-store cache
   - ModelExpress server-cache loading
   - ModelExpress GPU-to-GPU P2P loading
   - Scale-out to multiple replicas
   - Failure and fallback scenarios

This project aligns with ModelExpress's direction toward predictable startup,
readiness, fallback, and observability. It also complements AIPerf's work on
Kubernetes scale, accuracy, telemetry, and reproducibility.

## Potential Upstream Contributions

Before implementing a large cross-project feature, socialize a short public
design proposal and confirm ownership with the ModelExpress and AIPerf
maintainers.

### ModelExpress

- Emit structured lifecycle timestamps for discovery, source selection,
  transfer, runtime installation, and readiness.
- Report the selected loading path, bytes transferred, transfer rate, and
  fallback reason.
- Add reproducible cold-cache, warm-cache, P2P, and scale-out benchmark recipes.
- Add failure-injection coverage proving that fallback cannot serve a partial or
  mixed model version.

### AIPerf

- Define a lifecycle benchmark or artifact schema that records deployment
  start, full-replica readiness, first successful inference, and stable-SLO
  time.
- Add Kubernetes readiness validation that distinguishes one responsive
  endpoint from all expected workers being ready.
- Produce comparison reports combining startup time with steady-state TTFT,
  throughput, goodput, error rate, energy, and accuracy.
- Document a reproducible Dynamo and ModelExpress onboarding recipe using a
  public model and a fully specified infrastructure configuration.

The objective should be one coherent capability with tests, documentation,
benchmark data, and maintainer adoption. A few substantial contributions that
enable the end-to-end story are more valuable to the proposal than many small,
unrelated changes.

## Measurement Contract

| Operator question | Measurement |
| --- | --- |
| How soon can traffic safely begin? | Deployment start to all replicas ready |
| When can the first user receive output? | Deployment start to first token |
| When is the service truly usable? | Deployment start to sustained SLO |
| Does faster loading affect serving? | Steady-state throughput, TTFT, ITL, and goodput |
| Is the intended model loaded? | Pinned model revision and accuracy checks |
| Are the results repeatable? | Multiple trials with confidence intervals |
| What happens when the fast path fails? | Fallback latency, correctness, and availability |
| Does the approach scale? | Results across replica counts and model sizes |

Every published result should include:

- Model and tokenizer revisions
- Dynamo, ModelExpress, inference engine, and AIPerf versions
- GPU, NIC, storage, network, and Kubernetes topology
- Cache state and selected ModelExpress loading path
- Replica count and tensor/pipeline parallel configuration
- Workload, concurrency, warmup, and AIPerf configuration
- Raw result artifacts and enough instructions to reproduce the experiment

## Experiment Matrix

Start with the smallest experiment that proves the methodology, then expand it.

| Phase | Comparison | Primary outcome |
| --- | --- | --- |
| 1 | Native cold load vs. ModelExpress P2P | Validate the lifecycle timestamps and end-to-end speedup |
| 2 | Cold, warm-cache, server-cache, and P2P | Explain which loading path wins under each condition |
| 3 | One to several replicas | Measure readiness convergence and scale-out behavior |
| 4 | vLLM, SGLang, and TensorRT-LLM where supported | Demonstrate an engine-neutral methodology |
| 5 | Unavailable peer, transport failure, and stale revision | Quantify fallback behavior and correctness |
| 6 | Representative onboarding accuracy suite | Verify that the accelerated path serves the intended model correctly |

Avoid expanding the matrix until the maintainers agree on the timestamps,
readiness definition, and result schema. Otherwise, early results may not be
comparable with the final implementation.

## Talk Positioning

### Working Title

**From Model Arrival to Serving SLO: Reproducible Model Onboarding with NVIDIA
Dynamo, ModelExpress, and AIPerf**

### Draft Abstract

Large models are not production-ready merely because a container is running or
one health probe succeeds. Weight acquisition, GPU loading, replica convergence,
runtime warmup, and performance validation form a single onboarding lifecycle.
We present an open, reproducible workflow combining NVIDIA Dynamo and
ModelExpress with AIPerf to measure time from model arrival to sustained serving
SLO. We compare cold-storage, cached, and GPU-to-GPU loading paths across
scale-out scenarios, show how incomplete readiness distorts benchmark results,
and demonstrate automated validation of latency, throughput, accuracy, and
fallback behavior. Attendees leave with reusable Kubernetes recipes, benchmark
artifacts, and practical guidance for accelerating model onboarding without
sacrificing measurement integrity.

### Proposed Audience Takeaways

- Why endpoint readiness and full serving readiness are different
- How to measure model-loading speed without contaminating steady-state results
- When cold storage, local cache, server cache, or GPU P2P is the appropriate
  loading path
- How to combine startup, performance, reliability, and accuracy into one
  onboarding gate
- How to reproduce the workflow using upstream NVIDIA open source projects

## Execution Plan

### Phase 1: Maintainer Alignment

1. Discuss the lifecycle benchmark with the current ModelExpress collaborators.
2. Write a concise proposal defining events, timestamps, readiness, experiments,
   and artifact ownership.
3. Ask the AIPerf maintainers whether lifecycle orchestration belongs in AIPerf,
   an AIPerf Kubernetes integration, or a separate harness that emits AIPerf
   artifacts.
4. Agree on one upstream deliverable in each repository and one shared recipe.

### Phase 2: Minimum End-to-End Result

1. Use one public model, one inference runtime, and one cluster topology.
2. Compare a native cold load with ModelExpress P2P.
3. Record deployment-to-ready, deployment-to-first-token, deployment-to-SLO,
   and steady-state AIPerf metrics.
4. Repeat enough trials to report variance and confidence intervals.
5. Publish the implementation, recipe, raw data, and limitations.

### Phase 3: Production-Quality Evidence

1. Add cache and fallback paths.
2. Add replica scale-out and convergence measurements.
3. Add another engine or model only when it reveals a meaningful systems
   difference.
4. Validate model identity and quality with AIPerf accuracy evaluation.
5. Present preliminary results to both maintainer groups and incorporate their
   feedback.

### Phase 4: Submission

1. Base the abstract on completed public work and measured results.
2. Ask a ModelExpress, Dynamo, or AIPerf maintainer to co-present after the shared
   work has landed.
3. Replace generic claims in the abstract with two or three defensible numbers.
4. Include links to merged contributions and reproducible recipes where the
   submission form permits them.
5. Obtain employer approval for publishing architecture, costs, configurations,
   and benchmark data.

## What Will Strengthen the Submission

- Merged upstream code used by people beyond the original project
- A cross-project contract accepted by the relevant maintainers
- Reproducible public benchmark artifacts, not screenshots alone
- Results that include variance, failures, and limitations
- A surprising but practical lesson, such as the gap between endpoint and
  full-replica readiness
- A co-speaker who can add a distinct maintainer, platform, or adopter
  perspective
- Evidence that the workflow prevented an incorrect onboarding or deployment
  decision

## Risks to Avoid

- Framing the talk primarily as a product announcement
- Collecting impressive loading numbers without verifying post-load serving
  behavior
- Comparing different cache states, model revisions, or workload configurations
- Building a large feature before agreeing with maintainers on repository scope
- Claiming generality after testing only one favorable topology
- Submitting promised future work without public results
- Treating small contribution count as a proxy for technical impact

## Public References

- [ModelExpress repository and benchmark overview](https://github.com/ai-dynamo/modelexpress)
- [ModelExpress 1.0 vision](https://github.com/ai-dynamo/modelexpress/issues/594)
- [AIPerf public roadmap](https://github.com/ai-dynamo/aiperf/issues/1009)
- [Example of incomplete readiness distorting AIPerf results](https://github.com/NVIDIA/aicr/issues/1181)
- [NVIDIA GTC presenter interest form](https://www.nvidia.com/gtc/call-for-submissions/presenter-interest-form/)

The public presenter page currently available is an interest form and does not
provide reliable GTC 2027 submission dates. Register for notifications, but do
not wait for the call for submissions before developing the contributions and
results.
