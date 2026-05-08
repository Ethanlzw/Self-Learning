# Fault Tolerant Control 完全版复习笔记

## 严格按 Curriculum 六主题划分的核对表

| Curriculum 正式主题 | Curriculum 小项 | 本文对应位置 | 核对结论 |
|---|---|---|---|
| 1. Introduction to Fault and Statistical Signal Processing | Bayes' Theorem and PDF estimation; Linear Discriminant Analysis (LDA); Logistic Regression | Topic 1 | 严格归入 Topic 1。Threshold、FAR/MAR、MLE/MAP、Parzen、kNN 是 PDF estimation 和 statistical signal processing 的 lecture 扩展，不单独成主题。 |
| 2. Dimensionality Reduction Approaches | PCA and variants; Fisher Discriminant Analysis (FDA); MDS and Isomap | Topic 2 | 严格归入 Topic 2。KPCA、SPCA、Kernel MDS、Kernel Isomap 是对应 variants / extensions。 |
| 3. Support Vector Machines (SVM) and Perceptron | Hard Margin SVM; Soft Margin SVM; Kernel Trick and Perceptron | Topic 3 | 严格归入 Topic 3。KKT、dual form、slack variables 是 hard/soft margin 的推导内容。 |
| 4. Model based fault detection | Model based residual generation; parity method in frequency/time domain; observer-based methods (UIO, KF, ESO, NDO); CUSUM and ARL | Topic 4 | 严格归入 Topic 4。Observer 内容只作为 diagnosis/residual，不在这里讲 controller reconfiguration。 |
| 5. Fault Tolerant Control Strategy | Robust/Adaptive control; Reconfigurable Control Systems; FTC methods: Model Matching, Transient Elimination, Virtual Actuator/Sensor | Topic 5 | 严格归入 Topic 5。State-space/LQR/observer 只作为 reconfigurable FTC 的控制基础，不单独作为考试主题。 |
| 6. Safe/Secure Control for Cyber-Physical Systems (CPS) | Attack/Anomaly Detection; Resilient Control for CPS; Safe Control Synthesis | Topic 6 | 严格归入 Topic 6。MPC/terminal set/offset-free MPC 属于 safe control synthesis；LESO/UIO+MPC 属于 resilient control；VAE 属于 anomaly detection。 |

因此本文不是按 lecture 编号划分，而是按截图中的 6 个 Curriculum 主题划分；lecture 内容只作为知识点来源被归入对应主题。

---

## 导读：全课程总逻辑

Fault Tolerant Control 不是单一算法，而是一条完整工程链：

```text
Raw sensor / model data
-> Feature or residual generation
-> Fault detection
-> Fault isolation
-> Fault identification / estimation
-> Control reconfiguration / accommodation
-> Safe and resilient operation under constraints
```

6 个考试主题可以理解成三层：

1. Topic 1-3：数据驱动 FDD  
   从信号、概率、降维和分类角度判断 normal / fault / fault type。

2. Topic 4：模型驱动 FDD  
   用系统模型、残差、观测器和统计决策判断故障。

3. Topic 5-6：故障后的控制  
   Topic 5 讲 FTC strategy 和 reconfiguration；Topic 6 讲 CPS 中带约束、安全和攻击/异常的 resilient control。

### 0.1 基础术语

- Fault：系统组件至少一个特性偏离允许状态。例如 actuator loss、sensor bias、plant parameter drift。
- Failure：系统无法完成规定功能，是 fault 的严重后果。
- Disturbance：外部或内部未知输入，不一定代表组件损坏，例如风、负载变化。
- Noise：测量或过程随机扰动，通常不代表真实物理故障。
- Anomaly：观测到的异常行为，可能来自 fault、attack、disturbance 或工况变化。
- Attack：CPS 中人为恶意注入或篡改，例如 false data injection、actuator command manipulation。

### 0.2 Diagnosis 的三层

- Detection：是否有故障。二分类：normal vs fault。
- Isolation：故障在哪里或属于哪一类。多分类：normal / sensor fault / actuator fault / component fault 等。
- Identification / estimation：故障大小、时间、方向或参数变化是多少。例如估计 actuator loss `f_a`。

### 0.3 FTC 的两部分

FTC 可以写成：

```text
Fault diagnosis + Control reconfiguration
```

诊断提供“发生了什么”，重构决定“如何继续控制”。

---

# Topic 1: Introduction to Fault and Statistical Signal Processing

## 1.1 主题边界

Topic 1 讲的是：如何从传感器信号和统计概率角度判断系统是否故障。

完整覆盖：

- signal-based fault detection
- threshold detection
- time-domain / frequency-domain features
- FAR、MAR、confusion matrix、sensitivity、specificity
- supervised / unsupervised learning
- detection vs isolation
- Bayes theorem、MAP、LRT
- Gaussian PDF、optimal threshold、risk-based decision
- MLE、MAP parameter estimation
- histogram、Parzen window、kNN density estimation
- LDA、QDA、logistic regression

对应 Curriculum 小项：

- Bayes' Theorem and PDF estimation：覆盖 Bayes、MAP、LRT、Gaussian PDF、MLE/MAP、histogram、Parzen、kNN。
- Linear Discriminant Analysis (LDA)：完整放在 1.25；QDA 是 LDA 的自然对比扩展。
- Logistic Regression：完整放在 1.27。

不在 Topic 1 完整展开：

- PCA、FDA、MDS、Isomap 的降维理论放 Topic 2。
- SVM 最大间隔理论放 Topic 3。
- 残差和观测器放 Topic 4。

## 1.2 Signal-based fault detection 的基本问题

系统有一组传感器：

```math
S_1(t), S_2(t), \dots, S_n(t)
```

传感器可能测压力、温度、振动、电流、速度、位置等。Fault detection 要判断系统状态：

```math
X(t) =
\begin{cases}
Normal, & \text{system is healthy} \\
Fault, & \text{fault occurred}
\end{cases}
```

最简单的方法是 threshold alarm：

```math
\text{Alarm if } x(t) > J_{th}
```

但单纯 amplitude threshold 不够：

- 噪声会造成误报。
- 故障可能只改变频率，不改变幅值。
- 故障可能改变 variance、kurtosis、spectral energy，而 mean 不变。
- 工况变化会让正常数据越限。

所以课程引入 signal processing pipeline。

## 1.3 Signal diagnosis pipeline

标准流程：

```text
Raw signal
-> Feature extraction
-> Feature selection / dimensionality reduction
-> Classifier
-> Decision
```

含义：

- Raw signal：原始时间序列，维度高、噪声多。
- Feature extraction：把信号转换成更有诊断意义的统计量或频谱量。
- Feature selection：保留对故障敏感、对噪声不敏感的特征。
- Classifier：把 feature vector 映射到 normal/fault/fault type。
- Decision：根据阈值、概率或风险给出最终报警。

## 1.4 Time-domain features

给定一段信号样本：

```math
x_1, x_2, \dots, x_N
```

常用特征：

Mean：

```math
\mu = \frac{1}{N}\sum_{i=1}^{N}x_i
```

表示 bias 或 DC offset。适合检测传感器偏置、压力/温度均值漂移。

Variance：

```math
\sigma^2 = \frac{1}{N}\sum_{i=1}^{N}(x_i-\mu)^2
```

表示波动强度。适合检测振动增强、不稳定、噪声增大。

RMS：

```math
x_{rms} = \sqrt{\frac{1}{N}\sum_{i=1}^{N}x_i^2}
```

表示总能量或强度。常用于 vibration / current monitoring。

Peak-to-peak：

```math
x_{pp} = \max(x) - \min(x)
```

表示最大摆动范围。

Skewness：

```math
Skew = E\left[\left(\frac{x-\mu}{\sigma}\right)^3\right]
```

描述分布不对称性。

Kurtosis：

```math
Kurt = E\left[\left(\frac{x-\mu}{\sigma}\right)^4\right]
```

描述尖峰和重尾，适合冲击类故障，例如轴承局部损伤。

Crest factor：

```math
CF = \frac{\max |x_i|}{x_{rms}}
```

高 crest factor 表示瞬态冲击明显。

## 1.5 Frequency-domain features

频域方法适合“时间域幅值不明显，但频率结构改变”的故障。

Fourier transform：

```math
S(\omega)=\int_{-\infty}^{\infty}s(t)e^{-j\omega t}dt
```

FFT 是快速计算离散频谱的方法。

Power Spectral Density：

```math
PSD(\omega) \propto |S(\omega)|^2
```

常用特征：

- dominant frequency：主频。
- band energy：某个频段能量。
- harmonic content：谐波结构。
- spectral peak shift：峰值频率偏移。
- wavelet coefficients：时频局部特征，适合瞬态故障。

使用场景：

- bearing fault
- rotating machinery
- resonance detection
- motor current signature analysis

## 1.6 Dimensionality reduction 在 Topic 1 中的作用

在 Topic 1 中只需理解作用，不展开 PCA 推导。

原始信号可能是：

```math
x = [x_1, x_2, \dots, x_{1000}]^T
```

feature extraction 后可能变成：

```math
F = [\mu, \sigma^2, x_{rms}, Kurt]^T
```

作用：

- 减少计算量。
- 去除噪声和冗余。
- 让 normal 与 fault 在 feature space 中更容易分离。
- 让分类器可训练。

## 1.7 Supervised vs unsupervised

Supervised learning：

- 数据有 label，例如 normal / fault。
- 学习映射：

```math
h: X \rightarrow Y
```

- 典型方法：LDA、QDA、logistic regression、SVM、neural networks。
- 适合已知 fault types。

Unsupervised learning：

- 没有 fault label。
- 学 normal data distribution 或 cluster structure。
- 新样本若远离 normal cluster，则判为 anomaly。
- 典型方法：PCA、clustering、autoencoder、VAE。
- 适合未知故障。

## 1.8 Detection vs isolation

Detection 是二分类：

```math
Y=\{Normal, Fault\}
```

Isolation 是多分类：

```math
Y=\{Normal, Fault_1, Fault_2, \dots, Fault_K\}
```

Detection 只回答“有没有故障”。  
Isolation 回答“是哪种故障”。

## 1.9 Performance metrics

Confusion matrix：

```text
                    Predicted Normal    Predicted Fault
Actual Normal       TN                  FP
Actual Fault        FN                  TP
```

False Alarm Rate：

```math
FAR = \frac{FP}{TN+FP}
```

含义：系统正常却报警。高 FAR 导致停机、经济损失、operator alarm fatigue。

Missed Alarm Rate：

```math
MAR = \frac{FN}{TP+FN}
```

含义：系统故障却没报警。高 MAR 在安全系统中最危险。

Sensitivity / recall：

```math
Sensitivity = \frac{TP}{TP+FN}=1-MAR
```

Specificity：

```math
Specificity = \frac{TN}{TN+FP}=1-FAR
```

Accuracy：

```math
Accuracy=\frac{TP+TN}{TP+TN+FP+FN}
```

为什么 accuracy 会骗人：如果 90% 样本是 normal，一个永远输出 normal 的 detector 也能有 90% accuracy，但 fault detection rate 为 0。

Detection delay：

```math
DD = t_{alarm}-t_{fault}
```

理想值为 0，但实际总是大于 0。

## 1.10 Threshold trade-off

若 residual 或 feature 越大越像故障：

- 提高 threshold：更难报警，FAR 降低，MAR 升高。
- 降低 threshold：更容易报警，MAR 降低，FAR 升高。

安全系统一般更关心低 MAR；生产系统也要控制 FAR，避免频繁停机。

## 1.11 Bayes theorem

目标是求：

```math
P(F\mid x)
```

即观察到数据 `x` 后系统故障的后验概率。

Bayes theorem：

```math
P(F\mid x)=\frac{f(x\mid F)P(F)}{f(x)}
```

各项含义：

- `P(F|x)`：posterior，看到数据后系统故障概率。
- `f(x|F)`：likelihood，若系统故障，观察到 x 的可能性。
- `P(F)`：prior，故障先验概率。
- `f(x)`：evidence，归一化常数。

Normal 类同：

```math
P(N\mid x)=\frac{f(x\mid N)P(N)}{f(x)}
```

## 1.12 MAP decision rule

Maximum A Posteriori rule：

```math
\text{Decide Fault if } P(F\mid x)>P(N\mid x)
```

代入 Bayes theorem，证据 `f(x)` 可消去：

```math
f(x\mid F)P(F)>f(x\mid N)P(N)
```

含义：

- 不只看数据像不像 fault，也看 fault 是否先验上常见。
- 若故障极少，prior 会让 detector 更保守。
- 若安全代价很高，可通过 risk 修改 threshold。

多类 isolation：

```math
h^*(x)=\arg\max_k P(y=k\mid x)
```

也就是选择 posterior 最大的 fault class。

## 1.13 Continuous variables and PDF

传感器通常连续，单点概率为 0：

```math
P(X=x)=0
```

必须用 density：

```math
P(a<X<b)=\int_a^b f(x)dx
```

Bayes 连续形式：

```math
P(N\mid x)=\frac{f(x\mid N)P(N)}{f(x)}
```

## 1.14 Gaussian assumption

工程中常假设 normal/fault data 服从 Gaussian：

```math
x\mid N \sim \mathcal N(\mu_N,\sigma_N^2)
```

