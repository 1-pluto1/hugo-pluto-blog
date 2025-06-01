---
title: "s1: Simple test-time scaling"
date: 2025-02-16T11:30:03+00:00
tags:
  - LLM
  - NLP
  - ReasoningModel
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


单位： Stanford

代码：https://github.com/simplescaling/s1

基座模型： Qwen2.5 32B-Instruct

原文地址：https://arxiv.org/abs/2501.19393

### 主要内容：

提出了一种test-time scaling方法，用于在推理任务中提升大模型性能。

**小规模高质量数据集（s1K）的构建** 基于难度、多样性和质量三大标准，整理了一个包含1,000道问答及推理过程痕迹（reasoning traces）的数据集（s1K），并通过消融实验验证了这三大标准对模型性能的重要性。

**“预算强制”（Budget Forcing）策略** 为了灵活控制模型在推理时所用的计算预算，作者引入了“预算强制”技术：

- 当模型过早结束思考时，强制它进一步推理（例如多次添加“Wait”），从而鼓励模型反复检查和修正思路。
    
- 在需要限制生成时，则可强行截断推理过程，从而实现对推理时长或计算预算的约束。
    

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=NDU5OGJmMGMwNzUwMGRmZWRiOGViM2ExYmZkNGRmYzhfUVlHeWNqd2lJZ1V6bEoxNk1qaVk4TlpFMThjN2dlczZfVG9rZW46Q3B5RGI5V0N4b3FYZFl4TmJMMGNSenp1bnplXzE3NDg3NTg4Njk6MTc0ODc2MjQ2OV9WNA)

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=MjQ0NGU3ODczNWMyYWM1YjJhODFiMGI1ZTZlZGUzMDZfbk9oakYweXlPZmdLbW1tVWhzNUtmRFZKdzhCSFBrYUlfVG9rZW46SHBJa2JpZ0NLbzVXODl4ZEU2NWN0UGllbk5kXzE3NDg3NTg4Njk6MTc0ODc2MjQ2OV9WNA)

**显著提升了模型在数学竞赛题目上的表现** 利用上述数据集和“预算强制”方法，对大模型（Qwen2.5 32B-Instruct）进行了监督微调并命名为 s1-32B。在数学竞赛数据集（MATH 和 AIME24）上，s1-32B 相比 OpenAI 提供的 o1-preview 模型在准确率上最高提升可达 27%。同时，通过在推理过程中灵活地应用“预算强制”，可以让模型取得比常规推理更高的准确率（从 50% 提升到 57%）。

  

### 数据集构建

#### 原则

质量、难度和多样性。

#### 59K数据集的构建

##### 数据来源

- **现有数据集**：包括来自 NuminaMATH、历史 AIME 题目、OlympicArena、OmniMath、AGIEval 等不同领域和考试类型的问题，涵盖数学、天文、化学、计算机科学、法律、逻辑等诸多方向。
    
- **自行新建数据集**：包括来自斯坦福大学统计系博士资格考试的概率题（s1-prob）以及源自面试脑筋急转弯题的高难度题目（s1-teasers）。
    

##### **生成推理过程与答案**

- 利用 Google Gemini Flash Thinking API 为每道问题生成推理过程与最终解答，从而得到包含问题、推理过程和答案的三元组。
    

##### **数据去重与“污染”检测**

- 对数据进行去重处理，并通过 8-grams 检测与目标评测数据（如 MATH500、GPQA Diamond、AIME24）重叠的样本，最终保证评价结果的客观性和可靠性。
    

  

#### s1K数据集的构建

##### **质量筛选（Quality）**

- 排除出现 API 错误的题目，将规模从 59K 减至 54,116。
    
- 进一步删除格式混乱、不完整或存在异常的样本后，剩余 51,581 道题目。
    
- 从这部分数据中挑选出公认质量较高、不需进一步过滤的数据集，共计 384 道题目直接进入最终的 1,000 道题之列。
    

##### **难度筛选（Difficulty）**

- 通过在两个不同规模的模型（Qwen2.5-7B-Instruct 和 Qwen2.5-32B-Instruct）上评测每道题目是否能正确解答，如果任一模型能轻易解出，则认为该题过于简单予以剔除。
    
- 结合推理过程的长度（使用 Qwen2.5 的 tokenizer 统计）作为另一项难度指标，筛去简短或较易的题目，将规模进一步缩减至 24,496 道。
    

##### **多样性筛选（Diversity）**

- 借助 Claude 3.5 Sonnet 将每道题目按美国数学学会（AMS）数学学科分类（MSC）等进行领域标注，这些领域不仅包含数学，也覆盖其他科学学科。
    
- 最终从 24,496 道题目中，先按领域随机选择一个学科，然后在该学科下按推理长度偏好抽取一道题目（偏好较长推理过程），重复该流程，直至达到 1,000 道题目。
    
- 该子集覆盖了 50 个不同领域，兼具广度与深度。
    

  

  

### test-time scaling

1. **顺序式（Sequential）**：后续计算依赖前面的中间结果（例如在推理过程中生成更长的思考过程，并不断基于前面内容进行迭代修正）；
    
2. **并行式（Parallel）**：不同计算互相独立地并行进行（例如多数表决，majority voting）。
    

  

##### **预算强制（Budget forcing）**

- 该策略通过在解码阶段强制模型生成最少或最多一定数量的“思考 token”，来实现对推理时间或推理计算量的灵活控制。
    
- **强制最大 token**：在到达指定上限时，自动插入结束推理的标记（`end-of-thinking token`）以及“Final Answer:”来使模型提前停止推理，并输出当前的最佳答案。
    
- **强制最少 token**：禁止模型提前结束推理，同时可在推理过程中追加“Wait”以提示模型“继续思考”，从而延长推理过程。
    

