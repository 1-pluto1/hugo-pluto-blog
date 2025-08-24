---
title: Qwen-Image Technical Report
date: 2022-09-15T11:30:03+00:00
tags:
  - 深度学习
  - LLM
  - MLLM
  - NLP
categories:
  - AI
  - MLLM
  - Research
author: ZhaoYang
showToc: true
TocOpen: true
draft: true
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

### Abstract

我们推出了Qwen-Image，它是Qwen系列中的一款图像生成基础模型，在复杂文本渲染和精准图像编辑方面取得了显著进展。

为应对复杂文本渲染的挑战，我们设计了一套全面的数据处理流程，涵盖了大规模数据收集、筛选、标注、合成与平衡。此外，我们采用了一种渐进式训练策略，该策略从非文本到文本的渲染开始，由简至繁地处理文本输入，并逐步扩展至段落级别的描述。这种课程学习（curriculum learning）方法极大地增强了模型原生的文本渲染能力。因此，Qwen-Image不仅在英语等字母语言上表现出色，在更具挑战性的中文等象形文字（logographic languages）上也取得了卓越的进展。

为了提升图像编辑的一致性，我们引入了一种改进的多任务训练范式。该范式不仅包含了传统的文本到图像（T2I）和文本-图像到图像（TI2I）任务，还加入了图像到图像（I2I）的重建任务，从而有效地对齐了Qwen2.5-VL和MMDiT之间的潜在表征（latent representations）。此外，我们将原始图像分别输入到Qwen2.5-VL和VAE编码器中，以分别获取语义表征和重建表征。这种双重编码机制使得编辑模块能够在保持语义一致性与维持视觉保真度之间达到平衡。

我们在多个公开基准测试上对Qwen-Image进行了全面评估，包括用于通用图像生成的GenEval、DPG和OneIG-Bench，以及用于图像编辑的GEdit、ImgEdit和GSO。Qwen-Image取得了业界顶尖（state-of-the-art）的性能，展现了其在图像生成和编辑两方面的强大能力。此外，在LongText-Bench、ChineseWord和CVTG-2K上的测试结果表明，Qwen-Image在文本渲染方面表现卓越——尤其是在中文文本生成上——以显著优势超越了现有的顶尖模型。这凸显了Qwen-Image作为一个领先图像生成模型的独特定位：它既具备广泛的通用能力，又拥有卓越的文本渲染精度。



### Introduction

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20250824153323672.png" alt="image-20250824153323672" style="zoom:50%;" />

图像生成模型——涵盖文本到图像生成（T2I）(Rombach et al., 2021; Wu et al., 2022; Liang et al., 2022; OpenAI, 2023; Podell et al., 2023; Chen et al., 2024c; Li et al., 2024b; Esser et al., 2024; BlackForest, 2024; Gao et al., 2025; Gong et al., 2025; Cai et al., 2025) 和图像编辑（TI2I）(Brooks et al., 2023; Zhang et al., 2023; Wang et al., 2025; Deng et al., 2025; Labs et al., 2025; Wu et al., 2025b; Liu et al., 2025b; Cai et al., 2025; OpenAI, 2025)——已成为现代人工智能的基础组成部分，它使机器能够根据文本提示合成或修改视觉上引人入胜且语义上连贯的内容。在过去几年里，这一领域取得了显著进展，特别是随着基于扩散的模型架构（Ho et al., 2020; Liu et al., 2022）的出现，这些架构能够生成高分辨率图像，同时捕捉精细的语义细节。

尽管取得了这些进步，但两个关键挑战依然存在。首先，对于文本到图像生成任务，如何使模型输出与复杂、多方面的提示词对齐仍然是一个重大障碍。我们的评估显示，即便是像GPT Image 1 (OpenAI, 2025) 和 Seedream 3.0 (Gao et al., 2025) 这样业界顶尖的商业模型，在面对需要多行文本渲染、非字母语言（如中文）渲染、局部文本插入或文本与视觉元素无缝融合的任务时，也表现得力不从心。其次，对于图像编辑任务，实现编辑后输出与原始图像的精准对齐带来了双重挑战：(i) **视觉一致性**，即只应修改目标区域，同时保留所有其他视觉细节（例如，改变头发颜色而不改变面部细节）；以及 (ii) **语义连贯性**，即在进行结构性改变时必须保持全局语义（例如，修改人物姿势时保持其身份和场景的连贯性）。

