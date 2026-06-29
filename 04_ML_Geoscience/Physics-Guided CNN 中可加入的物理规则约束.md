## 1. 总体思路

在 CNN 提取地震局部特征时，物理约束一般不是直接加在卷积核上，而是加在以下几个位置：

- 输入数据预处理阶段
    
- CNN 特征提取阶段
    
- 模型输出层
    
- Loss function 损失函数
    
- 训练标签构建
    
- 后处理与验证阶段
    

核心目标是：

> 让 CNN 不只是从数据中学习统计相关性，而是尽量学习符合地球物理、岩石物理和地质规律的储层特征。

在地震储层预测中，CNN 主要负责提取局部地震纹理，例如反射轴、振幅异常、断层错断、砂体边界、薄层调谐等。物理约束的作用是限制模型输出结果不能违背基本物理规律。

---

# 2. 阻抗约束：Acoustic Impedance Constraint

## 2.1 基本公式

声波阻抗 Acoustic Impedance，简称 AI：

```text
AI = Vp × ρ
```

其中：

```text
AI = 声波阻抗
Vp = 纵波速度
ρ = 岩石体积密度，通常对应 RHOB
```

如果测井中有声波时差 DT，可以由 DT 计算 Vp：

```text
Vp = 1 / DT
```

需要注意单位转换。如果 DT 单位是 μs/ft，需要转换为 m/s。

---

## 2.2 约束形式

如果模型预测 AI，同时也输入或预测 Vp、RHOB，可以加入：

```text
L_AI = MSE(AI_pred, Vp_pred × RHOB_pred)
```

如果 Vp 和 RHOB 来自测井实测值，也可以写成：

```text
L_AI = MSE(AI_pred, Vp_log × RHOB_log)
```

---

## 2.3 地质与地球物理意义

这个约束保证模型预测的阻抗结果符合速度和密度的基本关系。

也就是说：

```text
CNN 可以从局部地震纹理中学习阻抗变化，
但它不能输出一个和 Vp、RHOB 矛盾的 AI。
```

---

## 2.4 推荐程度

强烈推荐。

这是最基础、最稳妥、最容易解释的物理约束之一，适合作为 Physics-guided CNN 的核心约束。

---

# 3. 反射系数约束：Reflectivity Constraint

## 3.1 基本公式

地震反射主要来自上下地层阻抗差异。反射系数近似为：

```text
R_i = (AI_{i+1} - AI_i) / (AI_{i+1} + AI_i)
```

其中：

```text
R_i = 第 i 个界面的反射系数
AI_i = 上一层声波阻抗
AI_{i+1} = 下一层声波阻抗
```

---

## 3.2 约束形式

如果 CNN 输出局部 AI 曲线或 AI patch，可以根据 AI 计算反射系数：

```text
AI_pred → R_pred
```

然后与井上计算得到的反射系数或地震反射特征进行比较：

```text
L_R = MSE(R_pred, R_log)
```

如果没有可靠的 R_log，也可以进一步进行地震正演。

---

## 3.3 地质与地球物理意义

这个约束保证：

```text
如果模型预测某个位置存在明显阻抗突变，
那么地震上也应该出现相应反射响应。
```

它可以防止 CNN 把噪声或处理痕迹误认为真实地层界面。

---

## 3.4 推荐程度

推荐。

适合与地震正演一致性约束一起使用。

---

# 4. 地震正演一致性约束：Seismic Forward Consistency Constraint

## 4.1 基本公式

传统地震正演关系可以简化为：

```text
Seismic = Wavelet * Reflectivity
```

即：

```text
阻抗模型
    ↓
反射系数
    ↓
与子波卷积
    ↓
合成地震记录
```

---

## 4.2 约束形式

如果模型预测 AI，则可以计算：

```text
AI_pred → R_pred → Synthetic_seismic
```

然后让合成地震与真实地震接近：

```text
L_seis = MSE(S_real, S_synthetic)
```

完整流程：

```text
输入地震 patch
        ↓
CNN
        ↓
预测 AI / PHIE
        ↓
由 AI 计算反射系数
        ↓
与子波卷积生成合成地震
        ↓
与真实地震 patch 对比
```

---

## 4.3 地质与地球物理意义

这个约束非常重要。它要求模型预测的地下参数能够正演回真实地震响应。

这可以避免出现：

```text
预测孔隙度图很好看，
但由预测结果生成的合成地震无法解释真实地震。
```

---

## 4.4 推荐程度

强烈推荐。

这是 Physics-guided seismic inversion 中最有代表性的约束之一。

