---
title: Wave Equation Basics
tags: [wave-equation, seismology, physics]
aliases: [波动方程, acoustic wave equation]
---

# 🌊 Wave Equation（波动方程）

波动方程是描述**扰动在介质中传播规律**的偏微分方程，是地震学与地球物理正演/反演的核心基础。

---

## 1. 标量波动方程（最基础形式）


$$\frac{\partial^2 u}{\partial t^2} = v^2 \nabla^2 u$$


- \(u(x,t)\)：波场（位移/压力）
- \(v\)：波速（常数或空间变量）
- \(\nabla^2\)：拉普拉斯算子

---

## 2. 物理意义

> 波动 = 惯性项 vs 恢复力 的平衡

- 左边：加速度（惯性）
- 右边：空间曲率（弹性恢复）

---

## 3. 在地震中的意义

波动方程控制：

- 地震波传播
- 反射/折射
- 绕射（diffraction）
- 成像基础（RTM / FWI）

---

## 4. 常见假设

- 线性介质
- 小应变
- 各向同性（isotropic）
- 无耗散（或弱衰减）

---

## 5. 与地震数据关系

地震记录：

$$
d(t,x) = W(t) * R(x,z) * G(x,z)
$$

本质来自 wave equation 的 Green's function 解。

---

## 💡 关键理解

> 地震记录不是“直接看到地下结构”，而是 wave equation 的响应结果