---
title: "Motus: A Unified Latent Action World Model"
aliases: []
created: '2026-03-27 01:00:16'
updated: '2026-03-27 01:00:16'
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
year: 2025
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

一句话写清这篇论文做了什么、为什么重要。

### Problem

- 这篇论文要解决什么问题？
- 为什么这个问题重要？

### Core Idea

- 核心方法是什么？
- 和已有方法相比，关键差异是什么？

### Main Contributions

1. 
2. 
3. 

### Method Notes

- Input / Output:
- Model / Pipeline:
- Training:
- Evaluation:

### Results

- 关键实验结果：
- 最强基线是谁：
- 作者声称的优势：

### Strengths

- 

### Limitations

- 

### My Thoughts

- 这篇论文对我当前研究的启发是什么？
- 可以和哪些已有笔记关联？

## Linking

### Related Notes

- [[相关主题笔记]]
- [[相关论文笔记]]

## Quotes / Snippets

> 记录一段关键原文或你自己的改写。