```math
x\mid F \sim \mathcal N(\mu_F,\sigma_F^2)
```

PDF：

```math
f(x\mid N)=\frac{1}{\sqrt{2\pi\sigma_N^2}}
\exp\left(-\frac{(x-\mu_N)^2}{2\sigma_N^2}\right)
```

阈值 `T`：

```math
x<T \Rightarrow Normal
```

```math
x>T \Rightarrow Fault
```

False alarm probability：

```math
P_{FA}=\int_T^\infty f(x\mid N)dx
```

Missed alarm probability：

```math
P_{MA}=\int_{-\infty}^{T} f(x\mid F)dx
```

Total error：

```math
P(E)=P(N)P_{FA}+P(F)P_{MA}
```

Optimal threshold 出现在：

```math
f(x\mid F)P(F)=f(x\mid N)P(N)
```

## 1.15 Likelihood Ratio Test

把 data 和 prior 分开：

```math
\Lambda(x)=\frac{f(x\mid N)}{f(x\mid F)}
```

普通 MAP 下：

```math
\Lambda(x) > \frac{P(F)}{P(N)} \Rightarrow Normal
```

或等价形式也可写成 fault likelihood over normal likelihood，只要方向一致。

若 prior 不知道：

1. Laplace principle：假设各类等可能。

```math
P(N)=P(F)=0.5
```

2. Neyman-Pearson criterion：固定最大 FAR，例如 `FAR <= 0.05`，再选 threshold。

## 1.16 Risk-based decision

普通 Bayes classifier 默认所有错误代价相同。但 FTC 中 missed alarm 往往比 false alarm 严重得多。

定义 loss matrix：

```math
\lambda_{ij} = \text{cost of deciding class } \omega_i
\text{ when truth is } \omega_j
```

二分类例子：

- `lambda_NN = 0`：真实 normal，判断 normal。
- `lambda_FF = 0`：真实 fault，判断 fault。
- `lambda_FN`：真实 normal，判断 fault，false alarm cost。
- `lambda_NF`：真实 fault，判断 normal，missed alarm cost。

Conditional risk：

```math
R(\alpha_i\mid x)=\sum_{j=1}^{c}\lambda_{ij}P(\omega_j\mid x)
```

Bayes risk rule：

```math
\text{Choose } \alpha_i \text{ if } R(\alpha_i\mid x)<R(\alpha_k\mid x),\ \forall k\neq i
```

二分类且正确判断代价为 0：

```math
R(\alpha_N\mid x)=\lambda_{NF}P(F\mid x)
```

```math
R(\alpha_F\mid x)=\lambda_{FN}P(N\mid x)
```

Decide fault if：

```math
\lambda_{FN}P(N\mid x)<\lambda_{NF}P(F\mid x)
```

代入 Bayes：

```math
\lambda_{FN}f(x\mid N)P(N)<\lambda_{NF}f(x\mid F)P(F)
```

因此 risk-adjusted LRT threshold：

```math
\frac{f(x\mid N)}{f(x\mid F)}
<
\frac{\lambda_{NF}P(F)}{\lambda_{FN}P(N)}
```

解释：如果 missed alarm cost `lambda_NF` 很大，系统会更敏感，宁可增加 false alarm，也要减少 missed fault。

## 1.17 MLE parameter estimation

Bayes rule 需要知道 `f(x|N)` 和 `f(x|F)`，现实中通常不知道，只能从 training data 估计。

给定 i.i.d. 数据：

```math
D=\{x_1,\dots,x_n\}
```

模型参数：

```math
\theta
```

Likelihood：

```math
L(\theta)=P(D\mid \theta)=\prod_{k=1}^{n} f(x_k\mid \theta)
```

MLE：

```math
\hat\theta_{MLE}=\arg\max_\theta L(\theta)
```

Log-likelihood：

```math
l(\theta)=\log L(\theta)=\sum_{k=1}^{n}\log f(x_k\mid \theta)
```

Gaussian mean：

```math
\hat\mu_{MLE}=\frac{1}{n}\sum_{k=1}^{n}x_k
```

Gaussian variance：

```math
\hat\sigma^2_{MLE}=\frac{1}{n}\sum_{k=1}^{n}(x_k-\hat\mu)^2
```

注意：MLE variance 用 `1/n`，是 biased estimator；统计中常用 unbiased sample variance：

```math
s^2=\frac{1}{n-1}\sum_{k=1}^{n}(x_k-\bar x)^2
```

MLE 优点：

- 数据充分时高效。
- 不需要 prior。
- 公式清楚，适合 Gaussian detector。

MLE 缺点：

- 小样本会过拟合。
- 没见过的事件可能被估计为概率 0。
- 完全依赖模型假设。

## 1.18 MAP parameter estimation

MAP 在 MLE 基础上加入 prior：

```math
P(\theta\mid D)=\frac{P(D\mid\theta)P(\theta)}{P(D)}
```

MAP objective：

```math
\hat\theta_{MAP}
=\arg\max_\theta P(D\mid\theta)P(\theta)
```

Log form：

```math
\hat\theta_{MAP}
=\arg\max_\theta
\left[
\log P(D\mid\theta)+\log P(\theta)
\right]
```

`log P(theta)` 是 regularizer。

Gaussian mean MAP 例子：

Likelihood：

```math
x_k\sim \mathcal N(\mu,\sigma^2)
```

Prior：

```math
\mu\sim \mathcal N(\mu_0,\sigma_0^2)
```

结果：

```math
\hat\mu_{MAP}
=
\frac{n\sigma_0^2}{n\sigma_0^2+\sigma^2}\bar x
+
\frac{\sigma^2}{n\sigma_0^2+\sigma^2}\mu_0
```

解释：

- 数据多时，`bar x` 权重变大，MAP 接近 MLE。
- 数据少时，prior mean `mu_0` 影响更大。

MAP 使用场景：

- fault data 稀缺。
- 有工程经验或物理先验。
- 希望避免 small-sample overfitting。

## 1.19 Recursive estimation

实时 FTC 中数据流连续到来，不能每次存全部数据。

Mean recursive update：

```math
\hat\mu_n
=
\hat\mu_{n-1}
+
\frac{1}{n}(x_n-\hat\mu_{n-1})
```

其中：

- `x_n - mu_{n-1}` 是 innovation。
- `1/n` 是 gain，数据越多，新样本影响越小。

这个思想是 Kalman filter 的基础：预测 + innovation correction。

## 1.20 Parametric vs non-parametric density estimation

Parametric：

- 假设分布形状，例如 Gaussian。
- 只估计少量参数，例如 `mu, sigma`。
- 优点：数据需求少，计算快。
- 缺点：假设错则模型错。

Non-parametric：

- 不假设具体形状。
- 让数据本身决定 PDF。
- 典型方法：histogram、Parzen window、kNN density。
- 优点：可表示多峰、偏斜、不规则分布。
- 缺点：需要大量数据，容易受维度灾难影响。

## 1.21 Histogram density estimation

对 bin `R`：

```math
P(X\in R)=\int_R p(x)dx
```

若 bin 很小，近似：

```math
P \approx p(x)\Delta x
```

若 `N_x` 个样本落入 bin：

```math
P\approx \frac{N_x}{N}
```

所以：

```math
\hat p(x)=\frac{N_x}{N\Delta x}
```

优点：

- 简单直观。
- 可用于 arbitrary normal distribution。

缺点：

- bin width 过大：oversmoothing，高 bias。
- bin width 过小：spiky，高 variance。
- 依赖 grid 位置。
- 高维时 bin 数爆炸。

Fault detection 用法：

1. 用 normal data 建 histogram。
2. 新样本落在 low density bin，则 anomaly。
3. 阈值可设置为覆盖 normal PDF 95% 或 99% 区域。

## 1.22 Parzen window

基本估计式：

```math
\hat p(x)=\frac{k}{NV}
```

Parzen 思路：

- 固定 window volume `V`。
- 在查询点 `x` 周围放一个窗口。
- 统计有多少训练样本落在窗口中。

Hypercube window：

```math
V=h^d
```

Window function：

```math
\phi(u)=
\begin{cases}
1, & |u_j|\le \frac12,\ \forall j \\
0, & otherwise
\end{cases}
```

Parzen estimator：

```math
\hat p(x)
=
\frac{1}{N}\sum_{i=1}^{N}\frac{1}{h^d}
\phi\left(\frac{x-x_i}{h}\right)
```

Smooth kernel version：

```math
\hat p_N(x)
=
\frac{1}{N}\sum_{i=1}^{N}\frac{1}{h^d}
K\left(\frac{x-x_i}{h}\right)
```

Gaussian kernel：

```math
K(u)=\frac{1}{(2\pi)^{d/2}}\exp\left(-\frac12\|u\|^2\right)
```

Kernel 条件：

```math
K(u)\ge 0,\qquad \int K(u)du=1
```

Bandwidth `h`：

- 小 `h`：undersmoothing，spiky，高 variance。
- 大 `h`：oversmoothing，blurred，高 bias。

为什么对 FTC 有用：

- normal data 可能多峰，例如机器有多个正常速度。
- Parzen 能表示 valley；落在 valley 的样本可能是 anomaly。

限制：

- 每个 test point 要和所有 training samples 计算距离。
- 高维样本需求指数增长。

## 1.23 kNN density estimation

kNN 反转 Parzen 的逻辑：

- Parzen 固定 volume `V`，看里面有多少点 `k`。
- kNN 固定 neighbor count `k`，看需要多大 volume `V_k(x)`。

令 `R_k(x)` 为 x 到第 k 个近邻的距离，d 维 hypersphere 体积：

```math
V_k(x)=c_dR_k(x)^d
```

kNN density：

```math
\hat p(x)=\frac{k}{N V_k(x)}
=
\frac{k}{N c_d R_k(x)^d}
```

解释：

- 高密度区：第 k 个邻居很近，`V` 小，density 高。
- 低密度区：第 k 个邻居很远，`V` 大，density 低。

k 选择：

- 小 k：高 variance，受噪声影响。
- 大 k：高 bias，过度平滑。
- 常见经验：`k ≈ sqrt(N)`。

优点：

- 局部自适应。
- tail behavior 比固定 Parzen window 好。
- 直观地把 anomaly detection 转为 distance-to-normal-cluster。

缺点：

- 存储全部数据。
- 在线计算昂贵。
- 高维下距离失去区分度。

## 1.24 Classification formulation

给定历史数据：

```math
(x_1,y_1),\dots,(x_n,y_n)
```

其中：

```math
x_i\in\mathbb R^d
```

binary detection：

```math
y_i\in\{0,1\}
```

fault isolation：

```math
y_i\in\{0,1,\dots,K\}
```

Detector：

```math
h:X\rightarrow Y
```

True error：

```math
L(h)=Pr(h(x)\neq y)
```

Training error：

```math
\hat L_n=\frac1n\sum_{i=1}^{n}\mathbf 1(h(x_i)\neq y_i)
```

## 1.25 LDA

LDA 是 generative classifier。

假设：

```math
x\mid y=k \sim \mathcal N(\mu_k,\Sigma)
```

所有类共享 covariance：

```math
\Sigma_0=\Sigma_1=\cdots=\Sigma
```

Class prior：

```math
\hat\pi_k=\frac{n_k}{n}
```

Mean：

```math
\hat\mu_k=\frac{1}{n_k}\sum_{i:y_i=k}x_i
```

Covariance：

```math
\hat\Sigma_k
=
\frac{1}{n_k-1}
\sum_{i:y_i=k}(x_i-\hat\mu_k)(x_i-\hat\mu_k)^T
```

Pooled covariance：

```math
\hat\Sigma
=
\frac{\sum_{k}n_k\hat\Sigma_k}{\sum_k n_k}
```

Discriminant：

```math
\delta_k(x)
=
x^T\Sigma^{-1}\mu_k
-
\frac12\mu_k^T\Sigma^{-1}\mu_k
+
\log\pi_k
```

Decision：

```math
h(x)=\arg\max_k \delta_k(x)
```

为什么边界是线性的：

Gaussian log density 中的 quadratic term `x^T Sigma^{-1} x` 对所有类相同，所以相互比较时会消掉。

使用场景：

- 类别近似 Gaussian。
- fault 改变 mean，但噪声结构相似。
- 样本不多，需要稳定线性 classifier。

优点：

- 简单，可解释。
- 参数较少。
- 在线计算快。

缺点：

- shared covariance 假设强。
- 非线性边界无法表达。

## 1.26 QDA

QDA 假设每类有自己的 covariance：

```math
x\mid y=k \sim \mathcal N(\mu_k,\Sigma_k)
```

Discriminant：

```math
\delta_k(x)
=
-
\frac12\log|\Sigma_k|
-
\frac12(x-\mu_k)^T\Sigma_k^{-1}(x-\mu_k)
+
\log\pi_k
```

由于 covariance 不同，quadratic term 不会消掉，所以边界为：

```math
x^TAx+b^Tx+c=0
```

使用场景：

- fault 不仅改变 mean，也改变 variance/correlation。
- 类别分布形状明显不同。

优点：

- 比 LDA 更灵活。
- 可以表达二次边界。

缺点：

- 参数更多，需要更多数据。
- covariance matrix 估计不稳时容易过拟合。

## 1.27 Logistic regression

Logistic regression 直接估计：

```math
P(y=1\mid x;\theta)
```

