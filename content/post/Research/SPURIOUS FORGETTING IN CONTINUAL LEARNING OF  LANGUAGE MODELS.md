---
title: SPURIOUS FORGETTING IN CONTINUAL LEARNING OF  LANGUAGE MODELS
date: 2025-02-22T11:30:03+00:00
tags:
  - LLM
  - NLP
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
ICLR 2025

单位： 华南理工大学

代码：https://github.com/zzz47zzz/spurious-forgetting

基座模型：

原文地址：https://arxiv.org/abs/2501.13453

  

### overview

最近在大语言模型（LLMs）的持续学习研究中揭示了一个令人困惑的现象：尽管进行了大量的训练，模型的性能却显著下降，这引发了关于任务对齐和潜在知识保留的疑问。本研究首先探讨了“虚假遗忘”的概念，提出这种性能下降往往反映了任务对齐的减弱，而非知识的丢失。通过使用合成数据集进行控制实验，我们研究了模型在新任务初始训练阶段的性能动态，发现早期的优化步骤可能会破坏先前建立的任务对齐。我们的理论分析将这些变化与模型权重的正交更新联系起来，为理解这种行为提供了一个稳健的框架。最终，我们引入了一种冻结策略，固定模型的底层，从而在四种持续学习场景中实现了显著的改进。我们的发现强调了任务对齐与知识保留之间的关键区别，为持续学习中更有效的策略铺平了道路。源代码已公开提供1。

  

在总结中，我们的贡献包括：(1) 我们是第一个识别出语言模型持续学习中虚假遗忘现象的；(2) 我们发现虚假遗忘是由任务对齐的丧失而非底层知识的丢失引起的；(3) 我们从理论上分析了虚假遗忘的原因；(4) 我们提出了冻结策略作为缓解虚假遗忘的有效方法。

### A CLOSER LOOK AT SPURIOUS FORGETTING

#### CONTROLLED SETTINGS UNDER SYNTHETIC DATASET

##### 数据集

传记数据集由 200,000 个合成个体组成，每个个体具有六个属性：出生日期、出生城市、就读大学、专业、公司名称和公司城市。该数据集分为两个子集：预训练数据和微调数据。预训练数据包含描述每个个体属性的陈述。例如，Curtis Chase Emley 在 1952 年 5 月 28 日庆祝他的生日。微调数据由用于知识提取的问答对组成，例如 “Curtis Chase Emley 的出生日期是什么？/n 答案：1952 年 5 月 28 日。” 除非另有说明，我们计算数据集的精确匹配准确率。更多细节和示例见附录 B。

##### 持续学习设置

最初，模型在 100,000 个个体上进行预训练，以建立坚实的知识基础。随后，我们在来自相同个体的问答数据上对模型进行微调（记为任务 0）。然后，我们引入一个新任务（记为任务 1），该任务包括模型不熟悉的额外 20,000 个个体。预训练和微调的初始学习率分别设置为 1 × 10^-3 和 5 × 10^-6。训练步骤配置为预训练 80K 步和微调 62.5K 步。这种小学习率与大量优化步骤相结合，确保了模型的全面训练。图 2b 提供了这种训练设置的示意图。我们在更多任务（附录 G.1.1）、不同个体数量（附录 G.1.2）、不同任务类型（附录 G.1.3）以及不同优化器和学习率（附录 G.1.4）上进行了额外实验，以表明虚假遗忘存在于持续学习的一般设置中。

##### 使用合成数据集的理由

现实世界的数据集，例如 TRACE 基准测试中的数据集，可能在以下两种情况下表现出知识重叠：(1) 预训练和微调期间获得的知识；(2) 微调任务之间的知识。相比之下，我们构建的传记数据集通过严格控制预训练和微调过程来规避这些问题，确保任务之间的知识不重叠。这对于隔离导致虚假遗忘的无关因素至关重要。使用合成数据集使我们能够分解任务对齐和底层知识的学习过程。具体而言，如图 2b 所示，在任务 0 中，模型学习任务对齐而不获取新知识。当过渡到任务 1 时，模型必须同时获取与任务 1 相关的新知识，同时为这个新任务建立任务对齐，因为任务 1 中的个体对模型来说是全新的。

