## TLDR

**GLaD** 框架通过将 **VGGT** 的 3D 几何特征蒸馏至 **LLM 最终隐藏状态**，解决了 VLA 模型缺乏空间推理和易受视觉扰动影响的问题。提出了**后期隐藏状态对齐（Late-stage Hidden State Alignment）**，将几何先验深度耦合进多模态决策过程，而非仅仅作为视觉输入。在 **LIBERO** 达到 **94.1%** 成功率（SOTA），在 **LIBERO-PRO** 物体扰动测试中比基线 UniVLA 提升了 **19%~60%**。

## Metadata

- **发表期刊/会议**：arXiv

- **论文作者**：Minghao Guo, Meng Cao, Jiachen Tao, Rongtao Xu, Yan Yan, Xiaodan Liang, Ivan Laptev, Xiaojun Chang

- **研究机构**：University of Illinois Chicago,MBZUAI

- **论文链接**：https://arxiv.org/abs/2512.09619

- **关键词**：Vision-Language-Action Models, Pretraining, Geometry Distillation, Robot Manipulation, Spatial Reasoning.

- **Code & Dataset & Weight：**

- **BibTeX**：

- ```
  @misc{guo2025gladgeometriclatentdistillation,
        title={GLaD: Geometric Latent Distillation for Vision-Language-Action Models}, 
        author={Minghao Guo and Meng Cao and Jiachen Tao and Rongtao Xu and Yan Yan and Xiaodan Liang and Ivan Laptev and Xiaojun Chang},
        year={2025},
        eprint={2512.09619},
        archivePrefix={arXiv},
        primaryClass={cs.RO},
        url={https://arxiv.org/abs/2512.09619}, 
  }
  ```

  

## Problem Definition

### **研究问题**

如何让仅依赖 2D 视觉编码器（如 SigLIP）的 VLA 模型获得理解 3D 空间结构、物体关系和深度的能力？

### **形式化定义**

**输入**：RGB 图像序列 $O$ + 自然语言指令 $L$。

**输出**：机器人控制动作序列 $A$。

**约束**：在 LLM 的推理过程中，嵌入几何约束 $F_{3d}$。

## Challenges

### **核心挑战**

2D 预训练编码器（CLIP/SigLIP）擅长语义识别（这是什么），但不擅长几何定位（这在哪里、多远、什么形状）。

论文重点证明了通过几何蒸馏，可以防止模型过度依赖颜色、纹理等肤浅视觉特征（Pattern Matching），转向真正的物体属性理解。



## Angle & Motivation

### **切入角度**

**知识蒸馏（Knowledge Distillation）**。引入预训练的几何基座模型（VGGT）作为教师，在训练过程中通过 Loss 约束，将 3D 先验“注射”进 VLA。

### **合理性与重要性**

文中通过图 1 证明了普通 VLA 会在物体堆叠时产生错误的 Attention。由于机器人操作本质上是 3D 任务，引入 3D 归纳偏置（Inductive Bias）是必然趋势。

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125045320041.png" alt="image-20260125045320041" style="zoom:67%;" />

## Methodology

### **实现细节**



<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125043244078.png" alt="image-20260125043244078" style="zoom:67%;" />

**Teacher**: 冻结的 VGGT，提取单帧 3D 表征（深度、点云、相机参数）。

**Student**: LLaMA-2-7B + DINOv2/SigLIP。

**Alignment**: 通过一个两层 MLP，将 LLM 第 32 层（最后一层）对应视觉 Token 的隐藏状态投影到 VGGT 特征空间。

**Loss**: $L_{total} = L_{VLA} + \lambda \cdot ||H_{aligned} - F_{3d}||^2_2$

### **性能提升的本质**

通过强迫 LLM 内部状态对齐几何特征，模型学会了关注物体的 3D 骨架（Affordance），从而在物体外观改变时依然能找准抓取点。

## Experiments

### **实验设置与指标**

LIBERO (130 任务) & LIBERO-PRO (鲁棒性)，指标为 Success Rate (%)。

### 对比实验

在LIBERO上的鲁棒性实验：

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125043313264.png" alt="image-20260125043313264" style="zoom:67%;" />





<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125043413991.png" alt="image-20260125043413991" style="zoom:67%;" />



在LIBERO基准上的对比实验：

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125043327920.png" alt="image-20260125043327920" style="zoom:67%;" />

### 消融实验



模型架构的消融实验：

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125043358993.png" alt="image-20260125043358993" style="zoom:67%;" />





## Summary & Evaluation

### **总体评价**

它没有改动 VLA 的基础算力结构，而是通过巧妙的蒸馏方案，低成本地解决了 2D VLA 的致命伤，实验论证逻辑严密。

### **值得 Follow 的点**

- **Hidden State 对齐方案**：这种“不增加推理开销，只在训练时加约束”的思路非常适合工程落地。
- **VGGT 作为特征源**：VGGT 提供的几何特征比纯深度图更适合机器人。