Sigmoid：

```math
h_\theta(x)
=
g(\theta^Tx)
=
\frac{1}{1+e^{-\theta^Tx}}
```

Decision：

```math
h_\theta(x)\ge 0.5 \Rightarrow Fault
```

Decision boundary：

```math
\theta^Tx=0
```

Cross-entropy cost：

```math
J(\theta)
=
-
\frac1m
\sum_{i=1}^{m}
\left[
y^{(i)}\log h_\theta(x^{(i)})
+
(1-y^{(i)})\log(1-h_\theta(x^{(i)}))
\right]
```

为什么不用 MSE：sigmoid + MSE 会使 cost non-convex，gradient descent 可能卡在 local minimum。Cross-entropy 来自 MLE，具有 convex 优势。

Gradient descent：

```math
\theta_j
:=
\theta_j
-
\alpha
\frac{1}{m}
\sum_{i=1}^{m}
(h_\theta(x^{(i)})-y^{(i)})x_j^{(i)}
```

Newton-Raphson：

```math
\theta^{(t+1)}
=
\theta^{(t)}
-
H^{-1}\nabla_\theta J
```

优点：

- 输出概率，便于 risk management。
- 参数权重可解释。
- 计算快，适合 embedded / online monitoring。
- 是复杂模型前的强 baseline。

缺点：

- 原始形式为线性边界。
- 特征工程很重要。
- 类别严重重叠时性能有限。

非线性扩展：加入 polynomial features，例如：

```math
\theta_0+\theta_1x_1^2+\theta_2x_2^2=0
```

---

# Topic 2: Dimensionality Reduction Approaches

## 2.1 主题边界

Topic 2 讲的是：如何把高维传感器数据压缩到低维空间，同时保留对 fault detection / isolation 有用的信息。

完整覆盖：

- PCA
- direct PCA and dual PCA
- SPE / Q statistic and Hotelling T²
- Kernel PCA
- Supervised PCA
- HSIC and Kernel SPCA
- Metric learning and sufficient dimension reduction
- FDA
- MDS
- Generalized / kernel MDS
- Isomap and Kernel Isomap

对应 Curriculum 小项：

- PCA and its variants：PCA、dual PCA、KPCA、SPCA。
- Fisher Discriminant Analysis (FDA)：完整放在 2.15。
- Multidimensional Scaling (MDS) and Isometric Mapping (Isomap)：完整放在 2.16-2.19。

不完整展开：

- LDA/QDA/logistic regression 的分类理论在 Topic 1。
- SVM kernel classification 在 Topic 3。

## 2.2 为什么 FDD 需要降维

现代工业系统可能有几十到几百个传感器。高维数据问题：

- 变量强相关。
- 噪声和冗余多。
- fault signature 可能隐藏在多个变量的组合方向。
- 高维空间距离和密度估计困难。
- 分类器容易过拟合。
- 可视化困难。

降维作用：

- 减少计算复杂度。
- 提取 latent variables。
- 建立 normal operating subspace。
- 用 residual subspace 捕捉 fault。
- 让 fault clusters 在 2D/3D 更明显。

## 2.3 PCA 核心思想

PCA 是无监督线性降维。目标：找到最大方差方向。

给定 fault-free historical data matrix：

```math
X=[x_1,\dots,x_n]\in\mathbb R^{d\times n}
```

样本协方差：

```math
S=\frac{1}{n}XX^T
```

Eigenvalue problem：

```math
S u_i = \lambda_i u_i
```

Eigenvalues 排序：

```math
\lambda_1\ge \lambda_2\ge \cdots \ge \lambda_d
```

第 i 个主成分方向 `u_i` 的投影方差：

```math
Var(u_i^Tx)=u_i^TSu_i=\lambda_i
```

PCA variance decomposition：

```math
\sum_{i=1}^{d}Var(u_i^TX)
=
\sum_{i=1}^{d}\lambda_i
=
Tr(S)
=
Var(X)
```

## 2.4 Normal subspace and residual subspace

选择前 p 个 PCs：

```math
U_p=[u_1,\dots,u_p]
```

Normal operating subspace：

```math
\mathcal S_p = span(U_p)
```

Projection / score：

```math
y=U_p^Tx
```

Reconstruction：

```math
\hat x=U_p y = U_pU_p^Tx
```

Residual：

```math
r=x-\hat x=(I-U_pU_p^T)x
```

故障检测逻辑：

- 正常数据主要落在 principal subspace。
- 故障可能破坏变量相关结构，使 reconstruction residual 增大。
- residual subspace 对 fault sensitive。

## 2.5 PCA monitoring statistics

Squared Prediction Error / Q statistic：

```math
SPE=Q=\|x-\hat x\|^2
```

意义：样本距离 normal subspace 的距离。大 SPE 表示异常或未建模变化。

Hotelling T²：

```math
T^2 = y^T \Lambda_p^{-1} y
```

其中：

```math
\Lambda_p=diag(\lambda_1,\dots,\lambda_p)
```

意义：样本在 principal subspace 内是否过远。大 T² 表示虽然还在 normal subspace 内，但 operating condition 异常。

SPE vs T²：

- SPE：检测 normal subspace 外的异常。
- T²：检测 normal subspace 内的异常强度。
- 两者互补。

## 2.6 Direct PCA algorithm

Offline training：

1. 收集 fault-free data `X`。
2. Center / standardize。
3. 计算 `XX^T` 或 covariance。
4. Eigen-decomposition，取 top p eigenvectors `U_p`。
5. 计算正常 score、SPE、T² limits。

Online monitoring：

1. 新样本 `x` 标准化。
2. Score：

```math
y=U_p^Tx
```

3. Reconstruction：

```math
\hat x=U_pU_p^Tx
```

4. Residual：

```math
r=x-\hat x
```

5. 判断 SPE/T² 是否越限。

## 2.7 Dual PCA

SVD：

```math
X=U\Sigma V^T
```

关系：

```math
XV=U\Sigma
```

所以：

```math
U=XV\Sigma^{-1}
```

Dual PCA 使用 `X^TX` 的 eigenvectors `V` 代替 `XX^T` 的 eigenvectors `U`。

优点：

- 当 sensor dimension `d` 很大，而 samples `n` 较小时，`X^TX` 更小。
- 适合大型 sensor arrays。

缺点：

- 某些 reconstruction step 仍依赖原始高维 `d`。
- Online explicit reconstruction 可能昂贵。

## 2.8 选择主成分数 p

常见方法：

Cumulative variance：

```math
\frac{\sum_{i=1}^{p}\lambda_i}{\sum_{i=1}^{d}\lambda_i}
\ge \rho
```

例如 `rho=90%` 或 `95%`。

Scree plot：

- 找 eigenvalue 曲线 elbow。

Cross-validation：

- 选择 fault detection performance 最优的 p。

注意：

- p 太小：normal variation 被丢掉，FAR 高。
- p 太大：fault direction 被吸进 normal subspace，MAR 高。

## 2.9 PCA 优劣势和使用场景

优点：

- 简单、稳定、可解释。
- 无监督，只需 fault-free data。
- 能去噪和去相关。
- 适合线性相关强的过程。

缺点：

- 只看 variance，不看 fault label。
- 最大 variance 不一定是故障信息。
- 微小但关键 fault 可能在低 variance direction。
- 只能处理线性 subspace。

使用场景：

- 工业过程监控。
- 初步 feature compression。
- fault-free normal model 建模。
- 与 SPE/T² 结合做 online monitoring。

## 2.10 Kernel PCA

普通 PCA 处理线性结构。若 normal data 位于 nonlinear manifold，需要 KPCA。

映射：

```math
\Phi:x\rightarrow \mathcal H
```

Kernel：

```math
K(x_i,x_j)=\Phi(x_i)^T\Phi(x_j)
```

KPCA 流程：

```text
Original sensor space X
-> Feature space H by implicit mapping
-> PCA in H
-> low-dimensional nonlinear normal subspace
```

常见 kernel：

Linear：

```math
K(x_i,x_j)=x_i^Tx_j
```

Polynomial：

```math
K(x_i,x_j)=(1+x_i^Tx_j)^p
```

RBF：

```math
K(x_i,x_j)=\exp\left(-\frac{\|x_i-x_j\|^2}{2\sigma^2}\right)
```

或：

```math
K(x_i,x_j)=\exp(-\gamma\|x_i-x_j\|^2)
```

KPCA online projection：

```math
y
=
\Sigma^{-1}V^T K(X,x)
```

因为：

```math
\Phi(X)^T\Phi(x)=K(X,x)
```

优点：

- 能处理非线性 process dynamics。
- 不需显式计算高维 feature。
- 对 nonlinear fault detection 更强。

缺点：

- Kernel 和参数选择敏感。
- Explicit reconstruction 难。
- Kernel matrix 随 training samples 增大。
- 新工况 extrapolation 可能不稳定。

## 2.11 Supervised PCA

普通 PCA 不看 label：

```text
PCA asks: which directions have maximum variance?
```

但 FDD 更关心：

```text
Which directions are related to fault?
```

SPCA 使用 `(X,Y)`：

- `X`：sensor data。
- `Y`：fault labels、fault indicator 或 degradation index。

目标：找低维 subspace `S=U^TX`，尽量保留与 `Y` 相关的信息。

SPCA 简单算法：

1. 对每个 sensor feature `j` 计算与 fault target 的 regression coefficient：

```math
s_j=\frac{X_j^TY}{\sqrt{X_j^TX_j}}
```

2. 保留：

```math
|s_j|>\delta
```

3. 在保留的 fault-sensitive sensors 上做 PCA。
4. 用提取的 PCs 做 regression 或 classification。

优点：

- 比 PCA 更关注故障信息。
- 可筛掉与故障无关的大方差正常变化。

缺点：

- 需要 label。
- label 噪声会影响结果。
- 只看单变量相关时可能遗漏组合故障特征。

## 2.12 HSIC-SPCA

HSIC 衡量两个随机变量是否相关。核心思想：若两个变量独立，则它们在 RKHS 中所有连续有界函数也不相关。

Centered matrix：

```math
H=I-\frac{1}{n}ee^T
```

Kernel matrices：

```math
K_{ij}=k(x_i,x_j),\qquad B_{ij}=b(y_i,y_j)
```

HSIC estimator：

```math
HSIC(Z)=\frac{1}{(n-1)^2}Tr(KHBH)
```

SPCA objective：

```math
\max_U Tr(U^TXHBHX^TU)
```

subject to：

```math
U^TU=I
```

解：

```math
XHBHX^T
```

的 top eigenvectors。

如果 `B=I`，则退化为标准 PCA 形式。因此 HSIC-SPCA 是 PCA 的监督推广。

## 2.13 Kernel SPCA

若 dependence 是非线性的，使用 kernel：

```math
\beta=\Phi(X)U
```

Kernel SPCA 最终得到 generalized eigenvalue problem，解与 kernel matrices 有关。

优点：

- 同时处理 label relevance 和 nonlinearity。

缺点：

- 更复杂。
- 参数选择难。
- 需要足够 labelled data。

## 2.14 Metric learning and SDR

Standard Euclidean distance：

```math
d(x_i,x_j)=\|x_i-x_j\|
```

问题：所有方向等权，可能把正常工况变化和故障微小变化同等看待。

Metric learning 学习：

```math
d_A(x_i,x_j)
=
\sqrt{(x_i-x_j)^TA(x_i-x_j)}
```

其中 `A` 是 positive semidefinite matrix。

若：

```math
A=UU^T
```

则：

```math
d_A(x_i,x_j)^2
=
(U^Tx_i-U^Tx_j)^T(U^Tx_i-U^Tx_j)
```

所以 metric learning 等价于学习一个 fault-sensitive projection。

Sufficient Dimension Reduction：

希望：

```math
p(y\mid x)=p(y\mid U^Tx)
```

也就是降维后不丢失对 fault label 的全部预测信息。

## 2.15 FDA

FDA 是 supervised dimensionality reduction。

目标：投影后 class separation 最大，class scatter 最小。

Binary case：

```math
z=w^Tx
```

两类均值：

```math
\mu_0=\frac{1}{n_0}\sum_{i:y_i=0}x_i
```

```math
\mu_1=\frac{1}{n_1}\sum_{i:y_i=1}x_i
```

Between-class scatter：

```math
S_B=(\mu_0-\mu_1)(\mu_0-\mu_1)^T
```

Within-class scatter：

```math
S_W=\Sigma_0+\Sigma_1
```

Objective：

```math
J(w)=\frac{w^TS_Bw}{w^TS_Ww}
```

Generalized eigenvalue problem：

```math
S_Bw=\lambda S_Ww
```

Multi-class：

```math
W=[w_1,\dots,w_k]
```

Maximize：

```math
Tr(W^TS_BW)
```

Minimize：

```math
Tr(W^TS_WW)
```

优点：

- 明确用于 class separation。
- 适合 labelled fault isolation。
- 投影后维度低，分类器更简单。

缺点：

- 需要 label。
- 类内 covariance 估计要可靠。
- 对非 Gaussian / nonlinear 分布有限。

PCA vs FDA：

- PCA：无监督，最大 total variance。
- FDA：有监督，最大 class separation。

## 2.16 MDS

Multidimensional Scaling 目标：在低维空间中保持样本间距离或相似度。

给定：

