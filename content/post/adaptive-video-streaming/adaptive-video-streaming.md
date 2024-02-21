---
title: "Adaptive Video Streaming System"
date: 2024-02-05
# weight: 1
# aliases: ["/first"]
tags: ["video streaming", "edge computing"]
author: "Yuheng"
showtoc: true
tocopen: true
draft: true
math: true
comments: false
description: "deliver live video stream intelligently"
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: false
ShowWordCount: true
ShowRssButtonInSectionTermList: false
ShowShareButtons: false
---

## Adaptive Video Streaming

## Distream

**Sensys 20**, **Skimmed**

### Resource

paper: [`Distream: Scaling Live Video Analytics withWorkload-Adaptive Distributed Edge Intelligence`](https://dl.acm.org/doi/pdf/10.1145/3384419.3430721)  
code: [`github`](https://github.com/AIoT-MLSys-Lab/Distream)

### Summarization

In this paper, the authors argue that the exsiting DL-based live video analytics systems are agnostic to the workload dynamics in real-world deployments. To achieve _(1)low latency_, _(2)high throughput_, and _(3)scalable live video analytics_ in real world, they propose Distream which can adaptively balance the workloads across smart cameras and partition the workloads between cameras and the edge clusters. To evaluate the performance, this paper adopts _(1)throughput_, _(2) latency_, and _(3)latency service level objective(SLO) miss rate_.

### Problem Setting

In Distream, the authors consider a task-specific DAG live video analytics pipeline which firstly filters the regions of interest(ROI) and then feed these extracted objects to the different branches of downstream analyzing models.

{{< figure src="/post/adaptive-video-streaming/distream-problem-setting-1.png" caption="Live video analytics pipeline of Distream" >}}

Distream considers a multi-cameras single-edge-server scenario where both cameras and edge servers could handle the analytics task. This brings three challenges _(1) cross-camera workload balancing_, _(2) camera-cluster workload balancing_, and _(3) adaption to workload dynamics_.

### Methodology & System Design

To address the challenge 1, Distream proposes a LSTM model, which considers the cross-camera workload correlation, heterogeneous compute capacity and overhead of workload balancing, which is able to predict the incoming workloads so as to avoid the possible high workloads.

To address the challenge 2, Distream pratitions the pipeline in a _stochastic_ scheme to balance the workload ratio of both camera and cluster sides.

To address the challenge 3, Distream incorperates a controller which could (1)trigger the cross-camera workload balancer when the cross-camera workload is imbalanced and (2) jointly optimize the pipeline partitioning so as to maximize the overall system throughput.

{{< figure src="/post/adaptive-video-streaming/distream-method-1.png" caption="Architecture of Distream" >}}

### QA

1. How to define the so called workload?

2. How to quantize the three evaluation metrics? (throughput, latency, and SLO miss rate)

3. How much overheads does Distream bring?

## Magic-Pipe

**Middleware 21**, **Skimmed**

### Resource

paper: [`Magic-Pipe: Self-optimizing video analytics pipelines`](https://dl.acm.org/doi/pdf/10.1145/3464298.3484504)  
code: `NOT FOUND`

### Summarization

The paper focuses on enhancing microservices-based video analytics systems by dynamically optimizing resource allocation and microservice parameters based on video content. The problem addressed is the inefficiency in static resource allocation and configuration, which fails to adapt to the changing content and computational demands of video analytics tasks. The motivation stems from the observation that optimal resource distribution and parameter settings can significantly vary in short periods due to dynamic video content. The system design introduces adaptive techniques for resource allocation, parameter tuning, and computation reduction through AI-driven methods, including reinforcement learning and graph-theoretic approaches. Experimental results demonstrate that the Magic-Pipe system significantly improves application response times and maintains high accuracy without additional hardware, compared to traditional static pipelines.

## Nexus

**SOSP 19**, **Thoroughly**

### Resource

paper: [`Nexus: a GPU cluster engine for accelerating DNN-based video analysis`](https://dl.acm.org/doi/10.1145/3341301.3359658)  
code: [`github`](https://github.com/uwsampl/nexus)

### Summarization

Nexus aims to maximize the utilization of GPUs in the DNN-based video analysis cluster while maintaining the latenct SLOs constrains. To do so, Nexus proposes a cluster-scale resource management system which is able to (1) perform detailed scheduling of GPUs, (2) groups and co-schedules the invocations and (3) support "fragment" execution of DNN model other than the conventional whole DNN execuation.

### Problem Setting

In the typical video processing framework, cluster receives the larget amount of video streams and samples ciritical frames among the stream. Then these aggregated frames are firstly pre-processed by some lightweight models to identify what parts(_window_) are worth to further analysis. Next, different _query_(maybe a sequence of models) may assign to these windows. All the outputs are aggregated as the final outputs of the corresponding video stream and task.

{{< figure src="/post/adaptive-video-streaming/nexus-problem-setting-1.png" caption="A typical video stream processing pipeline" >}}

The backend GPUs are expensive and very-high-capacity. Cost-savings lie in how can we maintain high utilization. A naive idea is to shard the incoming workload via a distributed frontend onto the backend GPUs. However, there are three challenges (1) How to maximize the combined throughput of different models deployed on the same device while satisfying the latency SLOs? (2) Applications may consist of _groups or sequence_ of models feed to each other, how to specify theses groups and schedule the group execution? (3) In the serving stage, different streams requires different models, how to batch these features so as to facilitate the advantage of _batching_ of neural network?

### Methodology

#### Squishy Bin Packing

As shown below, the processing cost of an input is varied with the size of the batch within which that input is processed.

{{< figure src="/post/adaptive-video-streaming/nexus-squishy-bin-packing-1.png" caption="Batching profile for different models. Lat: latency(ms), Req/s: throughput" >}}

Considering this property, Nexus splits the bin-packing into two scenarios. Fisrt, if the requests are high so that multiple GPUs are required to handle each task (**Saturate Workload**). Within the constrain of SLO, the batch execution cost can't exceed half of the task's latency SLO (this is because a task may be missed and would be excuted as part of the next batch). For this case, the required number of GPUs for each task is $r_{task} / Req$.

Another case (**Residual Workload**) is where each task requires less than one GPU. In this case, it's neccessary to aggregate different tasks into one GPU while maintaining the SLO.

{{< figure src="/post/adaptive-video-streaming/nexus-squishy-bin-packing-2.png" caption="Explanation of Saturate workload and Residula workload" >}}

Here defines the notations.

{{< figure src="/post/adaptive-video-streaming/nexus-squishy-bin-packing-notation.png" caption="Notation" >}}

The requests for a given model and latency SLO are refered as a _session_. _session_ specifies $(S_i, M_{k_{i}}, L_i, R_i, \mathcal{l}_{k_i}(b))$ under the assumption that the throughput is non-decreasing with batch size $b$.

For large sessions (Saturate Workload), Nexus directly picks the largest batch size which could satisfy the SLO constrain. For residual workload, Nexus seeks for a greedy scheduling algorithm in which the scheduler inspects each residual session in isolation and computes the largest batch size and the corresponding duty cycle in order to meet the throughput and SLO needs. The algorithm then attempts to merge multiple sessions within a GPU’s duty
cycle.

{{< figure src="/post/adaptive-video-streaming/nexus-squishy-bin-packing-algo.png" caption="" >}}

#### Complex Query Scheduling

A more practical system is to apply groups or sequence of models to the task where the outputs of one may be the input of another model. For example, for detection and recognition pipeline, it firstly extracts objects and then recognizes each objects. But the developer only specifies the SLO for the entire query.

Nexus extracts the dataflow dependency graph between model invocations in application code and fomulates the scheduling of queries as an optimization problem.Suppose the query involves a set of models $M_i$ with request rate $R_i$ , and the end-to-end latency SLO is $L$. The objective is to find the best latency SLO split $L_i$ for each model $M_i$ to minimize the total number of GPUs that are required for the query. Because latency $L_i$ is determined by batch size $b_i$, the optimization problem is equivalent to finding the best batch sizes that minimizes GPU count, while meeting the latency SLO along every path from the root model ($M_{root}$) to the leaf models.

#### Miscellaneous Optimization

1. Overlapping GPU and CPU computation
2. GPU multiplexing: the Nexus node runtime manages the execution of all models on a GPU, so it is able to pick batch sizes and execution schedules for all models in a round-robin fashion to make sure models abide by their latency SLOs
3. Prefix batching: Nexus computes the hash of every sub-tree of the model schema and compares it with the existing models in the database to identify common sub-trees when a model is uploaded. At runtime, models with known common sub-trees are loaded partially in the backend and batched at the sub-tree (or prefix) granularity
4. Adaptive Batching: early drop policy where the dispatcher scans through the queue using a sliding window whose length is the batch size determined by the global scheduler for a given session. It stops at the first request that has enough budget for batched execution latency of the entire window and drops all earlier requests.

### Evaluation

Nexus is evaluated by answering four questions:

(1) Does using Nexus result in better cluster utilization while meeting SLOs with respect to existing systems?  
(2) Does high performance persist when Nexus is used at a large scale?  
(3) How do the new techniques in Nexus contribute to its performance?  
(4) What determines how well each of these techniques work?