---

# 5. 孔隙度-密度约束：Porosity-Density Constraint

## 5.1 基本公式

体积密度和孔隙度之间的关系为：

```text
ρb = φρf + (1 - φ)ρma
```

其中：

```text
ρb = 体积密度 RHOB
φ = 孔隙度
ρf = 孔隙流体密度
ρma = 岩石骨架密度
```

常见骨架密度：

```text
砂岩 sandstone: ρma ≈ 2.65 g/cm³
石灰岩 limestone: ρma ≈ 2.71 g/cm³
白云岩 dolomite: ρma ≈ 2.87 g/cm³
```

---

## 5.2 约束形式

如果模型预测 PHIE，同时输入或预测 RHOB，可以加入：

```text
L_phi_density = MSE(RHOB_pred, PHIE_pred × ρf + (1 - PHIE_pred) × ρma)
```

也可以从 RHOB 反算密度孔隙度：

```text
φ_density = (ρma - ρb) / (ρma - ρf)
```

然后约束：

```text
L_phi = MSE(PHIE_pred, φ_density)
```

---

## 5.3 地质意义

一般来说：

```text
RHOB 较低 → 孔隙度可能较高
RHOB 较高 → 孔隙度可能较低
```

但这个关系受岩性、流体、泥质含量和井眼条件影响。

---

## 5.4 注意事项

这个约束适合作为 soft constraint，不适合设置过强权重。

适用场景：

```text
较干净砂岩储层
岩性相对单一
RHOB 曲线质量可靠
```

不适用或需谨慎：

```text
复杂碳酸盐岩
强泥质砂岩
井径垮塌严重井段
多矿物混合储层
```

---

# 6. 孔隙度-声波速度约束：Porosity-Sonic Constraint

## 6.1 基本公式

常见经验关系之一是 Wyllie time-average equation：

```text
Δt = φΔtf + (1 - φ)Δtma
```

其中：

```text
Δt = 声波时差 DT
φ = 孔隙度
Δtf = 流体声波时差
Δtma = 岩石骨架声波时差
```

---

## 6.2 约束形式

如果模型预测 PHIE，同时输入或预测 DT，可以加入：

```text
DT_pred ≈ PHIE_pred × DT_fluid + (1 - PHIE_pred) × DT_matrix
```

也可以由 DT 估算声波孔隙度，再与 PHIE_pred 对比：

```text
L_sonic_phi = MSE(PHIE_pred, PHI_sonic)
```

---

## 6.3 地质意义

一般趋势为：

```text
PHIE ↑ → Vp ↓ → DT ↑
```

也就是说，孔隙度升高时，岩石速度通常降低，声波时差增大。

---

## 6.4 注意事项

该关系受以下因素影响：

```text
压实作用
胶结作用
岩性变化
压力变化
裂缝
流体类型
泥质含量
```

因此也应作为 soft constraint。

---

# 7. 泥质含量约束：VSH / GR Constraint

## 7.1 基本地质规律

泥质含量 VSH 或自然伽马 GR 可以反映砂泥岩变化。

一般来说：

```text
VSH 高 → 泥质含量高 → 有效储层质量较差
VSH 低 → 可能为较干净砂岩 → 储层质量较好
```

---

## 7.2 约束形式

可以对高 VSH 区域的有效孔隙度或储层概率加入惩罚：

```text
L_vsh = penalty(PHIE_pred high when VSH high)
```

也可以设置随 VSH 增大的孔隙度上限：

```text
PHIE_max_allowed = a - b × VSH
```

然后：

```text
L_vsh = mean(max(0, PHIE_pred - PHIE_max_allowed))
```

---

## 7.3 地质意义

该约束防止模型在明显泥质层中预测出异常高的有效储层质量。

更合理的表达不是：

```text
VSH 高 → PHIE 一定低
```

而是：

```text
VSH 高 → 有效孔隙度、渗透率、储层概率通常降低
```

---

## 7.4 推荐程度

中等推荐。

适合储层分类、有效孔隙度预测、储层概率预测。

---

# 8. Archie 饱和度约束：Archie Water Saturation Constraint

## 8.1 基本公式

对于较干净砂岩，可以使用 Archie 公式：

```text
Sw^n = aRw / (φ^m Rt)
```

其中：

```text
Sw = 含水饱和度
φ = 孔隙度
Rt = 地层真实电阻率
Rw = 地层水电阻率
a, m, n = Archie 参数
```

---

## 8.2 约束形式

如果模型预测 Sw 和 PHIE，并且输入中有 Rt，可以计算：

