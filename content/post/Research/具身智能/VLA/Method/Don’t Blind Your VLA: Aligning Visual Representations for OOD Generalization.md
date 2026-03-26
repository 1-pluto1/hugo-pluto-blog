## TLDR

**OpenVLA** 模型通过 **视觉表示对齐（Visual Representation Alignment）** 方法解决了机器人微调过程中因 **表示崩溃（Representation Collapse）** 导致的视觉常识丢失与泛化性下降问题 。引入轻量级对齐机制，强制 VLA 的中间层特征与冻结的视觉教师模型（Vision Teacher）保持一致，锚定通用视觉语义。在 **Simpler** 分布外（OOD）基准测试上，比标准 SFT 成功率提升了高达 **10%** 的相对增益 。

## Metadata

- **发表期刊/会议**：arXiv

- **论文作者**：[Nikita Kachaev](https://tttonyalpha.github.io/)1, [Mikhail Kolosov](https://github.com/idkolosofff)2, [Daniil Zelezetsky](https://scholar.google.com/citations?user=6OY6gWcAAAAJ)2, [Alexey K. Kovalev](https://scholar.google.com/citations?user=N1zWb74AAAAJ)12, [Aleksandr I. Panov](https://grafft.github.io/)12

- **研究机构**：Cognitive AI Lab  2IAI MIPT

- **论文链接**：https://arxiv.org/abs/2510.25616

- **关键词**：

- **Code & Dataset & Weight：**  https://blind-vla-paper.github.io/#BibTeX

- **BibTeX**：

-           @misc{kachaev2025dontblindvlaaligning,
                  title={Don't Blind Your VLA: Aligning Visual Representations for OOD Generalization}, 
                  author={Nikita Kachaev and Mikhail Kolosov and Daniil Zelezetsky and Alexey K. Kovalev and Aleksandr I. Panov},
                  year={2025},
                  eprint={2510.25616},
                  archivePrefix={arXiv},
                  primaryClass={cs.LG},
                  url={https://arxiv.org/abs/2510.25616}, 
            }


  ​        

## Problem Definition

### **研究问题**

VLA 模型在针对特定机器人任务进行监督微调（SFT）时，会丢失预训练时期继承的通用视觉-语言理解能力 。

### **形式化定义**

**输入**：RGB 图像 $I$ 与语言指令 $l$ 。

**输出**：预测动作令牌 $y_t$ 。

**目标**：在最小化动作损失 $\mathcal{L}_{VLA}$ 的同时，保持内部表示 $h_{1:k}^{i^*}$ 的语义完整性 。





![image-20260125165231934](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125165231934.png)



![image-20260125165345791](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125165345791.png)



![image-20260125165240438](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125165240438.png)



![image-20260125165251733](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125165251733.png)



## Challenges

### **核心挑战**

当前 VLA 模型在处理视觉和语言复杂的任务时难以保持泛化性，尤其是微调数据多样性有限时极易发生过拟合 。

重点解决了 **表示退化（Representation Degradation）** 。作者认为，标准微调会将多样化的内部特征压缩到狭窄空间（即表示崩溃），通过正则化手段纠正这一趋势是非常合理的

## Angle & Motivation

### **切入角度**

基于 **柏拉图式表示假设（Platonic Representation Hypothesis）**，即强大的视觉和语言模型最终会收敛到共享的现实模型空间 。

## Methodology

### **实现细节**



<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125165651970.png" alt="image-20260125165651970" style="zoom:67%;" />





<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125165638556.png" alt="image-20260125165638556" style="zoom:67%;" />

1. 提取冻结视觉老师（如 C-RADIOv3）的 patch 特征 $z$ 。
2. 选取 VLA 骨干网中间层作为对齐目标层 。
3. 通过一个冻结的 MLP 投影器 $P_{\phi}$ 将 VLA 特征映射到老师的空间 。
4. 最小化补丁间余弦相似度损失 $\mathcal{L}_{align}$ 。

### **性能提升的本质**

通过强制对齐，模型在学习机器人动作（$\mathcal{L}_{VLA}$）时，被强制维持了原本清晰的、面向物体的 **注意力模式** 。

## Experiments

### **实验设置与指标**

基于 **Simpler** 的机器人仿真环境，包含视觉、语义和执行三个泛化轴 。

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125165557003.png" alt="image-20260125165557003" style="zoom:67%;" />

### 对比实验

基于 **Simpler** 仿真环境 。包含三个泛化轴线：

- **语义 (Semantic)**：包含 Carrot, Instruct, MultiCarrot, MultiPlate, Plate 等变体 。
- **视觉 (Vision)**：包含 VisionImg, Tex03, Tex05, Whole03, Whole05 等扰动变体 。
- **执行 (Execution)**：包含 Position, EEPose, PosChange To 等动作变体 。

**指标**：**成功率 (Success Rate)**，以“平均值 ± 标准差 (mean ± SD)”形式报告 。

![image-20260125165405741](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125165405741.png)



本文提出的 **VL-Think 诊断套件** 。涵盖 8 个知识领域：箭头 (Arrow)、颜色 (Color)、洗涤标志 (Laundry)、数字奇偶 (Parity)、公共信息 (PublicInfo)、形状 (Shape)、交通标志 (Traffic) 和天气 (Weather) 。

**指标**：**成功率 (Success Rate)** 。VLA 评估物体是否放置正确，VLM 评估对目标位置的语言描述是否准确 。

![image-20260125165415345](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125165415345.png)

### 

线性探针：

**数据集/基准**：**ImageNet-100**（ImageNet 的子集） 。

**指标**：**Top-1 准确率 (%)** 

![image-20260125165429053](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125165429053.png)



### 消融实验



直接可视化不同模型的不同层的注意力图，align后模型的注意力最集中：

![image-20260125165544659](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125165544659.png)



对比vla模型和其backbone的vlm模型，可以看到vla模型的特征的可分性较差：

![image-20260125165531159](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125165531159.png)







**数据集**：Simpler 仿真环境 。

**指标**：在语义、视觉、执行三个维度的**平均成功率**以及 **p 值 (p-value)**（使用配对 Wilcoxon 符号秩检验进行统计显著性分析） 。



![image-20260125165436896](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125165436896.png)

**对齐方法与层级对比**

- **数据集**：Simpler 仿真环境 。
- **指标**：三个维度的**平均成功率**和 **p 值** 。



![image-20260125165448421](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125165448421.png)



**投影器 (Projector) 类型对比**

- **数据集**：Simpler 仿真环境 。
- **指标**：三个维度的**平均成功率**和 **p 值** 。

![image-20260125165458453](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125165458453.png)

对齐层数对比：

- **数据集**：Simpler 仿真环境 。
- **指标**：三个维度的**平均成功率**和 **p 值** 。



![image-20260125165508243](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125165508243.png)

**损失函数与对齐系数对比**

- **数据集**：Simpler 仿真环境 。
- **指标**：三个维度的**平均成功率**和 **p 值** 。



![image-20260125165516193](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125165516193.png)

## Summary & Evaluation

### **总体评价**

通过严谨的 interpretability 探针（t-SNE、注意力图、线性探测）透彻分析了退化原因，方法简洁而优雅 。？

### **值得 Follow 的点**

- **VL-Think 任务集**：用于评估机器人“脑力”流失的绝佳工具 。
- **对齐层级选择**：聚焦中间层（融合层）而非顶层或底层 。