---
title: "Collaborative, Adaptive and Efficient Edge Systems"
date: 2024-02-05
# weight: 1
# aliases: ["/first"]
tags: ["collaborative system", "edge computing", "system"]
author: "Yuheng"
showtoc: true
tocopen: true
draft: true
math: true
comments: false
description: "recording for papers review"
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: false
ShowWordCount: true
ShowRssButtonInSectionTermList: false
ShowShareButtons: false
---

## Distream

**Sensys 20**

### Resource

paper: [`Distream: Scaling Live Video Analytics withWorkload-Adaptive Distributed Edge Intelligence`](https://dl.acm.org/doi/pdf/10.1145/3384419.3430721)  
code: [`github`](https://github.com/AIoT-MLSys-Lab/Distream)

### Summarization

In this paper, the authors argue that the existing DL-based live video analytics systems are agnostic to the workload dynamics in real-world deployments. To achieve _(1)low latency_, _(2)high throughput_, and _(3)scalable live video analytics_ in real world, they propose Distream which can adaptively balance the workloads across smart cameras and partition the workloads between cameras and the edge clusters [without sacrificing accuracy. ?doubts here]. To evaluate the performance, this paper adopts _(1)throughput_, _(2) latency_, and _(3)latency service level objective(SLO) miss rate_.

### Problem Setting

In Distream, the authors consider a task-specific DAG live video analytics pipeline which firstly filters the regions of interest(ROI) and then feed these extracted objects to the different branches of downstream analyzing models.

{{< figure src="/post/edge-systems/distream-problem-setting-1.png" caption="Live video analytics pipeline of Distream" >}}

Distream considers a multi-cameras single-edge-server scenario where both cameras and edge servers could handle the analytics task. This brings three challenges _(1) cross-camera workload balancing_, _(2) camera-cluster workload balancing_, and _(3) adaption to workload dynamics_.

### Methodology & System Design

To address the challenge 1, Distream proposes a LSTM model, which considers the cross-camera workload correlation, heterogeneous compute capacity and overhead of workload balancing, which is able to predict the incoming workloads so as to avoid the possible high workloads. The intuition here is if the current device suffers from a high workload, it is highly likely for its neighbors to have a high workload in the near future.

To address the challenge 2, Distream partitions the pipeline in a _stochastic_ scheme to balance the workload ratio of both camera and cluster sides. Compared with the _deterministic_ partition method, it embraces more flexibility to adapt to the edge computing resource variance.

To address the challenge 3, Distream incorporates a controller on the edge cluster which could (1)trigger the cross-camera workload balancer when the cross-camera workload is imbalanced and (2) jointly optimize the pipeline partitioning so as to maximize the overall system throughput.

{{< figure src="/post/edge-systems/distream-method-1.png" caption="Architecture of Distream" >}}

### QA

1. How to define the so called workload?
> workload: going through each classifier within the DAG is regarded as an individual workload

2. How to quantize the three evaluation metrics? (throughput, latency, and SLO miss rate)
> throughput: num of inferences processed per second (IPS)
> latency: sum of (1) network latency (2) workload queuing latency (3) workload processing latency
> latency SLO: 3s

3. How much overheads does Distream bring?
> overhead comes from three sources (1) solving the optimization problem for cross-camera workload balancing (2) figuring out the optimal DAG partition point. (3) RPC call for mitigating  workloads across cameras
> For 6 cams and 1 GPU, the overheads are 16us, 4us respectively and for 24 cams and 4 GPUs are 21us, 8us.
> For RPC call, it takes advantage of _batching_ RPC calls to reduce the overhead.

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

Nexus aims to maximize the utilization of GPUs in the DNN-based video analysis cluster while maintaining the latency SLOs constrains. To do so, Nexus proposes a cluster-scale resource management system which is able to (1) perform detailed scheduling of GPUs, (2) groups and co-schedules the invocations and (3) support "fragment" execution of DNN model other than the conventional whole DNN execution.

### Problem Setting

In the typical video processing framework, cluster receives the largest amount of video streams and samples critical frames among the stream. Then these aggregated frames are firstly pre-processed by some lightweight models to identify what parts(_window_) are worth to further analysis. Next, different _query_(maybe a sequence of models) may assign to these windows. All the outputs are aggregated as the final outputs of the corresponding video stream and task.

{{< figure src="/post/edge-systems/nexus-problem-setting-1.png" caption="A typical video stream processing pipeline" >}}

The backend GPUs are expensive and very-high-capacity. Cost-savings lie in how can we maintain high utilization. A naive idea is to shard the incoming workload via a distributed frontend onto the backend GPUs. However, there are three challenges (1) How to maximize the combined throughput of different models deployed on the same device while satisfying the latency SLOs? (2) Applications may consist of _groups or sequence_ of models feed to each other, how to specify theses groups and schedule the group execution? (3) In the serving stage, different streams requires different models, how to batch these features so as to facilitate the advantage of _batching_ of neural network?

### Methodology

#### Squishy Bin Packing

As shown below, the processing cost of an input is varied with the size of the batch within which that input is processed.

{{< figure src="/post/edge-systems/nexus-squishy-bin-packing-1.png" caption="Batching profile for different models. Lat: latency(ms), Req/s: throughput" >}}

Considering this property, Nexus splits the bin-packing into two scenarios. First, if the requests are high so that multiple GPUs are required to handle each task (**Saturate Workload**). Within the constrain of SLO, the batch execution cost can't exceed half of the task's latency SLO (this is because a task may be missed and would be executed as part of the next batch). For this case, the required number of GPUs for each task is $r_{task} / Req$.

Another case (**Residual Workload**) is where each task requires less than one GPU. In this case, it's necessary to aggregate different tasks into one GPU while maintaining the SLO.

{{< figure src="/post/edge-systems/nexus-squishy-bin-packing-2.png" caption="Explanation of Saturate workload and Residual workload" >}}

Here defines the notations.

{{< figure src="/post/edge-systems/nexus-squishy-bin-packing-notation.png" caption="Notation" >}}

The requests for a given model and latency SLO are referred as a _session_. _session_ specifies $(S_i, M_{k_{i}}, L_i, R_i, \mathcal{l}_{k_i}(b))$ under the assumption that the throughput is non-decreasing with batch size $b$.

For large sessions (Saturate Workload), Nexus directly picks the largest batch size which could satisfy the SLO constrain. For residual workload, Nexus seeks for a greedy scheduling algorithm in which the scheduler inspects each residual session in isolation and computes the largest batch size and the corresponding duty cycle in order to meet the throughput and SLO needs. The algorithm then attempts to merge multiple sessions within a GPU’s duty
cycle.

{{< figure src="/post/edge-systems/nexus-squishy-bin-packing-algo.png" caption="" >}}

#### Complex Query Scheduling

A more practical system is to apply groups or sequence of models to the task where the outputs of one may be the input of another model. For example, for detection and recognition pipeline, it firstly extracts objects and then recognizes each objects. But the developer only specifies the SLO for the entire query.

Nexus extracts the dataflow dependency graph between model invocations in application code and formulates the scheduling of queries as an optimization problem.Suppose the query involves a set of models $M_i$ with request rate $R_i$ , and the end-to-end latency SLO is $L$. The objective is to find the best latency SLO split $L_i$ for each model $M_i$ to minimize the total number of GPUs that are required for the query. Because latency $L_i$ is determined by batch size $b_i$, the optimization problem is equivalent to finding the best batch sizes that minimizes GPU count, while meeting the latency SLO along every path from the root model ($M_{root}$) to the leaf models.

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



## When2Com

**CVPR 20**

### Resource
paper: [When2com: Multi-Agent Perception via Communication Graph Grouping](https://openaccess.thecvf.com/content_CVPR_2020/papers/Liu_When2com_Multi-Agent_Perception_via_Communication_Graph_Grouping_CVPR_2020_paper.pdf)  
code: [github](https://github.com/GT-RIPL/MultiAgentPerception)

### Problem settings
This paper focuses on the multi-agent distributed perception task in which different agents have different angle of views. These agents are able to share information between each other to tackle problems such as _occlusion_, _degradation_, etc. However, challenges lie in (1) how to share information in the limited bandwidth and (2) which kind of information are worth to share. To tackle these problem, _when2com_ proposes a communication framework which learns to construct communication groups and decide when to communicate.

{{< figure src="/post/edge-systems/when2com_problem_setting.png" caption="illustration of multi-agent collaborative perception" >}}

### Methodology

#### Group Communication
Instead of full connected with all agents, _when2com_ learns how to group agents in a bandwidth efficient and easily scalable manner. The intuition behind this are (1) not all the information from other agents are useful (2) in a larger scale scenario, the network complexity and bandwidth usage increase shapely which is not practical in the real world.

{{< figure src="/post/edge-systems/when2com_group_com.png" caption="illustration of group communication" >}}

When2Com firstly apply _handshake communication_ to determine the weights of connections, and then prune less important connections with an _activation function_. Specifically, inspired by attention mechanism, it interprets the relationship between agents as a adjacency matrix. 

Following the _who2com_ paper, it firstly leverages a three-stage handshake communication mechanism - request, match, select. Each agent compresses its local observation $x_i$ into a compact query vector $\mu_i$ and a key vector $k_i$. It further broadcasts query to all other agents. (query is a compact feature which consumes little bandwidth, key is larger than query here). To decide _who_ to communicate, agent computes the matching score $m_{i,j}$ between agent $i$(requester) and $j$(supporter). The intuition of this adjacency matrix is the score implies the correlation between agent i and agent j. The amount of score delivers the information the supporter agent j can provide for the requester agent i.

{{< figure src="/post/edge-systems/when2com_adjacency_matrix.png" caption="Illustration of adjacency matrix. Blue arrow represents intra-agent communication whereas red arrow denotes inter-agent communication" >}}

#### When to communicate
Until now, agents learns how to group its supporters and  requesters. But in a more bandwidth efficient communication, if a agent has enough information locally, there is no need to request for further information from others. To take this into consideration, the agent should learn when it needs to request information from its group. To this end, _when2com_ adopts a method similar to self-attention in which it uses the correlation between the _key_ and _query_ to determine if a agent needs more information. $m_{i,j} = \phi(u_i, k_i)$, $m_{i,i} \approx 1$ represents that the agent has sufficient information and doesn't need to communicate. Note that the _key_ here is different in size from the _query_ which is shared between agents. The shareable _key_ is processed in an asymmetric message method to reduce the dimension of _query_ so as to reduce the demand for bandwidth.

Based on the cross attention $m_{i,j}$ above, _when2com_ derives the _matching matrix_ $M$: $M = \sigma(m_{i,j})$ where $\sigma$ is a row-wise softmax. To construct communication groups, it prunes the less important connections with an _activation function_ which zeros out the elements less than $\delta$, in paper $\delta = \frac{1}{num\ of\ agents}$

#### How to fuse information

{{< figure src="/post/edge-systems/when2com_model.png" caption="(a) Multiple-Request Multiple-Support Model (b) Handshake-Communication(H-Com) Module" >}}

The agents fuse the information from the supporter by weighting each feature based on the value of matching matrix.

With this learning representation feature, the downstream tasks can target to any kinds of tasks which shares the cooperative property.

### Evaluation

The primary objective for evaluation is to validate the effectiveness of _Group Communication_ and _When to communicate_. To achieve this, the evaluation is designed around two tasks _collaborative semantic segmentation_ and _multi-agent 3D classification_. I will only cover the former one here. The task can be described as follows: Given the RGB images, depth maps, record pose of each agent, the goal is to produce an accurate 2D semantic segmentation mask for every agent involved. During the evaluation, the authors considers three variants of collaboration: (1) single-requester multi-supporter (2) multi-requester multi-supporter (3) multi-requester multi-partial-supporter. (1) under the consumption that if an agent is degraded, then its original, non-degraded information will be presented in one of the supporting agents. (2) multi agents can suffer from degradation and follow the setting of 1 (3) remove the assumption in 1 and the degraded agent should select the most informative views of variable degree of relevance. The key results are as follows: 

{{< figure src="/post/edge-systems/when2com_result1.png" caption="" >}}
{{< figure src="/post/edge-systems/when2com_result2.png" caption="" >}}