```text
Sw_archie
```

然后加入：

```text
L_archie = MSE(Sw_pred, Sw_archie)
```

---

## 8.3 地质意义

一般趋势：

```text
高孔隙度 + 高电阻率 → 通常对应较低含水饱和度
低电阻率 → 通常对应较高含水饱和度或泥质导电影响
```

---

## 8.4 注意事项

Archie 公式适用条件较严格。

适用：

```text
干净砂岩
孔隙结构较简单
VSH 较低
Rw 可靠
Rt 曲线质量好
```

需谨慎：

```text
泥质砂岩
低阻油层
复杂碳酸盐岩
裂缝性储层
导电矿物发育
Rw 不确定区域
```

---

## 8.5 推荐程度

初期不建议作为核心约束。

可以作为后续扩展实验，尤其是在数据质量较高、储层较干净的情况下。

---

# 9. 参数取值范围约束：Physical Range Constraint

## 9.1 基本范围

储层参数必须处于物理合理范围内：

```text
0 ≤ PHIE ≤ 0.4 或 0.5
0 ≤ Sw ≤ 1
0 ≤ VSH ≤ 1
Vp > 0
RHOB > 0
AI > 0
K ≥ 0
```

---

## 9.2 实现方式

可以通过输出层激活函数控制：

```text
PHIE: sigmoid 输出后缩放到 0–0.4
Sw: sigmoid 限制在 0–1
VSH: sigmoid 限制在 0–1
AI: softplus 保证为正
```

也可以通过 penalty loss 实现：

```text
L_range = penalty(output outside physical range)
```

---

## 9.3 地质意义

避免模型输出物理上不可能的结果，例如：

```text
PHIE = -0.05
Sw = 1.3
VSH = -0.2
AI < 0
```

---

## 9.4 推荐程度

强烈推荐。

这是最基础、最容易实现的约束。

---

# 10. 垂向连续性约束：Vertical Smoothness Constraint

## 10.1 基本思想

同一地层内部，储层参数通常具有一定垂向连续性，不应无意义地剧烈震荡。

---

## 10.2 约束形式

一阶平滑：

```text
L_smooth = mean(|y_{i+1} - y_i|)
```

二阶平滑：

```text
L_smooth2 = mean(|y_{i+1} - 2y_i + y_{i-1}|)
```

---

## 10.3 地质意义

相邻深度点之间的 PHIE、AI、Sw 不应频繁出现无物理意义的突变。

---

## 10.4 注意事项

不能过度平滑，因为真实地层中存在：

```text
砂泥岩突变
层序界面
断层
不整合面
薄层
流体接触面
```

更合理的是：

```text
层内平滑
层界面允许突变
断层附近允许突变
```

---

# 11. 横向连续性约束：Lateral Smoothness Constraint

## 11.1 基本思想

同一层位内，储层参数通常具有横向连续性。

如果 CNN 在二维剖面或三维地震体上预测储层参数，可以加入空间平滑约束。

---

## 11.2 约束形式

二维情况下：

```text
L_spatial = |y(x+1,z) - y(x,z)| + |y(x,z+1) - y(x,z)|
```

三维情况下：

```text
L_spatial = differences along inline, crossline, time/depth
```

---

## 11.3 地质意义

同一砂体或同一地层单元内部，孔隙度和阻抗通常不会随机跳变。

---

## 11.4 注意事项

断层、尖灭、相变边界附近应允许参数发生明显变化。

---

# 12. 层位约束：Horizon / Stratigraphic Constraint

## 12.1 基本思想

地震数据具有成层性。储层参数通常沿层位变化，而不是简单沿水平线变化。

因此，CNN 提取局部 patch 时应尽量尊重地层结构。

---

## 12.2 实现方式

可以采用：

```text
沿层位提取窗口
沿等时面提取 patch
在目标层上下固定时间窗内采样
输入 horizon map
输入 relative geological time，RGT
只在目标层段内训练
对同一层位内输出做连续性约束
```

---

## 12.3 地质意义

避免 CNN 把不同地层混在一个固定矩形窗口中，从而学习到错误关系。

---

## 12.4 推荐程度

推荐。

尤其适合目标储层层位明确的研究区。

---

# 13. 断层约束：Fault-Aware Constraint

## 13.1 基本思想

断层会切断地层连续性，因此模型不应在断层两侧强行平滑储层参数。

---

## 13.2 约束形式

可以使用断层概率体或相干体作为权重：

