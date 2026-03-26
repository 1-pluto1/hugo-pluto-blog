## TLDR



本文证明了 VLM 的通用评测分数与其在机器人控制任务中的表现**并不正相关**。视觉编码器是 VLA 性能的核心瓶颈，且现有的“具身 VQA”微调对实际动作控制提升微乎其微。

**VLM4VLA** 通过构建极简的 1% 参数适配插件，设计了标准化的 **VLM4VLA** 流水线，剥离了策略头架构的干扰，纯粹评估 VLM 骨干网络对控制性能的贡献。一共评估了 24 种 VLM 变体，系统性地揭示了通用 VLM 的能力与具身控制性能之间的**非线性关系**，并指明视觉模块是决定 VLA 成败的核心瓶颈。

## Metadata

- **发表期刊/会议**：arXiv
- **论文作者**：Jianke Zhang1, Xiaoyu Chen1, Qiuyue Wang2, Mingsheng Li2, Yanjiang Guo1, Yucheng Hu1 Jiajun Zhang2, Shuai Bai2, Junyang Lin2, Jianyu Chen1
- **研究机构**：Institute for Interdisciplinary Information Sciences, Tsinghua University、 Qwen Team, Alibaba Inc.
- **论文链接**：https://arxiv.org/abs/2601.03309
- **关键词**：VLA, VLM, Robotic Manipulation、GAP between VLA and VLM
- **Code & Dataset & Weight：**https://cladernyjorn.github.io/VLM4VLA.github.io/
- **BibTeX**：
- ```
  @misc{zhang2026vlm4vlarevisitingvisionlanguagemodelsvisionlanguageaction,
        title={VLM4VLA: Revisiting Vision-Language-Models in Vision-Language-Action Models}, 
        author={Jianke Zhang and Xiaoyu Chen and Qiuyue Wang and Mingsheng Li and Yanjiang Guo and Yucheng Hu and Jiajun Zhang and Shuai Bai and Junyang Lin and Jianyu Chen},
        year={2026},
        eprint={2601.03309},
        archivePrefix={arXiv},
        primaryClass={cs.CV},
        url={https://arxiv.org/abs/2601.03309}, 
  }
  ```

  

## Problem Definition

### **研究问题**

现有的 VLA 模型普遍将预训练 VLM 作为核心骨干，但业界缺乏系统性的研究来回答：**VLM 的选择及其通用能力如何量化地转化为下游机器人的操作性能？**

### **形式化定义**

输入为单视角图像 $I$ 与自然语言指令 $L$，输出为连续的动作块 $A \in \mathbb{R}^{T \times D}$。研究重点在于函数 $f_{VLA}(I, L)$ 中 $f_{VLM}$ 组件的贡献度。

$$ \text{action} = \text{MLP}\left( \text{VLM}\left( \left[ \langle \text{img} \rangle, \dots, \langle \text{img} \rangle, \langle \text{text} \rangle, \dots, \langle \text{text} \rangle, \langle \text{ActionQuery} \rangle \right] \right) \right) $$

### **价值与意义**

通过厘清 VLM 与 VLA 的解耦关系，指导后续研究避开“盲目扩大模型规模”或“无效辅助微调”的误区。

## Challenges

### **本文针对性解决的挑战**

解决了**评估不公平**的问题。通过设计一个统一的 1% 参数插件的评估架构，排除了因策略头设计复杂程度不同而导致的性能差异。

## Angle & Motivation

### **切入角度**

**极简适配**。作者不使用复杂的扩散模型或流匹配为评估造成不必要的干扰，而是采用简单的 MLP 结构，尽可能消除策略头的影响，以观察 VLM 原始权重的表征潜力。

### **创新性**

长期以来，学术界倾向于认为 VLM 在通用语义理解上的提升会自然转化为具身控制能力的增强。本文通过严谨的分析，打破了这一固有范式，证明了通用语义能力与低层动作控制之间存在显著的解耦现象。目前部分研究热衷于通过“具身问答”或“几何感知任务”对 VLM 进行后训练。本文通过多组对比实验证明，此类任务对下游控制的提升微乎其微，指出了感知语义与动作执行之间的特征空间分叉。本文提出的 VLM4VLA 框架以“极简主义”为核心，通过剥离复杂的策略头（Policy Head），实现了对 VLM 骨干网络（Backbone）性能的**直接溯源**。

