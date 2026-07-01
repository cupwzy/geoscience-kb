---
type: concept
domain: ml-geoscience
status: active
created: 2026-07-01
tags:
  - type/concept
  - domain/ml-geoscience
aliases: [神经网络架构, Deep Learning Architectures]
---

# Neural Network Architectures

> [!abstract] 选择原则
> 架构不是模型名称竞赛。应从数据结构、目标输出、所需感受野、样本量和计算预算出发，选择合适的归纳偏置。

## 1. 架构对比

| 架构 | 核心归纳偏置 | 擅长的数据 | 地学例子 | 主要风险 |
|---|---|---|---|---|
| MLP | 全局特征组合 | 表格、井级属性 | 测井属性回归 | 忽略空间/顺序结构 |
| 1D CNN | 局部模式、平移共享 | 曲线、时间序列 | 测井曲线、地震道 | 长程关系需堆叠多层 |
| 2D/3D CNN | 局部空间结构 | 剖面、体数据 | 断层、盐体、相分类 | 3D 内存成本高 |
| U-Net | 多尺度编码—解码、跳连 | 像素/体素级输出 | 地震分割、属性反演 | 边缘伪影、过度平滑 |
| RNN/LSTM/GRU | 递归顺序依赖 | 序列 | 曲线预测、层序序列 | 并行度低、长依赖困难 |
| Transformer | 全局注意力 | 序列、多模态、patch | 井震融合、长程关联 | 数据和显存需求高 |
| GNN | 图上的邻接关系 | 非规则空间结构 | 井网、断层网络 | 图构建决定结果上限 |
| Autoencoder | 压缩后重建 | 无监督表示 | 降噪、异常检测 | 可能只学会平滑复制 |
| PINN/Physics-guided | 物理方程或规则 | 有控制方程的问题 | 波动方程、反演约束 | 损失刚性与假设失配 |

## 2. MLP：表格数据的基线

MLP 将输入向量经过若干全连接层：

$$
\mathbf h^{(l)}
=\phi\left(\mathbf W^{(l)}
\mathbf h^{(l-1)}+\mathbf b^{(l)}\right).
$$

适合：

- 每个样本已经表示为固定长度特征；
- 特征位置没有显式空间含义；
- 需要建立比树模型更灵活的多任务预测。

对于小型表格数据，树模型、线性模型往往是必须比较的强基线；神经网络不一定占优。

## 3. CNN：局部模式与参数共享

一维离散卷积可写为

$$
y_i
=\sum_{k=0}^{K-1}w_kx_{i+k}+b.
$$

相同卷积核在不同位置共享，使 CNN 能学习局部反射形态、纹理和边缘。关键概念包括：

- kernel size：单层观察的局部范围；
- stride：卷积核移动步长；
- padding：边界如何补齐；
- dilation：在不显著增加参数量时扩大感受野；
- channel：不同特征图；
- pooling：降采样并扩大有效感受野。

> [!warning] 平移等变不等于地质合理
> 地震垂向、inline 和 crossline 方向的物理含义及采样间隔不同，不应不加判断地使用完全对称的卷积核。

## 4. 感受野

感受野是某个输出位置理论上能够使用的输入范围。若局部地震 patch 小于薄层调谐、断层错断或目标砂体的特征尺度，模型根本看不到完整上下文。

理论感受野大也不代表远处信息被有效利用。应通过：

- 不同 patch 尺寸对比；
- 遮挡或扰动测试；
- 特征归因；
- 跨尺度消融实验

验证有效感受野。

## 5. U-Net：像素级和体素级预测

U-Net 由两部分组成：

- encoder：逐级降低分辨率，提取高级语义；
- decoder：逐级恢复分辨率；
- skip connection：把高分辨率细节传到对应解码层。

适合断层/盐体分割、地震相分类和稠密属性预测。需要注意：