```math
X=[x_1,\dots,x_n]\in\mathbb R^{d\times n}
```

希望找到：

```math
Y=[y_1,\dots,y_n]\in\mathbb R^{p\times n}
```

其中：

```math
p\le d
```

Classical MDS objective：

```math
\min_Y \sum_{i=1}^{n}\sum_{j=1}^{n}
(x_i^Tx_j-y_i^Ty_j)^2
```

矩阵形式：

```math
\min_Y \|X^TX-Y^TY\|_F^2
```

Eigen-decomposition：

```math
X^TX=V\Delta V^T
```

最终 embedding：

```math
Y=\Delta^{1/2}V^T
```

若从 distance matrix 出发，用 double centering：

```math
K=-\frac12HDH
```

其中：

```math
H=I-\frac{1}{n}ee^T
```

MDS 在 FDD 中：

```text
High-dimensional sensor data
-> distance matrix
-> MDS embedding
-> visual fault cluster / isolation
```

优点：

- 可直接用 distance / dissimilarity matrix。
- 适合可视化。
- Classical Euclidean MDS 与 PCA 有深层等价。

缺点：

- Classical MDS 本质线性。
- 对距离定义敏感。
- 新样本 out-of-sample 处理不如 PCA 直接。

## 2.17 Generalized / Kernel MDS

Classical MDS 使用 Euclidean distance。Generalized MDS 可以替换为更一般的 kernel/dissimilarity。

Kernel matrix：

```math
K=-\frac12HDH
```

Eigen-decomposition：

```math
K=V\Delta V^T
```

Embedding：

```math
Y=\Delta^{1/2}V^T
```

与 PCA/KPCA 的关系：

- Euclidean classical MDS 等价于 PCA。
- Generalized classical MDS 与 Kernel PCA 对应。

## 2.18 Isomap

MDS 的限制：用 Euclidean distance，只能表达线性结构。若 data 位于 curved manifold 上，例如 Swiss roll，Euclidean distance 会穿过 manifold，而非沿 manifold 表面。

Isomap 使用 geodesic distance。

Geodesic distance：沿 manifold 表面的最短路径距离。

Isomap algorithm：

1. Build kNN graph：
   每个点连接 k 个最近邻。

2. Edge weight：
   用局部 Euclidean distance。

3. Approximate geodesic：
   用 shortest path：

```math
D_{ij}^{(g)}
=
\min_r
\sum_{\ell}
\|r_{\ell}-r_{\ell+1}\|
```

4. Double centering：

```math
K=-\frac12HD^{(g)}H
```

5. Eigen-decomposition：

```math
K=V\Delta V^T
```

6. Embedding：

```math
Y=\Delta^{1/2}V^T
```

FDD 逻辑：

- Normal data 可能位于 nonlinear manifold。
- Fault data 可能偏离 manifold 或形成新 cluster。
- Isomap 展开 manifold 后，fault clusters 更容易分离。

优点：

- 能处理 nonlinear manifold。
- 保持 global geodesic structure。
- 适合 nonlinear fault isolation。

缺点：

- k 太小会断图，k 太大会破坏 manifold。
- 对噪声和 outliers 敏感。
- 计算 shortest paths 成本高。
- 新样本 embedding 不直接。

## 2.19 Kernel Isomap

标准 Isomap 的 geodesic distance matrix 可能导致 kernel 非 positive semidefinite，产生 complex eigenvalues。

课程给出调整：

```math
K' = K(D^2)+2cK(D)+\frac12c^2H
```

选择：

```math
c\ge c^*
```

其中 `c*` 来自特定 `2n x 2n` block matrix 的最大实特征值。

作用：保证 kernel positive semidefinite，使 embedding 数值稳定。

---

# Topic 3: Support Vector Machines and Perceptron

## 3.1 主题边界

Topic 3 讲的是最大间隔分类器。它和 Topic 1 的概率分类器不同：

- Topic 1：Bayes / LDA / QDA / logistic regression，偏概率和 distribution。
- Topic 3：SVM / Perceptron，偏几何间隔和 constrained optimization。

对应 Curriculum 小项：

- Hard Margin SVM：3.5-3.7。
- Soft Margin SVM：3.8-3.11。
- Kernel Trick and Perceptron：3.12 和 3.14。

## 3.2 SVM 在 FDD 中的位置

FDD pipeline：

```text
Sensor signal
-> Feature extraction / dimensionality reduction
-> Feature vector x
-> SVM classifier
-> normal / fault / fault type
```

SVM 适合：

- feature dimension 高。
- training samples 相对有限。
- fault classes 在 feature space 中有清晰边界。
- 希望分类器有强泛化能力。

## 3.3 基本概念

Hyperplane：

```math
\beta^Tx+\beta_0=0
```

Decision function：

```math
f(x)=\beta^Tx+\beta_0
```

Label：

```math
y_i\in\{-1,+1\}
```

分类规则：

```math
\hat y=sign(\beta^Tx+\beta_0)
```

Support vectors：

- 离 hyperplane 最近的样本。
- 决定 margin 和 hyperplane 位置。
- 删除其他非 support vectors，边界不变。

Margin：

```math
d_i=\frac{y_i(\beta^Tx_i+\beta_0)}{\|\beta\|}
```

SVM 目标：

```math
\max_{\beta,\beta_0}\min_i d_i
```

## 3.4 SVM 优劣势

优点：

- 高维空间有效。
- 只依赖 support vectors，内存相对节省。
- Kernel 灵活，可处理非线性边界。
- Convex optimization，有全局最优。

缺点：

- 大数据训练慢。
- 噪声大、类别重叠严重时性能下降。
- 原生不输出概率。
- Kernel、C、gamma 选择敏感。

## 3.5 Hard Margin SVM

适用条件：数据完全线性可分，且噪声很小。

Scale invariance：

```math
\beta^Tx+\beta_0=0
```

乘任意常数边界不变。为了固定 scale，要求 closest points 满足：

```math
\min_i y_i(\beta^Tx_i+\beta_0)=1
```

于是所有点满足：

```math
y_i(\beta^Tx_i+\beta_0)\ge 1
```

Margin：

```math
M=\frac{1}{\|\beta\|}
```

最大化 margin 等价于：

```math
\min_{\beta,\beta_0}\frac12\|\beta\|^2
```

subject to：

```math
y_i(\beta^Tx_i+\beta_0)\ge 1,\quad i=1,\dots,N
```

## 3.6 Lagrangian and KKT

Hard margin Lagrangian：

```math
L(\beta,\beta_0,\alpha)
=
\frac12\|\beta\|^2
-
\sum_{i=1}^{N}
\alpha_i
\left[
y_i(\beta^Tx_i+\beta_0)-1
\right]
```

KKT conditions：

1. Stationarity：

```math
\nabla_{\beta,\beta_0}L=0
```

2. Primal feasibility：

```math
y_i(\beta^Tx_i+\beta_0)\ge1
```

3. Dual feasibility：

```math
\alpha_i\ge0
```

4. Complementary slackness：

```math
\alpha_i
\left[
y_i(\beta^Tx_i+\beta_0)-1
\right]
=0
```

由 stationarity：

```math
\frac{\partial L}{\partial \beta}=0
\Rightarrow
\beta=\sum_{i=1}^{N}\alpha_iy_ix_i
```

```math
\frac{\partial L}{\partial \beta_0}=0
\Rightarrow
\sum_{i=1}^{N}\alpha_iy_i=0
```

解释：最优 hyperplane normal vector 是 training points 的线性组合；只有 support vectors 的 `alpha_i>0`。

## 3.7 Hard margin dual

Dual objective：

```math
\max_\alpha
\sum_{i=1}^{N}\alpha_i
-
\frac12
\sum_{i=1}^{N}\sum_{j=1}^{N}
\alpha_i\alpha_jy_iy_jx_i^Tx_j
```

subject to：

```math
\alpha_i\ge0,\qquad
\sum_{i=1}^{N}\alpha_iy_i=0
```

Matrix form：

```math
\max_\alpha
\alpha^Te-\frac12\alpha^TS\alpha
```

where：

```math
S_{ij}=y_iy_jx_i^Tx_j
```

Decision function：

```math
f(x)
=
\sum_{i=1}^{N}\alpha_iy_i x_i^Tx+\beta_0
```

实际只需 support vectors：

```math
f(x)
=
\sum_{i\in SV}\alpha_iy_i x_i^Tx+\beta_0
```

## 3.8 Soft Margin SVM

真实 fault data 有噪声和重叠，hard margin 常无解或过拟合。

引入 slack variable：

```math
\xi_i\ge0
```

Relaxed constraint：

```math
y_i(\beta^Tx_i+\beta_0)\ge1-\xi_i
```

Objective：

```math
\min_{\beta,\beta_0,\xi}
\frac12\|\beta\|^2
+
C\sum_{i=1}^{N}\xi_i
```

subject to：

```math
y_i(\beta^Tx_i+\beta_0)\ge1-\xi_i,\quad
\xi_i\ge0
```

解释：

- 第一项让 margin 大。
- 第二项惩罚 margin violation。
- `C` 控制二者权衡。

## 3.9 Slack variable interpretation

```math
\xi_i=0
```

正确分类，且在 margin 外或 margin 上。

```math
0<\xi_i<1
```

进入 margin，但仍正确分类。

```math
\xi_i=1
```

落在 decision boundary 上。

```math
\xi_i>1
```

错误分类。

## 3.10 C parameter

Low C：

- 错分惩罚小。
- margin 更宽。
- 容忍训练错误。
- 泛化可能更好。
- 适合 noisy data。

High C：

- 错分惩罚大。
- margin 更窄。
- 尽量拟合训练数据。
- 对 outlier 敏感，容易过拟合。

## 3.11 Soft margin dual

Soft margin dual objective 与 hard margin 相同：

```math
\max_\alpha
\sum_i\alpha_i
-
\frac12
\sum_i\sum_j
\alpha_i\alpha_jy_iy_jx_i^Tx_j
```

区别在 constraint：

```math
0\le\alpha_i\le C
```

and：

```math
\sum_i\alpha_iy_i=0
```

这个 box constraint 来自 slack variable 的 KKT：

```math
\mu_i=C-\alpha_i
```

且：

```math
\mu_i\ge0 \Rightarrow \alpha_i\le C
```

## 3.12 Kernel trick

若原空间线性不可分，映射到高维：

```math
\Phi(x)
```

Dual 中只出现 dot product：

```math
x_i^Tx_j
```

替换为：

```math
K(x_i,x_j)=\Phi(x_i)^T\Phi(x_j)
```

Kernelized decision：

```math
f(x)=
\sum_{i\in SV}\alpha_iy_iK(x_i,x)+\beta_0
```

优点：不显式计算高维坐标。

常见 kernels：

Linear：

```math
K(x_i,x_j)=x_i^Tx_j
```

Polynomial：

```math
K(x_i,x_j)=(\gamma x_i^Tx_j+c)^d
```

RBF：

```math
K(x_i,x_j)=\exp(-\gamma\|x_i-x_j\|^2)
```

Sigmoid：

```math
K(x_i,x_j)=\tanh(\alpha x_i^Tx_j+c)
```

RBF 参数：

- `gamma` 大：单个样本影响范围小，边界复杂，可能过拟合。
- `gamma` 小：影响范围大，边界平滑，可能欠拟合。

## 3.13 Hard vs Soft Margin 总结

Hard margin：

- 必须完全线性可分。
- 不允许 slack。
- Constraint：`alpha_i >= 0`。
- 对 outliers 很敏感。

Soft margin：

- 允许 margin violation。
- 有 slack variables。
- Constraint：`0 <= alpha_i <= C`。
- 更适合真实故障数据。

## 3.14 Perceptron

Perceptron 是线性分类器，是 SVM 的基础对比。

Decision：

```math
\hat y=sign(w^Tx+b)
```

若误分类：

```math
y_i(w^Tx_i+b)\le0
```

Update：

```math
w\leftarrow w+\eta y_ix_i
```

```math
b\leftarrow b+\eta y_i
```

优点：

- 简单。
- 线性可分时可收敛。
- 在线学习容易。

缺点：

- 没有最大 margin。
- 对噪声敏感。
- 线性不可分时不收敛。

Perceptron vs SVM：

- Perceptron 找一个可分边界。
- SVM 找最大间隔边界。
- SVM 泛化通常更好。

---

# Topic 4: Model-Based Fault Detection

## 4.1 主题边界

Topic 4 讲模型驱动故障检测：利用系统模型产生 residual 或 state estimate，再做统计判断。

完整覆盖：

- residual generation
- parity method in time/frequency domain
- observer-based residual
- Luenberger observer
- Kalman filter
- ESO / LESO
- UIO
- NDO
- CUSUM
- ARL

对应 Curriculum 小项：

- Model based residual generation：4.2-4.3。
- Parity method in frequency/time domain：4.4。
- Observer-based methods (UIO, KF, ESO, NDO)：4.5-4.10。
- CUSUM and Average Run Length (ARL)：4.12-4.16。

控制重构本身不在 Topic 4 完整展开，而在 Topic 5/6。

## 4.2 Model-based FDD 基本模型

连续时间 LTI 模型：

```math
\dot x=Ax+Bu+Ed
```

```math
y=Cx
```

其中：

- `x`：state。
- `u`：known input。
- `d`：unknown disturbance / fault。
- `y`：measured output。