#### SPURIOUS FORGETTING FROM PERFORMANCE PERSPECTIVE

我们首先重现了在安全对齐和持续指令调优场景中观察到的虚假遗忘现象。如图 2a 所示，在学习任务 1 之后，我们在任务 0 上的性能出现了急剧下降，在最初的 150 步优化步骤内，从接近 100% 下降到大约 10%。直观上，期望任务 0 的底层知识在仅仅 150 步内消失是不合理的。

基于这一观察，我们尝试恢复任务 0 的性能。恢复实验的步骤如图 2c 所示。具体而言，对于预训练、任务 0 和任务 1 期间的任何检查点，我们在任务 0 的一半数据上对模型进行一个周期的微调，并在剩余的一半数据上进行评估。虽然训练需要任务 0 的一半数据，但需要注意的是，这个训练集与测试数据不重叠。例如，如果模型缺乏测试集中的知识，恢复后的性能将接近零。相反，如果模型保留了这些知识，恢复后的性能应接近 100%。

我们对从预训练到任务 0 再到任务 1 的所有检查点进行了恢复实验。结果绘制为图 2a 中的虚线（恢复后的任务 0 准确率）。我们发现，在任务 1 训练的前 150 步内，恢复性能几乎保持在 100%，到任务 1 结束时略微下降到 96%。相比之下，任务 0 的准确率在 150 步后下降到大约 10%，之后略有回升至 20%。这一结果进一步支持了我们的假设，即虚假遗忘并非由于知识的实际丧失。基于这些观察，我们在附录 E 中提供了虚假遗忘的正式定义。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=OTNjZjJlYzM4NDYwMzQ3YzFlOGI0NjAyYjIxNjE4NjhfZm5odEtYOERyRlQ3ODJWZnYxY0lTVTNVb0JITU8zTWpfVG9rZW46SkVzV2JnRlVHb0VPcWV4TW0zdGNndFpvbjRiXzE3NDg3NTkxMTg6MTc0ODc2MjcxOF9WNA)

#### LOSS LANDSCAPE PERSPECTIVE

为了更好地理解前 150 步期间发生的情况，我们在由初始 150 步和后续步骤的权重更新方向所构成的二维空间中对测试损失进行了可视化。结果汇总于图 3a。最初，我们观察到任务 1 的损失景观急剧下降，同时任务 0 的损失显著增加。这一现象解释了任务 0 性能的急剧下降。

图 4 显示了模型权重更新之间的角度。∆PT、∆Task0 和 ∆Task1 分别表示预训练、微调任务 0 和微调任务 1 阶段的权重更新。∆Task0150 表示从第 150 步的权重减去第 0 步的权重所得到的权重更新。图 (a) 和 (b) 分别比较了预训练和任务 0 期间以及任务 0 和任务 1 期间的权重更新角度。完整结果见附录 G.3。

在前 150 步之后，训练轨迹中出现了一个关键的转折点：模型向右移动并达到了任务 1 的局部最小值。从这一分析中，我们得出以下两个关键见解：

1. **优化方向的矛盾**：在学习任务 1 的初期，任务 0 和任务 1 的梯度方向相反，这表明在仅使用任务 1 的数据进行微调时（即 SEQ），任务 0 和任务 1 的优化路径在训练开始时是矛盾的。
    
2. **两阶段训练轨迹**：整个训练轨迹可以分为两个阶段。第一阶段包括最初的 150 步，结合图 2a 中显示的恢复性能，我们得出这些步骤有效地撤销了任务 0 的对齐。第二阶段从第 150 步到训练结束，在此期间，模型同时学习了（1）任务 1 的对齐和（2）与任务 1 相关的知识。在这个第二阶段，我们观察到任务 0 的损失先略有下降，然后又上升。考虑到图 2a 中显示的任务 0 和任务 1 的准确率，我们假设任务 0 损失的下降对应于学习任务 1 对齐的效果，而随后的上升则对应于获取任务 1 知识的效果。不幸的是，学到的任务 1 对齐与任务 0 的对齐方向不一致，导致了任务 0 的虚假遗忘现象（如图 6 所示）。
    

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=MGYzYjBkMmE1OThjOTIwNDkzZGUwODVkNWRiNDg3OTlfZjBZMmtkWUdqYzRHSTdDNzNOQjAwNTVVZlRubjA5TlBfVG9rZW46TWZ6SWJMTEd4b1ZheHp4WjB2OWM5QTZjbmtiXzE3NDg3NTkxMTg6MTc0ODc2MjcxOF9WNA)

