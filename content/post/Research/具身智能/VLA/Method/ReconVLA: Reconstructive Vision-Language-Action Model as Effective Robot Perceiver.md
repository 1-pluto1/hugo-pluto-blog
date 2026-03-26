## TLDR

**ReconVLA** 通过引入基于 **Diffusion Transformer** 的**局部注视区域（Gaze Region）重建**任务，解决了传统 VLA 模型**视觉注意力分散**的问题，实现了机器人从“看个大概”到“精准注视”的跨越。在 CALVIN “堆叠方块”（最具挑战性的微操任务）上成功率提升了 **20.2%**；在未见物体的现实泛化实验中，成功率远超现在的SOTA。

## Metadata

- **发表期刊/会议**：AAAI
- **论文作者**：Wenxuan Song1, Ziyang Zhou1, Han Zhao2,3, Jiayi Chen1, Pengxiang Ding2,3, Haodong Yan1, Yuxin Huang1, Feilong Tang4, Donglin Wang2, Haoang Li1
- **研究机构**：1The Hong Kong University of Science and Technology (Guangzhou) 2Westlake University 3Zhejiang University 4Monash University
- **论文链接**：https://arxiv.org/abs/2508.10333
- **关键词**：VLA, Diffusion Transformer, Visual Grounding, Precision Manipulation
- **Code & Dataset & Weight：** https://github.com/OpenHelix-Team/ReconVLA
- **BibTeX**：
- ```
  @article{song2025reconvla,
    title={ReconVLA: Reconstructive Vision-Language-Action Model as Effective Robot Perceiver},
    author={Song, Wenxuan and Zhou, Ziyang and Zhao, Han and Chen, Jiayi and Ding, Pengxiang and Yan, Haodong and Huang, Yuxin and Tang, Feilong and Wang, Donglin and Li, Haoang},
    journal={arXiv preprint arXiv:2508.10333},
    year={2025}
  }
  ```

  

## Problem Definition

### **研究问题**

传统的 VLA 模型（如 OpenVLA）在执行动作时存在“视觉注意力漂移”现象。模型往往关注背景或无关区域，导致在需要极高精度的任务（如对准、堆叠）中表现糟糕。

### **形式化定义**

**输入**：当前图像 $I$、文本指令 $S$、机器人本体感受（Proprioception）。

**输出**：可执行动作 $A$（Action Tokens）。

**附加任务**：重建注视区域的潜在标记 $z_0$（Gaze Region Latent Tokens）。

### **价值与意义**

解决了端到端机器人模型“理解力强、执行力差（微操烂）”的痛点

## Challenges

### **核心挑战**

如何在不引入高延迟外部检测器（如 YOLO）的情况下，让模型自主、实时地在内部建立起“物体-像素”的精准映射？

### **本文针对性解决的挑战**

通过**隐式视觉重建**取代**显式坐标回归**。避免了 LLM 对数字（坐标）不敏感的问题，转而发挥 LLM 在特征表征上的优势。

## Angle & Motivation

### **切入角度**

**模拟人类注视（Human Gaze）**。人类在操作时，视野中心（中央凹）极其清晰，周围模糊。

### **合理性与重要性**



<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125012848386.png" alt="image-20260125012848386" style="zoom:67%;" />



作者通过可视化证明，常规 VLA 的 Attention Map 是分散的。这种统计学上的“注意力溃散”是导致操作失败的元凶。

### **创新性**

打破了“感知归感知，动作归动作”的固有范式，提出通过**生成（Generation）**任务来倒逼**感知（Perception）**能力的提升。

这是已有的一些范式：

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125012932582.png" alt="image-20260125012932582" style="zoom:50%;" />

## Methodology

### **实现细节**

1. **Backbone**：LLaVA-7b (Qwen2 + SigLIP)。

2. **Gaze Reconstruction**：提取目标物体的局部 Patch，通过 VAE 编码为 $z_0$。

3. **DiT Head**：一个轻量级的扩散 Transformer，以 LLM 的隐藏状态 $h_R$ 为条件，学习从噪声 $z_t$ 还原 $z_0$。

4. **Loss 函数**：

   $$\mathcal{L}_{ReconVLA} = \mathcal{L}_{VLA}^{action} + E_{t,\epsilon} [\|D(z_t; h_R, t) - \epsilon\|^2]$$



<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125012951967.png" alt="image-20260125012951967" style="zoom:67%;" />



### **性能提升的本质**

**“梯度约束力”**。扩散重建任务是一个极其“稠密”的信号。为了能画出那个物体，LLM 必须强迫自己的注意力矩阵“咬死”那个物体的像素。

## Experiments



![image-20260125013016797](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125013016797.png)

### **实验设置与指标**

**Benchmark**：CALVIN (仿真长程任务)、Real-world (AgileX PiPer 机械臂)。

**指标**：子任务成功率 (Success Rate)、平均完成长度 (Avg. Length)。

### 对比实验

**VS 显式定位 (EG)**：ReconVLA 减少了冗余输入，性能更好。

**VS 思维链定位 (CG)**：ReconVLA 避免了坐标预测的误差。

![image-20260125013055518](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125013055518.png)



**VS 生成式规划 (GR-1)**：在当前帧理解上更胜一筹，微操精度更高。

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125013111250.png" alt="image-20260125013111250" style="zoom:67%;" />



真实世界实验：

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125013242233.png" alt="image-20260125013242233" style="zoom:67%;" />



<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125013221625.png" alt="image-20260125013221625" style="zoom:67%;" />

### 消融实验

没有 2M 规模预训练，模型在未见物体上会彻底“失明”。

重建“局部（Gaze）”的效果显著优于重建“全图（Entire Image）”。



<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125013133995.png" alt="image-20260125013133995" style="zoom:67%;" />







## Summary & Evaluation

### **总体评价**

没有通过堆参数量来解决问题，而是通过精妙的**辅助任务设计**，利用 Diffusion 切开了 VLA 的感知黑盒。

### **值得 Follow 的点**

1. **数据自动化**：利用 Grounding DINO 自动生成大量带局部标注的数据。
2. **模块化设计**：Diffusion Head 可以作为一个插件，移植到任何 VLM 上增强其空间感知力。

### **局限性与机会**

- **延迟问题**：虽然是轻量级 DiT，但扩散模型的多次去噪步数（Sampling Steps）在超高频控制中可能仍是瓶颈。
- **未来方向**：把动作头和重建头统一替换为 **流匹配（Flow Matching / $\pi_0$ 风格）**，或许能在保持精度的同时大幅提升推理速度。