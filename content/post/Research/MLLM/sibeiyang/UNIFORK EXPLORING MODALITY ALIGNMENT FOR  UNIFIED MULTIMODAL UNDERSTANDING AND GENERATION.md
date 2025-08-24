---
title: UNIFORK EXPLORING MODALITY ALIGNMENT FOR  UNIFIED MULTIMODAL UNDERSTANDING AND GENERATION
date: 2022-09-15T11:30:03+00:00
tags:
  - MLLM
  - LLM
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

在多模态人工智能领域，统一的图像理解与生成已成为一种前景广阔的新范式。尽管近期取得了一些进展，但这类统一模型的最佳架构设计仍是一个开放性挑战。

在这项工作中，我们首先分析了用于理解和生成的任务专属专家模型以及当前统一模型的模态对齐行为。我们的分析揭示了一个关键的观察结果：**理解任务**受益于随网络深度逐渐增强的模态对齐，这有助于构建语义信息以实现更好的理解；相反，**生成任务**遵循一种不同的趋势——模态对齐在浅层网络中增强，但在深层网络中减弱，以恢复空间细节。

这些存在差异的对齐模式在完全共享的 Transformer 主干网络中造成了根本性的冲突，其中统一的表示流通常会导致两种任务之间的性能折衷。

受此发现启发，我们引入了 **UniFork**，这是一种新颖的 **Y 形架构**，它共享浅层网络以进行跨任务表示学习，同时在深层网络中采用任务专属的分支以避免任务间的干扰。这种设计有效地平衡了共享学习和任务专业化。通过广泛的消融实验，我们证明了 Unifork 的性能持续优于传统的完全共享 Transformer 架构，并且其性能达到或超过了任务专属模型。

我们的代码已在 https://github.com/tliby/UniFork 开源。



### INTRODUCTION

近期的工作（Xie et al., 2024; Li et al., 2025a; Deng et al., 2025; Zhang et al., 2025）在统一的多模态生成与理解方面取得了显著进展。通过将语言和视觉信号都投射到一个共享的嵌入空间，并以不同方式进行排列，在单一的 Transformer 架构内同时执行图像理解和生成任务变得可行。然而，尽管共享这样一个通用范式，生成和理解任务的目标本质上是不同的（Wu et al., 2025; Chen et al., 2025b）。图像生成强调视觉输出的保真度和美学质量，专注于像素级别的细节，如纹理和颜色。相反，图像理解则侧重于高层次的语义理解，例如识别物体、解释空间关系和对场景内容进行推理。这种根本性的差异使得统一这两项任务变得极具挑战性。

为了解决任务间的差异问题，一些近期的方法（Wu et al., 2025; Chen et al., 2025b）分别采用了为理解和生成任务量身定制的、不同的语义和空间图像表示。其他方法则引入了扩散优化目标（Xie et al., 2024; Zhou et al., 2024）或外部模型（Ge et al., 2024a; AI et al., 2025）来为图像生成解码空间特征。尽管这些设计可以提升特定任务的性能，但它们往往破坏了大型语言模型（LLM）中原始的“下一词元预测”（Next-Token Prediction, NTP）范式的简洁性和优雅性。此外，在监督微调（Supervised Fine-Tuning, SFT）过程中，通常需要进行细致的数据平衡以维持各项任务的性能。再者，生成与理解之间的内在关系在很大程度上仍未被探索，这引发了关于这些任务如何在统一框架内相互补充的重要问题。

在这项工作中，我们从图像和语言词元（token）之间特征对齐的视角，研究了图像理解与生成之间的关系。我们发现这两项任务表现出截然不同的对齐模式：图像理解受益于随网络深度逐渐增强的对齐以构建语义表示，而图像生成则依赖于浅层的强对齐以及随后在深层减弱的耦合，以实现细粒度的视觉合成。此外，在 NTP 建模范式下采用完全共享的 Transformer 主干网络，会迫使两种任务在表示上做出折衷。这些发现强调了在设计统一模型时，考虑理解和生成任务的不同对齐模式以实现两项任务最佳性能的重要性。

