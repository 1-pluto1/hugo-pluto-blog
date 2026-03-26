## 0. TLDR

这一篇是专门针对 VLA 的**Action Tokenization** **的工作**，核心是网络输出action token形式的优化，可以用于优化PI0加速训练。与pi0（diffusion VLA）不同的是，这里是自回归（AR）的VLA，这类工作先前还有RT-1,RT-2,OpenVLA。论文提出了 **FAST**，一种基于**离散余弦变换 (DCT)** 和 **字节对编码 (BPE)** 的动作分词方案，通过将动作信号从时域转为频域并压缩，解决了高频机器人数据在自回归训练中因信息冗余导致的“训练崩坏”问题。在训练速度上比基于扩散模型的 $\pi_{0}$ 快 **5 倍**，且在复杂的“叠衣服”等灵巧任务上表现持平，并实现了首个在 DROID 数据集上的**零样本** 泛化策略。在 VLA 研究中，动作的**表征质量**与模型架构同等重要；频域分析为处理高频时序信号提供了一个极其简单且高效的新视角。

## 1. Metadata

- **发表期刊/会议**：arXiv
- **论文作者**：Physical Intelligence
- **研究机构**：Physical Intelligence
- **论文链接**：https://arxiv.org/abs/2501.09747
- **关键词**：VLA, Action Tokenization, Discrete Cosine Transform (DCT), Autoregressive Transformers
- **Code & Dataset & Weight：** https://github.com/Physical-Intelligence/openpi
- **BibTeX**：

## 2. Problem Definition

### **研究问题**

基于Transformer的 VLA 模型在学习复杂且可泛化的机器人行为方面非常有效。但必须对连续的机器人动作信号进行“Token”（分词）（LLM 的文本生成形式就是预测next token），将**连续动作映射为离散的token**。(RT-2、openvla类似的做法)，**当前主流的action Token化方法，即简单的“逐维度、逐时间步分bin”**，在处理来自高频控制的灵巧操作时表现很差。如何为自回归 VLA 模型设计一种高效的动作 Token 化方案，使其能够处理**高频** 且**灵巧** 的机器人动作 ？

### **形式化定义**

策略 $\pi(a_{1:H}|o)$ 将观测 $o$ 映射到长度为 $H$ 的动作序列（Action Chunk） 。Token 化过程 $\mathcal{T}_a: a_{1:H} \rightarrow [T_1, \dots, T_n]$ 需要将连续动作序列转化为离散 Token 序列，且需保证重构误差最小。

### **价值与意义**

打破了AR VLA 只能做低频简单任务的局限，使其在计算效率和泛化性能上能与 Flow-based 的方法一较高下 。

## 3. Challenges

### **核心挑战**

在高频控制下，相邻时间步的动作**高度相关，导致每个Token 的**边际信息量**趋近于零 。

### **本文针对性解决的挑战**

传统的“逐时间步分 bin”方案在处理高频数据时，模型容易陷入局部最优——它会学会简单的“复制上一个动作”来降低 Loss，而无法提取有意义的控制逻辑。这种选择非常合理，因为本质问题不在模型容量，而在数据表征的信噪比。

## 4. Angle & Motivation

### **切入角度**

**时间序列压缩**（Time-series Compression）。作者借鉴了图像压缩（如 JPEG）的逻辑：平滑变化的信号在频域中可以用极少数系数表达。

### **合理性与重要性**

论文通过一个简单的三次样条曲线插值实验证明：随着采样频率增加，Naive 方案的预测误差呈指数级上升，而基于 DCT 的方案误差始终保持低位稳定 。

### **创新性**

它打破了“动作预测即分类”的固有范式，引入了类似语言模型 BPE 的思想来压缩连续控制信号。

## 5. Methodology

### **实现细节**



![image-20260123103008734](/Users/yangchao/Library/Application Support/typora-user-images/image-20260123103008734.png)



![image-20260123103034227](/Users/yangchao/Library/Application Support/typora-user-images/image-20260123103034227.png)



1. **归一化**：将动作映射到 $[-1, 1]$ 区间 。
2. **离散余弦变换 (DCT)**：将动作序列从时域转换到频域 。
3. **量化与稀疏化**：通过 `scale-and-round` 操作保留显著的频率系数，抹除不重要的噪声 。
4. **分层展平**：优先排列所有维度的低频系数（决定大体轨迹），后排列高频系数（细化动作） 。
5. **BPE 编码**：使用 BPE 算法将稀疏的整数序列进一步压缩为密集的 Token 。

### **逻辑闭环**

该方法通过 DCT 移除了冗余，通过 BPE 提升了每个 Token 的信息密度，解决了第 4 部分提到的“边际信息量低”的问题 。

### **性能提升的本质**

更紧凑的表征让模型在相同的训练步数内看到了更高效的信号，加快了收敛速度 。

## 6. Experiments

### **实验设置与指标**

在 6 个真实机器人任务（如叠 T 恤、收餐具、装杂物袋）和 1 个模拟环境（Libero）中评估。主要指标是**任务成功率**和**任务进度** 。

### 对比实验

**Baseline**：Naive Binning (OpenVLA 风格)、FSQ (基于分级向量量化)。

**SOTA 对比**：与基于扩散模型的 $\pi_{0}$ 相比，FAST 能够以 **1/5 的计算量**达到同等性能 

### 消融实验

证明了 **BPE 步骤**至关重要。如果没有 BPE，序列中充满了大量的 `0`，会稀释学习信号并显著拖慢推理速度 。

## 7. Summary & Evaluation

### **总体评价**

你认为这篇论文的质量如何？

### **值得 Follow 的点**

**FAST+ 通用分词器**：官方发布的预训练分词器可以直接用于其他 VLA 的动作空间。

![image-20260123103117457](/Users/yangchao/Library/Application Support/typora-user-images/image-20260123103117457.png)

### **局限性与机会**

还有哪些坑没填？如果让你来改进，你会从哪里下手？