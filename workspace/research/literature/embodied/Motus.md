---
title: "Motus: A Unified Latent Action World Model"
aliases: []
created: '2026-03-27 01:00:16'
updated: '2026-03-27 01:36:43'
content_type: research
note_type: literature
domain: embodied
status: reading
paper_title: "Motus: A Unified Latent Action World Model"
authors:
  - "Hongzhe Bi"
  - "Hengkai Tan"
  - "Shenghao Xie"
  - "Zeyuan Wang"
  - "Shuhe Huang"
  - "Haitian Liu"
  - "Ruowen Zhao"
  - "Yao Feng"
  - "Chendong Xiang"
  - "Yinze Rong"
  - "Hongyan Zhao"
  - "Hanyu Liu"
  - "Zhizhong Su"
  - "Lei Ma"
  - "Hang Su"
  - "Jun Zhu"
year: ""
venue: "arXiv"
doi: "10.48550/arXiv.2512.13030"
url: "https://doi.org/10.48550/arXiv.2512.13030"
zotero_key: "biMotusUnifiedLatent2025"
zotero_link: ""
pdf_path: "/Users/yangchao/Zotero/storage/RPZ4ICPA/Bi 等 - 2025 - Motus A Unified Latent Action World Model.pdf"
related_topics: []
publish_candidate: false
tags:
  - research/literature
categories: []
---

# Literature Note

## Abstract

While a general embodied agent must function as a unified system, current methods are built on isolated models for understanding, world modeling, and control. This fragmentation prevents unifying multimodal generative capabilities and hinders learning from large-scale, heterogeneous data. In this paper, we propose Motus, a unified latent action world model that leverages existing general pretrained models and rich, sharable motion information. Motus introduces a Mixture-of-Transformer (MoT) architecture to integrate three experts (i.e., understanding, video generation, and action) and adopts a UniDiffuser-style scheduler to enable flexible switching between different modeling modes (i.e., world models, vision-language-action models, inverse dynamics models, video generation models, and video-action joint prediction models). Motus further leverages the optical flow to learn latent actions and adopts a recipe with three-phase training pipeline and six-layer data pyramid, thereby extracting pixel-level "delta action" and enabling large-scale action pretraining. Experiments show that Motus achieves superior performance against state-of-the-art methods in both simulation (a +15% improvement over X-VLA and a +45% improvement over Pi0.5) and real-world scenarios(improved by +11~ 48%), demonstrating unified modeling of all functionalities and priors significantly benefits downstream robotic tasks.

## Annotations

%% Optional: filled manually or by future tooling if you want to sync Zotero highlights. %%

## Attachment Notes

- `/Users/yangchao/Zotero/storage/RPZ4ICPA/Bi 等 - 2025 - Motus A Unified Latent Action World Model.pdf`
- `/Users/yangchao/Zotero/storage/JVI3YERM/2512.html`

## Skill Summary

### One-Sentence Take

Motus 试图把理解、视频生成和动作建模统一进一个 latent action world model，用共享注意力、统一调度器和大规模异构数据预训练来提升 embodied policy 的泛化能力。

### Problem

- 现有 embodied foundation model 往往把 VLA、world model、inverse dynamics、video generation 等能力拆成多个孤立范式，导致系统无法真正形成统一的认知与决策闭环。
- 机器人学习又高度依赖异构数据，但不同 embodiment 的动作空间差异很大，很多互联网视频又没有动作标签，导致大规模预训练很难直接迁移到 policy learning。

### Core Idea

- 用一个 Mixture-of-Transformer (MoT) 架构把 understanding expert、video generation expert 和 action expert 放进同一个统一模型中，并通过共享 self-attention 做跨模态融合。
- 用类似 UniDiffuser 的调度器统一处理视频和动作等不同模态，从而支持 world model、VLA、IDM、video generation 和 joint prediction 等多种推理模式切换。
- 用 optical flow 学 latent action，把视觉动态压缩成可迁移的“delta action”，再配合三阶段训练流程和六层数据金字塔，把互联网视频、人类演示、仿真与机器人轨迹一起纳入预训练。

### Main Contributions

1. 提出 Motus，一个试图统一五类 embodied modeling paradigm 的 latent action world model，而不是只在单一范式上做增强。
2. 提出 latent action 设计，用 optical flow 对齐跨 embodiment 的运动模式，让无动作标签的视频数据也能参与 action pretraining。
3. 给出三阶段训练流程和六层数据金字塔，并在仿真和真实机器人实验中验证统一建模与大规模先验融合的有效性。

### Method Notes

- Input / Output: 输入当前观测、语言指令以及不同模式下的动作/未来视觉信息；输出可以是动作 chunk、未来视觉状态，或两者的联合预测。
- Model / Pipeline: 核心是 Tri-model Joint Attention，把 video generator、action expert 和 understanding expert 用共享注意力层耦合起来。
- Training: 采用 video pretraining -> latent action pretraining -> embodiment-specific action finetuning 的三阶段流程；latent action 通过 optical flow 和少量动作标签联合学习。
- Evaluation: 评估覆盖 simulation 和 real-world setting，并和 X-VLA、Pi0.5 等方法比较。

### Results

- 关键实验结果：论文摘要中给出的代表性结果是 simulation 上相对 X-VLA 提升约 15%，相对 Pi0.5 提升约 45%；real-world 场景提升约 11% 到 48%。
- 最强基线是谁：从摘要和方法定位看，X-VLA、Pi0.5 以及其他现有 VLA / world model / video policy 方法是核心对比对象。
- 作者声称的优势：统一建模没有牺牲多模态先验，反而能把 general multimodal priors 和 domain-specific robotics priors 融合起来，提升 policy 泛化能力。

### Strengths

- 问题定义很清楚，明确指出 embodied intelligence 目前最大的缺口是“统一性”与“大规模异构预训练”的矛盾。
- 方法上不是简单堆模块，而是围绕统一五类分布建模目标组织架构、latent action 和训练 recipe，整体叙事比较完整。
- 从摘要和前几节看，作者对“为什么 optical flow 适合作为跨 embodiment 动作中介”给出了较强的动机。

### Limitations

- 模型系统复杂度很高，三专家架构、统一调度器、latent action 和分阶段训练叠加后，训练与调参成本可能非常高。
- 统一了多种范式，但不同能力是否真的都被同一组共享表示稳健支撑，还需要更细的消融和 failure case 分析。
- latent action 强依赖 optical flow 表达质量，若场景噪声大、遮挡强或视觉动态和真实动作不一致，迁移效果可能会受影响。

### My Thoughts

- 这篇工作很适合放到 `embodied / VLA / world model` 交叉线上看，它不是单纯做 policy finetuning，而是在追求一个更接近“统一 embodied foundation model”的方向。
- 如果后续你要整理 topic note，可以把它和 X-VLA、Pi0.5、F1、UWM 一类工作放在一起，对比“统一建模程度”“动作表示方式”“预训练数据来源”这三个维度。

## Linking

### Related Notes

- [[相关主题笔记]]
- [[相关论文笔记]]

## Quotes / Snippets

> 记录一段关键原文或你自己的改写。
