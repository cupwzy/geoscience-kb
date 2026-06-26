---
type: concept
domain: geophysics
status: draft
created: 2026-06-26
tags:
  - type/concept
  - domain/geophysics
  - concept/zoeppritz
  - concept/avo
---

# Zoeppritz Equation Derivation

## Purpose

卓普利兹方程描述平面 P 波或 S 波入射到弹性界面时，反射波和透射波振幅之间的关系，是 AVO/AVA 分析的理论基础。

## Starting Assumptions

- 两侧介质为均匀、各向同性、线弹性介质。
- 界面为平面界面。
- 入射波为平面波。
- 界面两侧满足位移连续和应力连续。
- 不考虑吸收、散射和各向异性。

## Notation

- 上覆介质：$V_{P1}, V_{S1}, \rho_1$
- 下伏介质：$V_{P2}, V_{S2}, \rho_2$
- 入射角：$\theta_1$
- 反射 P 波角：$\theta_1$
- 反射 S 波角：$\phi_1$
- 透射 P 波角：$\theta_2$
- 透射 S 波角：$\phi_2$

## Boundary Conditions

1. 法向位移连续
2. 切向位移连续
3. 法向应力连续
4. 切向应力连续

## Matrix Form

在弹性界面上，反射和透射系数可以写成矩阵方程：

$$
\mathbf{A}\mathbf{x} = \mathbf{b}
$$

其中：

$$
\mathbf{x} =
\begin{bmatrix}
R_{PP} \\
R_{PS} \\
T_{PP} \\
T_{PS}
\end{bmatrix}
$$