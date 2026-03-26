## TLDR

**ActiveVLA** 通过 **Coarse-to-fine（由粗到精）** 的主动感知框架，解决了现有 VLA 模型因静态视角导致的**遮挡**与**分辨率不足**问题。引入了基于 3D 点云的**主动视角选择（Active View Selection）**与**虚拟 3D 缩放（Active 3D Zoom-in）**机制，使机器人具备了“想看哪里看哪里”的能动性。在 **RLBench** 上达到了 **91.8%** 的 SOTA 成功率；在 **COLOSSEUM** 鲁棒性测试中以 **65.9%** 领跑。



## Metadata

- **发表期刊/会议**：arXiv
- **论文作者**：Zhenyang Liu1,2, Yongchong Gu1, Yikai Wang3,* Xiangyang Xue1,† Yanwei Fu1,
- **研究机构**：Fudan University 2Shanghai Innovation Institute 3Nanyang Technological University
- **论文链接**：https://arxiv.org/html/2601.08325v1
- **关键词**：
- **Code & Dataset & Weight：** https://zhenyangliu.github.io/ActiveVLA/
- **BibTeX**：
- ```
  @misc{liu2026activevlainjectingactiveperception,
        title={ActiveVLA: Injecting Active Perception into Vision-Language-Action Models for Precise 3D Robotic Manipulation}, 
        author={Zhenyang Liu and Yongchong Gu and Yikai Wang and Xiangyang Xue and Yanwei Fu},
        year={2026},
        eprint={2601.08325},
        archivePrefix={arXiv},
        primaryClass={cs.RO},
        url={https://arxiv.org/abs/2601.08325}, 
  }
  ```

  

## Problem Definition

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125033633672.png" alt="image-20260125033633672" style="zoom:67%;" />

### **研究问题**

如何提升机器人由于视角受限而在复杂、精细化长时程任务中的操作成功率？

### **形式化定义**

**输入**：多视角 RGB-D 图像 $o$ + 自然语言指令 $l$。

**输出**：由 6-DoF 位姿 $T$、夹爪状态 $g$ 和碰撞检测 $c$ 组成的连续动作序列。

## Challenges

### **核心挑战**

**感知的被动性**。静态相机在手部接近物体时必然产生遮挡，且固定分辨率无法兼顾“全局语义”与“局部细节”。

## Angle & Motivation

### **切入角度**

**主动视觉（Active Vision）**。借鉴心理学家 Richard Gregory 的“感知即假设检验”理论，将机器人从“被动观察者”转变为“主动信息搜集者”

### **创新性**

它打破了“VLA = 大模型推理 + 静态图输入”的固有范式，引入了**几何约束**下的反馈调节机制。

## Methodology

### **实现细节**

![image-20260125033619309](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125033619309.png)



#### 1. 粗略阶段 (Coarse Stage)：关键区域定位

- **7 通道渲染**：将 3D 点云投影为包含 RGB (3) + Depth (1) + World Coords (3) 的 2D 图像，确保 VLM 能理解 3D 结构。
- **热力图预测**：利用凸上采样（Convex Upsampling）从 VLM 的 Token 中恢复高分辨率热力图 $H = U(\text{Rearrange}\{t_i\})$。

#### 2. 精细阶段 (Fine Stage)：主动感知优化

- **视角选择 (A-VS)**：在球面上均匀采样，通过公式评估：

  $$s_i = w_{vis} \cdot s_{vis} + w_{dis} \cdot s_{dis} + w_{div} \cdot s_{div}$$

  平衡**可见性、距离、多样性**，选出最互补的 $K$ 个视角。

- **3D 缩放 (A-3Z)**：根据 $W(z) = 2d \tan \frac{\alpha}{2z}$ 调整视场角，实现无损的虚拟光学变焦，获取高密度特征。

#### 3. 动作预测 (Action Prediction)

- **空间投票**：通过反向投影在离散 3D 网格上进行多视图评分累加 $S(g) = \sum w_v h_v$。
- **特征融合**：拼接全局语义 Token 与局部 ROI Token，通过 MLP 预测离散化的旋转角度。



## Experiments

### **实验设置与指标**

, , 。

### 对比实验

COLOSSEUM

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125033548485.png" alt="image-20260125033548485" style="zoom:67%;" />



RLBench

<img src="/Users/yangchao/Library/Application Support/typora-user-images/image-20260125033606029.png" alt="image-20260125033606029" style="zoom:67%;" />

GemBench

![image-20260125033652056](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125033652056.png)



### 消融实验



![image-20260125033703295](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125033703295.png)



![image-20260125033715688](/Users/yangchao/Library/Application Support/typora-user-images/image-20260125033715688.png)



## Summary & Evaluation

### **值得 Follow 的点**

1. **7 通道投影技术**：这是将 3D 结构喂给 2D VLM 的极佳方案。
2. **多目标评分函数**：其视角多样性评分公式可直接用于其他多相机系统。

