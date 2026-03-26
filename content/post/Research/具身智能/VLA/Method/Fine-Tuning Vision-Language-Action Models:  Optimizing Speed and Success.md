## TLDR

论文主要针对VLA任务中对LLM的微调策略进行研究，作者以OpenVLA为基础模型，探索并对比了几种设计方式（其中原版的OpenVLA为 autoregressive + discrete + next-token prediction）：

- action decoding 方式：autoregressive vs. parallel generation
- action的表示：discrete vs. continuous
- 学习目标：next-token prediction vs. L1 regression vs. diffusion

作者最终得到优化后的微调策略，包含四个关键设计：

- **并行解码架构**：提升推理效率
- **Action Chunking**：增强时序动作的连贯性
- **连续动作表示**：提高动作空间灵活性
- **L1回归目标函数**：简化学习目标，优化训练稳定性



## Metadata

- **发表期刊/会议**：arXiv

- **论文作者**：Moo Jin Kim1 Chelsea Finn1 Percy Liang1

- **研究机构**：Stanford

- **论文链接**：https://arxiv.org/abs/2502.19645

- **关键词**：Vision-Language-Action Models, Fine-Tuning, Robot Learning, Real-time Control 

- **Code & Dataset & Weight：** https://github.com/moojink/openvla-oft?tab=readme-ov-file

- **BibTeX**：

- ```
  @article{kim2025fine,
    title={Fine-Tuning Vision-Language-Action Models: Optimizing Speed and Success},
    author={Kim, Moo Jin and Finn, Chelsea and Liang, Percy},
    journal={arXiv preprint arXiv:2502.19645},
    year={2025}
  }
  ```

  

## Problem Definition

### **研究问题**

尽管预训练的 VLA 模型（如 OpenVLA）在泛化性上表现出色，但在适配新机器人或新任务时，现有的微调策略（通常沿用预训练时的逐 Token 自回归预测）推理速度极慢（仅 3-5 Hz），且在复杂的双臂操作中表现不佳 。

### **形式化定义**

**形式化定义**：

- **输入**：多视角图像（三方视角 + 腕部视角）、机器人本体状态（Joint Angles）、语言指令 。
- **输出**：连续动作块（Action Chunks），即未来 $K$ 个时间步的 $D$ 维动作序列 。

### **价值与意义**

使庞大的视觉语言模型（7B 参数量级）能够部署在需要高频反馈（25-50+ Hz）的真实机器人任务中，且无需额外的低级控制器 。

## Challenges

### **核心挑战**

自回归生成固有的延迟。OpenVLA 原本生成 1 个时间步的动作需要 7 次串行推理，生成一个 $K$ 步长的动作块会导致延迟呈 $K$ 倍增长，无法满足实时性。

## Methodology

### **实现细节**

- OpenVLA基于 Prismatic 7B VLM 为基础，采用自回归解码和离散动作表示，有以下问题：

1. **推理效率低**：自回归解码导致动作生成延迟高（单步0.33秒），无法支持高频控制（25-50+ Hz）。
2. **Action Chunking困难**：自回归生成计算量较大，若需预测未来K个时间步的动作，自回归方式需进行 K×D 次forward，难以实现实时Action Chunking。
3. **语言指令跟随弱**：多视角输入下模型易受视觉信息干扰，忽视语言指令关键信息。
4. **双臂任务性能较差**

因此作者尝试针对这些问题进行改进：

- **Parallel decoding：**将自回归模型的因果注意力（causal attention）掩码改为双向注意力（允许所有token间相互关注），解除生成顺序限制，在输入中预先插入“空Action Embeddings”作为占位符，模型通过一次前向传播直接得到完整动作序列
- **连续动作表示：**原版OpenVLA将连续动作离散化为256个bin。作者通过将离散动作的softmax分类头替换为MLP动作头即可实现连续动作表示。MLP的输入为LLM最后一层Hidden States，输出为D维连续的动作。如果是Diffusion形式则输出noise的预测。
- **额外的输入输出：**原版OpenVLA只支持单相机图像输入且没有state的输入，作者改进了pipeline，对于多个相机图像都用相同的vision encoder编码，对于state的输入也是直接投影到LLM的embedding space即可。
- **改进版FiLM：**VLA中常常会遇到机器人没有follow语言指令的问题，在多个视角图像输入情况下更明显。这是因为模型因视觉输入中的伪相关性干扰（如物体位置、背景纹理等无关特征），并且任务进行整个过程中只有小部分时间动作与语言指令强相关。因此模型常常会直接无视语言指令，仅condition on视觉输入。针对这个问题，作者FiLM来强化language following。作者将FiLM中的γ和β作用于一个图像特征图的所有patch同一通道维度，以实现全局语义对齐。FiLM插入在视觉Encoder每个块的自注意力层后、FFN层前。FiLM是嵌入在网络中的变换层，根据语言输入将embedding空间进行变换，从而实现以语言为条件的输出。FiLM早在2017年就被提出并广泛使用。RT-1中有用到过FiLM，图像视频生成中经典的DiT使用的adaLN层也是类似的思想。