基于这一观察，我们提出了 **UniFork**，一种用于统一图像理解与生成的 **Y 形架构**。具体来说，Transformer 主干网络的浅层在两个任务间共享，以实现跨任务的语义学习。在深层网络中，我们引入了任务专属的分支——两个结构相同但参数独立的模块。理解分支用于提炼语义表示，而生成分支则用于重建空间细节。通过在深层解耦特定任务的表示学习，UniFork 有效地缓解了由不同对齐模式引起的表示冲突。UniFork 的另一个优势在于其训练的灵活性。在最终的 SFT 阶段，任务专属的参数可以使用各自的数据集独立优化，从而无需进行精细的数据比例调整。为了验证我们设计的有效性，我们进行了广泛的消融研究，结果表明 UniFork 的性能优于完全共享的架构，并达到了与任务专属专家模型相当的水平。此外，基于 Qwen2.5-0.5B LLM（Yang et al., 2025）进行适度扩展后，UniFork 的性能超过了在相似规模上训练的当前最先进的统一模型。

我们的主要贡献总结如下：

- 我们分析了专家模型中特定任务的模态对齐模式，强调了图像理解和生成的不同需求，并为统一模型的设计提供了见解。
- 我们提出了 UniFork，一种 Y 形架构，它在深层解耦了特定任务的学习，同时在浅层保留了共享的语义表示学习。这种设计实现了有效的跨任务学习，并缓解了任务间的性能冲突。
- 全面的消融研究表明，UniFork 的性能优于完全共享的 Transformer 架构。通过适度扩展，我们的方法在理解和生成任务上都取得了显著的性能提升。



### METHOD

#### OBSERVATION AND ANALYSIS

近期的工作致力于在单一框架内统一图像理解与生成，并提出了多种架构。然而，很少有工作研究这两项任务之间的内在关系。在本研究中，我们从**模态对齐**的视角来探究它们的差异。

给定一张图像 $X$ 及其对应的文本提示 $T$，我们将第 $l$ 个 Transformer 层提取的视觉特征表示为 $V_{\text{gen}}^l \in \mathbb{R}^{n_v \times c}$，并将最后一个 Transformer 层的文本提示特征表示为 $T \in \mathbb{R}^{n_t \times c}$。其中，$n_v$ 和 $n_t$ 分别代表视觉和文本词元（token）的数量，$c$ 是特征的通道大小。对于**生成任务**，我们从 Geneval (Ghosh et al., 2023) 数据集中采样了 500 个提示。在每一层 $l$，我们使用 mutual k-nearest neighbors (mutual-kNN) 计算模态对齐分数 $A_{\text{gen}}^l$，这是一种常用于评估表示对齐的指标 (Huh et al., 2024)：
$$A_{\text{gen}}^l = \text{mutual-kNN} \left( \left( \frac{1}{n_v} \sum_{i=1}^{n_v} V_{\text{gen}}^{l,i}[b] \right)_{b=1}^{500}, \left\{ \frac{1}{n_t} \sum_{j=1}^{n_t} T_j[b] \right\}_{b=1}^{500} \right).$$
对于**理解任务**，我们将生成的图像连同查询“为这张图片提供一句单句描述：（Provide a one-sentence caption for the image:）”一起输入模型。我们从每一层提取视觉特征 $V_{\text{und}}^l$，并计算 $V_{\text{und}}^l$ 与其对应提示特征之间的对齐分数：
$$A_{\text{und}}^l = \text{mutual-kNN} \left( \left( \frac{1}{n_v} \sum_{i=1}^{n_v} V_{\text{und}}^{l,i}[b] \right)_{b=1}^{500}, \left\{ \frac{1}{n_t} \sum_{j=1}^{n_t} T_j[b] \right\}_{b=1}^{500} \right).$$

##### 生成与理解中存在差异的对齐模式

