---
title: "Embodied Foundation Models and VLA Generalization"
aliases: []
created: '2026-03-27 01:55:00'
updated: '2026-03-27 02:03:00'
content_type: research
note_type: topic
domain: embodied
status: active
topic: "Embodied Foundation Models and VLA Generalization"
scope: "围绕统一 embodied foundation model、VLA 表征保持与 OOD 泛化的研究脉络整理"
related_domains:
  - "mllm"
  - "llm"
seed_notes:
  - "workspace/research/literature/embodied/Motus.md"
  - "workspace/research/literature/embodied/Don't Blind Your VLA.md"
target_post_path: "content/post/Research/具身智能/"
tags:
  - research/topic
categories: []
---

# Topic Note

## Topic Definition

- Topic: Embodied Foundation Models and VLA Generalization
- Scope: 关注 embodied foundation model 是否应该统一建模，以及 VLA 在动作微调后如何保持原有视觉-语言表征能力与 OOD 泛化能力。
- Why now: 这一方向正处于从“单一 policy 模型”向“统一世界模型 / foundation model”迁移的阶段，值得尽早梳理主线。

## Core Questions

1. 一个真正统一的 embodied foundation model 应该统一哪些能力，而不是只把 policy learning 做大？
2. 当 VLM/VLM-like backbone 被适配到动作空间后，原有视觉-语言表征会损失多少？
3. 大规模通用先验和 embodiment-specific 训练信号应该如何结合，才能提升真实泛化能力？

## Key Papers

- [[Motus]]
- [[Don't Blind Your VLA]]
- [[content/post/Research/AI4SE/The Hitchhikers Guide to Production-ready Trustworthy Foundation Model powered Software (FMware)]]

## Main Threads

### Thread 1

- `Motus` 代表的是“统一建模”路线：尝试把 understanding、video generation、action 等多种 embodied capability 融到一个 latent action world model 里。
- 这条路线关心的核心不是单一任务性能，而是能否在一个 shared architecture 里同时容纳 VLA、world model、IDM、video generation 等多种分布建模目标。

### Thread 2

- `Don't Blind Your VLA` 代表的是“能力保真”路线：重点不是再做更大的统一模型，而是研究 VLA 微调后，原始视觉表征是否退化，以及如何修复这种退化。
- 这条路线强调：如果动作微调破坏了预训练 VLM 的视觉-语言先验，那么再大的 embodied model 也可能在 OOD 场景下失去泛化基础。

### Thread 3

- 两篇论文共同指向一个核心问题：embodied intelligence 不只是 action prediction，本质上是在平衡通用感知先验、动作适配、和真实泛化三者之间的关系。
- 从研究脉络看，可以把这一主题理解为两层耦合问题：
  1. 模型层面是否要统一更多能力；
  2. 表示层面如何在统一或适配过程中避免遗忘已有先验。

## Agreement / Disagreement Map

- 共同点：
  两篇论文都把泛化问题当成 embodied intelligence 的核心瓶颈，并且都默认“保留预训练视觉-语言能力”对下游机器人任务是有价值的。
- 分歧点：
  `Motus` 关注的是如何把更多 embodied capability 融到同一个统一模型中；`Don't Blind Your VLA` 关注的是在现有 VLA 微调过程中，如何避免视觉表示退化。
- 方法侧差异：
  `Motus` 倾向于通过更统一的建模对象、latent action 和大规模数据配方来解决问题；`Don't Blind Your VLA` 倾向于通过表示诊断与轻量 alignment loss 做 targeted fix。
- 还没搞清楚的点：
  统一模型的收益到底来自更好的 inductive bias，还是来自更大的训练规模与更多模态先验；同时，表示保持是否会和动作特化之间存在系统性冲突，也还没有被彻底回答。

## Important Methods / Terms

- Mixture-of-Transformer (MoT)
- UniDiffuser-style scheduler
- latent action
- optical flow as action proxy
- visual representation retention
- OOD generalization
- attention sink
- representation collapse
- VL-Think
- visual alignment loss

## Open Problems

- 如何量化 unified embodied model 中不同子能力之间的真正共享程度，而不是只看最终 success rate？
- 如何在不牺牲动作适配能力的前提下保留 VLM 的视觉-语言表示，并判断这种保留是否真能转化为真实任务泛化？
- 如何让大规模视频先验、机器人轨迹和 world model 训练更系统地统一起来，而不是简单堆叠数据与模块？
- 对不同 embodiment、不同视觉 backbone、不同动作空间，表示退化和统一建模收益是否具有一致规律？

## Writing Angle

- 如果要写成博客，核心观点是什么？
  embodied foundation model 的关键不只是“把更多模块塞一起”，而是如何在统一建模与表示保持之间取得平衡。
- 目标读者是谁？
  关注 VLA、world model、robot foundation model 的研究者和入门读者。
- 最适合发到哪个栏目？
  `content/post/Research/具身智能/`
- 博客结构建议：
  先交代为什么“统一建模”成为趋势，再引出为什么“表示退化”会成为这一趋势下的新问题，最后用 `Motus` 和 `Don't Blind Your VLA` 做对照式展开。

## Related Notes

- [[Motus]]
- [[Don't Blind Your VLA]]
- [[workspace/research/literature/embodied/Motus]]
- [[workspace/research/literature/embodied/Don't Blind Your VLA]]