## Experiments

### 仿真实验

- **实验环境和任务**：作者在 LIBERO 仿真环境进行实验（Franka机械臂），包含4个task suites，评测空间布局、物体、目标、长时程任务上的泛化能力。
- **数据**：每个suites包含10个任务500条轨迹。
- **对比方法**：原版OpenVLA、Diffusion Policy、Octo、DiT Policy、Seer、MDT、pi0。
- **改进方案**：Parallel decoding+Action Chunking、连续动作（L1回归/扩散模型）。
- **输入输出**：单第三视角图像+语言指令，部分实验增加腕部相机和机器人状态输入。
- **实验结论：**
  - Parallel decoding+Action Chunking：成功率提升14%（平均76.5%→ 90.2%）。
  - 连续动作：成功率再提升5%（95.3%），L1回归与扩散模型（95.4%）非常接近。指标超过除了pi0以外的所有baseline方法。
  - 输入扩展（多视角+状态输入）：OpenVLA-OFT平均成功率97.1%，超越pi0的94.2%。
  - 推理效率：Parallel decoding使效率提升4倍（4.2 Hz → 15.9 Hz），加上Action Chunking后可达到100Hz，使用Diffusion则降低为10Hz



### 真机实验

- **实验环境和任务**：在ALOHA平台上进行实验，双臂ViperX 300 S，25 Hz控制频率，任务包括双臂叠衣服/短裤、用勺子舀取xx、把xx放入锅里，测试形变物体、长程任务、工具使用、指令follow的能力
- **数据**：
  - 折叠短裤20条， 评估基础折叠动作成功率
  - 折叠T恤30条， 评估长程任务连贯性
  - 用勺子舀取xx， 3种物体共45条 ，考察工具使用能力
  - 把xx放入锅里，3种物体共300条，考察指令follow的能力，以及out-of-distribution的泛化能力
- **对比方法**：原版OpenVLA、在双臂数据上预训练过的 RDT-1B 和pi0，另外还有ACT和Diffusion Policy从头训练
- **改进方案**：OpenVLA-OFT+，即Parallel decoding+Action Chunking+改进版FiLM
- **输入输出**：3个摄像头（头部+腕部）输入和14维关节状态输入。
- **实验结论：**
  - ACT < Diffusion Policy < RDT-1B < pi0 < OpenVLA-OFT+
  - 去掉FiLM之后成功率大幅降低，甚至低于ACT+FiLM的性能，因此证明FiLM确实可以提高指令follow的能力
  - 推理效率：ACT的频率最高（400Hz）延迟最低，pi0接近300Hz，OpenVLA-OFT+和RDT1B接近（80Hz左右，延迟0.3s左右）

## Summary & Evaluation

这篇论文创新点较少，主要贡献是验证了一些已知的结论。论文实验中的一些有趣的点值得讨论和思考：

- 模型输出action的延迟问题：真机测试中OpenVLA-OFT+和RDT1B高达0.3s，这意味着模型输出action的时刻对应0.3s以前的观测，导致很难完成含有动态的任务，只有当物体都是静止的时候才能借助Action Chunking提高等效的输出频率。相比之下原版的ACT0.058s的延迟就可以满足动态任务的需要。
- 语言指令follow的能力：实验中专门考察了模型的指令follow的能力。VLA中不follow指令的原因可能是视觉相关的token数量远远大于language token数量，导致模型更有可能学会让action输出condition on视觉输入，这篇论文提到的FiLM可以以较小的计算代价提升模型指令follow能力，如果实践中遇到此类问题除了提高数据数量和质量外也可以考虑这种方案