#### MODEL WEIGHT PERSPECTIVE

为了进一步剖析初始训练阶段的权重更新情况，我们评估了两个训练阶段的权重更新之间的角度 θ(∆A, ∆B)，这两个阶段的权重更新分别记为 ∆A 和 ∆B。该角度有助于我们了解这些阶段的权重是否在同一空间内进行更新。例如，对于 MLP 输出层的矩阵，我们首先使用奇异值分解（SVD）计算 ∆A 和 ∆B 的列空间。随后，在一个列空间的基向量与其在另一个列空间上的投影之间计算角度。接近零的角度表明权重在同一空间内更新，而接近 90 度的角度则表明更新发生在几乎正交的空间中。更多细节见附录 G.3。我们在图 4 中总结了结果。其他模型组件也观察到了类似的趋势，详见附录 G.3。

在图 4a（蓝色）中，不同预训练阶段更新之间的角度较小，这表明预训练更新发生在一致的空间内。同一图中的橙色部分显示，任务 0 的更新空间几乎与预训练相同，输入嵌入除外，这表明输入嵌入在任务 0 的对齐中起着重要作用。

图 4b（蓝色）表明，任务 1 的前 150 步更新的权重与任务 0 的空间接近。这表明这些初始步骤的主要作用是撤销任务 0 的对齐。同一图中的橙色部分显示，任务 1 后续步骤更新的权重处于明显不同的空间，尤其是影响底层，包括输入嵌入层。

图 5：主成分中特征的偏移。案例 1：微调任务 0（第 0 步 - 最终）；案例 2：微调任务 1（第 100-150 步）；案例 3：微调任务 1（第 200 步 - 最终）；案例 4：微调任务 1（第 0 步 - 最终）。完整结果见附录 G.4。

基于第 3.3 节的发现，我们得出结论：包括输入嵌入层在内的底层对于任务对齐至关重要。换句话说，这些层中几乎正交的权重更新导致了任务 0 和任务 1 之间对齐的差异，最终导致了任务 0 中观察到的虚假遗忘现象（如图 6 所示）。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=NDU2ZDJkNTBjMmJmMjJjZDFhYmU1YmU0ZjI1NDlhYmFfNjlyd1k0T01ybEt2bG5nOFpNWWNON1N4Nk4xV0xLeHFfVG9rZW46TTZtUGJRelZJb2VuVXh4alR2OGNRSXVwbjNlXzE3NDg3NTkxMTg6MTc0ODc2MjcxOF9WNA)

#### FEATURE PERSPECTIVE

在本节中，我们研究了在虚假遗忘背景下特征表示的变化情况。具体而言，对于每个训练阶段，我们计算了每个 Transformer 层的隐藏状态（特征）的差异以及这些特征的主成分的差异。主成分的小幅度偏移表明，特征变化的方向与原始特征几乎正交，这暗示修改后的特征可能在很大程度上保留了其先前的表示，并且可以通过逆转偏移来恢复。

结果汇总于图 5。值得注意的是，我们在前三种情况中观察到主成分的显著偏移（从图 5a 到 5f），而图 5g 和 5h 几乎没有偏移。这一模式表明，任务对齐的学习和遗忘——发生在任务 0、任务 1 的前 150 步以及任务 1 的后续步骤中——通常会导致特征表示的变化，这在主成分的偏移中得到了反映。

有趣的是，当我们考虑前 150 步和后续步骤的综合影响时，主成分的偏移消失了。这表明，尽管任务对齐不同，但两个任务的特征表示中存在共享模式，因为它们都将模型对齐到问答任务。换句话说，在任务 0 和任务 1 的情况下，主成分的偏移可以相互抵消，导致在考虑任务 1 的整个学习过程中主成分没有净变化。这表明任务 0 和任务 1 的任务对齐并非根本对立。此外，我们观察到主成分的偏移似乎起源于底层，并传播到上层。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=NDNiOWZlYTVlZDVkNzI2ZDgxZWU1YTFkZmI2MDlmMmFfSDEybDBnaVM5TXQ3bVBoQ2hhSDRzbkFuY1FvMjhONWNfVG9rZW46TllPWmJmVnVJb0FsTk54cXdQU2NVb0F4blZiXzE3NDg3NTkxMTg6MTc0ODc2MjcxOF9WNA)

