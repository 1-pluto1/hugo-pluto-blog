## TLDR

**EmbodiedMAE** 通过在增强的 **DROID-3D** 数据集上进行**多模态掩码自编码预训练（RGB+Depth+PC）**，解决了机器人操作中 3D 空间感知缺失和领域鸿沟的问题。提出了首个原生支持 3D 多模态、可扩展（Scalable）且专为具身智能设计的视觉基础模型（VFM）。在 70 个仿真任务和 20 个真实世界任务中持续超越 DINOv2、SPA 等 SOTA 模型；尤其是 **Large-scale RGBD 模型性能甚至超过了 Giant-scale 仅 RGB 模型**。

## Metadata

- **发表期刊/会议**：arXiv

- **论文作者**：Zibin Dong♡, Fei Ni♡, Yifu Yuan♡, Yinchuan Li♢, Jianye Hao∗♡,

- **研究机构**：Tianjin University, ♢Huawei Noah’s Ark Lab

- **论文链接**：https://arxiv.org/abs/2505.10105

- **关键词**：Embodied AI, VLA, Masked Autoencoder, 3D Representation, Robot Manipulation

- **Code & Dataset & Weight：** 没开源

- **BibTeX**：

- ```
  @misc{dong2025embodiedmaeunified3dmultimodal,
        title={EmbodiedMAE: A Unified 3D Multi-Modal Representation for Robot Manipulation}, 
        author={Zibin Dong and Fei Ni and Yifu Yuan and Yinchuan Li and Jianye Hao},
        year={2025},
        eprint={2505.10105},
        archivePrefix={arXiv},
        primaryClass={cs.RO},
        url={https://arxiv.org/abs/2505.10105}, 
  }
  ```

  

## Problem Definition

### **研究问题**

现有的视觉基础模型（VFM）大多在互联网图片上训练，缺乏对机器人操作至关重要的 3D 空间感知和“厘米级”的深度理解。

### **形式化定义**

**输入：** 机器人多模态观测序列 $\{RGB, Depth, Point Cloud\}$。

**输出：** 统一的跨模态潜在表征 $h$，用于下游策略网络 $\pi$ 生成动作序列 $a$。

## Challenges

### **核心挑战**

**数据鸿沟：** 现有的 3D 数据集要么是室外大场景，要么是根据 3D 图像重建的深度，精度极差。

**架构难点：** 简单堆叠深度通道往往会导致性能退化，缺乏有效的多模态融合机制。

### **本文针对性解决的挑战**

重点攻克了 **“高质量 3D 具身数据匮乏”** 和 **“原生 3D 表征学习效率低”** 的问题。选择在 DROID 基础上用硬件 SDK 恢复深度，这种做法比纯算法预测更合理、更具物理真实性。

## Angle & Motivation

### **切入角度**

自监督掩码自编码（MAE）+ 跨模态特征对齐。

### **创新性**

引入 **Dirichlet 分布掩码策略**，强迫模型进行跨模态“补全”推理。这种从 2D 重建 3D 或从 3D 推理 2D 的能力，打破了以往模态孤立训练的范式。

## Methodology

### **实现细节**



![image-20260125154646152](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125154646152.png)



![image-20260125154803827](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125154803827.png)



**DROID-3D 构建：** 利用时间融合（Temporal Fusion）和硬件校准提取 76K 轨迹的高保真点云。

**编码器：** 使用 ViT 架构，配合专用的点云分块器（DP3 编码器）。

**解码器：** 采用交叉注意力进行显式融合，共享 Transformer 组件以节省计算量。

**知识蒸馏：** 将 Giant 模型的“解题思路”通过顶、中（3/4层）、底三处对齐，传授给能在 4090 上跑的 Small/Base/Large 模型。

$$L_{MAE} = \mathbb{E}_{(I,D,P) \sim \mathcal{D}, Dir(\alpha)} \left[ \underbrace{\|g(h_I, h) - I_2\|_2^2}_{\text{RGB}} + \underbrace{\|g(h_D, h) - D_2\|_2^2}_{\text{深度}} + \underbrace{\|g(h_P, h) - P_2\|_2^2}_{\text{点云}} \right]$$



使用如下的策略网络来生成动作：

![image-20260125154833826](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125154833826.png)



## Experiments

### **实验设置与指标**

LIBERO (40任务), MetaWorld (30任务), 真机 (20任务)。核心指标是 **任务成功率（Success Rate）**。

### 对比实验

模型在不同情况下的视觉预测：

![image-20260125154946641](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125154946641.png)



在LIBERO上的指标

![image-20260125155032100](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125155032100.png)



在MetaWorld上的指标



![image-20260125155042553](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125155042553.png)





真机指标：

![image-20260125155056528](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125155056528.png)



### **消融实验**

对Masking Ratio、Feature Alignment、Loss Ratio β的消融实验：

![image-20260125155110282](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125155110282.png)



## Summary & Evaluation

### **值得 Follow 的点**

**3/4 深度对齐法**：在做模型压缩或蒸馏时可以直接套用。

**Dirichlet 掩码**：处理多模态不平衡输入时的绝佳策略。