如以下例子：

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=N2I2YTM0OGQzZGY5NjkzYjcwMzIxMDg2YzY5ODU1M2ZfUW5KYkF5S0dyZlJWT1h3OHJuNW92aXpGUUJoN3A1VHVfVG9rZW46VDdvMWJEQk50b3NYZm94VUE5WWNQejRYbnRkXzE3NDg3NTg4Njk6MTc0ODc2MjQ2OV9WNA)

##### **Baselines**

作者将提出的预算强制方法与以下两类方法进行了对比：

1. **条件长度控制（Conditional length-control）**
    
    1. **基于 token 粒度**：在提示中明确指定允许的最大思考 token 数。
        
    2. **基于推理步骤（step）**：指定一个上限步数，每步大约包含 100 个 token。
        
    3. **基于类别（short/long）**：分别提供两种泛化提示（如“简短思考”或“深入思考”）。
        
2. **拒绝采样（Rejection sampling）**
    
    1. 在生成过程中，对当前生成的长度进行检测，若超出设定的预算就丢弃重新采样，直到符合预算为止，等同于在生成长度条件下寻找最优解的近似过程。
        

##### Metrics

作者不仅关注最终能够取得的最高准确率，还同样注重方法在推理计算量（thinking tokens）方面的可控性以及随计算量增大时性能的提升趋势。为此，作者提出了如下三项指标来全面衡量方法的质量：

1. **可控性（Control）**
    

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=MjFmMTExN2U3OWMxNmQxNzA1NTNkYmNhZDRlMzE1N2NfODRPM2J4SWJkaHZESWpzOGZtMHlyNTdQVEdPcU1XbzNfVG9rZW46TlJjNGJsbW9Sb29SVmp4MU1KQWM1dnVjbnViXzE3NDg3NTg4Njk6MTc0ODc2MjQ2OV9WNA)

2. 其中，$$\alpha_{\min}$$ 与 $$\alpha_{\max}$$分别代表预先设定的最小和最大思考 token（即推理计算预算），$$\mathcal{A$$ 为不同预算取值的集合。指标值以百分比形式呈现，若该方法在整个预算范围内都能稳定地生成对应长度的思考内容，则可控性可达 100%。这一指标衡量了方法能否在测试阶段有效地按照预期的“预算”来控制推理过程长度。
    

  

3. **扩展性（Scaling）**
    

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=YWFjZjE5NjM5MzQ2MTk1NTg5Nzk1ODQ2ZDAwMjQwOTZfSUNudG4yTFU0UjZHYThsWmttbFptRUFxc3NpcU5oY21fVG9rZW46UlhEamJ6enAwb1VzeUl4cmJOZGNGZ3BybjZmXzE3NDg3NTg4Njk6MTc0ODc2MjQ2OV9WNA)

4. 该指标取决于随着思考 token 的增加，模型在准确率$$f(\cdot$$上能否取得更高的提升，并且提升速度（斜率）有多大。它用 piece-wise linear 函数的平均斜率来衡量方法的整体扩展能力。若指标为正且数值越大，说明增加推理预算能更有效地提升模型表现。
    
5. **性能上限（Performance）**
    

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=NDIzMmI4ODlmNzk1NzczYzRjOWU2YjAwNzJiMzZkNmNfV3JhajZRc0w5a0JyTXdySlhMTDlESUh3QkZlNWRZdmVfVG9rZW46UmxZeGJkdWtrbzZiVFp4NHMwN2M4U0JXbnlmXzE3NDg3NTg4Njk6MTc0ODc2MjQ2OV9WNA)

6. 即在给定的测试预算范围内，该方法所能达到的最高准确率。尽管理想情况下，如果模型在预算增加时准确率能持续上升，理论上可达到“100%”或者无限逼近最高值，但现实中还受可控性、上下文窗口大小等因素的限制，最终会出现增长趋缓或被迫停止的情况。
    

  

##### Ablations

**预算强制（Budget forcing）**

- 预算强制方法不仅在控制上表现完美（即可以精准控制推理的计算量），而且在扩展性和性能方面也表现出色，尤其在 **AIME24** 数据集上，获得了最佳得分。因此，作者决定在后续实验中（如 **s1-32B** 模型的评估）采用该方法。
    
- 对于性能外推（extrapolating performance）的不同字符串比较，发现使用 “Wait” 字符串时，性能最为优越。
    

**类别条件控制（Class-conditional control）** 在对类别条件控制的评估中，作者总结了以下几点：

- **Token-conditional control** 由于模型无法可靠地计数 token，因此在没有预算强制的情况下会失败。即使训练时有要求，模型仍无法有效控制生成的 token 数量。
    
- **Step-conditional control** 中，尽管设置了不同的步骤目标（step targets），但模型会通过在少数步骤中生成大量 token 或在许多步骤中生成少量 token 来“规避”计算约束，因此这种方法的可控性较差。
    
- **Class-conditional control** 方法通过简单地告知模型“思考更长时间”来增加推理计算量，从而提升性能。该方法在扩展性方面表现较好，并且能显著改善测试时的计算控制和模型的性能。
    

  

  

  

### S1.1

生成思维链的从Gemini换到了DeepSeek R1

![](https://dppemvhuzp.feishu.cn/space/api/box/stream/download/asynccode/?code=OGM4OTMwMzUxZjhlZjg5NTBiZTJlMTAzYjg3OGI1YTBfMDN4QkhUZWNWVlk4azRCb2Y5ajM2blBvVlludmhWMTZfVG9rZW46Q2g3bWI0Zjl4b2JnZzh4enVtbmNiaU1CblhlXzE3NDg3NTg4Njk6MTc0ODc2MjQ2OV9WNA)