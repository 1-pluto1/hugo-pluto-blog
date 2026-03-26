---
title: "Don't Blind Your VLA: Aligning Visual Representations for OOD Generalization"
aliases: []
created: '2026-03-27 01:53:13'
updated: '2026-03-27 01:53:13'
content_type: research
note_type: literature
domain: embodied
status: reading
paper_title: "Don't Blind Your VLA: Aligning Visual Representations for OOD Generalization"
authors:
  - "Nikita Kachaev"
  - "Mikhail Kolosov"
  - "Daniil Zelezetsky"
  - "Alexey K. Kovalev"
  - "Aleksandr I. Panov"
year: 
venue: "arXiv"
doi: "10.48550/arXiv.2510.25616"
url: "https://doi.org/10.48550/arXiv.2510.25616"
zotero_key: "kachaevDontBlindYour2025"
zotero_link: ""
pdf_path: "/Users/yangchao/Zotero/storage/TL3PVS9A/Kachaev 等 - 2025 - Don't Blind Your VLA Aligning Visual Representations for OOD Generalization.pdf"
related_topics:
  - "Embodied Foundation Models and VLA Generalization"
publish_candidate: false
tags:
  - "research/literature"
categories: []
---

# Literature Note

## Abstract

The growing success of Vision-Language-Action (VLA) models stems from the promise that pretrained Vision-Language Models (VLMs) can endow agents with transferable world knowledge and vision-language (VL) grounding, laying a foundation for action models with broader generalization. Yet when these VLMs are adapted to the action modality, it remains unclear to what extent their original VL representations and knowledge are preserved. In this work, we conduct a systematic study of representation retention during VLA fine-tuning, showing that naive action fine-tuning leads to degradation of visual representations. To characterize and measure these effects, we probe VLA's hidden representations and analyze attention maps, further, we design a set of targeted tasks and methods that contrast VLA models with their counterpart VLMs, isolating changes in VL capabilities induced by action fine-tuning. We further evaluate a range of strategies for aligning visual representations and introduce a simple yet effective method that mitigates degradation and yields improved generalization to out-of-distribution (OOD) scenarios. Taken together, our analysis clarifies the trade-off between action fine-tuning and the degradation of VL representations and highlights practical approaches to recover inherited VL capabilities. Code is publicly available: https://blind-vla-paper.github.io

## Annotations

%% Optional: filled manually or by future tooling if you want to sync Zotero highlights. %%

## Attachment Notes

- `/Users/yangchao/Zotero/storage/TL3PVS9A/Kachaev 等 - 2025 - Don't Blind Your VLA Aligning Visual Representations for OOD Generalization.pdf`
- `/Users/yangchao/Zotero/storage/JGJFHE67/2510.html`

## Skill Summary

### One-Sentence Take

这篇论文聚焦一个很关键但常被忽略的问题：VLM 适配成 VLA 后，原有视觉-语言表示会不会退化；作者通过表示分析和一个轻量 visual alignment 方法，尝试在动作微调时保住这种通用视觉能力并提升 OOD 泛化。

### Problem

- VLA 模型通常继承自预训练的 VLM，但在面向机器人动作任务做 supervised fine-tuning 之后，这些模型原本的视觉-语言表示可能会塌缩、退化，导致在 OOD 场景下泛化变差。
- 这个问题重要，因为 VLA 的核心承诺就是“利用 VLM 的通用语义先验去提升 embodied generalization”；如果 fine-tuning 过程本身破坏了这些先验，VLA 的基础假设就会被削弱。

### Core Idea

- 先系统性分析 VLA fine-tuning 是否导致视觉表示退化，方法包括 attention map 检查、隐藏表示分析和一个名为 VL-Think 的诊断任务集。
- 在此基础上，提出一个轻量的 Visual Representation Alignment 方法，在动作微调阶段把 VLA 的视觉表示约束到一个通用视觉 teacher 的表征空间附近。
- 关键不是再做一个更复杂的 VLA 架构，而是在 task-specific SFT 阶段用很低开销保住原始 VLM 的视觉语义能力。

### Main Contributions

1. 系统展示了 naive VLA fine-tuning 会导致表示塌缩、attention sink 和 domain-specific forgetting，而不只是简单的性能波动。
2. 提出了 VL-Think 诊断任务集，用来评估 VLM 到 VLA 过程中视觉-语言知识迁移和遗忘情况。
3. 提出一个简单高效的 visual alignment 方法，在不明显增加复杂度的前提下缓解表示退化并提升 OOD 泛化。

### Method Notes

- Input / Output: 输入是图像和语言指令，目标是在机器人动作微调过程中保持视觉表示与 teacher visual model 的对齐，同时继续优化 action objective。
- Model / Pipeline: 先比较 VLM 与对应 VLA 的 attention maps、隐藏表示和 VL 能力，再在 fine-tuning 阶段加入 visual alignment loss。
- Training: 方法嵌入在 task-specific supervised fine-tuning 中，不强调重型额外监督，而强调一个轻量的正则约束。
- Evaluation: 评估重点是 OOD generalization，并从 attention、representation 和任务表现三个层面验证表示是否被保留。

### Results

- 关键实验结果：论文称在 Simpler-based benchmark 上，相比标准 SFT 能带来最高约 10% 的相对 OOD 泛化提升。
- 最强基线是谁：对照的核心对象是标准的 naive SFT VLA，以及其起点 VLM / OpenVLA-7B 等代表性模型。
- 作者声称的优势：方法几乎不增加训练复杂度，却能在动作微调时保住更多视觉语义结构，从而改善 OOD 表现。

### Strengths

- 选题非常扎实，切中了 VLA 研究里一个“大家默认存在，但很少被系统验证”的关键问题。
- 论文不是直接提出一个更大的模型，而是先用 attention map、t-SNE 和诊断任务把“表示退化”这个问题说清楚，论证路径比较完整。
- 方法轻量，这是它相对很多大规模多任务联合训练方法的现实优势，更容易作为现有 VLA 流程中的补丁接入。

### Limitations

- 论文主要解决的是“表示保持”问题，而不是统一建模、长时规划或更广义的 embodied foundation model 目标，因此能力范围相对聚焦。
- visual alignment 是否对不同机器人平台、不同视觉 backbone 都稳定有效，还需要更多跨模型验证。
- 如果动作任务本身确实需要某些表示重组，那么过强的表示对齐也可能限制模型对特定动作域的适配能力。

### My Thoughts

- 这篇论文很适合和 `Motus` 放在一起看：`Motus` 关注如何把更多 embodied capability 统一起来，而这篇关注的是在动作微调过程中如何保住预训练视觉表示，二者共同构成了“统一建模 vs 表示保真”的张力。
- 如果你后续写 topic note，这篇可以作为“VLA generalization 不只是训练规模问题，也包括 fine-tuning 过程中的表示退化问题”的核心证据。

## Linking

### Related Notes

- [[Embodied Foundation Models and VLA Generalization]]
- [[Motus: A Unified Latent Action World Model]]

## Quotes / Snippets

> 记录一段关键原文或你自己的改写。
