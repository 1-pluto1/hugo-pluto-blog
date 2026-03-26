## TLDR

InternVLA-M1 认为机器人要先学会在图像里空间定位，才能在物理世界执行动作。它通过 300 万条空间数据的训练，让大模型拥有了几何直觉。**InternVLA-M1** 采用**空间引导的双系统框架**，通过将任务解耦为“具身无关的空间定位”预训练与“空间引导的动作”后训练，解决了通用机器人策略在复杂环境下泛化性差的问题。提出了**空间引导训练范式（Spatially Guided Training）**，利用大规模 Web 数据构建空间先验，并通过**梯度衰减机制**平衡感知与动作的学习。在 **SimplerEnv Google Robot** 上成功率提升 **14.6%**，在 **WidowX** 上提升 **17%**，在现实世界未见过物体任务中实现 **+20.6%** 的飞跃。

## Metadata

- **发表期刊/会议**：arXiv

- **论文作者**：Intern Robotics, Shanghai AI Laboratory

- **研究机构**：Intern Robotics, Shanghai AI Laboratory

- **论文链接**：https://arxiv.org/abs/2510.13778

- **关键词**：Vision-Language-Action (VLA), Spatial Grounding, Embodied AI, Generalist Robot Policy

- **Code & Dataset & Weight：**https://internrobotics.github.io/internvla-m1.github.io/

- **BibTeX**：

- ```
  @article{internvlam1,
    title   = {InternVLA-M1: A Spatially Guided Vision-Language-Action Framework for Generalist Robot Policy},
    author  = {InternVLA-M1 Contributors},
    journal = {arXiv preprint arXiv:2510.13778},
    year    = {2025}
  }
  ```

  

## Problem Definition

### **研究问题**

如何弥补抽象的语言指令与具体的 3D 物理动作之间的鸿沟？当前的 VLA 模型往往在学会抓取的同时，丧失了对物体相对位置、几何关系的深刻理解。

### **形式化定义**

**输入**：RGB 图像 $V$ + 任务指令 $L$ + 空间提示 $P$。

**输出**：低层电机控制序列（6DoF 增量动作） $A$。

### **价值与意义**

实现了可扩展的通用智能。通过这种框架，机器人不再局限于特定的实验室场景，而是能在杂乱、未见过的真实家庭/工业环境中稳健执行长程任务。

## Challenges

### **核心挑战**

**数据极度稀缺**：带动作标签的机器人数据远少于 Web 端的视觉文本数据。

**泛化性瓶颈**：传统的端到端 VLA 容易过拟合于细微的电机行为，无法理解“左边”、“后面”这种高层空间概念。

### **本文针对性解决的挑战**

重点攻克了**“如何高效利用非机器人数据来增强机器人的空间感知”**。互联网上有无穷无尽的“带框图片”，这比昂贵的“机器人抓取视频”更容易获取。

## Angle & Motivation

### **切入角度**

**空间定位是指令与动作之间的“桥梁”。** 作者认为机器人应该“先定位，后行动”

### **合理性与重要性**

文中通过 **PSS (Projection-space Similarity)** 指标证明，如果没有空间引导，动作梯度和感知梯度会发生 0.25 的低相似度冲突；引入空间引导后，一致性提升至 0.42。

### **创新性**

它打破了“纯端到端硬练”的暴力范式，引入了类似人类认知的“双系统”逻辑，让学术界重新审视“几何先验”在具身智能中的决定性作用。

## Methodology

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260124092329870.png" alt="image-20260124092329870" style="zoom:50%;" />





<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260124092350138.png" alt="image-20260124092350138" style="zoom:50%;" />





<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260124092505721.png" alt="image-20260124092505721" style="zoom:50%;" />





<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260124092521213.png" alt="image-20260124092521213" style="zoom:50%;" />



#### 1. 数据集构建方法 (The Scalable Synthetic Data Engine)

作者在 **Isaac Sim** 和 **GenManip** 之上构建了一套全自动流水线，解决了数据多样性问题：

- **规划与渲染解耦 (Decoupled Pipeline)**：先在物理引擎中生成不带画面的纯动作轨迹（极速计算），记录关节和物体坐标；随后由渲染器在随机光照、1.6K 种纹理、87 种穹顶光下进行多视角“重演”。
- **特权信号利用 (Privileged Signals)**：利用仿真环境中的上帝视角（物体网格、精确位姿）自动生成 2D 边界框、3D 点和末端执行器轨迹，作为 **VLM 预训练的真值**。
- **资产多样性**：资产库包含 1.4 万个标注物体、211 张桌子，确保了模型对未见过物体的泛化能力。

#### 2. 模型实现细节 (The Dual-System Framework)

- **两阶段流水线**：
  - **Stage 1: 空间定位预训练**。利用 300 万条（2.3M 空间相关）QA 数据（点/框/轨迹）训练 VLM 大脑。
  - **Stage 2: 空间引导动作后训练**。联合训练动作专家（Diffusion Policy）。
- **梯度防火墙**：引入 **0.5 的梯度衰减因子**。在训练动作头时，减弱回传到 VLM 的梯度强度，防止针对特定动作的微调“洗掉”大模型原本的通用语义常识。
- **隐式空间提示 (Latent Spatial Prompting)**：在输入中加入“定位关键物体”等提示词，不要求模型输出文本，而是显式激活 VLM 内部的几何感知神经元，增强特征嵌入的空间指向性。

## Experiments

### **实验设置与指标**