在本文中，我们推出了Qwen-Image，这是Qwen系列中一款新颖的图像生成模型，旨在通过全面的数据工程、渐进式学习策略、增强的多任务训练范式以及可扩展的基础设施优化来克服这些挑战。

为应对复杂提示词对齐的挑战，我们开发了一个稳健的数据处理流程，涵盖了大规模收集、标注、筛选、合成数据增强和类别平衡。我们进一步采用了一种课程学习（curriculum learning）策略，从基础的文本渲染任务开始，逐步推进到段落级别和对布局敏感的描述。这种方法显著提升了模型遵循不同语言指令的能力，特别是像中文这样的象形文字语言。

为应对图像对齐的挑战，我们提出了一种增强的多任务学习框架，该框架在一个共享的潜在空间内无缝集成了T2I、I2I和TI2I的目标。具体来说，输入图像被编码为两种不同但互补的特征表示：通过Qwen-VL (Bai et al., 2025) 提取**语义特征**，以捕捉高层次的场景理解和上下文含义；同时通过VAE编码器获得**重建特征**，以保留低层次的视觉细节。这两组特征随后被共同作为条件信号输入到MMDiT架构 (Esser et al., 2024) 中。这种双重条件设计使模型能够同时保持语义连贯性和视觉一致性。

为了确保大规模训练的效率和稳定性，我们设计了一个利用TensorPipe进行分布式数据加载和预处理的**生产者-消费者（Producer-Consumer）框架**。生产者负责处理VAE编码和数据I/O等预处理任务，而消费者则使用Megatron (Shoeybi et al., 2019) 框架专注于分布式模型训练。我们还实施了广泛的监控工具，以确保在整个大规模训练过程中的可靠收敛和调试能力。

Qwen-Image在根据复杂文本提示生成高质量图像以及执行精确、具备上下文感知能力的图像编辑方面都取得了显著的进步。该模型能够解释复杂的语言结构，并生成在视觉上引人注目且同时符合语义意图和视觉约束的输出。为验证其有效性，我们在包括文本到图像生成和图像编辑在内的一系列多样化任务上对Qwen-Image进行了评估。

Qwen-Image的主要贡献可总结如下：

- **卓越的文本渲染能力：** Qwen-Image在复杂文本渲染方面表现出色，包括多行布局、段落级语义和精细细节。它能高保真地支持字母语言（如英语）和象形文字语言（如中文）。
- **一致的图像编辑效果：** 通过我们增强的多任务训练范式，Qwen-Image在编辑操作中，在保持语义含义和视觉真实性两方面都取得了卓越的性能。
- **跨基准测试的强劲性能：** 在多个基准测试上的评估表明，Qwen-Image在各种生成和编辑任务中持续优于现有模型，为图像生成领域建立了一个强大的基础模型。

### Model

在本节中，我们将介绍Qwen-Image模型的架构设计，并详细阐述其训练数据与训练细节。

#### Model Architecture

如图6所示，Qwen-Image架构建立在三个协同工作的核心组件之上，以实现高保真的文本到图像生成。

- **首先**，一个**多模态大语言模型（MLLM）\**作为\**条件编码器**，负责从文本输入中提取特征。
- **其次**，一个**变分自编码器（VAE）\**充当\**图像标记器（image tokenizer）**，将输入图像压缩成紧凑的潜在表征（latent representations），并在推理（inference）阶段将其解码还原。
- **第三**，一个**多模態擴散Transformer（MMDiT）\**作为\**骨干扩散模型**，在文本引导下，对噪声和图像潜在变量之间的复杂联合分布进行建模。

虽然本节概述了这些组件的一般作用，但具体的模型选择和架构细节将在后续章节中进行详细阐述。

#### Multimodal Large Language Model

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20250824164346816.png" alt="image-20250824164346816" style="zoom:50%;" />



Qwen-Image 采用 Qwen2.5-VL 模型 (Bai et al., 2025) 作为文本输入的特征提取模块，主要基于以下三个关键原因：

1. Qwen2.5-VL 的语言和视觉空间已经对齐，相比像 Qwen3 (Yang et al., 2025) 这样的纯语言模型，它更适合文本到图像任务；
2. Qwen2.5-VL 保留了强大的语言建模能力，与纯语言模型相比性能没有明显下降；
3. Qwen2.5-VL 支持多模态输入，从而使 Qwen-Image 能够解锁更广泛的功能，例如图像编辑 (Labs et al., 2025)。