## Methodology

![image-20260123190509305](/Users/yangchao/Library/Application Support/typora-user-images/image-20260123190509305.png)





![image-20260123181255141](/Users/yangchao/Library/Application Support/typora-user-images/image-20260123181255141.png)

### **实现细节**

**架构设计**：引入 $\langle ActionQuery \rangle$ Token，提取 VLM 最后的 Hidden State，接入简易 MLP。

**训练协议**：全参数微调，包含 LLM、Vision Encoder 及 Word Embeddings。

**损失函数**：采用 Huber Loss 处理连续动作 $a_{pos}$，BCE Loss 处理夹爪状态 $a_{end}$。

$$ \text{action} = \text{MLP}\left( \text{VLM}\left( \left[ \langle \text{img} \rangle, \dots, \langle \text{img} \rangle, \langle \text{text} \rangle, \dots, \langle \text{text} \rangle, \langle \text{ActionQuery} \rangle \right] \right) \right) $$

$$ L = \frac{1}{|B|} \sum_{B} \left( \frac{\| a_{pos} - \hat{a}_{pos} \|^2}{2} + \text{BCE}(a_{end}, \hat{a}_{end}) \right) \quad (1) $$





### **逻辑闭环**

方法通过严格的“控制变量”，确保所有性能增益均来源于 VLM 骨干权重的调整，直接回应了“VLM 能力转化”的研究命题。





## Experiments

#### 通用 VLM 性能基准测试 (Performance Analysis)

作者评估了 9 种开源 VLM（1B-30B 参数规模）在统一 VLM4VLA 框架下的表现。具体可见table1、table2

- **核心发现 1：模型规模不等于控制能力。** 在 **Simpler-Bridge** 和 **Libero-Long** 任务中，参数量最小的模型 **Kosmos-2** 取得了最高的成功率，显著优于规模更大的 Qwen 系列。这表明在底层控制任务中，基于坐标定位（Grounding）的预训练比通用语义推理更有效。
- **核心发现 2：Qwen 系列在语义泛化任务中占优。** 在 **Calvin ABC-D** 任务（对指令理解和场景泛化要求高）中，Qwen2.5/3VL 系列表现出极强的竞争力，其平均完成任务数接近目前的 SOTA 专家模型。



<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260123181710626.png" alt="image-20260123181710626" style="zoom: 67%;" />

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260123182046732.png" alt="image-20260123182046732" style="zoom:67%;" />



#### VLM 通用能力与 VLA 性能的相关性分析

作者将 VLM 在 18 个通用 VQA 榜单（如 MMMU, MathVista, GPQA 等）的平均分与 VLA 任务成功率进行线性回归分析。

- **Calvin 基准（高度正相关）**：拟合曲线显示出强线性关系，表明能处理复杂 VQA 任务的模型通常也能较好地完成 Calvin 中的语义指令任务。
- **Simpler & Libero 基准（无显著相关性）**：线性回归的 $R^2$ 极低，曲线近乎水平。
- **结论**：现有的 VLM 通用评测标准无法预测其在精密物理操纵任务中的潜力。



<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260123182105467.png" alt="image-20260123182105467" style="zoom:67%;" />



#### 辅助具身任务的影响 (Impact of Auxiliary Tasks)

为了验证“教模型说话是否能让它更会干活”，作者测试了 7 种辅助微调任务（SFT）的效果：

- **任务类型**：
  - **几何感知**：Robopoint（2D 坐标预测）、Vica（尺寸/距离估算）。
  - **动作逻辑**：Robo2vlm（动作轨迹判断）、Robobrain2。
  - **视觉生成**：Omni-Generation（深度图、语义分割图生成）。