#### SUMMARY

如图 6 所示，虚假遗忘的根本原因在于任务 0 和任务 1 对齐之间的差异。在第 5 节中，我们将证明数据重放或我们提出的冻结方法（Freeze）能够使模型学习到对齐的任务 0 和任务 1 对齐。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=ZmQyNDc1YWNhYWRlNjBhZWZlYmY3YzQ2MGI4ZTNkYjZfZDJIbU1oTVZvZzZ6T2hLdXpiZEFBSDNSeUFhMmYzRGZfVG9rZW46R2xuWWJsUWczb0NoVWN4Z2V0YWMwM2RnbkxYXzE3NDg3NTkxMTg6MTc0ODc2MjcxOF9WNA)

### method

#### 通过冻结底层来减轻虚假遗忘

之前的分析表明，现有的持续学习技术在应对虚假遗忘时存在困难，主要原因是它们未能有效缓解任务0的对齐撤销。这引发了一个问题：我们如何才能有效地实现这一目标？

**数据重放的启示**：为了探讨这一问题，我们回顾了第3.2节中关于任务0的恢复实验，其中在任务0数据的一部分上进行训练导致了性能的提升。这表明数据重放可能是对抗虚假遗忘的一种可行技术，因为在任务0数据的一个子集上进行训练可能有助于恢复任务0的对齐。表1证实了这一点，显示保留任务0的旧数据显著提高了任务0和任务1的性能。图3b中的损失景观图表明，尽管模型在优化新旧数据时最初撤销了任务0的对齐，但在任务1的学习过程中随后与任务0对齐，表明重新向任务0对齐。详细解释见附录G.2。

**模型更新的启示**：尽管存储了多达20%的旧数据，但在初始训练步骤中任务0的对齐撤销仍然不可避免。为应对这一挑战，我们借鉴了第3.4节中关于模型权重更新的见解。我们的研究发现，底层在学习和遗忘任务对齐的过程中起着关键作用。特征偏移的证据（图5）和命题4.9表明，特征的偏移起源于底层并向上累积。这导致了一个简单的解决方案：冻结底层的所有组件，包括输入嵌入层，记为冻结（Freeze）。

**缓解虚假遗忘的免费午餐**：为了验证这一假设，我们将冻结方法应用于传记数据集，结果汇总于表1。令人惊讶的是，冻结方法非常有效，将SEQ的性能从11%提升至44%，同时更新的参数不到一半。这种方法为缓解虚假遗忘提供了一个有效的解决方案，特别是在没有旧数据可用的情况下，可谓是一个宝贵的免费午餐。图7a显示了一个清晰的趋势：随着冻结层的数量从1增加到9，任务0的对齐撤销过程得到缓解。然而，这也减缓了任务1的学习并降低了模型容量。值得注意的是，随着更多层被冻结，在训练的后期阶段出现了显著的遗忘，表明稳定性和可塑性之间存在权衡。通过采用早期停止策略，在任务1的准确率超过99%时捕获模型，我们观察到了性能的提升，详细结果见表1。

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=ZjllOTA4NGRlZTNlMjlmNDRkZGQ4M2IzNGZmNTZhNDVfbHM2aFFmRjlQNVNFRHJDdjNIWGdVNUpjaENHYTE2ejFfVG9rZW46REVhaGJVeTFib2kySlh4cDZqdmNFZEhFbnplXzE3NDg3NTkxMTg6MTc0ODc2MjcxOF9WNA)

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=MTljZWI1YjMwNGRiNGNkNjc5ZmZjN2YwYTJjYTc2OGFfNHB5WW82MWJRdmpPNDBWYkx1eVRRNFF0RmkyRFE2dWVfVG9rZW46U0MyVmJNTkQ3b1Q4dVN4SkRCZWM5bTg2blhiXzE3NDg3NTkxMTg6MTc0ODc2MjcxOF9WNA)

