---
title: "Adaptive Video Streaming System"
date: 2023-11-29
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
**Sensys 20**, **skimmed**

### Resource
paper: [`Distream: Scaling Live Video Analytics withWorkload-Adaptive Distributed Edge Intelligence`](https://dl.acm.org/doi/pdf/10.1145/3384419.3430721)  
code: [`github`](https://github.com/AIoT-MLSys-Lab/Distream)

### Summarization
In this paper, the authors argue that the exsiting DL-based live video analytics systems are agnostic to the workload dynamics in real-world deployments. To achieve *(1)low latency*, *(2)high throughput*, and *(3)scalable live video analytics* in real world, they propose Distream which can adaptively balance the workloads across smart cameras and partition the workloads between cameras and the edge clusters. To evaluate the performance, this paper adopts *(1)throughput*, *(2) latency*, and *(3)latency service level objective(SLO) miss rate*.

### Problem Setting
In Distream, the authors consider a task-specific DAG live video analytics pipeline which firstly filters the regions of interest(ROI) and then feed these extracted objects to the different branches of downstream analyzing models.  

{{< figure src="/post/adaptive-video-streaming/distream-problem-setting-1.png" caption="Live video analytics pipeline of Distream" >}}

Distream considers a multi-cameras single-edge-server scenario where both cameras and edge servers could handle the analytics task. This brings three challenges *(1) cross-camera workload balancing*, *(2) camera-cluster workload balancing*, and *(3) adaption to workload dynamics*.

### Methodology & System Design

To address the challenge 1, Distream proposes a LSTM model, which considers the cross-camera workload correlation, heterogeneous compute capacity and overhead of workload balancing, which is able to predict the incoming workloads so as to avoid the possible high workloads.

To address the challenge 2, Distream pratitions the pipeline in a *stochastic* scheme to balance the workload ratio of both camera and cluster sides.

To address the challenge 3, Distream incorperates a controller which could (1)trigger the cross-camera  workload balancer when the cross-camera workload is imbalanced and (2) jointly optimize the pipeline partitioning so as to maximize the overall system throughput.

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