- 上采样可能产生棋盘格伪影；
- patch 边界会导致拼接缝；
- 跳跃连接可能让模型复制局部纹理而忽略全局语义；
- 输出分辨率不能超越输入数据真实支持的分辨率。

## 6. 残差网络

残差块学习

$$
\mathbf y=\mathcal F(\mathbf x)+\mathbf x.
$$

跳跃路径让梯度更容易跨越深层网络，并使网络学习相对输入的修正。残差连接改善可优化性，但不会自动解决数据不足或标签噪声。

## 7. RNN、LSTM 与 GRU

基本 RNN 的状态更新为

$$
\mathbf h_t
=\phi(\mathbf W_x\mathbf x_t
+\mathbf W_h\mathbf h_{t-1}
+\mathbf b).
$$

LSTM/GRU 用门控机制控制信息保留和遗忘，缓解长序列中的梯度问题。对测井曲线，它们能表达深度顺序，但必须明确：

- 输入是深度域还是时间域；
- 采样间隔是否统一；
- 双向结构是否使用了部署时不可获得的未来信息。

## 8. Transformer 与注意力

缩放点积注意力为

$$
\operatorname{Attention}(Q,K,V)
=\operatorname{softmax}
\left(\frac{QK^\mathsf T}{\sqrt{d_k}}\right)V.
$$

它根据 query 与 key 的相似度，对 value 加权汇总。优势是可直接连接远距离位置，并适合多模态融合；代价是：

- 标准自注意力随序列长度呈二次复杂度；
- 位置关系必须显式编码；
- 小数据下容易过拟合；
- 注意力权重不应被直接当作因果解释。

## 9. GNN：井网和非规则关系

当数据不是规则网格，而是井、层段或断层块组成的关系网络，可把对象表示为节点，把空间邻近、地层连通或构造关系表示为边：

$$
\mathbf h_i'
=\operatorname{Update}
\left(
\mathbf h_i,
\operatorname{Aggregate}_{j\in\mathcal N(i)}
\mathbf h_j
\right).
$$

图的构造本身就是强假设。仅按欧氏距离连边可能跨越断层或不同层位，造成虚假的信息传播。

## 10. Autoencoder

编码器与解码器分别为

$$
\mathbf z=f_\theta(\mathbf x),
\qquad
\hat{\mathbf x}=g_\phi(\mathbf z),
$$

通过重建损失学习低维表示。常见变体：

- denoising autoencoder：从受扰输入恢复干净信号；
- sparse autoencoder：限制隐变量稀疏；
- variational autoencoder：学习概率潜变量分布。

重建误差低只说明能复原输入，不代表隐变量一定具有地质可解释性。

## 11. Physics-guided、PINN 与混合模型

这几个概念应区分：

- **Physics-informed loss**：在损失中加入 PDE 残差、守恒关系或边界条件；
- **Physics-guided architecture**：把可微正演算子或已知关系嵌入网络；
- **Hybrid inversion**：物理反演与神经网络模块组合；
- **Data augmentation by simulation**：用正演数据扩充训练集。

对于储层预测，物理规律通常具有条件适用性，因此软约束常比硬编码更安全。具体规则见 [[Physics-Guided CNN 中可加入的物理规则约束]]。

## 12. 架构选择清单

1. 输入是表格、曲线、剖面、体还是图？
2. 输出是全局类别、逐点属性、分割掩码还是概率分布？
3. 目标现象需要多大空间/时间上下文？
4. 各轴的采样间隔和物理意义是否相同？
5. 训练时可用的信息在部署时是否仍可用？
6. 样本量是否足以支持该模型容量？
7. 是否有更简单的基线可以回答同一问题？
8. 架构复杂度增加后，验证集和盲测区是否稳定改善？

## 13. 相关笔记

- [[Neural Network Fundamentals]]
- [[Backpropagation and Optimization]]
- [[Neural Network Training and Generalization]]
- [[Physics-Guided CNN 中可加入的物理规则约束]]
