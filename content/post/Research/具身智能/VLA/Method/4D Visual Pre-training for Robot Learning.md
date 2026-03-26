## TLDR

**FVP** 模型通过**下一帧点云预测（Next-Point-Cloud-Prediction）作为自监督目标，利用条件扩散模型**解决了 3D 机器人学习中**时空特征对齐难**和**动态常识缺失**的问题。

将视觉预训练从 3D 静态重建升维到 **4D（3D+Time）动态预测**，强制编码器理解动作带来的物理反馈。在 12 个真实世界任务中将 DP3 的平均成功率提升了 **28%**；在 RDT-1B (VLA) 上实现了显著的**长程任务**与**空间感知**提升。

## Metadata

- **发表期刊/会议**：ICCV 2025

- **论文作者**：Chengkai Hou1, Yanjie Ze3, Yankai Fu1, Zeyu Gao4, Songbo Hu2, Yue Yu2 Shanghang Zhang1, Huazhe Xu

- **研究机构**：1Peking University 2Tsinghua University 3Shanghai Qizhi Institute 4 CASIA 5 Shanghai AI Lab

- **论文链接**：https://arxiv.org/abs/2508.17230

- **关键词**： Vision-Language-Action (VLA), 3D Point Cloud, Diffusion Models, 4D Pre-training, World Models.

- **Code & Dataset & Weight：** https://4d-visual-pretraining.github.io/

- **BibTeX**：

- ```
  @article{cheng2025fvp,
      author    = {Chengkai Hou and Yanjie Ze and Yankai Fu and Zeyu Gao and Yue Yu and Songbo Hu and Shanghang Zhang and Huazhe Xu},
      title     = {FVP: 4D Visual Pre-training for Robot Learning},
      journal   = {ICCV},
      year      = {2025},
    }
  ```

## Problem Definition

### **研究问题**

目前的机器人视觉表示大多依赖 2D Web 数据（如 CLIP），丢失了 3D 空间感；而现有的 3D 预训练（如 PointMAE）又是静态的，无法让机器人理解动作（Action）与环境变化（Dynamics）之间的因果关系

### **形式化定义**

**输入**：历史观测点云序列 $o_{1:t-1}$，当前/历史动作 $a_{1:t-1}$。

**输出**：预测下一帧点云 $o_t$。

## Challenges

### **核心挑战**

互联网缺乏大规模、高质量的带有动作标注的 3D 点云数据（不像 2D 图片那样随处可见）。

### **本文针对性解决的挑战**

利用自监督学习突破数据瓶颈。作者认为不需要无穷无尽的标注，只需通过“预测未来”就能倒逼模型学到最核心的物理表征。这种选择非常合理，因为它符合 **JEPA（联合嵌入预测架构）** 的前沿趋势。

## Angle & Motivation

### **切入角度**

**4D 时空预测**。作者不再满足于让模型知道“杯子在哪里”，而是要模型知道“我推一下，杯子会到哪里”。

## Methodology

### **实现细节**

可视化下一帧点云预测的任务

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125234538932.png" alt="image-20260125234538932" style="zoom:67%;" />

流程图

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125234519162.png" alt="image-20260125234519162" style="zoom:67%;" />



**特征提取**：利用 3D Encoder（如 PointNet++）将历史帧编码为潜在表示 $z$。

**生成式预测**：采用 **Point-Voxel Diffusion** 结构。将 $z$ 作为 Condition，配合高斯噪声，通过去噪过程恢复出下一帧点云。

**公式核心**：

$$\mathcal{L} = \mathbb{E}_{\epsilon \sim \mathcal{N}(0,I)} \left\| \epsilon - \epsilon_\theta(o_{t,T,+}, T) \right\|^2_2$$

其中 $o_{t,T,+}$ 融合了噪声点云与历史特征 $z$。

### **性能提升的本质**

