---
title: Wave Equation Numerical Modeling
tags: [fdm, seismic-modeling, numerical-method]
---

# 💻 波动方程数值模拟

地震模拟 = 解 wave equation

---

## 1. 基本流程

1. 输入 velocity model
2. 设置 source wavelet
3. 时间推进
4. 记录 receiver wavefield

---

## 2. 核心算法（FD）

二阶中心差分：

\[
\frac{\partial^2 u}{\partial x^2}
\approx
\frac{u_{i+1} - 2u_i + u_{i-1}}{\Delta x^2}
\]

---

## 3. 正演模拟流程

```text
velocity model → wave equation → synthetic seismogram