- **实验结果**： 与未经辅助微调的原始模型（Baseline）相比，**所有经过辅助任务微调的模型性能均出现退化**。
- **结论**：单纯在语义层面引入具身问答任务，无法弥合底层动作控制所需的精细表征差距。



<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260123182126478.png" alt="image-20260123182126478" style="zoom:67%;" />



<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260123182941673.png" alt="image-20260123182941673" style="zoom:80%;" />



#### 模态重要性与视觉鸿沟消融 (Modality-level Ablation)

##### 视觉 vs 语言模块

作者对比了冻结（Freeze）与微调（Unfreeze）不同模块的效果：如table3

- **视觉编码器（Vision Encoder）**：冻结该模块会导致所有模型性能断崖式下跌。即便 7B 模型的 LLM 部分保持训练，其表现依然不如视觉层全微调的 3B 模型。
- **语言模块与词嵌入**：冻结词嵌入对 VLA 性能几乎没有负面影响。
- **结论**：**视觉表征的重塑是 VLA 训练的核心。**

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260123182144317.png" alt="image-20260123182144317" style="zoom:67%;" />



##### 视觉鸿沟的本质分析

为了探究视觉退化的原因，作者进行了两组进阶实验：

1. **分辨率实验**：将分辨率从 224 提升至 768，结果显示性能提升极微小。证明瓶颈不在于像素清晰度，而在于特征的对齐方式。table 7
2. **Real-to-Sim 验证**：在全真实的 BridgeV2 图像上进行控制任务训练。实验发现即使没有模拟器的视觉噪声，冻结视觉层依然无法实现有效控制。解冻后，通过微调向视觉编码器里注入控制特征，成功率显著提高。 table 4
3. **结论**：VLM 学习的是“理解型特征”，而 VLA 需要“控制型特征”，两者在表征空间的轨迹会发生分叉。





<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260123182247151.png" alt="image-20260123182247151" style="zoom:67%;" />



<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260123182200691.png" alt="image-20260123182200691" style="zoom:67%;" />

##### 视觉表征的“分歧”实验 

除了 Figure 4，论文中另一个极具理论价值的图表是 **Figure 5**，它从表征学习的角度解释了为什么 Figure 4 会出现负面结果。

- **图表含义**：Figure 5 描绘了 VLM 预训练路径与 VLA 动作学习路径在特征空间中的轨迹。
- **轨迹分叉论**：
  - **初期一致**：在训练初期，两者的空间方向大致相同（都需要理解世界、识别物体）。
  - **后期分叉**：随着任务深入，VLM 向“语义推理”靠拢，而 VLA 向“精细控制”靠拢。这解释了为什么只做语义层面的 VQA 微调（Figure 4 的内容）无法弥合最终的控制鸿沟。



![image-20260123182219909](/Users/yangchao/Library/Application Support/typora-user-images/image-20260123182219909.png)











#### 训练下限：从零开始训练 (From Scratch)

为了证明 VLM 预训练的必要性，作者随机初始化了相同架构的模型并直接在机器人数据上训练：

- **结果**：所有模型在所有基准测试中均表现出**灾难性的性能崩溃**（Success Rate 趋近于 0）。
- **结论**：VLM 预训练虽不足以支撑控制，但它提供了机器人理解世界、泛化任务所必需的**底层语义先验**。



<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260123182304969.png" alt="image-20260123182304969" style="zoom:67%;" />





## Summary & Evaluation

### **总体评价**

作者通过多次实验给出一个“负面”结论（通用能力不等于控制能力），很有批判性价值。通过剥离 99% 的参数干扰，只用 1% 的极简插件（Minimalist Adaptation）做横向对比，这种**控制变量**的思维值得学习。

有时间再看一遍。

### **值得 Follow 的点**

从特征学习的角度来看，VLM训练中表示学习的整体空间方向大致与VLA训练一致。然而，在训练的某个阶段，两条路径会分岔到不同的区域，这导致了目前观察到的VLM和VLA之间的潜在差距。这也解释了为什么VLM预训练对VLA至关重要，但两者之间仍存在明显差距。



### **局限性与机会**

没有真机测试。