## TLDR

**Hi Robot** 模型通过**分层视觉-语言-行动（Hierarchical VLA）架构**和**合成交互数据生成方法**，解决了机器人难以理解开放式指令及实时处理环境反馈的问题。借鉴“系统 1/系统 2”理论，构建了一个高层推理 VLM（生成子指令）与底层执行 VLA（生成动作）的分层控制架构，并通过 VLM 自动回推标注生成大规模合成交互数据集。在平均表现上，Hi Robot 的指令准确率（IA）比 GPT-4o 高出 **40%** 以上，且在任务进度（TP）上显著优于扁平化（Flat）策略模型。该论文为 **VLA 模型研究** 提供了将互联网规模的推理能力与物理执行解耦的范式参考，尤其是其**自动化合成标注**的思路对解决机器人数据稀缺问题极具启发。

## Metadata

- **发表期刊/会议**：arXiv
- **论文作者**：Lucy Xiaoyang Shi、Brian Ichter、Michael Equ
- **研究机构**：Physical Intelligence、Stanford、University of California, Berkeley
- **论文链接**：https://arxiv.org/abs/2502.19417
- **关键词**：Hierarchical VLA, Open-Ended Instruction Following, Situated Feedback, Synthetic Data Generation 
- **Code & Dataset & Weight：**https://github.com/Physical-Intelligence/openpi
- **BibTeX**：
- ```
  - @misc{shi2025hirobotopenendedinstruction,
          title={Hi Robot: Open-Ended Instruction Following with Hierarchical Vision-Language-Action Models}, 
          author={Lucy Xiaoyang Shi and Brian Ichter and Michael Equi and Liyiming Ke and Karl Pertsch and Quan Vuong and James Tanner and Anna Walling and Haohuan Wang and Niccolo Fusai and Adrian Li-Bell and Danny Driess and Lachy Groom and Sergey Levine and Chelsea Finn},
          year={2025},
          eprint={2502.19417},
          archivePrefix={arXiv},
          primaryClass={cs.RO},
          url={https://arxiv.org/abs/2502.19417}, 
    } 
  ```

  

## Problem Definition

### **研究问题**

如何使机器人在真实世界中理解复杂的、多阶段的开放式人类指令（如“做个素食三明治，不要番茄”），并能实时根据环境反馈（如“那个不是垃圾”）动态调整行为 。

### **形式化定义**

系统输入为多视角图像 $I_{t}^{1},...,I_{t}^{n}$、机器人状态 $q_{t}$ 以及开放式语言提示 $l_{t}$ 。输出为连续的动作块 $A_{t} = [a_{t}, ..., a_{t+H-1}]$，通过策略分布 $p(A_{t}|o_{t})$ 进行表示 。

### **价值与意义**

将机器人从只能执行“捡起杯子”等原子化指令，提升到能够进行人类水平的灵活交互与推理。

## Challenges

### **核心挑战**

**语义复杂性**：长程任务（Long-horizon tasks）涉及多步推理、逻辑约束和对环境感知的深层理解。

**数据稀缺性**：带有复杂交互、纠错反馈的真实机器人演示数据极难大规模采集。

### **本文针对性解决的挑战**

重点解决了**复杂指令的泛化理解**与**交互数据的规模化生成**。

## Angle & Motivation

### **切入角度**

借鉴 Kahneman 的 **"System 1 & System 2"** 认知理论 。

- **System 2 (高层推理)**：负责解析复杂语义、处理反馈并拆解目标 。
- **System 1 (底层动作)**：负责快速、自动化的物理执行 。

### **合理性与重要性**

文中通过实验证明，直接训练一个“扁平（Flat）”的端到端 VLA 在处理需要细微特征辨识和排除逻辑的任务时表现极差 。

### **创新性**

打破了“一味追求端到端”的范式，证明了在机器人领域，适度的**分层解耦**辅以**大规模 VLM 驱动的数据增强**能释放出比纯扩大模型规模更强的适应性 。

## Methodology

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260123141807476.png" alt="image-20260123141807476" style="zoom:50%;" />



<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260123141826709.png" alt="image-20260123141826709" style="zoom:50%;" />

### **实现细节**

**分层推理框架**：

- **High-Level Policy (VLM)**：基于 PaliGemma-3B，接收图像和复杂 Prompt，产生中级原子语言指令（Atomic Commands）及语音回复 。
- **Low-Level Policy (VLA)**：基于 $\pi_{0}$ 模型，将中级指令转化为高频动作流 。

**数据生成流**：收集人类遥操作数据并分段，利用 SOTA VLM（如 GPT-4V 或相似模型）“反向想象”出可能触发该动作的复杂指令、约束或纠错，从而构建合成数据集 $\mathcal{D}_{syn}$ 。

### **逻辑闭环**

该方法通过高层 VLM 继承了互联网规模的常识，完美弥补了机器人原生数据对复杂语言理解能力的不足。

### **性能提升的本质**

在于**将推理负担从物理控制层剥离**。高层模型可以使用更强的语言推理能力（即使频率较低），而底层模型专注于执行被“翻译”好的简单指令 。

## Experiments

### **实验设置与指标**

**Benchmark**：餐桌清理（Table Bussing）、三明治制作（Sandwich Making）、超市购物（Grocery Shopping）。

**指标**：指令准确率（Instruction Accuracy, IA）和任务进度（Task Progress, TP）。

### 对比实验



<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260123142910343.png" alt="image-20260123142910343" style="zoom:50%;" />

对比了 Flat VLA（无层级）、GPT-4o 直接驱动（无特定领域微调）以及人类专家引导（Oracle）。结果显示 Hi Robot 在 IA 和 TP 上全面超越基座 VLM 方案 。

### 消融实验

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260123142931635.png" alt="image-20260123142931635" style="zoom:50%;" />

移除合成数据后，模型的 IA 崩塌式下降，证明了**合成交互数据**是使模型具备“听得懂话”的能力的核心驱动力 。

## Summary & Evaluation

### **值得 Follow 的点**

**合成数据范式**：利用 LLM/VLM 对机器人轨迹进行反向 Prompt 标注的方法非常值得借鉴。

**异步运行频率**：高层策略每秒运行一次或按需触发，底层高频运行，这种异步机制在实时系统设计中很实用 。