<img src="/Users/yangchao/Downloads/Fig 2 Page 1.jpg" style="zoom:24%;" />

利用这一分析工具，我们首先获取了在分别针对生成和理解任务训练的专家模型 (Sun et al., 2024; Liu et al., 2024b) 中的对齐模式。如图 2(a) 所示，我们观察到在**生成任务**中，对齐分数在浅层网络中增加，但在深层网络中减少。这一趋势与 REPA (Yu et al., 2024) 研究在扩散模型上的观察结果一致，表明浅层网络专注于跨模态对齐和语义基础的构建，而深层网络则负责合成高频的视觉细节。相反，如图 2(b) 所示，**理解任务**在所有层中都表现出递增的对齐分数，这表明深层网络中的强跨模态对齐对于准确理解至关重要。这些发现揭示了这两项任务需要根本上不同的对齐行为。

##### NTP 范式下完全共享主干网络的表示折衷

接着，我们研究了 Emu3-base (Sun et al., 2023b)，这是一个在两项任务上联合预训练的原生多模态模型。如图 2(c) 所示，生成和理解任务的对齐曲线几乎重叠，都遵循一种**先增后减**的模式。这表明在联合训练中，理解任务（的表示模式）可能为了适应生成任务的目标而做出了妥协。为了验证这一点，我们分析了两个从 Emu3-base 微调而来的任务专属变体：Emu3-Gen (Sun et al., 2023b) 和 Emu3-Chat (Sun et al., 2023b)。有趣的是，如图 2(d) 所示，Emu3-Chat 恢复了理解任务所特有的**单调递增**的对齐趋势，而 Emu3-Gen 则保留了生成任务典型的**先升后降**模式。这进一步支持了我们的假设：这两项任务偏好不同的对齐动态，并且在 NTP 范式下简单地共享主干网络可能会导致表示上的冲突。

受这些观察的启发，我们提出了一种 **Y 形架构**，该架构共享浅层网络以进行联合语义学习，并解耦深层网络以适应特定任务的对齐需求。



#### ARCHITECTURE

UniFork 的整体架构如图 3 所示，它能够在统一的框架内同时实现跨任务学习和任务专属的特化。

<img src="/Users/yangchao/Pictures/UniFork Method.png" alt="0." style="zoom:22%;" />

##### **视觉分词器 (Visual Tokenizer)**

为了保持架构的简洁性，我们对理解和生成任务采用了同一个图像分词器（tokenizer）。我们早期的探索性实验表明，基于 VAE 的分词器在有限规模的训练下表现不佳，这与先前工作（Xie et al., 2024）的观察结果一致。因此，我们利用了 VILA-U（Wu et al., 2024b）中提出的分词器，该分词器在保持图像重建质量的同时，增强了文本与图像的对齐。给定一张输入图像，该分词器会将其以 16×16 的系数进行压缩，将得到的二维特征展平为一维的词元（token）序列，并在将这些词元送入语言模型之前，先通过一个轻量级的 MLP（多层感知机）。

##### **Transformer 主干网络 (Transformer Backbone)**

受我们对齐分析的启发，UniFork 采用了一种 **Y 形的 Transformer 架构**。给定一个总共有 (M + N) 层的 Transformer，前 M 层在两个任务间共享，以支持联合的语义表示学习。剩下的 N 层包含两个任务专属的分支：一个专用于图像理解的语义强化，另一个则专注于图像生成的空间细节重建。我们使用 Qwen2.5-0.5B LLM（Yang et al., 2025）的权重来初始化整个主干网络。值得注意的是，当 N = 0 时，UniFork 退化为具有完全参数共享的 Emu3（Wang et al., 2024）架构；当 M = 0 时，其结构变得与最近在 BAGEL（Deng et al., 2025）中提出的“混合 Transformer”（Mixture-of-Transformers）设计相似。

##### **生成视觉头 (Generation Vision Head)**

