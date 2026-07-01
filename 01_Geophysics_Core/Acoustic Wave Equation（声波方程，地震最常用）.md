---
title: Acoustic Wave Equation
tags: [wave-equation, seismic, forward-modeling]
---

# 🔊 Acoustic Wave Equation（声学波动方程）

用于描述**压力波在非弹性剪切介质中的传播**（地震最常用近似）。

---

## 1. 标准形式

\[
\frac{1}{v^2(x)} \frac{\partial^2 p}{\partial t^2} = \nabla^2 p + s(x,t)
\]

- \(p\)：压力场
- \(v(x)\)：速度模型
- \(s\)：震源项

---

## 2. 物理假设（非常重要）

Acoustic model 忽略：

- S-wave（剪切波）
- 各向异性
- 粘弹性衰减（默认）

👉 只保留 P-wave

---

## 3. 为什么地震常用 acoustic？

- 计算成本低
- 易做反演（FWI）
- 足够解释大部分反射数据

---

## 4. 数值求解核心思想

通常用：

- Finite Difference (FD)
- Spectral Method
- Pseudo-spectral

---

## 5. 离散形式（2nd order FD）

\[
p^{n+1} = 2p^n - p^{n-1} + \Delta t^2 v^2 \nabla^2 p^n
\]

---

## ⚠️ 数值稳定性

必须满足 CFL 条件：

\[
\Delta t < \frac{\Delta x}{v_{max}\sqrt{d}}
\]

---

## 🧠 地球物理意义

- velocity model → controls wave propagation
- reflectivity → emerges from velocity contrast
- imaging → inversion of this equation