Fault models：

Actuator fault：

```math
\dot x=Ax+B(u+f_a)
```

Sensor fault：

```math
y=Cx+F_sf_s
```

Plant fault：

```math
\dot x=A_fx+Bu
```

Model-based FDD 的核心是 residual：

```math
r=y-\hat y
```

Normal 时：

```math
r\approx0
```

Fault 时：

```math
r \text{ becomes large or structured}
```

## 4.3 Residual 的设计目标

好 residual 需要：

- 对 fault sensitive。
- 对 noise robust。
- 对 unknown disturbance insensitive。
- 可用于 isolation。
- 计算实时可行。

矛盾：

- 阈值低：检测快，但 FAR 高。
- 阈值高：FAR 低，但 MAR 和 delay 高。

## 4.4 Parity method

Parity relation 的思想：利用 input-output model 构造 nominal 情况下必须满足的代数关系。

一般形式：

```math
r = Wy - V u
```

或更抽象：

```math
r = Q(q)y - P(q)u
```

若系统正常且模型准确：

```math
r=0
```

若 fault 破坏关系：

```math
r\neq0
```

Time-domain parity：

- 用时域差分/状态空间关系构造 residual。
- 适合实时实现。

Frequency-domain parity：

- 用 transfer function 关系构造 residual。
- 可分析频率响应和故障敏感性。

优点：

- 不一定需要完整状态估计。
- 结构清楚。
- 可设计 residual direction 用于 isolation。

缺点：

- 依赖模型准确性。
- 噪声和未建模动态会造成 residual。
- 高阶系统 parity relation 构造复杂。

## 4.5 Observer-based residual generation

如果 state 不可测，用 observer：

```math
\dot{\hat x}=A\hat x+Bu+L(y-C\hat x)
```

Predicted output：

```math
\hat y=C\hat x
```

Residual：

```math
r=y-\hat y
```

Error：

```math
e=x-\hat x
```

Error dynamics：

```math
\dot e=(A-LC)e
```

选择 `L` 使 `A-LC` stable / Hurwitz。

优点：

- 可估计不可测 state。
- residual 与系统动态一致。
- 可调 observer poles。

缺点：

- 要求 observability / detectability。
- 对 unknown input / fault 未处理时 residual 和 state estimate 会被污染。

## 4.6 Kalman filter

Kalman filter 是在噪声下的最优 observer。

离散模型：

```math
x_{k+1}=Ax_k+Bu_k+w_k
```

```math
y_k=Cx_k+v_k
```

其中：

```math
w_k\sim \mathcal N(0,Q),\qquad v_k\sim \mathcal N(0,R)
```

Kalman filter 在 model prediction 和 measurement correction 之间权衡：

- model noise 小，信模型更多。
- measurement noise 小，信传感器更多。

适用：

- 噪声显著。
- 统计噪声 covariance 可估计。
- 需要 state estimate 和 residual。

限制：

- 线性 Gaussian 假设。
- Q/R 选错会影响 residual。
- fault 若被 filter 当作 noise 吸收，可能降低检测敏感性。

## 4.7 ESO / LESO

Extended State Observer 把 unknown input / disturbance 当作扩展状态。

系统：

```math
\dot x=Ax+Bu+Ed(t)
```

扩展状态：

```math
x_e=
\begin{bmatrix}
x\\d
\end{bmatrix}
```

假设：

```math
\dot d \approx 0
```

Augmented model：

```math
\begin{bmatrix}
\dot x\\
\dot d
\end{bmatrix}
=
\begin{bmatrix}
A&E\\
0&0
\end{bmatrix}
\begin{bmatrix}
x\\d
\end{bmatrix}
+
\begin{bmatrix}
B\\0
\end{bmatrix}u
```

优点：

- 可以估计 lumped disturbance / fault。
- 适合慢变扰动、step fault。
- 可与 MPC 做 active fault accommodation。

缺点：

- 假设 disturbance 慢变。
- 快速剧烈变化时估计滞后。
- 扩展系统 observability 可能不满足。

## 4.8 UIO

Unknown Input Observer 不尝试估计未知输入，而是让 error dynamics 与未知输入完全解耦。

系统：

```math
\dot x=Ax+Bu+Ed
```

```math
y=Cx
```

UIO structure：

```math
\dot z=Fz+TBu+Ky
```

```math
\hat x=z+Hy
```

Error：

```math
e=x-\hat x
```

由：

```math
\hat x=z+HCx
```

得到：

```math
e=(I-HC)x-z
```

希望未知输入 `d` 不进入 error dynamics。关键条件：

```math
(I-HC)E=0
```

等价于：

```math
HCE=E
```

解存在 iff：

```math
rank(E)=rank(CE)
```

若满足，可选：

```math
H=E(CE)^\dagger
```

Transformation：

```math
T=I-HC
```

选择：

```math
F=(I-HC)A-K_1C
```

```math
K=K_1+FH
```

再用 pole placement 选 `K_1` 使 `F` Hurwitz。

UIO 优点：

- 状态估计不被 unknown input 污染。
- 对任意变化的 `d(t)` 仍可保持估计解耦。
- 适合 resilient estimation。

UIO 缺点：

- rank condition 严格。
- 需要 enough output information。
- 被 decouple 的 fault 若要用于 control accommodation，需要另外 algebraic reconstruction。

## 4.9 NDO

NDO 是 Nonlinear Disturbance Observer，属于 Curriculum 中 observer-based methods 的一类。它的作用是针对非线性系统估计 lumped disturbance / fault，并据此生成 residual 或补偿量。

典型非线性系统可写成：

```math
\dot x=f(x,u)+E(x)d
```

```math
y=h(x)
```

其中：

- `f(x,u)` 是已知非线性动力学。
- `d` 是未知扰动、故障或 attack effect。
- `E(x)` 是未知输入进入系统的方向。

NDO 的目标是构造：

```math
\hat d \rightarrow d
```

使 disturbance estimation error：

```math
e_d=d-\hat d
```

收敛或有界。

一种通用理解是把 NDO 写成“非线性模型预测 + 输出误差校正”：

```math
\dot{\hat x}
=
f(\hat x,u)+E(\hat x)\hat d
+
L_x(y-\hat y)
```

```math
\dot{\hat d}
=
L_d(y-\hat y)
```

```math
\hat y=h(\hat x)
```

如果 `d` 缓慢变化或分段常值，并且 observer gains `L_x,L_d` 选得合适，则 `hat d` 可以跟踪未知扰动或故障。

NDO residual 可以取：

```math
r_d=\hat d
```

或：

```math
r_y=y-\hat y
```

诊断逻辑：

- 正常时 `hat d` 接近 0 或 nominal disturbance level。
- 故障/attack 时 `hat d` 出现显著偏移。
- 可以用 threshold、CUSUM 或 ARL 方法判断。

NDO vs ESO vs UIO：

- ESO/LESO：把 disturbance 当扩展状态估计，常假设慢变。
- NDO：面向 nonlinear dynamics，可以显式利用非线性模型结构估计 disturbance。
- UIO：不估计 unknown input，而是通过结构条件让 state estimation error 与 unknown input 解耦。

优点：

- 适合非线性 CPS、机械系统、机器人、飞行器等。
- 可同时用于 detection 和 compensation。
- 比单纯 linear observer 更贴近 nonlinear plant。

缺点：

- 依赖非线性模型质量。
- 增益设计和稳定性证明更复杂。
- 快速变化或不可辨识 disturbance 仍难估计。

## 4.10 Observability and detectability

Observer pole placement 要求 pair observable。

Observability matrix：

```math
\mathcal O=
\begin{bmatrix}
C\\CA\\ \vdots \\ CA^{n-1}
\end{bmatrix}
```

Full rank：

```math
rank(\mathcal O)=n
```

若不完全 observable，但不可观 modes 都稳定，则 detectable，observer error 仍可收敛。

UIO 中对应检查：

```math
rank
\begin{bmatrix}
C\\
\lambda I-A_1
\end{bmatrix}
=n
```

对所有 unstable eigenvalues `lambda`。

## 4.11 Statistical decision and hypothesis testing

模型残差或传感器序列仍有噪声，因此不能只靠单点阈值。引入 hypothesis testing。

Null hypothesis：

```math
H_0: \text{system is normal, parameter } \theta_0
```

Alternative hypothesis：

```math
H_1: \text{fault occurred, parameter } \theta_1
```

Likelihood ratio：

```math
r(x)=\frac{f_{\theta_1}(x)}{f_{\theta_0}(x)}
```

Log-likelihood increment：

```math
s(x)=\ln r(x)=
\ln\frac{f_{\theta_1}(x)}{f_{\theta_0}(x)}
```

解释：

- `s(x)>0`：当前数据支持 fault。
- `s(x)<0`：当前数据支持 normal。

## 4.12 CUSUM offline derivation

假设 fault 在未知时间 `n_c` 发生。对数据序列：

```math
x_1,x_2,\dots,x_k
```

若 fault time 已知，则 total log-likelihood ratio：

```math
S_{n_c}^{k}
=
\sum_{i=n_c}^{k}
\ln\frac{f_{\theta_1}(x_i)}{f_{\theta_0}(x_i)}
=
\sum_{i=n_c}^{k}s(x_i)
```

但 `n_c` 不知道，所以 maximum likelihood estimate：

```math
g(k)=
\max_{1\le n_c\le k}
\sum_{i=n_c}^{k}s(x_i)
```

Decision：

```math
g(k)>h \Rightarrow \text{alarm}
```

其中 `h` 是 threshold。

## 4.13 Recursive CUSUM

Offline CUSUM 每步要遍历所有历史数据，计算量太大。递推形式：

```math
g(k)=\max(0,g(k-1)+s(x_k))
```

解释：

- Normal 下，`s(x_k)` 平均为负，累加值被 max floor 限制在 0。
- Fault 后，`s(x_k)` 平均为正，`g(k)` 持续上升，直到超过 `h`。

Change-time counter：

```math
N(k)=N(k-1)\mathbf 1_{\{g(k-1)>0\}}+1
```

Alarm time：

```math
k_a
```

Estimated change time：

```math
\hat n_c=k_a-N(k_a)
```

## 4.14 Gaussian CUSUM

检测 Gaussian mean shift：

```math
H_0:x_k\sim\mathcal N(\mu_0,\sigma^2)
```

```math
H_1:x_k\sim\mathcal N(\mu_1,\sigma^2)
```

Increment：

```math
s(x_k)
=
\ln\frac{f_{\mu_1}(x_k)}{f_{\mu_0}(x_k)}
```

推导后：

```math
s(x_k)
=
\frac{\mu_1-\mu_0}{\sigma^2}
\left(
x_k-\frac{\mu_1+\mu_0}{2}
\right)
```

中点：

```math
M=\frac{\mu_1+\mu_0}{2}
```

若 `x_k` 越过中点向 fault mean 移动，increment 变正。

Under H1：

```math
\mu_s=
\frac{(\mu_1-\mu_0)^2}{2\sigma^2}>0
```

```math
\sigma_s^2=
\frac{(\mu_1-\mu_0)^2}{\sigma^2}
```

Under H0：

```math
\mu_s=
-
\frac{(\mu_1-\mu_0)^2}{2\sigma^2}<0
```

所以 CUSUM 在 normal 下贴近 0，在 fault 下正漂移。

## 4.15 Alarm time factors

Alarm delay：

```math
\tau_D=k_a-n_c
```

影响因素：

- Fault magnitude 大：`mu_1-mu_0` 大，drift 大，报警快。
- Fault magnitude 小：drift 小，报警慢。
- Noise 高：`sigma^2` 大，delay 不确定。
- Threshold 高：false alarm 少，但 detection delay 大。

## 4.16 ARL

Average Run Length 是阈值设计指标。

Average time to detect：

```math
\tau_D
```

在 H1 下计算，希望小。

Average time between false alarms：

```math
\tau_F
```

在 H0 下计算，希望大。

课程给出近似公式：

```math
\hat L(\mu_s,\sigma_s,h)
=
\frac{\sigma_s^2}{2\mu_s^2}
\left[
\exp(-arg)+arg-1
\right]
```

其中：

```math
arg=
\frac{2\mu_s h}{\sigma_s^2}
+
\frac{2.2232\mu_s}{\sigma_s}
```

设计流程：

1. 确定可接受 false alarm 间隔，例如 `tau_F >= 100000 samples`。
2. 在 H0 下计算 `mu_s, sigma_s`。
3. 调整 threshold `h`，使 ARL 满足 false alarm 要求。
4. 在 H1 下计算 `tau_D`，确认检测足够快。

## 4.17 Topic 4 方法选择

Parity：

- 适合 input-output model 清楚、状态估计不必要的系统。
- 简单直接。

Observer residual：

- 适合状态不可测但模型可靠的系统。
- 可结合 Luenberger / KF。

ESO/LESO：

- 适合 unknown disturbance 慢变、需要估计故障大小的系统。

UIO：

- 适合希望 state estimate 不受 unknown input 污染的系统。
- 需要满足 rank condition。

NDO：

- 适合 nonlinear plant 中估计 lumped disturbance / nonlinear fault effect。
- 需要较可靠非线性模型和可辨识的 disturbance channel。

CUSUM：

- 适合在线检测 abrupt change。
- 比单点阈值更能积累小故障证据。