由于该图像分词器（Wu et al., 2024b）使用了残差矢量量化（residual vector quantization）方法（Lee et al., 2022）将每个词元映射为多个离散编码，我们引入了一个图像头（image head）来预测这些编码。这个头接收来自 LLM 最后一层的输出特征，并以自回归（autoregressively）的方式为每个词元生成编码。

#### TRAINING PIPELINE

如图 4 所示，整体的训练过程可分为三个阶段：

<img src="/Users/yangchao/Downloads/Fig 4 Page 1.jpg" alt="Fi" style="zoom:24%;" />

##### **第一阶段：视觉对齐预训练 (Visual Alignment Pretraining)**

此阶段的目标是将视觉表示与预训练的 LLM 对齐。遵循先前的工作（Xie et al., 2024; Chen et al., 2025b），我们首先在 ImageNet-1K (Deng et al., 2009) 数据集上训练模型，以高效地捕捉像素级别的依赖关系。我们使用成对的图像和文本描述来构建学习任务，其中类别名称通过使用 OpenAI ImageNet 模板 (Radford et al., 2021) 被转换为自然语言提示。该数据被用于训练**图像字幕生成 (image captioning)** 和**文本到图像生成 (text-to-image generation)** 两个任务。随后，我们使用来自 Laion-En (Schuhmann et al., 2022) 的 3000 万样本和来自 COYO (Byeon et al., 2022) 的 1000 万样本的混合数据，对这两个任务进行训练。在此阶段，LLM 的权重被**冻结**，我们只训练随机初始化的视觉连接器 (visual connector) 和图像头 (image head)。生成任务遵循 `“<caption><image>”` 的格式，而字幕生成任务则使用 `“<image><caption>”` 的格式。

##### **第二阶段：联合优化 (Joint Optimization)**

此阶段旨在增强模型在图像理解和生成两方面的综合能力。我们**解冻** LLM，并联合优化主干网络、视觉连接器和图像头。对于多任务预训练，我们使用了来自 JourneyDB (Sun et al., 2023a)、SAM (Kirillov et al., 2023)、Unsplash (Unsplash, 2020) 以及一个内部数据集的 3250 万个图文对用于生成任务，并使用了 InternVL-1.5 (Chen et al., 2024b) 预训练数据的一个 1650 万的子集用于理解任务。

接着，我们进行**指令微调 (instruction tuning)**。在生成任务上，我们从 3250 万的数据集中采样一个子集，并与 BLIP3o-60k (Chen et al., 2025a) 数据集结合，总计 500 万样本。在理解任务上，我们使用 InternVL-1.5 SFT 数据集的一个 380 万的子集。生成任务的格式为：`“USER: <Input Message> ASSISTANT: <Response>”`。对于理解任务，我们采用基线 SFT 的对话格式 (Yang et al., 2025)。

##### **第三阶段：任务专属微调 (Task-Specific Fine-Tuning)**

UniFork 架构的一个重要优势是其优化的灵活性。在联合训练之后，我们通过**独立的微调**进一步提升特定任务的性能。在此阶段，只有任务专属的层会被更新，而所有共享组件保持冻结。我们复用第二阶段的指令微调数据集，并独立地对理解和生成分支进行微调。这最后一个阶段使得模型能够在不引入干扰的情况下专精于各项任务，从而有效地平衡了共享语义表示和任务专属优化。

#### TRAINING OBJECTIVE

UniFork 以自回归的方式对视觉和文本词元（token）进行建模。因此，我们对两个任务均采用交叉熵损失（cross-entropy loss），并且没有引入任何任务专属的损失权重：

$$L_{\text{total}} = - \sum_{i=1} \log P(\hat{x}_i = x_i | x_{<i}) \quad (1)$$

其中 $P$ 表示由 UniFork 网络建模的概率分布。$\hat{x}_i$ 和 $x_i$ 分别代表预测的词元和真实的词元（ground truth）。

