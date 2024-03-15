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

Before introducing pipeline, we first establish how we measure the performance of _CPU Time_ through sequential execution model.

The _CPU Time_ which is also called as _CPU execution time_ represents the time the CPU spends on one task, exclusive of I/O time. 

$$
\begin{align}
CPU\ time &= num\ clock\ cycles\ for\ a\ program * clock\ cycle\ time \\
&= \frac{num\ CPU\ clock\ cycles\ for\ a\ program}{clock\ rate}
\end{align}
$$

Alternatively, it can be understood as the product of instructions executed and the average clock cycle per instruction(**CPI**):

$$
\begin{align}
    CPU\ time &= Instruction\_ count * CPI * clock\_ cycle \\
    &= \frac{Instruction\_ count * CPI}{clock\_ rate}
\end{align}
$$

### Non-pipelined
In a single-cycle implementation, the CPI is 1, implying that the _clock cycle time_ equals the duration of the longest execution path. This model provides a baseline for assessing the improvements offered by pipelining.

### Pipelined

Pipelining's core principle involves concurrently executing non-interfering instructions to enhance the rate at which instructions are completed. Although each instruction may require more time in a pipelined process, the overall throughput—the number of instructions executed per time unit—increases. The cycle time in a pipeline is determined by the duration of the slowest pipeline stage, including any additional latching overheads. For optimal speedup, it is critical to evenly balance the execution time across all pipeline stages to mitigate bottlenecks.

{{< figure src="/post/computer-architecture/speedup_of_pipeline.png" caption="Speedup of pipelining" >}}

---

However, there are certain situations, known as _hazards_, that prevent the next instruction in the instruction stream from executing during its designated clock cycle which reduce the performance from the ideal speedup. 

#### Structural Hazards

_Structural Hazards_ occurs when multiple instructions attempt to use the same structure at the same clock cycle (but the structure allows only one access at a time).

Solution:
1. stall pipeline(inserting NOP instructions)
2. increasing the hardware resources (like adding ports to a memory unit)

#### Data Hazards

_Data Hazards_ occurs when there is a data dependence between pipelined instructions. eg. RAW(read after write)

Solution:
1. Bypassing(forwarding): pass the data once it is computed in the intermediate of pipeline
2. Compiler may do something, eg. rearrange the order without affecting the result
3. Pipeline stall(inserting NOP instructions)

#### Control Hazards

_Control Hazards_ happens when branch target is determined, there  is a delay between IF and EX. 

Solution:
1. Pipeline stall(inserting NOP instructions)
2. Speculation of branch outcome
   1. Branch prediction: assuming a given outcome and proceeding without waiting to see the actual branch outcome
   2. Branch target buffer: in the IF stage caches the branch target address, but we also need to fetch the next sequential instruction. The prediction bit in IF/ID selects which “next” instruction will be loaded into IF/ID at the next clock edge.

## Memory Hierarchy