使用了 **SimplerEnv (Google/WidowX)**, **LIBERO**, 以及基于 **Isaac-Sim** 的 200 个大规模任务。指标为**任务成功率 (Success Rate)** 和 **轨迹平均绝对误差 (MAE)**。







**Table 1** 记录了 InternVLA-M1 在 **SimplerEnv** 仿真评测集中的 **Google Robot** 平台上的表现。SimplerEnv 是目前业内公认最难的“视觉泛化”测试之一，它专门考察模型在面对光照变化、相机视角偏移、物体纹理改变时是否还能保持动作的准确性。

在这一表格中，主要对比了以下几个关键维度：

- **Google Robot-VM (Visual Matching)**：视角和光照剧烈变化下的成功率。
- **Google Robot-VA (Visual Aggregation)**：物体纹理和背景颜色多样化组合下的成功率。
- **对比基准 (Baselines)**：包括目前最强的开源模型 $\pi_0$、GR00T、OpenVLA 以及作者自己构建的 Vanilla VLA (基础版)。

![image-20260124102210863](/Users/yangchao/Library/Application Support/typora-user-images/image-20260124102210863.png)



### 

**Table 2** 记录了模型在 **WidowX** 机器人平台上的表现。WidowX 相比 Google Robot 来说，其动力学特性和常见的作业场景都有所不同。在这里，实验主要考察的是 **WidowX-VM (Visual Matching)**，即在视觉干扰（光照、背景、角度）下的任务成功率。

表格中展示了 InternVLA-M1 与一众顶尖 VLA 模型的同台竞技结果：

- **核心胜出**：InternVLA-M1 达到了 **+9.8%** 的平均成功率提升（相对于之前的 SOTA）。
- **惊人的涨幅**：最引人注目的是它与 **Vanilla VLA**（即没有经过空间引导预训练的版本）的对比，性能直接拔高了 **+17.0%**。
- **对手情况**：即使是像 $\pi_0$ 这种在 2025 年名声大噪的模型，在 WidowX 的视觉偏移场景下也表现出了疲态，而 InternVLA-M1 依然保持了极高的成功率。

![image-20260124102227245](/Users/yangchao/Library/Application Support/typora-user-images/image-20260124102227245.png)



##### **(a) 左图：空间定位表现 (Spatial Grounding - IoU@0.5)**

- **指标：** 在 **RefCOCOg** 数据集上的交并比（IoU）。
- **现象：** 蓝色曲线（带空间提示的协同训练）不仅起始点高，而且在整个训练过程中保持了极高的定位精度。
- **教授解读：** 这证明了 InternVLA-M1 的“大脑”非常清醒。即便在练动作的过程中，它也没有产生“灾难性遗忘”，其空间感知能力始终在线。

##### **(b) 中图：操作表现 (Manipulation - WidowX SR)**

- **指标：** 在 **SimplerEnv-WidowX** 上的任务成功率（SR）。
- **现象：** 蓝色曲线的斜率明显更陡，且最终达到的成功率上限（约 0.6-0.7 之间）高于灰色曲线（不带空间提示）。
- **教授解读：** 结合 (a) 图看，这说明**更强的空间感知能力直接转化为了更高的操作成功率**。模型不仅学得更快（收敛快），而且学得更好（上限高）。

##### **(c) 右图：梯度相似度 (Gradient Similarity - PSS)**

- **指标：** 投影空间相似度 (Projection-space Similarity)。
- **现象：** 使用空间引导后，PSS 从 0.25 显著提升至 0.42。



![image-20260124102247034](/Users/yangchao/Library/Application Support/typora-user-images/image-20260124102247034.png)



三个训练策略：**Vanilla VLA**、**Vanilla co-train**、**InternVLA-M1**。在多模态理解、空间定位、机器人操纵三个领域的实验结果：

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260124102315416.png" alt="image-20260124102315416" style="zoom:67%;" />



##### LIBERO (Franka) 基准测试

##### 1. 任务维度拆解

- **Spatial (空间)**：物体还是那些，但摆放位置变了（考查空间适应性）。
- **Objects (物体)**：摆放位置固定，但换了没见过的物体（考查视觉泛化）。
- **Goal (目标)**：物体和位置都差不多，但任务指令变了（考查语义理解）。
- **Long (长程)**：最难的一项，需要跨越多个步骤、多个物体和复杂操作。



![image-20260124102331613](/Users/yangchao/Library/Application Support/typora-user-images/image-20260124102331613.png)





**Figure 7** 展示了在 Isaac-Sim 仿真环境中，面对 200 个任务和 3000 多个物体时，模型在四种情况下的表现。

![image-20260124102343803](/Users/yangchao/Library/Application Support/typora-user-images/image-20260124102343803.png)



**Figure 10** 展示了在真实环境中模型的表现。



![image-20260124102357101](/Users/yangchao/Library/Application Support/typora-user-images/image-20260124102357101.png)





真实世界长程任务的模型表现：

![image-20260124102411690](/Users/yangchao/Library/Application Support/typora-user-images/image-20260124102411690.png)





长程任务中的任务规划能力：

![image-20260124102420454](/Users/yangchao/Library/Application Support/typora-user-images/image-20260124102420454.png)



## Summary & Evaluation

### **值得 Follow 的点**

1. **解耦渲染与规划**的数据生成思路。
2. **梯度衰减**在多任务联合训练中的保护作用。
3. **零动作填充 (Zero-action Padding)** 解决子任务切换边界感。