-   对于**图像生成任务**，损失仅在**视觉词元**上计算。
-   对于**图像理解任务**，损失仅在文本词元的**回复部分**上计算。

### EXPERIMENTS

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20250824124938927.png" alt="image-20250824124938927" style="zoom:50%;" />

在本节中，我们将展示全面的实验来评估所提出的 UniFork 架构。我们首先介绍实验设置（第 4.1 节），随后是通过消融研究来证明 Y 形架构的有效性（第 4.2 节）。在这些见解的指导下，我们适度地扩展了模型和数据规模，并将 UniFork 在图像理解和生成任务上与专家模型及近期的统一模型进行比较（第 4.3、4.4 节）。最后，我们分析了 UniFork 的模态对齐模式（第 4.5 节）。

#### EXPERIMENT SETUP

##### **实现细节 (Implementation Details)**

我们使用 Qwen2.5-0.5B LLM (Yang et al., 2025) 来初始化 UniFork 的 Transformer 主干网络。Transformer 的后半部分层被复制，以分别为图像理解和生成任务构建两个独立的分支。主干网络的总参数量为 12.1 亿（1.21B），其中 5 亿（0.5B）为理解任务的活动参数，7.6 亿（0.76B）为生成任务的活动参数。我们采用 VILA-U-256 (Wu et al., 2024b) 的分词器（tokenizer）来为每张图像获取与文本对齐的编码。该分词器的词汇表大小为 16,384，压缩系数为 16×16。

输入图像被调整到 384×384 的分辨率。对于**图像生成**任务，我们将较短的一边调整到 384，并对较长的一边进行中心裁剪。对于**图像理解**任务，我们将较长的一边调整到 384，并用背景色（RGB: 127, 127, 127）填充较短的一边，以形成 384×384 的输入。遵循先前的工作 (Sun et al., 2024; Chen et al., 2025b)，我们在图像生成过程中采用**无分类器指导 (classifier-free guidance)**。具体来说，在训练期间，10% 的输入提示被随机丢弃，并替换为一个特殊的填充词元 (padding token)。在推理期间，指导尺度 (guidance scale) 设置为 4.0，以平衡**保真度 (fidelity)** 和**多样性 (diversity)**。整个训练过程在 16 块 Nvidia A100 GPU 上进行，每个阶段的详细配置总结在表 1 中。

##### **评估基准 (Evaluation Benchmarks)**

为了评估 UniFork 在图像理解和生成两方面的有效性，我们将其与专为各项任务训练的专家模型以及近期的统一模型进行比较。

- **对于图像理解** 🧠，我们在五个广泛采用的基准上进行评估：**MME-P** (Fu et al., 2024)、**POPE** (Li et al., 2023b)、**SEED-I** (Li et al., 2023a)、**VQAv2** (Goyal et al., 2017) 和 **GQA** (Hudson & Manning, 2019)。这些基准共同评估了视觉理解的多个方面，包括**感知 (perception)**、**推理 (reasoning)** 和**定位 (grounding)**。
- **对于图像生成** 🎨，我们使用 **GenEval** (Ghosh et al., 2023) 和 **MJHQ-30K** (Li et al., 2024) 基准。**GenEval** 是一个以物体为中心的基准，它从六个维度评估文本到图像的对齐情况：“单个物体 (single object)”、“两个物体 (two objects)”、“计数 (counting)”、“颜色 (colors)”、“位置 (position)”和“颜色属性 (color attributes)”。而 **MJHQ-30K** 则专注于整体图像质量和视觉美学。它使用 **Fréchet Inception Distance (FID)** 指标来评估生成图像与一个包含 3 万张高质量参考图像的精选集之间的相似度。

#### ABLATION STUDY

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20250824125323065.png" alt="image-20250824125323065" style="zoom:50%;" />

##### **UniFork 架构的有效性 (Effectiveness of UniFork Structure)**

为了验证所提出的 UniFork 架构的有效性，我们使用四种模型变体进行了一项对比研究，所有变体均由 Qwen2.5-0.5B LLM (Yang et al., 2025) 初始化。这些变体包括：

