---
type: concept
domain: ml-geoscience
status: active
created: 2026-07-01
tags:
  - type/concept
  - domain/ml-geoscience
aliases: [反向传播与优化, Backpropagation, Gradient Descent]
---

# Backpropagation and Optimization

> [!abstract] 一句话理解
> 反向传播使用链式法则高效计算每个参数对损失的影响；优化器利用这些梯度寻找损失较低的参数。

## 1. 前向传播、损失与梯度

设网络预测为

$$
\hat{\mathbf y}=f_\theta(\mathbf x),
$$

损失为

$$
J(\theta)=\mathcal L(\hat{\mathbf y},\mathbf y).
$$

训练需要计算

$$
\nabla_\theta J
=\left[
\frac{\partial J}{\partial\theta_1},
\ldots,
\frac{\partial J}{\partial\theta_P}
\right],
$$

它表示每个参数发生微小变化时，损失如何变化。

## 2. 反向传播就是链式法则

对简单网络

$$
z=wx+b,\qquad
a=\phi(z),\qquad
J=\mathcal L(a,y),
$$

有

$$
\frac{\partial J}{\partial w}
=\frac{\partial J}{\partial a}
\frac{\partial a}{\partial z}
\frac{\partial z}{\partial w}
=\frac{\partial J}{\partial a}\phi'(z)x.
$$

反向传播从损失开始，按计算图的反方向传播局部导数。它不是一种优化算法，而是**计算梯度的方法**；SGD、Adam 等才是利用梯度更新参数的优化算法。

## 3. 梯度下降

最基本的更新为

$$
\theta_{t+1}
=\theta_t-\eta\nabla_\theta J(\theta_t),
$$

其中 $\eta$ 是学习率：

- 太大：损失震荡、发散或越过较优区域；
- 太小：收敛缓慢，有限训练时间内可能欠拟合；
- 合适的学习率通常比微调许多次要超参数更重要。

## 4. Batch、Epoch 与 Iteration

- **batch**：一次前向/反向传播使用的样本子集；
- **iteration/step**：完成一次参数更新；
- **epoch**：训练集中的样本大体被使用一遍。

若训练集大小为 $N$、批量大小为 $B$，每个 epoch 约有

$$
\left\lceil\frac NB\right\rceil
$$

次更新。地学 patch 常高度相关，“看过多少 patch”不等于“看过多少独立地质样本”。

## 5. 常用优化器

### SGD

$$
\theta_{t+1}
=\theta_t-\eta g_t.
$$

简单、内存占用低，但对学习率和特征尺度较敏感。

### Momentum

$$
\mathbf v_t
=\beta\mathbf v_{t-1}+(1-\beta)g_t,
$$

$$
\theta_{t+1}
=\theta_t-\eta\mathbf v_t.
$$

利用历史方向平滑梯度，有助于穿过狭长损失谷。

### Adam

Adam 同时维护梯度的一阶矩和二阶矩估计，并对不同参数自适应缩放步长。它通常容易起步，但默认参数并非对所有问题最佳，也不保证泛化一定优于 SGD。

### AdamW

AdamW 将权重衰减与梯度更新解耦，通常比把 L2 惩罚直接混入 Adam 梯度更符合预期。

## 6. 学习率策略

常见策略包括：

- warm-up：训练初期从小学习率逐渐升高；
- step decay：在预定 epoch 降低学习率；
- cosine decay：按余弦曲线逐渐衰减；
- reduce on plateau：验证指标长期不改善时降低；
- one-cycle：先升高再降低学习率与动量。

调度器的依据必须明确：监控训练损失、验证损失或任务指标会产生不同结果。

## 7. 梯度消失与梯度爆炸

深层链式乘法可能使梯度：

- 趋近于零：前层几乎得不到更新；
- 迅速增大：参数更新不稳定甚至出现 NaN。

常见缓解方法：

- ReLU/GELU 等激活函数；
- Xavier 或 He 初始化；
- 残差连接；
- BatchNorm、LayerNorm；
- 梯度裁剪；
- 合理的学习率与数据标准化。

梯度裁剪能阻止数值爆炸，但也可能只是掩盖错误的损失尺度、异常样本或实现问题。

## 8. 初始化

若所有神经元使用完全相同的初始权重，它们会收到相同梯度，无法学习不同特征。初始化通常以零均值随机值打破对称性，并控制各层激活方差：

- Xavier/Glorot：常用于 tanh 等对称激活；
- He/Kaiming：常用于 ReLU 系列；
- 偏置通常可从零或小常数开始。

## 9. 多损失优化

Physics-guided 模型常写成

$$
\mathcal L_{\text{total}}
=\mathcal L_{\text{data}}
+\sum_{k=1}^{K}\lambda_k\mathcal L_k.
$$

这里真正困难的是不同损失项的：

- 数值量级；
- 单位与归一化；
- 梯度方向是否冲突；
- 训练阶段中的相对重要性。

不能仅因为某项“物理上重要”就赋予巨大权重。若物理假设只在部分岩相成立，过强约束会把模型推向系统性偏差。相关实例见 [[Physics-Guided CNN 中可加入的物理规则约束]]。

## 10. 推荐的训练诊断

每次实验至少记录：

- 总损失及各子损失；
- 训练与验证指标；
- 学习率；
- 梯度范数；
- 参数或输出是否出现 NaN/Inf；
- 最优模型对应的 epoch；
- 随机种子、数据版本和代码版本。

这些内容应进入 [[Experiment Registry]]，否则“某次效果很好”通常无法复现。

## 11. 常见故障定位

| 现象 | 优先检查 |
|---|---|
| 损失从一开始就是 NaN | 输入异常、除零、log(0)、学习率、混合精度 |
| 损失完全不下降 | 标签错位、参数未加入优化器、梯度被 detach、学习率过小 |
| 训练下降而验证变差 | 过拟合、泄漏后的分布差异、BatchNorm、小样本 |
| 损失剧烈震荡 | 学习率、批量大小、异常样本、损失尺度 |
| 输出接近常数 | 类别不平衡、正则过强、激活饱和、标签方差小 |
| 物理损失低但数据拟合差 | 物理项权重过大或物理假设不适用 |

## 12. 自检问题

- 反向传播计算的是哪一个标量目标的梯度？
- 每个损失项是否经过可解释的尺度归一化？
- 优化器、学习率和调度器状态是否随 checkpoint 保存？
- 梯度异常是优化问题，还是数据/实现问题？
- 最优模型是按哪个验证指标选择的？

## 13. 相关笔记

- [[Neural Network Fundamentals]]
- [[Neural Network Training and Generalization]]
- [[Physics-Guided CNN 中可加入的物理规则约束]]
- [[Experiment Registry]]
