## TLDR



https://blog.csdn.net/v_JULY_v/article/details/149352338

大规模 VLA 算得太慢（延迟高），机器人要么停顿等结果，要么换块执行时由于动作不连贯而“抽搐”。这篇论文提出了一种异步新方法，让机器人一边做动作一边丝滑地构思下一步。**RTC (Real-Time Chunking)** 算法通过将异步动作块生成转化为**推理端图像修复 (Inpainting)** 问题，解决了大规模 VLA 模型在机器人控制中的高延迟与动作不连贯问题。将异步动作衔接建模为带引导的去噪过程，并引入**软掩码 (Soft Masking)** 机制来兼顾动作连贯性与环境反应速度。在 6 项极具挑战的双臂协作任务中，执行速度比同步推理提升了 **20%**，且在注入 **>300ms** 的极端延迟下仍能保持极高的任务成功率。

## Metadata

- **发表期刊/会议**：arXiv
- **论文作者**：Kevin Black、Manuel Y. Galliker、Sergey Levine
- **研究机构**：Physical Intelligence、UC Berkeley
- **论文链接**：https://arxiv.org/abs/2506.07339
- **关键词**：
- **Code & Dataset & Weight：** https://github.com/Physical-Intelligence/openpi
- **BibTeX**：
- ```
  @misc{black2025realtimeexecutionactionchunking,
        title={Real-Time Execution of Action Chunking Flow Policies}, 
        author={Kevin Black and Manuel Y. Galliker and Sergey Levine},
        year={2025},
        eprint={2506.07339},
        archivePrefix={arXiv},
        primaryClass={cs.RO},
        url={https://arxiv.org/abs/2506.07339}, 
  }
  ```

  

## Problem Definition

### **研究问题**

如何在大规模 VLA 模型推理延迟远高于控制器采样频率的情况下，实现流畅、实时且连贯的机器人闭环控制？

### **形式化定义**

**输入**：当前观察 $o_t$ 以及正在执行的上一动作块的剩余部分 $A_{prev}$。

**输出**：一个新的动作块 $A_{new}$，其前 $d$ 个动作需与正在执行的动作严格对齐，后部则根据新观察进行预测。

### **价值与意义**

VLAs 虽强但运行慢（如 $\pi_0$ 延迟常 >40ms），RTC 允许我们在不牺牲模型规模的前提下，让机器人在动态环境下保持反应灵敏且动作丝滑。

## Challenges

### **核心挑战**

高参数量模型的**推理延迟 (Latency)** 与机器人对**实时性 (Real-time)** 要求之间的矛盾。

### **本文针对性解决的挑战**

**异步动作块的不连续性**。在异步执行时，新生成的动作块往往会跳变到与旧动作块不同的执行模式（Mode-jumping），导致机器人剧烈抖动或任务失败。

## Angle & Motivation

### **切入角度**

将动作块的衔接视为**条件引导下的“修复” (Inpainting via Guidance)** 。

### **合理性与重要性**

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260123144641771.png" alt="image-20260123144641771" style="zoom:50%;" />

文中通过图 2 证明了朴素的异步方法会导致 OOD（分布外）的高加速度。流匹配模型在图像领域已证明了强大的 Inpainting 能力，迁移至动作序列生成具有天然的数学合理性 

## Methodology

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260123145007768.png" alt="image-20260123145007768" style="zoom:50%;" />



### **实现细节**

**Guided Inference**：在流匹配去噪过程中，加入一个梯度项，引导生成的动作向 $A_{prev}$ 靠拢 。

**Soft Masking**：使用指数衰减权重 $W$。对注定要执行的 $d$ 个动作给予权重 1，中间重叠区权重逐渐减小，末端新生成的动作为 0 。

**Weight Clipping**：引入 $\beta$ 裁剪引导权重，防止由于去噪步数过少导致计算不稳定 。

### **逻辑闭环**

该方法通过冻结即将发生的动作并平滑预测未来动作，完美解决了异步计算产生的时间差导致的动作跳变问题 。

### **性能提升的本质**

利用扩散/流匹配模型的多模态覆盖能力，在保持原有策略分布的同时，找到了与过去动作最兼容的采样路径。

## Experiments

### **实验设置与指标**

**Benchmark**：Kinetix 仿真器（12 个动态任务，如投掷、平衡）及 6 个真实世界双臂任务 。

**指标**：成功率 (Solve Rate)、任务吞吐量 (Throughput) 。

### 对比实验

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260123145051788.png" alt="image-20260123145051788" style="zoom:50%;" />



<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260123145208412.png" alt="image-20260123145208412" style="zoom:50%;" />







对比了同步推理 (Synchronous)、时间集成 (TE) 以及 BID 。实验涵盖了人为注入的 0-200ms 延迟维度 。RTC 在各种延迟下均表现出极强的鲁棒性，而 TE 在高延迟下会导致机器人触发保护性停机 

### 消融实验

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260123145234797.png" alt="image-20260123145234797" style="zoom:50%;" />

图 8 证实了**指数衰减 (Exponential Decay)** 掩码优于线性和硬掩码，且图 7 证明了 $\beta=5$ 的裁剪对防止动作发散至关重要

## Summary & Evaluation

### **总体评价**

它不改变模型训练逻辑，仅通过优雅的推理端数学变换，解决了 VLA 落地最头疼的延迟问题。 

### **值得 Follow 的点**

**推理端修复思想**：对于处理时序预测任务，这种锁定过去，修复未来的思路非常通用。

**软掩码调度策略**：如何平衡“历史一致性”与“实时反馈”的权重分配。