1. **生成专家 (Gen Expert)**：仅在生成数据上训练。
2. **理解专家 (Und Expert)**：仅在理解数据上训练。
3. **完全共享 LLM (Fully Shared LLM)**：对两个任务使用单一的 Transformer 主干网络，并带有一个 7000 万（0.07B）参数的视觉头用于图像生成。
4. **UniFork**：共享 Transformer 的前半部分层，但在后半部分采用独立的任务专属分支。

所有模型都在第一和第二阶段所用数据的一个子集上进行训练，输入图像分辨率设为 256×256。为确保公平比较，我们对每项任务都保持了相同的活动参数数量和训练配置。

如表 2 所示，**UniFork 在图像理解和生成任务上均持续优于“完全共享 LLM”模型**，并且其性能与任务专属的专家模型**相当甚至更优**。这些结果表明，Y 形 Transformer 架构在共享语义学习和任务专属表示之间取得了更有效的**权衡 (trade-off)**。通过解耦后半部分的层，UniFork 减少了任务间的干扰，并实现了有针对性的特征优化 (feature refinement)，从而带来了跨两种模态的整体性能提升。

##### **在更多数据集上的模态对齐分析 (Modality Alignment Analysis on More Datasets)**

在第 3.1 节中，我们使用 Geneval (Ghosh et al., 2023) 基准分析了模态对齐模式。该基准的提示相对较短，且主要集中在物体层面的描述。为避免潜在的数据集特定偏差，我们将对齐分析扩展到了更长的提示，这些提示更侧重于**整体场景描述**和**风格属性**。

我们从 MJHQ-30K (Li et al., 2024) 中随机采样了 500 个提示，平均提示长度从 Geneval (Ghosh et al., 2023) 的 7.3 个词增加到了 32.9 个词。如图 5 所示，观察到的趋势与图 2 中的趋势**保持一致**。具体来说，生成任务的对齐分数在 Transformer 的各层中仍然呈现出**先升后降**的模式，而理解任务的对齐分数则继续**单调递增**。

这些结果进一步支持了我们早前的发现：完全共享一个 Transformer 主干网络可能会导致两种任务之间的**表示折衷 (representational compromise)**。这凸显了 UniFork 架构的必要性，即在一个统一的框架内更好地适应图像生成和理解之间存在差异的对齐行为。

<img src="/Users/yangchao/Downloads/Fig Align Page 1.jpg" style="zoom:24%;" />



#### IMAGE UNDERSTANDING EVALUATION

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20250824125957364.png" alt="image-20250824125957364" style="zoom:50%;" />

如表 3 所示，尽管在图像理解任务上仅使用了 **5 亿（0.5B）** 的活动参数，UniFork 仍在所有相关基准上表现出强劲的性能。与近期的统一模型 Show-1 (Xie et al., 2024)（13亿/1.3B参数）相比，UniFork 在 MME-P 上取得了 **10%** 的相对提升，在 POPE 上取得了 **7.3%** 的相对提升。

值得注意的是，即使与更大规模的、仅用于理解任务的模型（如 MobileVLM (2.7B)、IDEFICS-9B 和 LLaVA (7B)）相比，UniFork 依然具有竞争力。它在 POPE 上的表现与 MobileVLM 相当（85.8 vs. 84.9），并且在 SEEDv1 上优于 IDEFICS-9B（55.2 vs. 45.0）。我们在图 7 中进一步提供了一些定性结果。

这些结果凸显了我们 **Y 形架构**的有效性。它有助于减少任务间的干扰，并使得 UniFork 即使在**参数预算有限**的情况下也能表现出色。

#### IMAGE GENERATION EVALUATION



<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20250824130105396.png" alt="image-20250824130105396" style="zoom:50%;" />



<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20250824130119028.png" alt="image-2025082413" style="zoom:50%;" />



