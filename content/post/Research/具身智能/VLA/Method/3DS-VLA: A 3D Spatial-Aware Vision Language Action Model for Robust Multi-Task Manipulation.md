## TLDR

**3DS-VLA** 通过 **2D-to-3D 位置对齐**和**序列化空间约束**，在不增加编码器参数的前提下，让预训练 2D VLM 获得了精准的 3D 空间操作能力。利用“非参数化 3D 分词”和“PE 对齐”机制，实现了 2D 语义先验与 3D 几何特征在同一坐标系下的无损融合。在 **RLBench** 26 项任务中刷新 SOTA（超过 **3D-Lotus** 4%+），且在真实世界中能承受高达 **10cm** 的坐标噪声。



## Metadata

- **发表期刊/会议**：CoRL
- **论文作者**：Xiaoqi Li1,2, Liang Heng1, Jiaming Liu3, Yan Shen1,2, Chenyang Gu3, Zhuoyang Liu3, Hao Chen4, Nuowei Han1, Renrui Zhang4, Hao Tang3, Shanghang Zhang3, Hao Dong1,2
- **研究机构**：**1CFCS, School of Computer Science, Peking University, 2PKU-Agibot Lab, 3State Key Laboratory of Multimedia Information Processing, School of Computer Science, Peking University 4CUHK**
- **论文链接**：https://openreview.net/forum?id=dT45OMevL5&referrer=%5Bthe%20profile%20of%20Shanghang%20Zhang%5D(%2Fprofile%3Fid%3D~Shanghang_Zhang4)#discussion
- **关键词**：Vision-Language-Action, Robotic Manipulation, Imitation Learning
- **Code & Dataset & Weight：** 暂无开源
- **BibTeX**：

## Problem Definition

### **研究问题**

目前的 VLA 模型存在两极分化：2D 模型懂语义但不懂深度，容易在接触物体的最后几厘米“抓瞎”；3D 模型（如 DP3）有深度但没数据，泛化性极差。

### **形式化定义**

**输入**：当前帧图像 $i_t$、点云 $p_t$、任务语言 $l$、3D 关键点约束 $k_t$、机器人状态 $r_t$。

**输出**：在 $SE(3)$ 空间中的下一帧动作预测 $\hat{a}_{t+1}$（含位置 $x \in \mathbb{R}^3$、四元数 $\theta \in \mathbb{R}^4$ 及夹爪状态 $g$）。

## Challenges

### **核心挑战**

**数据稀缺**：大规模 3D 机器人操作数据集极度匮乏。

**空间损耗**：现有的 3D-to-2D 投影或 2D-to-3D 特征提升（Lifting）都会造成几何细节丢失。

### **本文针对性解决的挑战**

重点攻克了 **“如何复用大规模 2D 预训练模型进行 3D 空间推理”**。通过让模型直接输入原始点云并对齐 2D 位置嵌入，规避了重头训练 3D 模型的成本。

## Angle & Motivation

### **切入角度**

**表征层面对齐 + 逻辑层面约束**。

### **合理性与重要性**

作者通过对比实验发现，2D VLA 的失败大多发生在“接触物体前的一瞬间”。这证明了显式的 3D 几何（点云）和显式的目标引导（关键点）是解决机器人任务的“最后一块拼图”。

### **创新性**

有启发性。它摒弃了复杂的 3D 神经网络，转而利用 Transformer 的**置换不变性**，通过 PE（位置编码）的变换实现了跨模态的统一。

## Methodology



![image-20260125022904092](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125022904092.png)



![image-20260125022917974](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125022917974.png)



### **实现细节**

**非参数化 3D Tokenizer**：通过 FPS（采样）和 kNN（聚类）将 2048 个点云聚合成 2D 规模的 Token，不增加计算参数。

**2D-to-3D 位置对齐**：通过相机参数将 3D Token 投影回 2D 像素，直接“借用”对应的 CLIP 预训练 PE。

**文本化约束**：将 3D 坐标写进 Prompt，利用 LLaMA 的自回归特性预测 200 分箱（bins）化的动作。

## Experiments

### **实验设置与指标**

**Benchmark**：RLBench (21 单臂 + 5 双臂) + 10 项真机任务。

**指标**：Success Rate (成功率)。

### 对比实验



RLBench 单臂实验

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125022936934.png" alt="image-20260125022936934"  />、、

RLBench 双臂实验



![image-20260125022952474](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125022952474.png)

真实世界实验：

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125023026009.png" alt="image-20260125023026009" style="zoom:67%;" />



### 消融实验

**关键点约束**：贡献最大，提升 **19%**。

**PE 对齐**：贡献 **2%**，但在精度任务中是质变的保障。

![image-20260125023005120](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125023005120.png)



## Summary & Evaluation

### **总体评价**

深刻理解了 Transformer 的特性，用最轻量化的方式解决了具身智能最重的痛点。

### **值得 Follow 的点**

1. **文本化坐标**：在做 VLA 模型时，考虑将几何特征转为文本输入，能利用 LLM 的预训练逻辑。
2. **2D-to-3D 对齐**：这种“不改参数、只改输入”的思路。