---

# Topic 5: Fault Tolerant Control Strategy

## 5.1 主题边界

Topic 5 讲故障发生后如何保持控制系统稳定和性能。

完整覆盖：

- passive FTC and active FTC
- robust / adaptive control
- redundancy
- state-space control foundations
- observer and separation principle
- reconfigurable control systems
- model matching
- exact / approximate model matching
- actuator / plant / sensor fault reconfiguration
- virtual actuator / virtual sensor
- bumpless transfer / elimination of transients

Topic 6 中的 MPC safe constraints 不在这里完整展开。

对应 Curriculum 小项：

- Introduction to Robust/Adaptive control：5.4-5.6。
- Reconfigurable Control Systems：5.10。
- Fault-Tolerant Control methods：Model Matching 在 5.11-5.15；Transient Elimination / Bumpless Transfer 在 5.17-5.20；Virtual Actuator/Sensor 在 5.16。

说明：5.7-5.9 的 state-space、LQR、observer 不是新增考试主题，只是为了理解 reconfigurable controllers、model matching 和 bumpless transfer 的必要控制基础。

## 5.2 Fault locations

标准闭环：

```text
Controller -> Actuator -> Plant -> Sensor -> Controller
```

故障位置：

Actuator fault：

```math
B\rightarrow B_f
```

含义：输入通道能力改变，例如 loss of effectiveness、stuck actuator。

Plant fault：

```math
A\rightarrow A_f
```

含义：内部动力学改变，例如 friction、mass、stiffness 变化。

Sensor fault：

```math
C\rightarrow C_f
```

含义：测量路径改变，例如 sensor loss、bias、scaling error。

## 5.3 Redundancy

FTC 的物理前提是 redundancy。

Hardware redundancy：

- 备用执行器。
- 多传感器。
- 冗余通道。
- 例子：飞机有多个液压 actuator。

Analytical redundancy：

- 利用数学模型和剩余传感器估计丢失信息。
- 例子：double tank 中一个水位传感器坏了，但系统仍可由另一个传感器和模型估计完整状态。

关键限制：

- 若 actuator fault 使系统不可控，无法通过算法恢复控制能力。
- 若 sensor fault 使系统不可观，无法通过算法恢复丢失信息。
- FTC 不能创造不存在的物理通道。

## 5.4 Passive FTC

Passive FTC 使用固定 controller：

```math
u=K(y,r)
```

controller 设计时就考虑一定范围内的 fault / uncertainty。

通常依赖 robust control。

优点：

- 不依赖实时故障诊断。
- 结构简单。
- 避免误诊导致错误切换。
- 实时实现风险低。

缺点：

- 保守。
- 只适合预设 fault range。
- 严重故障或未覆盖故障性能差。

使用场景：

- 故障范围可预估。
- 系统安全等级高，不希望频繁切换。
- 计算资源有限。

## 5.5 Active FTC

Active FTC 使用 FDD 结果重构控制：

```text
Residual generation
-> Decision system
-> Controller switching / reconfiguration
```

形式：

- 切换到 backup controller。
- 更新 controller gain。
- 重新分配 actuator command。
- 用 virtual sensor / actuator 适配 nominal controller。

优点：

- 可以处理更大范围和更严重故障。
- 故障特异性强。
- 性能可比 passive FTC 更好。

缺点：

- 依赖 FDD 准确性和速度。
- 误诊会导致错误 controller。
- 切换可能产生 transient。
- 架构复杂。

## 5.6 Robust vs adaptive control

Robust control：

- controller 固定。
- 对不确定性集合保持稳定。
- 把 fault 当 bounded uncertainty。
- 性能保守。

Adaptive control：

- controller parameters 随系统变化更新。
- 适合慢变参数。
- 对突变故障、非线性严重故障更难。

在 FTC 中：

- Robust control 常用于 passive FTC。
- Adaptive control 可作为 active / reconfigurable FTC 的一部分。

## 5.7 State-space control foundation

Nominal linear system：

```math
\dot x=Ax+Bu
```

State feedback：

```math
u=-Kx+Nr
```

Closed-loop：

```math
\dot x=(A-BK)x+BNr
```

Stability 由 eigenvalues of `A-BK` 决定。

Pole placement：

- 如果 `(A,B)` controllable，可任意配置 closed-loop poles。
- 用 desired settling time、overshoot 等转成 pole locations。

## 5.8 LQR

Pole placement 需要猜 poles。LQR 自动权衡状态误差和控制能量。

Cost：

```math
J=\int_0^\infty (x^TQx+u^TRu)dt
```

Control law：

```math
u=-Kx
```

解释：

- `Q` 惩罚 state deviation。
- `R` 惩罚 actuator effort。

优点：

- 适合 MIMO / 高阶系统。
- 得到稳定且能量合理的反馈。

缺点：

- 无硬约束。
- 需要完整 state 或 observer。

## 5.9 Observer and separation principle

很多 state 不可直接测量，因此：

```math
u=-K\hat x
```

Luenberger observer：

```math
\dot{\hat x}=A\hat x+Bu+L(y-C\hat x)
```

Error dynamics：

```math
\dot e=(A-LC)e
```

Rule of thumb：

- observer poles 通常比 controller poles 快 3 到 10 倍。

Observer + feedback：

```math
\dot{\hat x}
=
(A-BK-LC)\hat x
+
Ly
+
BNr
```

Dynamic controller standard form：

```math
\dot x_c=A_cx_c+B_cy_{ref}+E_cy
```

```math
u=C_cx_c+D_cy_{ref}+F_cy
```

这个形式用于后续 bumpless transfer。

## 5.10 Reconfigurable control systems

Active reconfiguration 类型：

Precomputed laws：

- gain scheduling
- multiple model control
- LPV controllers

Online synthesized：

- adaptive control
- eigenstructure assignment
- MPC-based reconfiguration

Supervisor：

- 根据 FDD 结果选择或计算 controller。

核心问题：

- 什么时候切换？
- 切换到哪个 controller？
- 切换是否平滑？
- 故障后剩余系统是否可控/可观？

## 5.11 Model matching

Model matching 是 active FTC 关键策略。

Nominal system：

```math
\dot x=Ax+Bu,\qquad y=Cx
```

Nominal controller：

```math
u=-Kx
```

Nominal closed-loop：

```math
\dot x=(A-BK)x
```

定义 desired closed-loop matrix：

```math
A_{des}=A-BK
```

Faulty system：

```math
\dot x=A_fx+B_fu
```

希望找 `K_f`：

```math
\dot x=(A_f-B_fK_f)x
```

并满足：

```math
A_f-B_fK_f=A-BK
```

含义：故障系统在新 controller 下表现得像健康系统。

## 5.12 Exact Model Matching

从目标：

```math
A_f-B_fK_f=A-BK
```

移项：

```math
B_fK_f=A_f-A+BK
```

这是矩阵方程：

```math
MX=N
```

解存在 iff：

```math
columns(N)\in Im(M)
```

所以 EMM 条件：

```math
Im(A_f-A+BK)\subseteq Im(B_f)
```

如果 actuator fault 让 `B_f` 降秩，某些 control directions 丢失，EMM 不可能。

## 5.13 Approximate Model Matching

若 exact impossible，求最接近：

```math
\min_{K_f}
\|(A_f-B_fK_f)-(A-BK)\|_F
```

等价于：

```math
\min_{K_f}\|B_fK_f-(A_f-A+BK)\|_F
```

Pseudo-inverse solution：

```math
K_f=B_f^\dagger(A_f-A+BK)
```

含义：

- 用剩余 actuator 尽可能模拟原 closed-loop。
- `B_f^\dagger` 像 control allocation distributor。

优点：

- 即使 exact 不可行，也能求最佳折中。
- 计算直接。

缺点：

- 不能保证完全恢复 nominal behavior。
- 若剩余控制能力太低，性能仍差。

## 5.14 Plant and actuator fault reconfiguration

Plant fault：

```math
A\rightarrow A_f
```

Model matching：

```math
K_f=B^\dagger(A_f-A+BK)
```

如果 `B_f=B`。

Actuator fault：

```math
B\rightarrow B_f
```

Model matching：

```math
K_f=B_f^\dagger(A-A+BK)
```

若同时 plant and actuator fault：

```math
K_f=B_f^\dagger(A_f-A+BK)
```

重点：actuator loss 可能减少 rank，导致不可控方向无法恢复。

## 5.15 Sensor reconfiguration

Sensor fault：

```math
y_f=C_fx
```

Nominal output feedback：

```math
u=-Ky=-KCx
```

Faulty output feedback：

```math
u=-K_fy_f=-K_fC_fx
```

希望：

```math
K_fC_f=KC
```

解：

```math
K_f=KCC_f^\dagger
```

若 `C_fC_f^T` invertible：

```math
C_f^\dagger=C_f^T(C_fC_f^T)^{-1}
```

Exact sensor matching condition：

```math
Ker(C_f)\subseteq Ker(C)
```

解释：

- 若某状态对 faulty sensors 不可见，但对 nominal sensors 可见，就无法重构 nominal measurement。
- 传感器故障后仍要有 analytical redundancy。

## 5.16 Virtual actuator and virtual sensor

Virtual actuator：

```text
nominal controller command u
-> virtual actuator
-> command u_f suitable for faulty plant
```

用途：nominal controller 不变，在其后加一个接口，把控制命令重新分配到剩余 actuator。

Virtual sensor：

```text
faulty measurement y_f
-> virtual sensor
-> reconstructed nominal measurement y
-> nominal controller
```

用途：nominal controller 不变，在其前加一个接口，清洗或重构传感器信号。

优点：

- 保留 nominal controller。
- 把 fault accommodation 封装在接口模块中。
- 工程上便于模块化。

缺点：

- 仍依赖 redundancy。
- virtual component 设计错误会污染 controller。

## 5.17 Switching transient problem

多个 dynamic controllers：

```math
K_1(s),K_2(s),\dots,K_N(s)
```

每个 controller 有 internal states，例如 integrator state 或 observer state。

当 Controller 1 active，Controller 2 disconnected 时，Controller 2 的 internal states 可能漂移或停在 0。

切换瞬间：

```math
u_1(t_s^-)\neq u_2(t_s^+)
```

输入 jump：

```math
\Delta u=(K_fC_f-KC)x
```

后果：

- actuator shock。
- mechanical stress。
- secondary fault。
- closed-loop instability。

## 5.18 Bumpless transfer principle

目标：让 passive controllers 始终跟踪 active controller output。

Controller i standard form：

```math
\dot x_{ci}
=
A_{ci}x_{ci}
+
B_{ci}y_{ref}
+
E_{ci}y
```

```math
u_i
=
C_{ci}x_{ci}
+
D_{ci}y_{ref}
+
F_{ci}y
```

引入 tracking error：

```math
e_u=u-u_i
```

Modified dynamics：

```math
\dot x_{ci}
=
A_{ci}x_{ci}
+
B_{ci}y_{ref}
+
E_{ci}y
+
L_i(u-u_i)
```

Active controller：

```math
u=u_i
```

所以 tracking term 为 0，不影响 active controller。

Passive controller：

```math
u\neq u_i
```

tracking feedback 驱动 internal states，使：

```math
u_i\rightarrow u
```

## 5.19 Bumpless transfer stability

代入：

```math
u_i=C_{ci}x_{ci}+D_{ci}y_{ref}+F_{ci}y
```

得到 passive dynamics：

```math
\dot x_{ci}
=
(A_{ci}-L_iC_{ci})x_{ci}
+
(B_{ci}-L_iD_{ci})y_{ref}
+
(E_{ci}-L_iF_{ci})y
+
L_iu
```

Tracking stability 要求：

```math
A_{ci}-L_iC_{ci}
```

Hurwitz。

如果 `D_ci` invertible，希望消除 reference influence：

```math
B_{ci}-L_iD_{ci}=0
```

所以：

```math
L_i=B_{ci}D_{ci}^{-1}
```

若 `D_ci` 不可逆或为 0，则用 pole placement / LQR 选择 `L_i`。

## 5.20 Bumpless transfer tuning

Fast poles / high gain：

- passive controller output 快速跟踪 active controller。
- 切换 bump 小。
- 但放大 measurement noise，internal state jitter。

Slow poles / low gain：

- 更平滑，抗噪。
- 但故障突发时 passive controller 可能来不及跟踪。

设计步骤：

1. 把所有 FTC controllers 写成 standard state-space form。
2. 给每个 controller 注入 `L_i(u-u_i)`。
3. 检查 `D_ci` 是否可逆。
4. 若可逆，用 `L_i=B_ciD_ci^{-1}`；否则 pole placement。
5. 验证切换时 `u_i≈u`。

## 5.21 Multi-controller and soft switching

多个 controllers 对应多个 fault modes：

```math
K_0,K_1,\dots,K_m
```

Hard switching：

```math
u=u_i
```

Soft switching / blending：

```math
u=\sum_i\omega_i u_i
```

where：

```math
\omega_i\in[0,1],\qquad \sum_i\omega_i=1
```

优点：

- 过渡平滑。
- 适合 uncertain diagnosis。

缺点：

- 稳定性分析更复杂。
- 权重设计不当会混合出不良控制行为。

---

# Topic 6: Safe and Secure Control for CPS

## 6.1 主题边界

