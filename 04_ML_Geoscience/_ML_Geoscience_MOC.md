---
type: moc
domain: ml-geoscience
status: active
created: 2026-06-26
tags:
  - moc/ml-geoscience
---

# ML Geoscience MOC

## Neural Network Foundations

- [[Neural Network Fundamentals]]：神经元、前向传播、激活函数与任务输出。
- [[Backpropagation and Optimization]]：链式法则、梯度、优化器与训练诊断。
- [[Neural Network Architectures]]：MLP、CNN、U-Net、RNN、Transformer、GNN 与混合物理模型。
- [[Neural Network Training and Generalization]]：数据划分、正则化、评价、校准与分布漂移。

## Core Topics

- [[Feature Engineering for Seismic and Logs]]
- [[Train Validation Split for Spatial Data]]
- [[Model Interpretability in Geoscience]]
- [[Uncertainty and Calibration]]
- [[Physics-Guided CNN 中可加入的物理规则约束]]
- [[Experiment Registry]]

## Suggested Learning Path

1. [[Neural Network Fundamentals]]
2. [[Backpropagation and Optimization]]
3. [[Neural Network Architectures]]
4. [[Neural Network Training and Generalization]]
5. [[Physics-Guided CNN 中可加入的物理规则约束]]

## Principles

- 空间泄漏比随机泄漏更隐蔽：井、线、区块、层位之间的划分要写清楚。
- 输入特征必须保留地质含义和处理历史。
- 解释模型前，先解释数据和标签。
- 复杂模型必须与简单基线比较，并通过盲井或盲区验证。
- 神经网络输出是条件于训练分布的统计预测，不等同于地下真值。
