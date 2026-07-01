---
title: Elastic Wave Equation
tags: [wave-equation, elasticity, geophysics]
aliases: [弹性波动方程, Elastic Wave Equation, Navier Equation]
---

# Elastic Wave Equation（弹性波动方程）

> [!abstract] 模型定位
> 弹性方程同时描述介质的体积变形和剪切变形，因此能够产生 P 波、S 波以及界面处的转换波。

## 1. 从三个基本关系出发

### 1.1 动量守恒

$$
\rho\frac{\partial^2u_i}{\partial t^2}
=\frac{\partial\sigma_{ij}}{\partial x_j}+f_i,
$$

其中 $\mathbf u$ 是位移，$\boldsymbol\sigma$ 是应力张量，$\mathbf f$ 是体力密度。重复下标表示求和。

### 1.2 小应变假设

$$
\varepsilon_{ij}
=\frac12\left(
\frac{\partial u_i}{\partial x_j}
+\frac{\partial u_j}{\partial x_i}
\right).
$$

它只保留位移梯度的一阶项，适用于地震波引起的微小形变。

### 1.3 各向同性线弹性本构关系

$$
\sigma_{ij}
=\lambda\,\varepsilon_{kk}\delta_{ij}
+2\mu\varepsilon_{ij},
$$

其中 $\lambda$、$\mu$ 是 Lamé 参数，$\mu$ 也是剪切模量。$\lambda$ 主要参与体积变形，$\mu$ 控制介质抵抗剪切变形的能力。

## 2. Navier 位移方程

将应变—位移关系和本构关系代入动量方程，且假设 $\lambda$、$\mu$ 在局部为常数，得到

$$
\rho\frac{\partial^2\mathbf u}{\partial t^2}
=(\lambda+\mu)\nabla(\nabla\cdot\mathbf u)
+\mu\nabla^2\mathbf u+\mathbf f.
$$

利用恒等式

$$
\nabla^2\mathbf u
=\nabla(\nabla\cdot\mathbf u)
-\nabla\times(\nabla\times\mathbf u),
$$

也可写为

$$
\rho\frac{\partial^2\mathbf u}{\partial t^2}
=(\lambda+2\mu)\nabla(\nabla\cdot\mathbf u)
-\mu\nabla\times(\nabla\times\mathbf u)+\mathbf f.
$$

这一直观地分开了体积变形与旋转/剪切变形。

> [!warning] 非均匀介质
> 当 $\lambda$、$\mu$ 空间变化时，不能把它们直接移出空间微分算子。更一般、也更稳妥的形式是 $\rho\ddot{\mathbf u}=\nabla\cdot\boldsymbol\sigma+\mathbf f$。

## 3. P 波与 S 波分解

对无源均匀介质的 Navier 方程分别取散度和旋度。

### P 波：体积变化

令 $\theta=\nabla\cdot\mathbf u$，则

$$
\frac{\partial^2\theta}{\partial t^2}
=V_p^2\nabla^2\theta,
\qquad
V_p=\sqrt{\frac{\lambda+2\mu}{\rho}}.
$$

P 波的质点运动方向与传播方向平行。

### S 波：剪切旋转

令 $\boldsymbol\omega=\nabla\times\mathbf u$，则

$$
\frac{\partial^2\boldsymbol\omega}{\partial t^2}
=V_s^2\nabla^2\boldsymbol\omega,
\qquad
V_s=\sqrt{\frac{\mu}{\rho}}.
$$

S 波的质点运动方向与传播方向垂直。流体中 $\mu=0$，因此 $V_s=0$，不能传播 S 波。

## 4. 常用弹性参数关系

各向同性介质只需两个独立弹性常数，再加密度即可确定波速。常用关系为

$$
\mu=\rho V_s^2,\qquad
\lambda=\rho(V_p^2-2V_s^2),
$$

$$
K=\lambda+\frac23\mu
=\rho\left(V_p^2-\frac43V_s^2\right),
$$

$$
\nu=\frac{\lambda}{2(\lambda+\mu)}.
$$

其中 $K$ 是体积模量，$\nu$ 是泊松比。

## 5. 界面现象与地震意义

弹性波到达界面时，边界两侧需满足位移/质点速度及牵引应力的连续条件。因此入射 P 波一般会产生：

- 反射 P 波；
- 反射 S 波；
- 透射 P 波；
- 透射 S 波。

这些振幅与角度关系由 [[Zoeppritz Equation Derivation|Zoeppritz 方程]]描述，是 [[AVO and AVA Basics|AVO/AVA]] 与多分量地震的基础。

## 6. 常见数值形式

实际计算常使用一阶速度—应力系统：

$$
\rho\frac{\partial v_i}{\partial t}
=\frac{\partial\sigma_{ij}}{\partial x_j}+f_i,
$$

$$
\frac{\partial\sigma_{ij}}{\partial t}
=\lambda\delta_{ij}\frac{\partial v_k}{\partial x_k}
+\mu\left(
\frac{\partial v_i}{\partial x_j}
+\frac{\partial v_j}{\partial x_i}
\right).
$$

速度与应力分量可布置在交错网格上，便于使用中心差分并减少数值伪影。

## 7. 何时需要弹性模型

适合：

- 多分量（3C/4C）数据与转换波；
- 弹性 RTM / FWI；
- 岩性、流体和 $V_p/V_s$ 敏感性研究；
- 强弹性对比、自由表面及模式转换明显的问题。

代价：

- 三维中要存储多个速度和应力分量；
- 参数增至 $V_p$、$V_s$、$\rho$ 或等价弹性参数；
- 不同参数的敏感核相互耦合，反演更容易出现串扰；
- 最高波速限制时间步长，最短 S 波波长又限制空间网格。

## 8. 模型边界

本页默认线性、小应变、各向同性、无耗散介质。若存在裂缝定向、薄互层、显著衰减或频散，应进一步考虑：

- 各向异性弹性方程；
- 粘弹性方程与品质因子 $Q$；
- 孔隙弹性（Biot）方程；
- 非线性或有限应变模型。

## 9. 自检问题

- Navier 方程中的参数在空间上是否可视为常数？
- 目标波场包含 P、SV、SH 中的哪些模式？
- 自由表面与界面条件是否正确表达了应力连续性？
- 参数化选择会不会导致 $V_p$、$V_s$ 与 $\rho$ 串扰？
- 网格是否同时解析了更短的 S 波波长？