如表 4 所示，UniFork 在 GenEval 上取得了 **46%** 的综合准确率，这比参数规模较小的消融变体提升了 **39%**。值得注意的是，UniFork 的性能超过了以往的、参数规模相似或更大的统一模型（如 LWM (Liu et al., 2024a) 和 Chameleon (Team, 2024)），并且在大多数类别上也超越了几个仅用于生成的基线模型（包括 LDM (Rombach et al., 2022a) 和 LlamaGen (Sun et al., 2024)）。

在 MJHQ-30K（表 5）上，UniFork 取得了 **10.6** 的 FID 分数，比其较小的变体提升了 **35%**。尽管 UniFork 使用的参数要少得多（**7.6亿 vs. 70亿以上**），但这一 FID 分数也超过了之前的统一模型，如 Show-1 (15.18) (Xie et al., 2024) 和 LWM (17.77)。

我们将这些性能提升归因于从消融研究中获得的结构性见解，该研究表明，在统一训练中，**任务专属的分支**对于解决模态对齐冲突至关重要。我们在图 1 和图 6 中进一步提供了一些定性结果。

通过将模型的活动参数从 5.7 亿适度扩展到 7.6 亿，我们在无需进行架构改变的情况下，解锁了显著的性能提升。我们期望通过使用更好的分词器、更大的参数量和更高质量的数据，获得进一步的提升。

#### MODALITY ALIGNMENT ANALYSIS

遵循第 3.1 节中介绍的方法，我们进一步可视化了 UniFork 在图像理解和生成两个任务上的模态对齐模式。

如图 8 所示，**理解任务**的对齐分数随网络深度的增加而**稳定上升** 📈，而**生成任务**的对齐分数则遵循**先升后降** 📉 的趋势。这些趋势与在专家模型中观察到的趋势一致，满足了这两个任务各自不同的表示需求。这一结果为 **UniFork 架构**的有效性提供了进一步的证据。通过解耦 Transformer 主干网络的后半部分层并分配任务专属的参数，模型可以在一个统一的框架内**调和 (reconcile)** 生成与理解任务之间存在差异的需求，而无需在表示质量上做出妥协。

### CONCLUSION

在这篇论文中，我们分析了专家模型以及基于 NTP 的统一模型在图像生成和理解任务中的**模态对齐模式**。我们发现，完全共享一个 Transformer 主干网络可能会导致任务间的干扰。受此发现启发，我们提出了 **UniFork 架构**，它共享浅层网络，并解耦深层网络以进行任务专属学习。消融研究验证了该设计的有效性。通过适度的扩展，UniFork 在两项任务上都取得了强劲的性能，展示了其作为未来统一多模态模型基线的潜力。

#### **局限性 (Limitations)** 🧐

尽管 UniFork 表现出强劲的性能，但它仍受三个主要因素的制约：**视觉分词器的质量**、**相对较小的模型规模**，以及**训练数据有限的质量**。这些局限性在图像生成任务中尤为突出。

具体而言，UniFork 中使用的分词器是在 256 分辨率下使用 Vision Transformer 编码器训练的，而模型则在 384 分辨率下运行，这可能导致空间上的不匹配。在这些方面中的任何一个进行改进，都有可能给整体性能带来显著的提升。

#### **未来工作 (Future Work)** 🚀

尽管 UniFork 有效地平衡了共享表示和任务专属表示，但这两种参数之间的**最佳比例**仍有待探索。这种平衡可能取决于任务的复杂性、训练数据的分布以及整体模型参数。未来的工作还应探索使用交错的视觉-语言数据来扩大训练规模，以更好地释放 UniFork 的**涌现能力**，尤其是在复杂的推理任务上。

此外，UniFork 的“**先共享后分离**”设计为扩展到视觉和语言之外的领域提供了一个灵活的基础。引入如音频、视频或 3D 数据等其他模态，可能会为跨模态对齐动态提供更深刻的见解，并支持开发一个真正的“**任意到任意 (any-to-any)**”的多模态架构。