```text
断层概率高 → 降低横向平滑约束
断层概率低 → 增强横向平滑约束
```

形式上：

```text
L_fault_aware_smooth = w × |y_i - y_j|
```

其中：

```text
w = 1 - fault_probability
```

---

## 13.3 地质意义

断层附近允许参数突变，非断层区域保持空间连续性。

---

## 13.4 推荐程度

推荐。

适合三维地震体预测，尤其是断层发育区。

---

# 14. 岩性分类约束：Facies-Conditioned Constraint

## 14.1 基本思想

不同岩性具有不同物性范围。

例如：

```text
砂岩：PHIE 可以较高，VSH 较低
泥岩：VSH 高，有效孔隙度低，渗透率低
碳酸盐岩：PHIE 与渗透率关系复杂
煤层：低密度、低阻抗，响应特殊
```

---

## 14.2 约束形式

如果模型同时预测岩性和储层参数，可以设置：

```text
如果 facies = shale，则 PHIE_eff 不应过高
如果 facies = clean sandstone，则 VSH 不应过高
如果 facies = tight sandstone，则 permeability 不应过高
```

也可以进行多任务学习：

```text
CNN 同时输出：
facies
PHIE
AI
Sw
```

然后加入多任务一致性约束。

---

## 14.3 地质意义

保证模型预测的储层参数与岩性解释相互一致。

---

## 14.4 推荐程度

中等推荐。

前提是有可靠岩性标签或解释相。

---

# 15. 多任务一致性约束：Multi-Task Consistency Constraint

## 15.1 基本思想

如果模型同时输出多个储层参数，它们之间不应互相矛盾。

例如：

```text
PHIE 高 → AI 通常偏低
VSH 高 → 有效孔隙度通常较低
RHOB 低 → PHIE 通常较高
RT 高 + PHIE 高 → Sw 通常偏低
Vp 高 + RHOB 高 → AI 高
```

---

## 15.2 约束形式

可以使用多任务 consistency loss：

```text
L_consistency = penalty(inconsistent predictions among PHIE, AI, VSH, Sw, RHOB, Vp)
```

也可以使用 ranking loss。

例如，在同一岩性内：

```text
如果 PHIE_i > PHIE_j，
那么 AI_i 通常应低于 AI_j。
```

可以写成：

```text
L_rank = max(0, margin + AI_i - AI_j)
```

---

## 15.3 地质意义

让模型输出参数之间保持合理的岩石物理趋势。

---

## 15.4 推荐程度

推荐，但需要谨慎设置，不宜过强。

---

# 16. 频带约束：Seismic Bandwidth Constraint

## 16.1 基本思想

地震数据频带有限，不能支持过高频率的储层细节。

例如地震频带可能只有：

```text
10–60 Hz
```

而测井数据具有远高于地震的垂向分辨率。

因此，CNN 不应从地震数据中预测出地震本身无法分辨的高频细节。

---

## 16.2 实现方式

可以采用：

```text
测井标签上尺度 upscaling
测井曲线 blocking
对预测结果进行频带限制
预测 AI 经过地震频带滤波后再与标签比较
```

训练标签构建建议：

```text
原始 PHIE_log
        ↓
按地震分辨率上尺度
        ↓
作为 CNN 训练标签
```

---

## 16.3 地质与地球物理意义

避免模型预测出看似精细但地震数据无法支持的细节。

---

## 16.4 推荐程度

强烈推荐。

这是井震联合预测中非常容易被忽略但极其重要的约束。

---

# 17. 振幅保真约束：Amplitude-Preservation Constraint

## 17.1 基本思想

如果使用地震振幅预测储层参数，前提是地震数据具有相对可靠的保幅处理结果。

如果振幅不保真，CNN 可能把处理痕迹误认为储层响应。

---

## 17.2 需要检查的内容

```text
地震振幅归一化方式是否一致
不同 inline / crossline 振幅尺度是否一致
井旁地震振幅是否与合成记录匹配
AVO / AVA 信息是否可靠
地震处理流程是否保幅
```

---

## 17.3 实现方式

这类约束不一定写成公式，更常体现在数据预处理和训练策略中。

可以采用：

```text
统一振幅归一化
井旁地震与合成地震匹配检查
剔除振幅异常的低质量区域
使用相对属性而非绝对振幅
```

---

## 17.4 推荐程度

推荐。

尤其当模型输入包含原始振幅或 AVO 属性时非常重要。

---

# 18. AVO / AVA 约束：AVO / AVA Constraint

## 18.1 基本思想