Topic 6 讲 CPS 中 fault、attack、anomaly、uncertainty 与 hard constraints 下的 safe and resilient control。

完整覆盖：

- Attack/Anomaly Detection 概念与三类方法
- constrained optimal control
- MPC
- state elimination and QP formulation
- finite-horizon stability
- terminal equality / terminal set / terminal cost
- reference tracking MPC
- target selector
- Resilient Control for CPS：offset-free MPC、LESO + MPC AFTC、UIO + MPC AFTC
- VAE generative anomaly detection

Topic 5 讲 reconfiguration strategy；Topic 6 更强调 constraints、safety envelope 和 CPS resilience。

对应 Curriculum 小项：

- Attack/Anomaly Detection：6.2 和 6.20-6.28。
- Resilient Control for CPS：6.16-6.19。
- Safe Control Synthesis：6.3-6.15。

## 6.2 CPS 中 fault、attack、anomaly

Fault：

- 组件自然退化或异常。
- 例如 sensor bias、actuator loss、friction increase。

Attack：

- 人为恶意行为。
- 例如 false data injection、actuator command manipulation、replay attack、denial of service。

Anomaly：

- 观测行为异常。
- 原因可能是 fault、attack、disturbance 或工况变化。

Topic 6 的核心不是只报警，而是：

```text
Even under anomaly, keep x and u inside safe constraints.
```

### 6.2.1 Attack/Anomaly Detection 的三类方法

Curriculum 中 Topic 6 的第一个小项是 Attack/Anomaly Detection。它可以按三条路线理解：

1. Model-based attack/anomaly detection  
   使用系统模型和 residual：

```math
r_k=y_k-\hat y_k
```

若攻击或异常污染 sensor / actuator / plant，则 residual 分布改变：

```math
\|r_k\|>J_{th}
\Rightarrow
\text{anomaly / possible attack}
```

适合：

- false data injection detection
- actuator command manipulation detection
- replay attack 导致预测与测量不一致的情况

优点：

- 物理解释强。
- 可与 observer、UIO、KF、CUSUM 结合。

缺点：

- stealthy attack 可能设计成绕过 residual。
- 模型误差可能造成 false alarm。

2. Data-driven anomaly detection  
   使用 normal data 学习正常分布或 manifold。新数据若低概率或远离 normal manifold，则报警：

```math
p(x_k\mid Normal)<\epsilon
```

或：

```math
AnomalyScore(x_k)>J_{th}
```

典型方法：

- PCA residual
- kNN / Parzen density
- Autoencoder / VAE

优点：

- 可检测未知故障或未知 attack pattern。
- 不一定需要完整物理模型。

缺点：

- 阈值校准困难。
- 工况变化可能被误判为 anomaly。

3. Sequence / change detection  
   使用 CUSUM 或 sequential likelihood ratio 检测统计分布变化：

```math
g(k)=\max(0,g(k-1)+s(x_k))
```

适合：

- 小幅持续攻击。
- 传感器 bias attack。
- actuator loss 逐步或突变发生。

优点：

- 能积累弱证据。
- 比单点阈值更适合 noisy CPS。

缺点：

- 需要设定 fault/attack 后的分布或至少设定 drift model。
- threshold 决定 detection delay 与 false alarm trade-off。

### 6.2.2 Attack、Fault、Disturbance 的区别与重叠

从检测器角度，它们都可能表现为 residual 或 anomaly score 变大；但控制响应不同：

- Disturbance：通常用 robust/offset-free control 补偿。
- Fault：需要 isolation 和 accommodation，例如 target shifting。
- Attack：还要考虑数据可信度、隔离被攻击通道、切换到安全模式或使用冗余信息。

因此 Topic 6 的检测结果必须进入 safe/resilient control，而不是只停留在报警。

## 6.3 为什么 LQR 不够

LQR cost：

```math
J=\sum (x_k^TQx_k+u_k^TRu_k)
```

但 LQR 默认无硬约束。真实系统有：

```math
x_k\in\mathcal X
```

```math
u_k\in\mathcal U
```

例如：

- valve opening 0%-100%。
- actuator torque limit。
- voltage/current limit。
- pressure/temperature safety ceiling。
- position/velocity constraints。

因此需要 constrained optimal control 和 MPC。

## 6.4 Constrained finite-horizon LQ problem

有限时域问题：

```math
\min_{u_0,\dots,u_{K-1}}
\sum_{k=0}^{K-1}
(x_k^TQx_k+u_k^TRu_k)
+
x_K^TSx_K
```

subject to：

```math
x_{k+1}=Ax_k+Bu_k
```

```math
x_0=x_{initial}
```

```math
x_k\in X_k
```

```math
u_k\in U_k
```

由于有 inequality constraints，不能直接用 Riccati equation，必须转成 QP。

## 6.5 Open-loop optimal control

一次性求解得到：

```math
u_0^*,u_1^*,\dots,u_{K-1}^*
```

这是 open-loop script。

缺点：

- 若 disturbance 使实际状态偏离 prediction，后续 input sequence 仍按旧计划执行。
- 没有反馈。
- 可能导致 constraint violation。

## 6.6 Receding Horizon Control / MPC

MPC loop：

1. Measure current state：

```math
x_t
```

2. Solve finite-horizon optimization from current state。

3. 得到 sequence：

```math
u_{t|t}^*,u_{t+1|t}^*,\dots,u_{t+N-1|t}^*
```

4. 只执行第一个：

```math
u_t=u_{t|t}^*
```

5. 下一时刻重新测量、重新优化。

核心：不断 re-plan，把 optimization 变成 feedback。

## 6.7 State prediction stacking

从：

```math
x_{k+1}=Ax_k+Bu_k
```

有：

```math
x_1=Ax_0+Bu_0
```

```math
x_2=A^2x_0+ABu_0+Bu_1
```

```math
x_3=A^3x_0+A^2Bu_0+ABu_1+Bu_2
```

Stacked form：

```math
X=\mathcal A x_0+\mathcal B U
```

其中：

```math
X=
\begin{bmatrix}
x_1\\x_2\\ \vdots \\x_N
\end{bmatrix}
```

```math
U=
\begin{bmatrix}
u_0\\u_1\\ \vdots \\u_{N-1}
\end{bmatrix}
```

`mathcal B` 是 lower triangular Toeplitz，因为未来 input 不能影响过去 state。

## 6.8 MPC cost to QP

Cost：

```math
J=X^TQX+U^TRU
```

代入：

```math
X=\mathcal A x_0+\mathcal B U
```

展开：

```math
J
=
U^T(\mathcal B^TQ\mathcal B+R)U
+
2x_0^T\mathcal A^TQ\mathcal B U
+
x_0^T\mathcal A^TQ\mathcal A x_0
```

标准 QP：

```math
\min_U
\frac12U^THU+f(x_0)^TU
```

where：

```math
H=2(\mathcal B^TQ\mathcal B+R)
```

```math
f(x_0)^T=2x_0^T\mathcal A^TQ\mathcal B
```

`H` 只依赖模型和权重，可 offline 计算；`f` 随当前状态变化。

## 6.9 Constraint translation

State constraints：

```math
C_xX\le D_x
```

Input constraints：

```math
C_uU\le D_u
```

代入 state elimination：

```math
C_x(\mathcal A x_0+\mathcal B U)\le D_x
```

得到：

```math
C_x\mathcal B U\le D_x-C_x\mathcal A x_0
```

Final QP：

```math
\min_U
\frac12U^THU+f(x_0)^TU
```

subject to：

```math
GU\le W(x_0)
```

where：

```math
G=
\begin{bmatrix}
C_x\mathcal B\\
C_u
\end{bmatrix}
```

```math
W(x_0)=
\begin{bmatrix}
D_x-C_x\mathcal A x_0\\
D_u
\end{bmatrix}
```

## 6.10 State elimination pros/cons

优点：

- 优化变量只有 U，变量少。
- 实现直观。
- 适合中小规模 MPC。

缺点：

- Hessian dense。
- 长 horizon 计算代价高，约 `O(N^3)`。
- `A^N` 可能造成数值病态，尤其 open-loop unstable system。

## 6.11 MPC stability problem

Finite horizon MPC 可能 short-sighted：

- 只优化 horizon 内 cost。
- 不关心 horizon 后系统是否会发散。
- 最优 cost 不一定每步下降。

需要 terminal design 来保证：

- recursive feasibility
- asymptotic stability

## 6.12 Terminal equality

最简单稳定约束：

```math
x_{t+N\mid t}=0
```

优点：

- Lyapunov proof 简单。
- 终点回到 origin，下一步可 shift sequence 并 append zero。

证明逻辑：

若当前最优序列为：

```math
\{u_0^*,u_1^*,\dots,u_{N-1}^*\}
```

下一步构造可行序列：

```math
\{u_1^*,\dots,u_{N-1}^*,0\}
```

因为旧 terminal state 为 0，append 0 仍可行。Optimal cost 下降：

```math
V^*(x_{t+1})-V^*(x_t)
\le
-(x_t^TQx_t+u_t^TRu_t)<0
```

缺点：

- 工程上常 infeasible。
- 短 horizon 内强行到 0 需要过大控制。
- actuator constraints 下难以满足。

## 6.13 Terminal set and terminal cost

更实用方法：

Terminal set：

```math
x_{t+N\mid t}\in X_f
```

Terminal cost：

```math
x_N^TPx_N
```

MPC objective：

```math
\min
\sum_{k=0}^{N-1}
(x_k^TQx_k+u_k^TRu_k)
+
x_N^TPx_N
```

Stability conditions：

1. 存在 local controller：

```math
\kappa_f(x)=K_fx
```

2. 在 terminal set 内满足 constraints：

```math
x\in X,\qquad K_fx\in U
```

3. Positive invariance：

```math
x\in X_f
\Rightarrow
(A+BK_f)x\in X_f
```

4. Terminal cost Lyapunov decrease：

```math
(A+BK_f)^TP(A+BK_f)-P+Q+K_f^TRK_f\le0
```

证明逻辑：

- 当前 horizon 末端进入 safe landing zone `X_f`。
- 下一步 shifted sequence 末端 append local control `K_fx_N`。
- Positive invariance 保证仍在 `X_f`。
- Terminal cost 近似 infinite tail cost。

结果：

- Recursive feasibility。
- Asymptotic stability。

## 6.14 Reference tracking MPC

Regulation MPC 让：

```math
x\rightarrow0,\qquad u\rightarrow0
```

但实际常要：

```math
y\rightarrow y_{spec}
```

例如温度维持 150 度、位置维持 4 m。

先求 steady-state pair：

```math
x_s=A x_s+B u_s
```

```math
C x_s=y_{spec}
```

矩阵形式：

```math
\begin{bmatrix}
I-A & -B\\
C & 0
\end{bmatrix}
\begin{bmatrix}
x_s\\u_s
\end{bmatrix}
=
\begin{bmatrix}
0\\y_{spec}
\end{bmatrix}
```

再定义 shifted variables：

```math
\tilde x=x-x_s
```

```math
\tilde u=u-u_s
```

Dynamic MPC 调节：

```math
\tilde x\rightarrow0,\qquad \tilde u\rightarrow0
```

则：

```math
x\rightarrow x_s,\qquad u\rightarrow u_s
```

## 6.15 Target selector

若 system overactuated：

```math
m>p
```

同一个 output target 可能有多个 `(x_s,u_s)`。需要 target selector QP：

```math
\min_{x_s,u_s}
\|x_s-x_{spec}\|_{Q_s}^2
+
\|u_s-u_{spec}\|_{R_s}^2
```

subject to：

```math
(I-A)x_s-Bu_s=0
```

```math
Cx_s=y_{spec}
```

```math
x_s\in X,\qquad u_s\in U
```

结构：

```text
Layer 1: Target selector
Layer 2: Dynamic MPC
```

Layer 1 找安全可达 steady-state，Layer 2 找到达该 steady-state 的最优轨迹。

## 6.16 Offset-free MPC

现实中模型不完全准确：

- constant disturbance
- drag
- load change
- mass mismatch
- heat loss

标准 MPC 会有 steady-state offset。

引入 disturbance model：

```math
x_{k+1}=Ax_k+Bu_k+B_dd_k
```

```math
d_{k+1}=d_k
```

```math
y_k=Cx_k+C_dd_k
```

Observer estimates：

```math
\hat x_k,\hat d_k
```

Observer update：

```math
\hat x_{k+1}=A\hat x_k+Bu_k+B_d\hat d_k+L_x(y_k-\hat y_k)
```

```math
\hat d_{k+1}=\hat d_k+L_d(y_k-\hat y_k)
```

Target selector 修正：

```math
(I-A)x_s-Bu_s=B_d\hat d_k
```

```math
Cx_s=y_{spec}-C_d\hat d_k
```

矩阵形式：

```math
\begin{bmatrix}
I-A&-B\\
C&0
\end{bmatrix}
\begin{bmatrix}
x_s\\u_s
\end{bmatrix}
=
\begin{bmatrix}
B_d\hat d_k\\
y_{spec}-C_d\hat d_k
\end{bmatrix}
```

执行：

```math
u_k=u_s+\tilde u_{0|t}^*
```

Offset-free execution loop：

1. Measure `y_k`。
2. Estimate `hat x_k, hat d_k`。
3. Target selection with disturbance estimate。
4. Dynamic MPC。
5. Actuate。