设 x 和 y 分别表示图像和文本输入。给定用户的输入（例如提示词和图像），我们采用 Qwen2.5-VL 模型来提取特征。为了更好地引导模型生成精炼的潜在表征（latent representation），并考虑到不同任务下输入模态的变化，我们分别为纯文本输入和“文本+图像”混合输入设计了不同的系统提示（system prompts）。我们在图7和图15中展示了该系统模板。

最终，我们利用 Qwen2.5-VL 语言模型主干的最后一层隐藏状态，将其作为用户输入的最终表征。

#### Variational AutoEncoder

一个强大的变分自编码器（VAE）表征对于构建强大的图像基础模型至关重要。当前的图像基础模型通常会在海量图像数据集上，利用2D卷积来训练一个图像VAE，以获得高质量的图像表征。与此不同，我们的工作旨在开发一种能同时兼容图像和视频的、更通用的视觉表征。然而，现有的图像-视频联合VAE，例如Wan-2.1-VAE (Wan et al., 2025)，通常会面临性能上的权衡，导致图像重建能力下降。

为此，我们采用了一种“**单编码器、双解码器**”的架构。该设计使用一个兼容图像和视频的共享编码器，同时为每种模态分别配备了独立的、专用的解码器，这使得我们的图像基础模型能够作为未来视频模型的骨干网络。具体来说，我们采用了Wan-2.1-VAE的架构，冻结其编码器，并**只对图像解码器进行微调**。

为了提升重建的保真度，尤其是在小文本和精细细节方面，我们在一个自研的、富含文本的图像语料库上训练解码器。该数据集包含了真实世界的文档（PDF、PPT、海报）以及合成的文本段落，内容涵盖了字母语言（如英语）和象形文字语言（如中文）。

在训练过程中我们观察到：(1) 平衡**重建损失（reconstruction loss）**与**感知损失（perceptual loss）**能有效减少网格状的伪影（grid artifacts），这种伪影常见于像灌木丛这样的重复性纹理中。(2) 随着重建质量的提升，**对抗性损失（adversarial loss）**会变得无效，因为判别器（discriminator）已无法提供有效的指导。基于这些观察，我们只使用重建损失和感知损失，并在微调过程中动态调整它们的比例。

有趣的是，我们发现，仅微调解码器就能有效地增强细节并改善小文本的渲染效果，从而为Qwen-Image的文本渲染能力打下了坚实的基础。相关的定量和定性结果在第5.2.1节中呈现。



#### Multimodal Diffusion Transformer

Qwen-Image 采用**多模态扩散Transformer (MMDiT)** (Esser et al., 2024) 来对文本和图像进行联合建模。这种方法已在一系列工作（如FLUX系列 (BlackForest, 2024; Labs et al., 2025) 和 Seedream系列 (Gong et al., 2025; Gao et al., 2025)）中被证明是有效的。1

在每个模块（block）中，我们引入了一种新颖的位置编码方法：**多模态可缩放旋转位置编码（Multimodal Scalable RoPE, MSRoPE）**。如图8所示，我们比较了多种文本-图像联合位置编码策略。在传统的MMDiT模块中，文本标记（token）被直接拼接到经过展平（flattened）的图像位置嵌入之后。而 Seedream 3.0 (Gao et al., 2025) 则引入了可缩放旋转位置编码（Scaling RoPE），其中图像的位置编码被移至图像中心区域，而文本标记则被视为形状为 `[1, L]` 的二维标记。然后，使用二维旋转位置编码（2D RoPE） (Heo et al., 2024) 进行图文联合位置编码。

尽管这种调整有助于分辨率的缩放训练，但文本和图像的某些行的位置编码（例如图8(B)中的第0个中间行）会变得同构（isomorphic），这使得模型更难区分文本标记和位于该行的图像潜在标记。此外，要确定一个合适的图像行来拼接文本标记也并非易事。

为了解决上述挑战，我们引入了多模态可缩放旋转位置编码（MSRoPE）。在这种方法中，文本输入被视为二维张量，并且其两个维度应用相同的位置ID。如图8(C)所示，这在概念上等同于将文本沿着图像的对角线进行拼接。这种设计使得MSRoPE能够在图像端利用分辨率缩放的优势，同时在文本端保持与一维RoPE（1D-RoPE）的功能等效性，从而避免了为文本确定最佳位置编码的需要。

我们在表1中展示了Qwen-Image的架构和配置。

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20250824183109336.png" alt="image-20250824183109336" style="zoom:50%;" />

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20250824183134348.png" alt="image-20250824183134348" style="zoom:50%;" />

### Data

#### Data Collection