如果有 near / mid / far stack 或 angle gather，可以利用振幅随偏移距或入射角变化的规律约束模型。

AVO / AVA 响应受以下因素影响：

```text
Vp
Vs
密度
孔隙度
流体类型
岩性
压力
```

---

## 18.2 实现方式

输入可以包括：

```text
Near stack
Mid stack
Far stack
```

模型应学习不同角度地震响应之间的物理关系。

可以加入：

```text
不同角度叠加数据之间的趋势约束
弹性参数一致性约束
Vp/Vs 与流体响应约束
```

---

## 18.3 地质意义

AVO / AVA 对流体和岩性识别更敏感，适合油气检测和弹性反演。

---

## 18.4 注意事项

该约束对数据质量要求较高，需要：

```text
高质量角道集
可靠角度叠加数据
保幅处理
较好的井震标定
可靠 Vp、Vs、RHOB 测井
```

---

## 18.5 推荐程度

初期不建议作为核心约束。

适合作为高级扩展方向。

---

# 19. 推荐的初始约束组合

对于 Physics-guided CNN / CNN-Transformer 井震联合反演，建议初始版本不要加入过多复杂约束。

最稳妥的组合是：

```text
1. 参数范围约束
2. AI = Vp × RHOB 阻抗约束
3. 反射系数 + 子波卷积的地震正演一致性约束
4. 垂向 / 横向平滑约束
5. 频带约束
```

---

## 19.1 推荐 Loss Function

可以设计为：

```text
L_total =
L_data
+ λ1 L_range
+ λ2 L_AI
+ λ3 L_seismic_forward
+ λ4 L_smooth
+ λ5 L_bandwidth
```

其中：

```text
L_data = 预测值与井上标签之间的误差
L_range = 参数物理范围约束
L_AI = 声波阻抗一致性约束
L_seismic_forward = 地震正演一致性约束
L_smooth = 空间或垂向平滑约束
L_bandwidth = 地震频带约束
λ1–λ5 = 各约束项权重
```

---

# 20. 针对 PHIE + AI 预测的推荐设计

## 20.1 输入数据

```text
地震 patch
井旁测井曲线
地震属性
层位信息
可选：传统反演 AI 体
```

---

## 20.2 模型输出

```text
PHIE_pred
AI_pred
```

---

## 20.3 推荐物理约束

```text
PHIE_pred 限制在 0–0.4 或 0–0.5
AI_pred 必须为正
AI_pred 应与 Vp × RHOB 一致
AI_pred 计算反射系数后应能正演生成合理地震
合成地震应与真实地震相似
PHIE 和 AI 在同一岩性内应保持合理负相关趋势
预测结果应符合地震频带分辨率
同一层位内应具有合理空间连续性
断层附近允许突变
```

---

# 21. 不建议初期强加的约束

初期不建议把以下约束作为核心：

```text
Archie Sw 约束
Gassmann 流体替换约束
复杂 AVO / AVA 约束
渗透率-孔隙度经验公式
强岩性先验约束
```

原因是这些约束对数据和地质背景要求较高。

例如 Sw 预测需要：

```text
可靠 Rt
可靠 Rw
泥质校正
Archie 参数 a, m, n
流体性质
岩性背景
生产或试油验证
```

如果这些条件不充分，物理约束可能反而误导模型。

---

# 22. 总结

在 CNN 提取局部地震特征时，可以加入的物理规则约束包括：

```text
阻抗约束：AI = Vp × RHOB
反射系数约束：R = (AI₂ - AI₁) / (AI₂ + AI₁)
地震正演约束：Seismic = Wavelet * Reflectivity
孔隙度-密度约束：ρb = φρf + (1 - φ)ρma
孔隙度-声波约束：DT 与 PHIE 的岩石物理关系
泥质约束：VSH / GR 与储层质量关系
饱和度约束：Archie 公式
范围约束：PHIE、Sw、VSH、AI 必须在合理范围内
连续性约束：同一层位内参数应连续
断层约束：断层附近允许突变
层位约束：沿地层结构提取和约束特征
频带约束：预测结果不能超过地震分辨率
振幅保真约束：防止模型学习处理痕迹
AVO / AVA 约束：用于角度叠加数据和流体识别
```

最推荐的初始版本是：

```text
CNN 提取局部地震纹理
+ 参数范围约束
+ AI = Vp × RHOB
+ 反射系数与地震正演一致性
+ 层内平滑约束
+ 地震频带约束
```

这个组合既有明确地球物理意义，又不会过度复杂，适合作为第一篇论文或 PhD proposal 的技术路线。