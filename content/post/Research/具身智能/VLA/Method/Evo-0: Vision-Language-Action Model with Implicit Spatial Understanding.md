## TLDR

**Evo-0** 模型通过引入一个**隐式几何特征融合模块 (VGGT)**，解决了现有 VLA 模型因 2D 预训练导致的 **3D 空间感知缺失**问题。设计了一个“即插即用”的几何感知支路，利用视觉几何基础模型（VGFM）提供深度感知，而无需依赖物理深度传感器。在 RLBench 模拟器中比基准模型 $\pi_0$ 成功率提升 **15%**；在现实世界任务中平均成功率提升 **28.88%**，且在干扰环境下表现出极强的鲁棒性。

## Metadata

- **发表期刊/会议**：arXiv

- **论文作者**：Tao Lin∗, Gen Li∗, Yilei Zhong, Yanwen Zou, Yuxin Du, Jiting Liu, Encheng Gu4, Bo Zhao†

- **研究机构**：1School of AI, Shanghai Jiao Tong University, 2EvoMind Tech, 3IAAR-Shanghai, 4University of Cambridge

- **论文链接**：https://arxiv.org/abs/2507.00416

- **关键词**：

- **Code & Dataset & Weight：** https://mint-sjtu.github.io/Evo-0.io/

- **BibTeX**：

- ```
  @article{lin2025evo,
    title={Evo-0: Vision-Language-Action Model with Implicit Spatial Understanding},
    author={Lin, Tao and Li, Gen and Zhong, Yilei and Zou, Yanwen and Zhao, Bo},
    journal={arXiv preprint arXiv:2507.00416},
    year={2025}
  }
  ```

## Problem Definition

### **研究问题**

现有的 VLA 模型（如 OpenVLA）在处理需要高精度空间对齐的任务（如“销钉入孔”）时经常失败，根本原因在于其骨干网络（VLM）是在 2D 互联网图片上预训练的，缺乏对物理世界深度和几何结构的理解。

### **形式化定义**

输入多视角 RGB 图像 $\{I_t^i\}$、指令 $L$ 和机器人状态 $S_t$；输出连续或离散动作 $A_t$。目标是最大化 $p(A_t | I_t^i, L, S_t)$。

### **价值与意义**

为机器人提供“空间感”，使其能处理复杂、精密的操作任务，同时避免增加昂贵的 3D 传感器，降低部署成本。

## Challenges

### **核心挑战**

**数据稀缺**：高质量的机器人 3D 标注数据（点云/深度）远少于 2D 互联网数据。

**传感器噪声**：深度相机在面对透明物体或反光表面时表现极差。

### **本文针对性解决的挑战**

如何在**仅使用 RGB 输入**的前提下，让模型“脑补”出 3D 几何特征。这种选择非常合理，因为它兼容了现有的所有单目/多目相机硬件。

## Angle & Motivation

### **切入角度**

**隐式 3D 先验注入**。不直接输入 3D 数据，而是借用一个已经学过“如何从 2D 重建 3D”的基础模型（VGGT）作为特征提取器。

### **合理性与重要性**

VGGT 在大规模 2D-3D 配对数据上练过，它对物体轨迹和遮挡关系有天生的直觉。

### **创新性**

打破了“3D 任务必须用 3D 传感器”或“语义和几何必须在同一个 Encoder 里学”的固有范式，采用了**并行双支路编码**。

## Methodology

### **实现细节**

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125040241154.png" alt="image-20260125040241154" style="zoom:67%;" />

**双编码器架构**：保持 $\pi_0$ (PaliGemma) 提取语义，增加一个冻结的 **VGGT** 提取 3D $Tokens$。

**Cross-attention 融合器**：以 2D 视觉 $Tokens$ 为 $Query$，去 $VGGT$ 的 3D $Tokens$ 中寻找空间对应信息。

**参数高效微调**：只训练融合模块和 VLM 的 LoRA 层，保护了原有的语义知识。

### **性能提升的本质**

通过**交叉注意力**将语义锚定（这是什么）与几何定位（在哪里）在特征层面进行了强行对齐。

## Experiments

### **实验设置与指标**

RLBench 仿真环境（5个任务）+ 现实世界（5个精密任务，含透明物体）。

### 对比实验

对比了 OpenVLA-OFT 和 $\pi_0$。在所有维度（成功率、泛化性）上均有提升。

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125040256117.png" alt="image-20260125040256117" style="zoom:67%;" />





真实世界任务：

![image-20260125040349819](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125040349819.png)

鲁棒性测试：

![image-20260125040416949](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125040416949.png)

### 消融实验

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125040321521.png" alt="image-20260125040321521" style="zoom:67%;" />

验证了训练步数对性能的影响，证明了 Evo-0 的**样本效率**极高（更短的训练时间达到更好的效果）。

## Summary & Evaluation

### **总体评价**

它没有去卷更大的算力或更多的数据，而是通过**巧妙的模块设计**解决了 VLA 的短板。
