---
title: Enhancing Reasoning through Process Supervision with Monte Carlo Tree Search
date: 2025-02-20T11:30:03+00:00
tags:
  - LLM
  - NLP
  - ReasoningModel
  - MCTS
categories:
  - AI
  - Research
author: ZhaoYang
showToc: true
TocOpen: true
draft: false
disableShare: false
hideSummary: false
searchHidden: false
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---


单位： AI Lab

代码：没有开源

基座模型：Llama-3.1-8B-Instruct deepseek-math-7b-instruct

原文地址：https://arxiv.org/abs/2501.01478

### 总览

![](https://cdn.jsdelivr.net/gh/1-pluto1/blog_imgs/20250601134647302.png)

所提出的方法如图1所示。我们采用自训练的方式训练大语言模型（LLMs）（Zhang et al. 2024b; Zelikman et al. 2022），即利用模型自身生成的数据进行训练。我们利用蒙特卡洛树搜索（MCTS）来采样和搜索逐步推理路径，并从构建的搜索树中收集训练数据。生成的数据随后用于对大语言模型进行监督微调（SFT）。这一生成与微调的过程会迭代重复，直到收敛。

  

### **数据生成**

我们生成一个训练数据集 $$D = \{(x_i, p_j^i, s_j^i, c_j^i)\}_{i=1}^$$，其中$$x_$$ 表示问题集 $$$$ 中的第 $$$$ 个问题，$$p_j^$$ 表示对问题 $$x_$$ 的第 $$$$ 个部分解，$$s_j^$$ 表示部分解 $$p_j^$$ 的下一步推理，$$c_j^$$ 表示对 $$s_j^$$ 分配的分数。

对于 PP 中的每个问题，我们将问题求解过程中的每个独立步骤视为一棵树上的节点，这些步骤由两个换行字符分隔。我们在由大型语言模型（LLM）生成的推理路径上，对每一步执行蒙特卡洛树搜索（MCTS），以探索下一步的最优动作。具体而言，对于问题 xix_i 的每个部分解 $$p_j^$$，我们遵循以下迭代过程：

1. **选择（Selection）**。从根节点（即 pjip_j^i）开始，选择 UCB（Upper Confidence Bound，Kocsis and Szepesvári 2006）值最高的子节点，一直向下搜索，直到当前节点未被完全扩展或该节点代表解答的最终一步（例如 “The final answer is 3.”）。
    
2. **扩展（Expansion）**。如果当前节点不是最终解且尚未完全扩展，则使用一个非零温度（temperature）的 LLM 采样下一步，作为一个新的子节点。
    
3. **模拟（Simulation）**。从新扩展的节点开始，使用 LLM 继续采样该部分解的后续推理，直到模型给出最终答案。
    
4. **回传（Backpropagation）**。模拟结束后，将生成的最终答案与标准答案对比。如果正确则奖励为 1.0，否则为 0.0。然后将该奖励沿着在树中访问过的节点回传，并更新它们的访问次数和累积奖励，以指导后续搜索。
    

在重复上述过程多次后，我们构建了一棵树，其中根节点的子节点表示 $$p_j^$$ 的下一步推理。对于第 $$$$ 个子节点 $$v_{j,k}^$$，其得分计算如下：

  

![image.png](https://cdn.jsdelivr.net/gh/1-pluto1/blog_imgs/20250601134724006.png)


其中 $$Q(\cdot$$ 表示节点的累积奖励，$$N(\cdot)$$ 表示节点的访问次数，$$\alpha$$ 是用于控制得分尺度的手动设置的常数，括号中的表达式体现了节点 $$v_{j,k}^i$$ 的“相对正确度”（relative correctness）。最终，我们将$${(x_i,p_j^i,s_{j,k}^i,r_{j,k}^i)}$$ 加入训练集，其中 $$s_{j,k}^$$ 是与节点 $$v_{j,k}^i$$ 对应的步骤（若该步骤得分为 0 则跳过）。接着，我们将 UCB 值最高的下一步附加到 $$p_j^$$ 中，得到一个新的部分解，并再次执行上述过程，直到下一步为最终步骤或达到最大解答步骤限制。

---

### **迭代训练**

在生成完训练数据后，我们使用上一轮训练生成的 LLM 对其进行训练，并重复多轮。第一轮从预训练的 LLM 开始。在第 $$$$ 轮中，我们从 $$$$ 中采样一定数量的问题用于数据生成，然后使用以下损失函数来训练 LLM：

![](https://cdn.jsdelivr.net/gh/1-pluto1/blog_imgs/20250601134750600.png)


其中 $$\pi_{\theta_i}$$ 表示第 ii 轮训练得到的 LLM。第一个项是加权的负对数似然（weighted negative log-likelihood）。受到来自人类反馈的强化学习（RLHF, Ouyang et al. 2022）的启发，我们在第二项中引入 KL 惩罚，以减轻分布漂移（distribution shift），这是离线强化学习中的一大挑战。实际上，我们的训练方法可以视为一种强化学习形式。

---

**表 1**：在 MATH 和 GSM8K 数据集上的性能。结果以均值 ± 标准误差的形式报告。

|   |   |   |
|---|---|---|
|Method|Llama-3.1-8B-Instruct|deepseek-math-7b-instruct|
||MATH|GSM8K|
|Zero-shot-CoT|47.07 ± 0.15|80.77 ± 0.25|
|RFT|47.96 ± 0.21|83.33 ± 0.25|
|Step-level DPO - Iteration 1|47.12 ± 0.31|82.43 ± 0.27|
|Step-level DPO - Iteration 2|47.31 ± 0.19|82.55 ± 0.17|
|Step-level DPO - Iteration 3|48.29 ± 0.18|/|
|Step-level DPO - Iteration 4|48.48 ± 0.27|/|
|Ours - Iteration 1|50.04 ± 0.07|85.35 ± 0.24|
|Ours - Iteration 2|50.84 ± 0.20|85.80 ± 0.22|
|Ours - Iteration 3|51.52 ± 0.18|/|
|Ours - Iteration 4|51.92 ± 0.20|/|

---

**表 2**：跨任务迁移实验的结果。模型在一个数据集上进行训练，然后在另一个数据集上进行测试。

|   |   |   |
|---|---|---|
|Method|Llama-3.1-8B-Instruct|deepseek-math-7b-instruct|
||GSM8K to MATH|MATH to GSM8K|
|Ours - Iteration 1|48.50 ± 0.31|85.21 ± 0.15|
|Ours - Iteration 2|48.74 ± 0.26|85.72 ± 0.14|
|Ours - Iteration 3|/|85.53 ± 0.16|
|Ours - Iteration 4|/|85.65 ± 0.21|

论文翻译：https://dppemvhuzp.feishu.cn/docx/W84qdm5AAoienmxFl17cU7OOnkg?from=from_copylink