## 6.17 LESO + MPC AFTC

Fault-injected model：

```math
x_{k+1}=Ax_k+B(u_k+f_{a,k})
```

```math
y_k=Cx_k+C_{fs}f_{s,k}
```

Actuator fault：

- 真实物理输入能力变化。
- 必须 accommodation。

Sensor fault：

- 测量是假的。
- 物理系统未必变。
- 必须 isolation / cleaning。

Augmented LESO state：

```math
X_{aug}=
\begin{bmatrix}
x\\f_a\\f_s
\end{bmatrix}
```

假设：

```math
f_{a,k+1}=f_{a,k},\qquad f_{s,k+1}=f_{s,k}
```

Augmented dynamics：

```math
\begin{bmatrix}
x_{k+1}\\
f_{a,k+1}\\
f_{s,k+1}
\end{bmatrix}
=
\begin{bmatrix}
A&B&0\\
0&I&0\\
0&0&I
\end{bmatrix}
\begin{bmatrix}
x_k\\f_{a,k}\\f_{s,k}
\end{bmatrix}
+
\begin{bmatrix}
B\\0\\0
\end{bmatrix}u_k
```

Measurement：

```math
y_k=
\begin{bmatrix}
C&0&C_{fs}
\end{bmatrix}
X_{aug,k}
```

MPC 使用：

Sensor fault handling：

- 不改变 target。
- 不信 raw sensor。
- 用 LESO cleaned state estimate `hat x_k`。

Actuator fault handling：

- 改变 target selector：

```math
(I-A)x_s-Bu_s=B\hat f_{a,k}
```

若 actuator loss 为 `-0.5`，target selector 会把 baseline input 增加 `+0.5`。

## 6.18 UIO + MPC AFTC

LESO 动态估计 actuator fault，可能有滞后。UIO 另一种思想：

- actuator fault 作为 unknown input。
- UIO 对它解耦，保证 state estimate 不被污染。
- 再通过代数公式重构 fault。

Sensor fault：

- augmented state，UIO 估计并隔离。

Actuator fault：

- unknown input `d_k`，进入：

```math
E_{aug}f_a
```

UIO decoupling：

```math
T E_{aug}=0
```

构造：

```math
H=E_{aug}(C_{aug}E_{aug})^\dagger
```

```math
T=I-HC_{aug}
```

结果：

- actuator fault 不进入 observer internal state。
- cleaned state estimate 给 MPC。

Algebraic reconstruction：

```math
\hat f_{a,k-1}
=
(C_{aug}E_{aug})^\dagger
\left(
y_k
-
C_{aug}A_{aug}\hat X_{aug,k-1}
-
C_{aug}B_{aug}u_{k-1}
\right)
```

然后 target selector 用：

```math
(I-A)x_s-Bu_s=B\hat f_a
```

LESO vs UIO：

- LESO：动态估计 fault，像 observer 追踪，适合慢变但可能滞后。
- UIO：结构解耦 + algebraic reconstruction，fault signature 更快，但条件更严格。

## 6.19 Safe/resilient control logic

完整 safe AFTC pipeline：

```text
Corrupted sensor y_k
-> Observer / UIO / LESO
-> cleaned state estimate
-> fault estimate / reconstruction
-> target selector shifts steady-state
-> dynamic MPC enforces constraints
-> safe control input
```

核心目标：

```math
y_k\rightarrow y_{spec}
```

while：

```math
x_k\in X,\qquad u_k\in U
```

even if：

```math
f_a\neq0,\qquad f_s\neq0
```

## 6.20 VAE generative anomaly detection

FTC_09/10 讲 VAE。它属于 Topic 6 的 anomaly detection，也连接 Topic 1 概率和 Topic 2 latent representation。

Discriminative model：

```math
p(y\mid x)
```

直接学 data 到 label。问题：新故障未见过时，可能自信错分。

Generative model：

```math
p(x)
\quad \text{or}\quad
p(x,y)
```

学习正常数据如何生成。对未知故障更有优势。

Fault detection 思路：

1. 只用 healthy data 训练 VAE。
2. VAE 学 normal manifold。
3. 新数据若不符合 normal manifold，则 anomaly。

## 6.21 Autoencoder and PCA relation

PCA：

```math
z=W^Tx
```

```math
\hat x=Wz
```

Fault detection：

```math
\|x-\hat x\|^2 \text{ large} \Rightarrow anomaly
```

Autoencoder 是 nonlinear PCA：

```text
x -> encoder -> z -> decoder -> x_hat
```

训练目标：

```math
\min \|x-\hat x\|^2
```

标准 AE 问题：

- latent space deterministic。
- 没有 regularization。
- 相似样本可能映射很远。
- latent empty space 采样会产生无意义输出。
- 机器平滑工况变化时可能穿过 empty region，造成 false alarm。

## 6.22 VAE probabilistic setup

VAE 假设 sensor data `x` 由 latent variables `z` 生成。

目标 posterior：

```math
p_\theta(z\mid x)
```

Bayes：

```math
p_\theta(z\mid x)
=
\frac{p_\theta(x\mid z)p_\theta(z)}{p_\theta(x)}
```

其中：

- `p_theta(x|z)`：decoder likelihood。
- `p_theta(z)`：latent prior，通常 `N(0,I)`。
- `p_theta(x)`：marginal likelihood。

Marginal likelihood：

```math
p_\theta(x)
=
\int p_\theta(x\mid z)p_\theta(z)dz
```

高维下不可计算。

引入 variational approximation：

```math
q_\phi(z\mid x)
```

由 encoder neural network 输出。

## 6.23 KL divergence

衡量两个分布差异：

```math
D_{KL}(q\|p)
=
\int q(z)\log\frac{q(z)}{p(z)}dz
```

性质：

```math
D_{KL}(q\|p)\ge0
```

且：

```math
D_{KL}(q\|p)\neq D_{KL}(p\|q)
```

不是真正距离，因为不对称。

## 6.24 ELBO derivation

从：

```math
\log p_\theta(x)
```

写成对 `q_phi(z|x)` 的期望：

```math
\log p_\theta(x)
=
E_{q_\phi(z\mid x)}[\log p_\theta(x)]
```

引入：

```math
p_\theta(x)=\frac{p_\theta(x,z)}{p_\theta(z\mid x)}
```

乘除 `q_phi(z|x)`：

```math
\log p_\theta(x)
=
E_q
\left[
\log
\frac{p_\theta(x,z)}{q_\phi(z\mid x)}
\right]
+
E_q
\left[
\log
\frac{q_\phi(z\mid x)}{p_\theta(z\mid x)}
\right]
```

第二项是：

```math
D_{KL}(q_\phi(z\mid x)\|p_\theta(z\mid x))
```

所以：

```math
\log p_\theta(x)
=
ELBO
+
D_{KL}(q_\phi(z\mid x)\|p_\theta(z\mid x))
```

因为 KL 非负：

```math
\log p_\theta(x)\ge ELBO
```

展开 ELBO：

```math
ELBO
=
E_{q_\phi}
[\log p_\theta(x\mid z)]
-
D_{KL}(q_\phi(z\mid x)\|p_\theta(z))
```

VAE objective 最大化：

```math
L
=
E_{q_\phi}
[\log p_\theta(x\mid z)]
-
D_{KL}(q_\phi(z\mid x)\|p_\theta(z))
```

两项含义：

- Reconstruction likelihood：decoder 能否重构原始数据。
- KL regularizer：latent distribution 是否接近 prior `N(0,I)`。

## 6.25 Reparameterization trick

Encoder 输出：

```math
\mu,\quad \log\sigma^2
```

若直接从 `q_phi(z|x)` 采样，随机节点阻断 backpropagation。

重参数化：

```math
\epsilon\sim\mathcal N(0,I)
```

```math
z=\mu+\sigma\odot\epsilon
```

这样随机性外部化，`mu` 和 `sigma` 到 `z` 是可微的。

为什么输出 `log sigma^2`：

- neural network 输出范围是 `(-inf,inf)`。
- variance 必须正。
- 输出 log-variance 后再 exponentiate 可保证正性。

## 6.26 Gaussian KL closed form

通常：

```math
p(z)=\mathcal N(0,I)
```

```math
q_\phi(z\mid x)=\mathcal N(\mu,diag(\sigma^2))
```

KL regularization 有解析式：

```math
-D_{KL}(q_\phi(z\mid x)\|p(z))
=
\frac12
\sum_{i=1}^{D}
\left(
1+\log\sigma_i^2-\mu_i^2-\sigma_i^2
\right)
```

训练时不需要 sampling estimate KL。

## 6.27 VAE anomaly score

VAE 给两类 anomaly metric：

1. Reconstruction likelihood / error：

```math
\log p_\theta(x\mid z)
```

若假设 decoder Gaussian，则等价于 MSE：

```math
\|x-\hat x\|^2
```

2. Latent deviation：

监控 encoder mean：

```math
\mu(x)
```

若：

```math
\|\mu(x)\|
```

远离 0，表示样本不属于 normal latent sphere。

综合：

```math
AnomalyScore
=
\alpha\|x-\hat x\|^2
+
\beta\|\mu(x)\|^2
```

或使用 negative ELBO。

## 6.28 VAE 优劣势

优点：

- 只需 healthy data。
- 能检测未知故障。
- latent space 连续、完整，减少 AE empty-space false alarm。
- 同时给 reconstruction 和 probabilistic latent score。

缺点：

- 训练复杂。
- threshold calibration 需要 normal validation data。
- posterior collapse。
- 输出可能 oversmooth / blurry。
- 深度模型解释性弱。

Posterior collapse：

```math
q_\phi(z\mid x)\approx p(z)
```

latent 不编码信息。

解决：

- KL annealing：先降低 KL weight，让模型先学 reconstruction，再逐步加 KL。
- Free bits：允许每个 latent dimension 至少编码一定信息再惩罚 KL。

Blurriness：

- VAE 可能覆盖空白区域，输出平均化。
- IWAE 使用多个 latent samples tightening bound。

---

# 附录 A：六个主题边界防重复总结

## Topic 1 vs Topic 2

Topic 1 讲统计检测和概率分类。  
Topic 2 讲 feature / representation / low-dimensional embedding。  
PCA 在 Topic 1 只作为 pipeline 中的降维环节提到，完整算法在 Topic 2。

## Topic 1 vs Topic 3

Topic 1：Bayes、LDA、QDA、logistic regression，概率路线。  
Topic 3：SVM、kernel SVM、Perceptron，最大间隔和几何优化路线。

## Topic 1 vs Topic 4

Topic 1：从数据 distribution / features 判断 fault。  
Topic 4：用 physical/model-based residual 和 observer 判断 fault。

## Topic 2 vs Topic 3

Topic 2：如何得到低维 fault-sensitive features。  
Topic 3：如何用 feature vectors 做 maximum-margin classification。

## Topic 4 vs Topic 5

Topic 4 到 diagnosis 为止：residual、observer、CUSUM、UIO detection。  
Topic 5 从 diagnosis 之后开始：controller reconfiguration、model matching、bumpless transfer。

## Topic 5 vs Topic 6

Topic 5：FTC strategy 本身，重点是重构、切换、虚拟执行器/传感器。  
Topic 6：CPS safety 和 constraints，重点是 MPC、terminal set、offset-free、resilient control、VAE anomaly detection。

---

# 附录 B：口试答题模板

如果抽到任一主题，可以按下面顺序回答：

1. 一句话定义该主题解决什么问题。
2. 说明它在 FTC chain 中的位置。
3. 给出核心数学模型。
4. 给出核心算法流程。
5. 给出最重要公式。
6. 解释公式背后的工程含义。
7. 讲优点、缺点。
8. 讲适用场景和不适用场景。
9. 说明与其他主题的边界。
10. 用一个 fault detection / control reconfiguration 例子收尾。

---

# 附录 C：最小必背清单

Topic 1：

- FAR, MAR, sensitivity, specificity。
- Bayes theorem, MAP, LRT, risk-based decision。
- MLE vs MAP。
- Histogram, Parzen, kNN。
- LDA/QDA/logistic regression 的假设和边界形状。

Topic 2：

- PCA normal subspace / residual subspace。
- SPE/Q and T²。
- KPCA kernel trick。
- SPCA/FDA supervised fault-sensitive projection。
- MDS preserves distance, Isomap uses geodesic distance。

Topic 3：

- Hard margin primal and dual。
- Soft margin slack variable and C。
- KKT and support vectors。
- Kernel trick and common kernels。
- Perceptron vs SVM。

Topic 4：

- Residual generation。
- Observer-based methods：UIO/KF/ESO/NDO（Luenberger 作为 observer residual 基础）。
- UIO rank condition and decoupling。
- CUSUM recursive formula。
- Gaussian CUSUM increment。
- ARL threshold trade-off。

Topic 5：

- Passive vs active FTC。
- Redundancy。
- Model matching equation。
- EMM vs AMM。
- Sensor reconfiguration kernel condition。
- Virtual actuator/sensor。
- Bumpless transfer tracking law。

Topic 6：

- Constraints and why LQR is insufficient。
- MPC QP formulation。
- Terminal equality / terminal set / terminal cost。
- Reference tracking and target selector。
- Offset-free MPC。
- LESO/UIO + MPC AFTC。
- VAE ELBO and anomaly score。