**特征的动力学对齐**。由于模型在预训练时被迫理解了物理演化，其提取的 $z$ 向量天然携带了空间距离、物体惯性和运动趋势等关键信息，极大减轻了下游策略模型的负担。

## Experiments

### **实验设置与指标**

**Benchmark**：仿真（Adroit, MetaWorld）、真实世界（12个任务，含单臂、双臂、人形机器人）。

**指标**：Success Rate（成功率）。

### 对比实验

对比2D数据上训练的模型：

![image-20260125234619098](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125234619098.png)

对比了 RDT-1B 在五种真实世界任务（PickSquare, PlaceBottle, PutBox, StackBowl, WipePlate）中的表现。

对比的四种配置（模型变体）：

- **2D Image Input**：原生的 RDT-1B，只吃 2D 图像特征。这是 VLA 的基准线。
- **2D Image Input by R3M**：在原生 2D 输入的基础上，把 Encoder 换成经过 R3M（2D 自监督预训练）强化过的。
- **3D point cloud Input**：给 RDT-1B 额外增加点云输入，但 Encoder 是**随机初始化或常规训练**的（没有经过 FVP 预训练）。
- **3D encoder pretrained by FVP**：本文的最终形态——RDT-1B + 点云输入 + **FVP 预训练过的 3D Encoder**。

![image-20260125234629450](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125234629450.png)

Table 3 是对 VLA 模型核心能力的“压力测试”，评估了四个维度：空间理解、知识迁移、语言理解和长程任务。

实验设置：

- **数据集**：这里的 FVP 是在 **Robomind（域外机器人数据集）** 上进行的预训练。
- **对比项**：2D 图像基准 vs. 3D 点云输入 vs. **FVP 增强版**。



![image-20260125234638162](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125234638162.png)





- **DP3 + FVP / RISE + FVP**：分别在两个最强的 3D 模仿学习骨干（Backbone）上应用了 FVP。
- **DP3 (Baseline)**：视觉部分没有经过任何预训练。
- **DP3 + PointMAE / STRL / C2P**：用了其他主流的 3D 预训练方法。



<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125234650080.png" alt="image-20260125234650080" style="zoom: 67%;" />



**2D 预训练：如 **MVP**、**R3M**（也就是 Table 1 里那些巨头）。

**结构改进**：如 **EquiBot**（等变性机器人）和 **EquiDiff**。这些模型试图通过改变神经网络的对称性来提升性能。

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125234600858.png" alt="image-20260125234600858" style="zoom:67%;" />



### 消融实验

**DP3 + FVP (Full Model)**：完整体，表现最好，几乎全满分。

**Current Frame Input (无历史帧)**：

- **变化**：在预训练和推理时只给当前这一帧点云，不给过去的信息。
- **结果**：成功率大幅下降（平均下降了 25%-35%）。
- **深度解读**：这证明了 **“4D”**（时间维度）是 FVP 的灵魂。没有历史帧，模型就无法理解物体的运动趋势（Velocity/Dynamics），只能算是一个 3D 版本的静态感知模型。

**Freeze Visual Encoder (冻结编码器)**：

- **变化**：预训练完后，在下游任务微调时，不准修改 Encoder 的参数。
- **结果**：表现最差（跌到了 50% 左右）。
- **深度解读**：这反映了目前机器人学的一个硬伤——**域差异（Domain Gap）**。即便 FVP 在 Robomind 上学到了物理常识，但由于预训练数据集和真实实验场景的传感器噪声、背景光照不同，如果不微调，特征就无法精准匹配新环境。

![image-20260125234708425](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125234708425.png)



预训练时，给模型看多长的历史（1 帧到 4 帧）最合适？

![image-20260125234718992](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125234718992.png)



## Summary & Evaluation

### **值得 Follow 的点**

**特征融合方式**：如何将 3D 点云特征作为 RDT 等 VLA 模型的额外 Token 注入。

**自监督目标**：利用预测下一帧作为通用的预训练任务。

