---
title: "Computer Architecture - A Quantitative Approach"
date: 2024-03-13
# weight: 1
# aliases: ["/first"]
tags: ["computer architecture", "course"]
author: "Yuheng"
showtoc: true
tocopen: true
draft: true
math: true
comments: false
description: "notes of KAIST cs510 computer architecture by Prof.Jaehyuk Huh"
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: false
ShowWordCount: true
ShowRssButtonInSectionTermList: false
ShowShareButtons: false
---

> Reference book: Computer Architecture - A Quantitative Approach Sixth Edition

## Pipeline

Before introducing pipeline, we define how we measure the performance of _CPU Time_ through sequential execution model.

The _CPU Time_ which is also called as _CPU execution time_ represents the time the CPU spends on one task, exclusive of I/O time. 


$$
\begin{align*}
CPU\ time &= num\ clock\ cycles\ for\ a\ program * clock\ cycle\ time \\

&= \frac{num\ CPU\ clock\ cycles\ for\ a\ program}{clock\ rate}
\end{align*}
$$

Another way to think about the measurement is that ti equals the number of instructions executed multiplied by the average clock cycle per instruction(**CPI**).

$$

\begin{align*}
    CPU\ time &= Instruction\_count * CPI * clock\_cycle \\
    &= \frac{Instruction\_count * CPI}{clock\_rate}
\end{align*}

$$


### Non-pipelined
For single-cycle implementation, the _CPI_ equals 1 where the _Clock Cycle Time_ is the latency of the longest execution path.

### Pipelined

The key idea of pipelining is to run different instruction which won't affect each other simultaneously to increase the number of instructions completed per unit of time. In other words, after pipelining, the individual instruction takes longer, but the _throughput_ increase. The _Cycle time_ becomes the slowest logic time + latching overheads. To speedup, ideal pipelining all stages in a balanced manner other than that the speedup is less.

{{< figure src="/post/computer-architecture/speedup_of_pipeline.png" caption="Speedup of pipelining" >}}

---

However, there are situations, called _hazards_, that prevent the next instruction in the instruction stream from executing during its designated clock cycle which reduce the performance from the ideal speedup. 

#### Structural Hazards

_Structural Hazards_ occurs when multiple instructions attempt to use the same structure at the same clock cycle (but the structure allows only one access at a time).

Solution:
1. stall pipeline
2. add one more port

#### Data Hazards

_Data Hazards_ occurs when there is a data dependence between pipelined instructions. eg. RAW(read after write)

Solution:
1. Bypassing(forwarding): pass the data once it is computed in the intermediate of pipeline
2. Compiler may do something, eg. rearrange the order without affecting the result
3. Pipeline install

#### Control Hazards

_Control Hazards_ happens when branch target is determined, there  is a delay between IF and EX. 

Solution:
1. Pipeline install
2. Speculation of branch outcome
   1. Branch prediction: assuming a given outcome and proceeding without waiting to see the actual branch outcome
   2. Branch target buffer: in the IF stage caches the branch target address, but we also need to fetch the next sequential instruction. The prediction bit in IF/ID selects which “next” instruction will be loaded into IF/ID at the next clock edge.

## Memory Hierarchy