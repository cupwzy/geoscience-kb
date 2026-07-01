---
title: Elastic Wave Equation
tags: [wave-equation, elasticity, geophysics]
---

# 🧱 Elastic Wave Equation（弹性波动方程）

描述真实地球中 **P波 + S波同时传播**

---

## 1. 基本形式（Navier equation）

\[
\rho \frac{\partial^2 \mathbf{u}}{\partial t^2}
=
(\lambda + \mu)\nabla(\nabla \cdot \mathbf{u}) + \mu \nabla^2 \mathbf{u} + \mathbf{f}
\]

- \(\mathbf{u}\)：位移矢量
- \(\lambda, \mu\)：Lamé参数
- \(\rho\)：密度

---

## 2. 波的分解

- P-wave：纵波（压缩）
- S-wave：横波（剪切）

---

## 3. 波速关系

\[
V_p = \sqrt{\frac{\lambda + 2\mu}{\rho}}
\]

\[
V_s = \sqrt{\frac{\mu}{\rho}}
\]

---

## 4. 地震中的意义

Elastic model 用于：

- 多分量地震（3C/4C）
- 各向异性研究
- 复杂构造（盐丘、断层）

---

## 5. 为什么工业中少用？

原因：

- 计算成本极高
- 参数更多（λ, μ, ρ）
- 反演更不稳定

---

## 💡 实际策略

| 模型 | 使用场景 |
|------|----------|
| Acoustic | 常规反演 / FWI |
| Elastic | 高精度研究 / 论文 |
