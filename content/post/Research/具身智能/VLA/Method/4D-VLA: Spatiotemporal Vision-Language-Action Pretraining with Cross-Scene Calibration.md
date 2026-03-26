## TLDR

**4D-VLA** 通过集成 **3D 空间坐标嵌入**与**自适应记忆库采样（MBS）**，有效解决了跨数据集预训练中的坐标系混乱与动作歧义问题。将具身智能从 2D 视觉提升至 **4D 时空表征**（RGB-D + 历史帧选择），并提出 **MV-Bench** 填补了视角泛化性评估的空白。在 LIBERO-LONG 长时程任务中比 **OpenVLA 成功率提升了 25.4%**；在 MV-Bench 跨视角评估中表现出极强的鲁棒性。

## Metadata

- **发表期刊/会议**：NeurIPS 2025 poster
- **论文作者**：Jiahui Zhang1∗ Yurui Chen1∗ Yueming Xu1 Ze Huang1 Yanpeng Zhou2 Yu-Jie Yuan2 Xinyue Cai2 Guowei Huang2 Xingyue Quan2 Hang Xu2 Li Zhang1†
- **研究机构**：1 School of Data Science, Fudan University 2 Huawei Noah’s Ark Lab
- **论文链接**：https://arxiv.org/abs/2506.22242
- **关键词**：Vision-Language-Action (VLA), 4D Representation, RGB-D, Spatiotemporal Reasoning, Embodied AI
- **Code & Dataset & Weight：** https://github.com/LogosRoboticsGroup/4D-VLA
- **BibTeX**： 
- ```
  @article{zhang2025vla,
      title={4D-VLA: Spatiotemporal Vision-Language-Action Pretraining with Cross-Scene Calibration},
      author={Zhang, Jiahui and Chen, Yurui and Xu, Yueming and Huang, Ze and Zhou, Yanpeng and Yuan, Yujie and Cai, Xinyue and Huang, Guowei and Quan, Xingyue and Xu, Hang and Zhang, Li},
      year={2025},
      journal={arXiv preprint arXiv:2506.22242},
  }
  ```

  

## Problem Definition

### **研究问题**

如何利用大规模、异构的机器人数据集进行高效预训练，同时解决**坐标系混乱（Coordinate System Chaos）\**和\**状态混乱（State Chaos）**？

### **形式化定义**

**输入**：语言指令 $l$，以及序列化的 RGB-D 图像 $\{I_{t-n}, \dots, I_t\}$。

**输出**：预测动作 $a_t = [\Delta x, \Delta \theta, g]$，包含平移、旋转偏移量和夹爪状态。

**核心逻辑**：学习函数 $F_{\theta}(input) \approx A_t(input)$。

## Challenges

### **核心挑战**

**空间不一致性**：不同数据集的动作定义在不同的坐标系（底座坐标、相机坐标等），导致模型难以对齐动作与视觉。

**动作歧义性**：单帧 RGB 无法区分相似状态下的不同意图（如抓取前 vs 抓取后），也无法感知深度，导致操作精度低。？

### **本文针对性解决的挑战**

通过 4D 信息（3D 坐标嵌入 + 历史序列）攻克**输入信息不足导致的分布方差过大**问题。

## Angle & Motivation

### **切入角度**

**升维。** 从传统的 2D 图像输入切换到具备空间锚点的 4D 时空 Token。

## Methodology



<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260126012913962.png" alt="image-20260126012913962" style="zoom: 67%;" />

### **实现细节**



<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260126012939071.png" alt="image-20260126012939071" style="zoom:67%;" />

##### **空间感知视觉 Token（Spatial-aware Tokens）**

利用 RGB-D 数据，通过相机内参 $K$ 和外参 $[R|T]$ 进行反投影：

$$P_w(\cdot, u, v) = R \cdot D(u, v) \cdot K^{-1} \begin{bmatrix} u \\ v \\ 1 \end{bmatrix} + T$$

将得到的 3D 世界坐标 $P_w$ 编码为空间位置嵌入 $E_S$，并与视觉特征 $E(I)$ 逐元素相加。

##### **记忆库采样（Memory Bank Sampling, MBS）**

这是一种自适应采样算法（算法 1）：

- **逻辑**：维护一个固定容量 $k$ 的队列，计算新历史帧与库内帧的特征相似度。
- **策略**：剔除冗余（相似度高）的帧，保留差异化（相似度低）的帧。
- **目的**：用极小的计算开销获取最大跨度的有效历史上下文。

##### **时间位置编码（Temporal Encoding）**

采用 **Concat（拼接）** 方式处理非均匀采样后的时间偏移：

$$X = \bigcup_{i \in H} [e_{T, i} | e_{ST, i}] \cup \{e_{text}\}$$

每个 Token 对包含显式的时间戳信息。

##### **损失函数优化**

引入了**方向损失 $L_d$**，增强模型对位移方向的敏感度：

$$L = L_{trans} + L_{rot} + L_{grip} + \lambda_d \|d(\hat{\Delta x}) - d(\Delta x)\|^2$$





<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260126013002331.png" alt="image-20260126013002331" style="zoom:67%;" />

## Experiments

### **实验设置与指标**

**数据集**：DROID（预训练）、LIBERO（微调/仿真）、Real-world（实测）。

**指标**：成功率（Success Rate）。

### 对比实验



<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260126013024941.png" alt="image-20260126013024941" style="zoom:67%;" />

**LIBERO-LONG**：比 OpenVLA 提升 **25.4%**，体现了处理长序列任务的绝对优势。



![image-20260126013049625](/Users/yangchao/Library/Application Support/typora-user-images/image-20260126013049625.png)

**MV-Bench（自建基准）**：在 In-View 和 Cross-View 下全面领先，成功率高出 OpenVLA **28.8%**。



真实世界多视角评估：

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260126013132691.png" alt="image-20260126013132691" style="zoom:67%;" />

### 消融实验



![image-20260126013105123](/Users/yangchao/Library/Application Support/typora-user-images/image-20260126013105123.png)

**组件贡献**：空间位置编码对**精确放置**（Task 3）至关重要；MBS 采样对**长程任务**（Task 4）贡献最大。

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260126013214382.png" alt="image-20260126013214382" style="zoom:67%;" />



**采样策略**：实验证明，增加历史窗口 $n$ 比增加采样帧数 $k$ 更能提升性能。





时间编码方法的消融实验：

![image-20260126013242045](/Users/yangchao/Library/Application Support/typora-user-images/image-20260126013242045.png)

## Summary & Evaluation

### **总体评价**

它没有盲目追求更大的 LLM 底座，而是解决了具身智能中最底层的**数据表征一致性**问题。

### **值得 Follow 的点**

**3D Token 融合方案**：这种将 Depth 转化为 Positional Embedding 的方式非常优雅且高效。

**MBS 抽帧算法**：在处理视频数据流时，这种算法可以作为通用的性能优化模块。

