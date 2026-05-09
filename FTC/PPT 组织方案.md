# FTC专题组织方案

## 总体 PPT 逻辑模板

每个 Topic 都建议使用同一套叙事结构：

1. 第 1 页：Topic title and big question。
2. 第 2 页：FTC 背景连接，说明这个 Topic 在整门课中的位置。
3. 第 3 到 11 页：核心知识点，由基础到方法，由方法到工程应用。
4. 第 12 页：方法比较，说明何时用哪种方法。
5. 第 13 页：常见错误和限制。
6. 第 14 页：本 Topic 总结，用 5 到 7 句话收束。
7. 第 15 页：Assignment problem statement。
8. 第 16 页：Assignment method and result。
9. 第 17 页：Assignment reflection and Q&A bridge。

每个专题开头不需要重复完整 FTC 基础概念。只需要用一页快速提醒：Fault detection, isolation, identification, diagnosis, reconfiguration, safety constraints。这样能节省页数，也能让考官看到主题边界清楚。

---

# Topic 1: Introduction to Fault and Statistical Signal Processing

## 一、主题定位

本专题回答的问题是：如何从传感器信号和统计概率角度判断 normal 或 fault。

核心范围包括：

1. Signal based FDD。
2. Time domain and frequency domain features。
3. Confusion matrix, FAR, MAR, sensitivity, specificity。
4. Bayes theorem, MAP decision, likelihood ratio。
5. Risk based decision。
6. PDF estimation。
7. MLE and MAP estimation。
8. Histogram, Parzen Window, kNN density estimation。
9. LDA, QDA, Logistic Regression。

本专题的逻辑主线：

从 raw signal 出发，提取 fault sensitive features，再用 probability model 或 classifier 做 decision。

## 二、17 页 PPT 结构

### Page 1: Title and Big Question

标题：Introduction to Fault and Statistical Signal Processing

页面内容：

1. Big question: How can sensor data be converted into a reliable fault decision?
2. 关键词：signal, feature, probability, threshold, classifier。
3. 输出目标：normal, fault, fault class, confidence。

讲解方式：

先说本 Topic 是 data driven FDD 的入口。后续 PCA、SVM、VAE 都建立在这里的 feature, probability, threshold 思想上。

考官视角：

这一页要让人知道你不会把 Topic 1 讲成纯数学课，你会把统计方法放回 fault detection。

### Page 2: From Signal Alarm to Statistical FDD

页面内容：

1. 最简单方法：signal exceeds limit, then alarm。
2. 问题：noise, operating point variation, multiple alarms, difficult isolation。
3. 改进：statistical decision with features and probability。

讲解方式：

用一句话串起来：单纯阈值看幅值，统计 FDD 看数据模式是否符合 normal distribution。

可能被问：为什么只看 amplitude 不够？

回答：故障可能体现为 variance, frequency, correlation, distribution shift，不一定体现为平均值变大。

### Page 3: Signal Diagnosis Pipeline

页面内容：

Raw signal → preprocessing → windowing → feature extraction → normalization → model or classifier → decision。

需要展示：

1. Preprocessing: filtering, synchronization, detrending。
2. Windowing: convert time series into samples。
3. Normalization: avoid scale dominance。

讲解方式：

强调 pipeline 是工程落地结构。后面的 Bayes, LDA, Logistic Regression 都只接收 feature vector，不直接理解原始机器。

可能被问：为什么要 normalization？

回答：不同传感器量纲不同，未标准化时，距离、协方差、密度估计都会被大尺度变量主导。

### Page 4: Time and Frequency Domain Features

页面内容：

Time domain:

1. Mean: bias and drift。
2. Variance: vibration and instability。
3. RMS: energy。
4. Peak to peak: impulse。
5. Skewness and kurtosis: asymmetry and heavy tail。
6. Crest factor: sharp peak。

Frequency domain:

1. FFT and PSD。
2. Dominant frequency。
3. Band energy。
4. Harmonics and sidebands。

讲解方式：

用机械系统例子讲：轴承故障可能先改变频谱，传感器漂移主要改变 mean，松动可能增大 variance 和 kurtosis。

可能被问：time domain 和 frequency domain 如何选择？

回答：平稳周期类故障优先 frequency domain；偏置、漂移、冲击、短窗口在线检测常用 time domain；非平稳信号可考虑 STFT 或 wavelet。

### Page 5: Confusion Matrix and Detection Metrics

页面内容：

1. TP: fault and alarm。
2. FP: normal but alarm。
3. TN: normal and silent。
4. FN: fault but silent。

公式：

```math
FAR=\frac{FP}{FP+TN}
```

```math
MAR=\frac{FN}{FN+TP}
```

```math
Sensitivity=\frac{TP}{TP+FN}
```

```math
Specificity=\frac{TN}{TN+FP}
```

讲解方式：

强调 fault detection 中 FN 和 FP 后果不同。FN 是安全风险，FP 是 nuisance alarm 和 alarm fatigue。

可能被问：为什么 accuracy 不够？

回答：normal 样本远多于 fault 样本时，全部判 normal 也可能 accuracy 很高，但 MAR 会很差。

### Page 6: Threshold Trade Off

页面内容：

1. Alarm rule: \(J(x)>h\)。
2. Low threshold: low MAR, high FAR。
3. High threshold: low FAR, high MAR。
4. Detection delay: \(D=k_{alarm}-k_{fault}\)。

讲解方式：

把阈值理解成 sensitivity knob。安全关键系统通常宁愿多一些误报，也要减少漏报。

可能被问：阈值如何工程选取？

回答：可以根据 normal data 的统计分布、指定 FAR、ARL 要求、安全风险代价、实验验证共同确定。

### Page 7: Bayes Theorem and MAP Decision

页面内容：

```math
P(C_i|x)=\frac{p(x|C_i)P(C_i)}{p(x)}
```

```math
\hat C=\arg\max_i p(x|C_i)P(C_i)
```

解释：

1. Prior: fault before seeing data。
2. Likelihood: how likely x under class。
3. Posterior: updated belief。
4. Evidence: normalization。

讲解方式：

说清楚 Bayes 是把 measurement 变成 belief update。

可能被问：为什么 MAP 中可以去掉 \(p(x)\)？

回答：对所有 class，\(p(x)\) 相同，不影响 argmax。

### Page 8: Gaussian Assumption and PDF

页面内容：

```math
x|N\sim \mathcal N(\mu_N,\sigma_N^2)
```

```math
x|F\sim \mathcal N(\mu_F,\sigma_F^2)
```

误报与漏报面积：

```math
P_{FA}=\int_T^\infty f(x|N)dx
```

```math
P_{MA}=\int_{-\infty}^T f(x|F)dx
```

讲解方式：

用两条重叠 Gaussian 曲线解释：只要分布重叠，零错误通常做不到。

可能被问：PDF 值是概率吗？

回答：连续变量中单点概率为零，PDF 是密度。概率来自区间积分。

### Page 9: Likelihood Ratio and Risk Based Decision

页面内容：

```math
\Lambda(x)=\frac{p(x|F)}{p(x|N)}
```

```math
\Lambda(x)>\eta \Rightarrow Fault
```

Risk:

```math
R(\alpha_i|x)=\sum_j \lambda(\alpha_i|C_j)P(C_j|x)
```

讲解方式：

普通 MAP 主要看哪类概率更大。Risk based decision 还看错判代价。

可能被问：为什么安全系统中阈值可能低于 MAP 阈值？

回答：漏报代价很高时，系统愿意承担更多 false alarm 来降低 missed alarm risk。

### Page 10: MLE and MAP Parameter Estimation

页面内容：

```math
\hat\theta_{MLE}=\arg\max_\theta p(D|\theta)
```

```math
\hat\theta_{MAP}=\arg\max_\theta p(D|\theta)p(\theta)
```

比较：

1. MLE uses data。
2. MAP uses data and prior。
3. With enough data, MAP approaches MLE。
4. MAP helps small sample situations。

讲解方式：

把 MLE 讲成找最能生成训练数据的参数，把 MAP 讲成在数据基础上加入工程先验。

可能被问：FDD 训练阶段如何用 MLE？

回答：分别用 normal data 和 fault data 估计 \(\mu_N,\Sigma_N,\mu_F,\Sigma_F\)，再构造 classifier 或 threshold。

### Page 11: Non Parametric Density Estimation

页面内容：

1. Histogram。
2. Parzen window。
3. kNN density。

公式：

```math
\hat p(x)=\frac1N\sum_{i=1}^N\frac1{h^d}K\left(\frac{x-x_i}{h}\right)
```

```math
\hat p(x)=\frac{k}{NV_k(x)}
```

讲解方式：

强调当 Gaussian assumption 不可靠时，non parametric 方法更灵活。

可能被问：Parzen 的 bandwidth 如何影响结果？

回答：小 \(h\) 容易 overfit 和 high variance；大 \(h\) 容易 oversmoothing 和 high bias。

### Page 12: LDA, QDA, Logistic Regression

页面内容：

LDA:

```math
x|C_i\sim \mathcal N(\mu_i,\Sigma)
```

QDA:

```math
x|C_i\sim \mathcal N(\mu_i,\Sigma_i)
```

Logistic Regression:

```math
P(y=1|x)=\frac1{1+e^{-(w^Tx+b)}}
```

讲解方式：

三者对比：LDA 共享协方差，线性边界；QDA 各类协方差不同，二次边界；Logistic Regression 直接估 posterior。

可能被问：LDA 和 Logistic Regression 有什么差别？

回答：LDA 是 generative，先建 \(p(x|C_i)\) 再用 Bayes；Logistic Regression 是 discriminative，直接建 \(P(y|x)\)。

### Page 13: Method Selection and Common Pitfalls

页面内容：

1. Gaussian and enough data: LDA, QDA。
2. Need probability output: Logistic Regression。
3. Unknown distribution: Parzen or kNN。
4. High dimension: avoid direct density estimation。
5. Class imbalance: do not rely on accuracy。

讲解方式：

这一页用于展示判断力。方法没有绝对优劣，核心是看数据分布、标签数量、维度、实时性和安全代价。

可能被问：为什么高维下密度估计困难？

回答：curse of dimensionality 让样本在高维空间极稀疏，局部密度估计需要指数级数据量。

### Page 14: Topic Summary

页面内容：

1. Signal based FDD starts from feature extraction。
2. Metrics must separate FAR and MAR。
3. Bayes converts likelihood and prior into posterior。
4. MLE and MAP estimate unknown model parameters。
5. Non parametric methods handle non Gaussian data。
6. LDA, QDA, Logistic Regression are practical classifiers。

讲解方式：

用一句话收束：Topic 1 的核心是把 sensor features 转换成 statistical decision。

### Page 15: Assignment Position 1

内容建议：

1. Problem statement。
2. Given data or signal。
3. Detection goal。
4. What must be calculated。

讲解方式：

只讲题目背景和输入输出。不要在这一页堆计算细节。

### Page 16: Assignment Position 2

内容建议：

1. Method choice。
2. Main formula。
3. Calculation steps。
4. Result table or decision boundary。

讲解方式：

强调为什么选这个方法。例如若题目给 Gaussian 分布，优先 Bayes or LRT；若给 training data，先估参数。

### Page 17: Assignment Position 3

内容建议：

1. Result interpretation。
2. FAR or MAR implication。
3. Limitation。
4. What could improve。

讲解方式：

把作业结果接回 Topic 总结，并准备回答参数选择和阈值合理性问题。

## 三、考官可能提问与回答

### Q1: Fault, failure, anomaly, disturbance, noise 有什么区别？

回答：Fault 是系统部件或变量偏离允许行为；failure 是功能无法完成；anomaly 是观测异常，可能由 fault, attack, disturbance, noise 或工况变化造成；disturbance 是外部或内部未控输入；noise 是随机测量或过程不确定性。

### Q2: FAR 和 MAR 哪个更重要？

回答：取决于场景。安全关键系统中 MAR 通常更关键，因为 missed alarm 可能导致事故。生产系统中 FAR 也重要，因为 false alarm 会导致停机和 alarm fatigue。

### Q3: MAP 和 risk based decision 的区别？

回答：MAP 选择 posterior 最大的类别。Risk based decision 计算每个 action 的 expected loss，当不同错误代价差异很大时更合理。

### Q4: 为什么 LDA 是线性边界？

回答：LDA 假设各类共享协方差。判别函数相减后，二次项抵消，只剩关于 \(x\) 的线性项。

### Q5: QDA 为什么更容易 overfit？

回答：QDA 每个类别都估计自己的 covariance matrix，参数更多。样本少或维度高时 covariance 估计不稳定，甚至奇异。

### Q6: Logistic Regression 输出的是概率，为什么还需要 threshold？

回答：模型输出 \(P(y=1|x)\)。最终 alarm 仍要用 threshold。默认 0.5 只适合对称代价，安全系统中 threshold 可根据 MAR 和 FAR 调整。

### Q7: Parzen 与 kNN density 的本质区别？

回答：Parzen 固定窗口大小，数窗口内样本；kNN 固定样本数，反推出局部体积。

### Q8: 统计方法为什么会受 operating condition 影响？

回答：如果训练分布只覆盖某些工况，新的正常工况可能落在低概率区域，被误判为 fault。因此需要工况分区、归一化、在线更新或条件模型。

## 四、本专题最终总结

Topic 1 的 PPT 应该突出三个层次：

1. 信号层：从 raw signal 提取 features。
2. 统计层：用 PDF, Bayes, likelihood, MLE, MAP 建立判决。
3. 分类层：用 LDA, QDA, Logistic Regression 输出 fault decision。

最重要的考试表达：

Fault detection is a statistical decision problem under uncertainty. A good detector must balance false alarms, missed alarms, and detection delay while using features and probability models that match the physical behavior of the system.

---

# Topic 2: Dimensionality Reduction Approaches

## 一、主题定位

本专题回答的问题是：如何把高维传感器数据压缩到低维空间，同时保留 normal structure 或 fault sensitive information。

核心范围包括：

1. PCA and variants。
2. PCA monitoring statistics。
3. Kernel PCA。
4. Supervised PCA, HSIC, Metric Learning, SDR 的思想。
5. FDA。
6. MDS。
7. Isomap。

本专题的逻辑主线：

高维传感器数据存在冗余和相关性。降维方法把数据投影到低维表示中，让 normal subspace, residual deviation, class separation 或 manifold structure 更清楚。

## 二、17 页 PPT 结构

### Page 1: Title and Big Question

标题：Dimensionality Reduction Approaches

页面内容：

1. Big question: How can high dimensional measurements be represented in a lower dimensional fault sensitive space?
2. 关键词：projection, subspace, residual, manifold, visualization。

讲解方式：

说明 Topic 2 关心 representation。分类器本身属于 Topic 1 或 Topic 3，模型残差属于 Topic 4。

### Page 2: Why FDD Needs Dimensionality Reduction

页面内容：

1. Many sensors and high \(d\)。
2. Strong correlation among variables。
3. Noise and redundant channels。
4. Density estimation suffers in high dimension。
5. Visualization and fault isolation are difficult。

公式：

```math
z=W^Tx,\qquad z\in R^p,\quad p\ll d
```

讲解方式：

强调降维不是为了漂亮图，而是为了提取系统主要变化模式和故障敏感方向。

可能被问：降维会不会丢 fault 信息？

回答：会有风险。因此 PCA 需要同时监控 retained subspace 和 residual subspace；监督降维如 FDA 会利用标签增强 fault sensitivity。

### Page 3: PCA Core Idea

页面内容：

1. Center data。
2. Compute covariance。
3. Eigen decomposition。
4. Select first \(p\) principal components。

公式：

```math
S=\frac1NXX^T
```

```math
Su_i=\lambda_i u_i
```

```math
z=U_p^Tx
```

讲解方式：

PCA 找的是 variance 最大的正交方向。正常工况中的主要协同变化由前几个 principal components 表示。

可能被问：为什么最大方差方向代表 normal structure？

回答：训练数据通常是 healthy data，最大方差方向代表健康系统最主要的运行变化模式。

### Page 4: Normal Subspace and Residual Subspace

页面内容：

```math
\hat x=U_pU_p^Tx
```

```math
\tilde x=x-\hat x
```

```math
SPE=\|\tilde x\|^2
```

解释：

1. Principal subspace: normal variation。
2. Residual subspace: unexplained variation。
3. Fault often appears as large reconstruction residual。

讲解方式：

把 PCA 看成正常数据骨架。新样本能被骨架解释，说明正常；解释不了，残差变大。

可能被问：PCA 是无监督方法，为什么能检测 fault？

回答：用 healthy data 训练 normal subspace，fault data 偏离该 subspace 时 residual 或 monitoring statistic 上升。

### Page 5: Choosing Number of Principal Components

页面内容：

1. Cumulative variance contribution。
2. Scree plot。
3. Cross validation for detection performance。
4. Engineering interpretability。

公式：

```math
\frac{\sum_{i=1}^p\lambda_i}{\sum_{i=1}^d\lambda_i}\ge \rho
```

讲解方式：

解释 \(p\) 太小会丢正常信息，太大可能把 noise 和 fault direction 放进 normal subspace。

可能被问：保留 95 percent variance 一定好吗？

回答：不一定。FDD 中低方差方向也可能包含 fault 信息，所以需要结合 SPE, T squared 和验证数据判断。

### Page 6: PCA Monitoring Statistics

页面内容：

Hotelling \(T^2\): variation inside principal subspace。

```math
T^2=z^T\Lambda_p^{-1}z
```

SPE or Q statistic: variation outside principal subspace。

```math
Q=\|x-U_pU_p^Tx\|^2
```

讲解方式：

\(T^2\) 看正常子空间内是否过大，SPE 看是否偏离正常子空间。

可能被问：\(T^2\) 和 SPE 的区别？

回答：\(T^2\) 监控主子空间内的异常幅值；SPE 监控残差子空间中的未解释变化。

### Page 7: Direct PCA and Dual PCA

页面内容：

1. Direct PCA: eigen decomposition of \(XX^T\)。
2. Dual PCA: eigen decomposition of \(X^TX\)。
3. When \(d\) large and \(N\) smaller, dual form may be cheaper。

讲解方式：

这一页不需要展开复杂推导。重点说明 dual form 为 KPCA 做铺垫，因为 kernel matrix 就是样本内积矩阵。

可能被问：为什么 KPCA 需要 dual form？

回答：KPCA 中 feature space 维度可能极高，只能通过样本间 inner product 的 kernel matrix 计算。

### Page 8: Kernel PCA Motivation

页面内容：

1. Linear PCA assumes linear subspace。
2. Real systems may lie on nonlinear manifold。
3. Map \(x\) to feature space \(\Phi(x)\)。
4. Perform PCA in feature space。

公式：

```math
K(x_i,x_j)=\Phi(x_i)^T\Phi(x_j)
```

讲解方式：

用弯曲流形解释：原空间中非线性，映射到高维后更接近线性。

可能被问：kernel trick 的核心是什么？

回答：用 kernel function 直接计算 feature space inner product，避免显式构造 \(\Phi(x)\)。

### Page 9: KPCA for Fault Detection

页面内容：

流程：

1. Build kernel matrix。
2. Center kernel matrix。
3. Eigen decomposition。
4. Encode nominal data。
5. Monitor new data by feature space statistics。

讲解方式：

强调 KPCA 和 PCA 的逻辑一致，只是空间不同。

可能被问：KPCA 可以直接计算 \(x-\hat x\) 吗？

回答：一般不方便，因为 \(\Phi(x)\) 可能未知或无限维。通常用 feature space distance 或 kernel based monitoring statistics。

### Page 10: Supervised Dimensionality Reduction

页面内容：

1. PCA only maximizes variance。
2. Fault relevant direction may have low variance。
3. Supervised methods use labels or dependency。
4. Examples: SPCA, HSIC, Metric Learning, SDR。

讲解方式：

强调监督信息能让 projection 更 fault sensitive。

可能被问：PCA 和 SPCA 的区别？

回答：PCA 关注 input variance；SPCA 关注与 output label 或 fault indicator 相关的 variance。

### Page 11: FDA Fisher Discriminant Analysis

页面内容：

目标：投影后类间远，类内紧。

```math
J(w)=\frac{w^TS_Bw}{w^TS_Ww}
```

```math
w\propto S_W^{-1}(\mu_1-\mu_0)
```

讲解方式：

FDA 是有监督降维。它的目标不是保留最大总方差，而是让 normal 和 fault 在低维投影上更容易分开。

可能被问：FDA 与 PCA 的本质差别？

回答：PCA 无监督，找最大方差；FDA 有监督，找最大类间分离与最小类内散布的方向。

### Page 12: MDS Multidimensional Scaling

页面内容：

1. Preserve pairwise distances or similarities。
2. High dimensional \(X\) to low dimensional \(Y\)。
3. Useful for visualization and fault cluster separation。

目标形式：

```math
\min \|X^TX-Y^TY\|_F^2
```

讲解方式：

MDS 的核心是几何结构保持。若 fault cluster 在原空间中远离 normal cluster，低维图中也应该保持这种距离关系。

可能被问：MDS 与 PCA 有什么关系？

回答：Classical MDS 在欧氏距离和 centered data 条件下与 PCA 有密切等价关系，都可由 Gram matrix 特征分解得到低维嵌入。

### Page 13: Isomap

页面内容：

流程：

1. Build kNN graph。
2. Edge weights are local Euclidean distances。
3. Compute shortest path distances。
4. Use shortest paths as geodesic distance approximation。
5. Apply classical MDS。

讲解方式：

Isomap 适合非线性流形。它把全局欧氏距离换成沿流形走的 geodesic distance。

可能被问：Isomap 最怕什么？

回答：k 选择不当。k 太小图可能断开，k 太大可能形成 shortcut，破坏 geodesic distance。

### Page 14: Topic Summary and Method Map

页面内容：

1. PCA: unsupervised linear normal subspace。
2. KPCA: nonlinear normal structure through kernel。
3. FDA: supervised class separation。
4. MDS: pairwise distance preservation。
5. Isomap: nonlinear manifold with geodesic distance。

讲解方式：

用一句话总结：Topic 2 的核心是找到更好的 low dimensional representation，使 fault detection, isolation, visualization 更可靠。

### Page 15: Assignment Position 1

内容建议：

1. Dataset and variables。
2. Objective: dimension reduction or fault detection。
3. Given matrices or samples。
4. Required outputs。

### Page 16: Assignment Position 2

内容建议：

1. PCA or FDA steps。
2. Eigenvalues and selected components。
3. Projection or reconstruction。
4. Fault statistic or plot。

### Page 17: Assignment Position 3

内容建议：

1. Interpretation of low dimensional plot。
2. Whether fault is detectable。
3. Discussion of chosen \(p\), threshold, or labels。
4. Limitations and improvement。

## 三、考官可能提问与回答

### Q1: PCA 为什么适合 FDD？

回答：正常工况下多传感器数据具有相关性，通常位于低维 normal subspace。PCA 通过主成分学习这个结构，新样本若偏离该结构，SPE 或 residual 会变大。

### Q2: PCA 的最大方差一定是 fault sensitive 吗？

回答：不一定。PCA 只看总方差，不看 fault label。某些 fault 信息可能在低方差方向，因此要同时监控 residual subspace 或使用 FDA 等监督降维。

### Q3: \(T^2\) 和 Q statistic 分别检测什么？

回答：\(T^2\) 检测 principal subspace 中 score 的异常幅值；Q statistic 检测样本不能被 principal subspace 解释的残差。

### Q4: KPCA 为什么能处理非线性？

回答：它通过 \(\Phi(x)\) 把数据映射到 feature space，再在该空间做线性 PCA。kernel trick 让这个过程无需显式计算 \(\Phi\)。

### Q5: FDA 为什么需要标签？

回答：FDA 的目标函数包含 class mean 和 within class scatter。如果没有 normal 或 fault 标签，就无法构造 \(S_B\) 和 \(S_W\)。

### Q6: MDS 和 Isomap 的区别？

回答：MDS 通常保持欧氏距离或相似性；Isomap 先用 kNN graph 估计 geodesic distance，再做 MDS，适合非线性流形。

### Q7: Isomap 为什么会受 k 影响？

回答：k 决定邻接图结构。k 太小导致图断开；k 太大导致错误 shortcut，使 geodesic distance 失真。

### Q8: 降维后还需要 classifier 吗？

回答：视任务而定。PCA monitoring 可以直接用 \(T^2\) 和 SPE 检测；如果要 fault isolation，常需在低维特征上接 LDA, Logistic Regression, SVM 等分类器。

## 四、本专题最终总结

Topic 2 的 PPT 应该突出四个判断：

1. PCA 讲 normal subspace。
2. KPCA 讲 nonlinear normal subspace。
3. FDA 讲 supervised fault separation。
4. MDS 和 Isomap 讲 distance or manifold preserving visualization。

最重要的考试表达：

Dimensionality reduction in FDD is not only compression. It is a way to expose normal subspace, fault residuals, class separation, and nonlinear manifold structures in high dimensional sensor data.

---

# Topic 3: Support Vector Machines and Perceptron

## 一、主题定位

本专题回答的问题是：如何用 maximum margin principle 对 normal 和 fault 进行分类。

核心范围包括：

1. Hyperplane, margin, support vectors。
2. Hard Margin SVM。
3. Soft Margin SVM and slack variables。
4. Primal and dual problem。
5. KKT interpretation。
6. Kernel trick。
7. Perceptron。
8. SVM in FDD。

本专题的逻辑主线：

从线性分类边界出发，选择 margin 最大的 boundary；当数据有 noise 或 overlap 时引入 slack；当边界非线性时引入 kernel；Perceptron 作为线性分类器的学习基线。

## 二、17 页 PPT 结构

### Page 1: Title and Big Question

标题：Support Vector Machines and Perceptron

页面内容：

1. Big question: How can we build a fault classifier with good generalization?
2. 关键词：hyperplane, margin, support vectors, slack, kernel。

讲解方式：

说明 SVM 与 Topic 1 的概率分类不同。SVM 重点是 geometric margin 和 constrained optimization。

### Page 2: SVM in FDD Pipeline

页面内容：

Sensor signals → features or PCA/FDA → feature vector \(x\) → SVM → normal or fault class。

适用条件：

1. High dimensional features。
2. Limited labeled samples。
3. Clear boundary after feature extraction。
4. Need robust classification。

讲解方式：

强调 SVM 通常接在 feature extraction 或 dimensionality reduction 后。

可能被问：SVM 能直接处理 raw signal 吗？

回答：可以把 signal window 展成向量，但工程上通常先做特征提取或降维，提高稳定性和可解释性。

### Page 3: Hyperplane and Margin

页面内容：

```math
\beta^Tx+\beta_0=0
```

```math
f(x)=\beta^Tx+\beta_0
```

```math
y_i\in\{-1,+1\}
```

```math
y_i(\beta^Tx_i+\beta_0)>0
```

讲解方式：

超平面是分类边界。乘以 \(y_i\) 后，正确分类样本都有正的 functional margin。

可能被问：为什么要乘 \(y_i\)？

回答：因为两类应位于超平面两侧。乘以标签后，两类正确分类都变成正数，便于统一约束。

### Page 4: Support Vectors

页面内容：

1. Support vectors are closest samples to boundary。
2. They determine the final hyperplane。
3. Far points often have \(\alpha_i=0\)。

讲解方式：

用图解释，真正拉住边界的是最近点。SVM 的稀疏性来自许多样本对最终决策没有直接贡献。

可能被问：删掉非 support vectors 会怎样？

回答：在理想情况下，决策边界通常不变，因为 boundary 由 support vectors 决定。

### Page 5: Hard Margin SVM

页面内容：

Assumption: linearly separable data。

```math
y_i(\beta^Tx_i+\beta_0)\ge 1
```

Optimization:

```math
\min_{\beta,\beta_0}\frac12\|\beta\|^2
```

subject to above constraints。

讲解方式：

解释最大化 margin 等价于最小化 \(\|\beta\|\)。标准化后 margin width 与 \(1/\|\beta\|\) 相关。

可能被问：为什么约束右边可以设为 1？

回答：超平面参数可整体缩放，不改变边界。通过缩放可令最近点 functional margin 等于 1。

### Page 6: Why Hard Margin Fails in Real FDD

页面内容：

1. Sensor noise。
2. Class overlap。
3. Outliers。
4. Operating condition variation。
5. Hard constraints may become infeasible。

讲解方式：

Hard Margin 要求所有点完全分对，工程数据常不满足。

可能被问：Hard Margin 对 outlier 为什么敏感？

回答：一个异常点也会强迫边界改变，可能大幅缩小 margin 或导致不可分。

### Page 7: Soft Margin and Slack Variables

页面内容：

```math
\xi_i\ge 0
```

```math
y_i(\beta^Tx_i+\beta_0)\ge 1-\xi_i
```

解释：

1. \(\xi_i=0\): no violation。
2. \(0<\xi_i<1\): inside margin but correct。
3. \(\xi_i=1\): on boundary。
4. \(\xi_i>1\): misclassified。

讲解方式：

把 slack 讲成允许样本违反 margin 的程度。

可能被问：\(\xi_i\) 是不是分类错误数？

回答：不是。它是 margin violation magnitude。\(\xi_i>1\) 才对应 misclassification。

### Page 8: Soft Margin Primal Problem

页面内容：

```math
\min_{\beta,\beta_0,\xi}\frac12\|\beta\|^2+C\sum_{i=1}^N\xi_i
```

subject to:

```math
y_i(\beta^Tx_i+\beta_0)\ge 1-\xi_i
```

```math
\xi_i\ge0
```

讲解方式：

目标函数有两部分：large margin 和 small violations。\(C\) 控制二者权衡。

可能被问：大 \(C\) 和小 \(C\) 的区别？

回答：大 \(C\) 重视训练误差，可能 margin 窄和过拟合；小 \(C\) 允许更多 violation，margin 宽，可能欠拟合。

### Page 9: Dual Form and KKT Intuition

页面内容：

Dual objective:

```math
\max_\alpha \sum_i\alpha_i-\frac12\sum_i\sum_j\alpha_i\alpha_jy_iy_jx_i^Tx_j
```

Constraints:

```math
0\le\alpha_i\le C
```

```math
\sum_i\alpha_iy_i=0
```

讲解方式：

Dual form 只出现 inner product \(x_i^Tx_j\)，这就是 kernel trick 的入口。

可能被问：\(\alpha_i\) 的含义？

回答：\(\alpha_i\) 表示样本对分类边界的影响强度。\(\alpha_i=0\) 的点不直接影响边界。

### Page 10: Support Vector Types in Soft Margin

页面内容：

1. \(\alpha_i=0\): outside margin, no direct effect。
2. \(0<\alpha_i<C\): on margin, standard support vector。
3. \(\alpha_i=C\): margin violator or misclassified。

讲解方式：

结合图示讲，SVM 的边界由有正 \(\alpha\) 的样本决定。

可能被问：所有 support vectors 都分错了吗？

回答：没有。落在 margin 上的 support vectors 可正确分类；只有 \(\xi_i>1\) 的点分错。

### Page 11: Kernel Trick

页面内容：

```math
K(x_i,x_j)=\Phi(x_i)^T\Phi(x_j)
```

Common kernels:

1. Linear。
2. Polynomial。
3. RBF。

讲解方式：

kernel trick 让模型在 feature space 中做 linear SVM，在原空间中表现为 nonlinear boundary。

可能被问：为什么 dual form 适合 kernel？

回答：dual objective 和 decision function 都只需要样本间 inner product，可以直接把 \(x_i^Tx_j\) 替换成 \(K(x_i,x_j)\)。

### Page 12: Nonlinear Fault Classification with RBF Kernel

页面内容：

RBF kernel:

```math
K(x_i,x_j)=\exp(-\gamma\|x_i-x_j\|^2)
```

参数：

1. \(C\): violation penalty。
2. \(\gamma\): local influence of samples。

讲解方式：

\(\gamma\) 太大边界很弯，容易过拟合；\(\gamma\) 太小边界过平，可能欠拟合。

可能被问：如何调 \(C\) 和 \(\gamma\)？

回答：用 validation 或 cross validation，评价时看 FAR, MAR, F1, recall，而不只看 accuracy。

### Page 13: Perceptron

页面内容：

Decision:

```math
\hat y=sign(w^Tx+b)
```

Update if mistake:

```math
w\leftarrow w+\eta y_ix_i
```

```math
b\leftarrow b+\eta y_i
```

讲解方式：

Perceptron 是线性分类器的经典在线学习算法。它找到某个分离超平面，但不最大化 margin。

可能被问：Perceptron 与 SVM 的区别？

回答：Perceptron 只要分对即可，边界不唯一；SVM 选择最大 margin 的边界，通常泛化更好。

### Page 14: Topic Summary and Method Selection

页面内容：

1. Hard Margin: ideal separable case。
2. Soft Margin: noisy and overlapping data。
3. Kernel SVM: nonlinear boundary。
4. Perceptron: simple online linear classifier。
5. SVM requires feature scaling and parameter tuning。

讲解方式：

总结时强调 SVM 是 geometric classifier，不直接输出 calibrated probability。

### Page 15: Assignment Position 1

内容建议：

1. Data and labels。
2. Feature space plot。
3. Linear or nonlinear separability。
4. Required classifier。

### Page 16: Assignment Position 2

内容建议：

1. Hard or Soft Margin formulation。
2. Parameters \(C\), kernel。
3. Support vectors。
4. Decision boundary or confusion matrix。

### Page 17: Assignment Position 3

内容建议：

1. Classification result。
2. FAR and MAR interpretation。
3. Why this kernel or \(C\)。
4. Limitations and improvement。

## 三、考官可能提问与回答

### Q1: SVM 与 Logistic Regression 的核心差别？

回答：SVM 优化 margin，输出 decision score；Logistic Regression 建模 posterior probability，输出 \(P(y=1|x)\)。SVM 更偏几何优化，Logistic Regression 更偏概率解释。

### Q2: 为什么最大 margin 有助于 generalization？

回答：更大的 margin 表示分类边界远离最近样本，对小扰动和噪声更稳健，因此在未见样本上的分类更稳定。

### Q3: Soft Margin 中 \(C\) 如何影响 FAR 和 MAR？

回答：大 \(C\) 更努力拟合训练故障标签，可能降低训练 MAR，但增加过拟合和 FAR 风险。小 \(C\) 边界更平滑，可能降低 FAR，但可能漏掉小故障。

### Q4: Kernel SVM 的优点和风险？

回答：优点是能处理非线性边界。风险是 kernel 和参数选择不当会过拟合，训练复杂度高，解释性较弱。

### Q5: 为什么 SVM 需要 feature scaling？

回答：margin 和 kernel 都依赖距离或 inner product。尺度大的变量会主导边界，导致模型忽视其他变量。

### Q6: Support vector 是什么？

回答：support vector 是边界附近且 \(\alpha_i>0\) 的训练样本。它们决定最终分类边界。

### Q7: Hard Margin 是否适合真实故障检测？

回答：通常只适合理论或非常干净的数据。真实传感器数据有噪声、outlier、class overlap，所以 Soft Margin 更常用。

### Q8: Perceptron 收敛条件是什么？

回答：若数据线性可分，标准 perceptron 在有限步内能找到一个分离超平面。若不可分，可能持续震荡。

## 四、本专题最终总结

Topic 3 的 PPT 应该突出：

1. SVM 的几何直觉。
2. Hard Margin 的理想假设。
3. Soft Margin 对真实数据的适应。
4. Kernel trick 对非线性边界的扩展。
5. Perceptron 作为线性分类学习基线。

最重要的考试表达：

SVM is a maximum margin classifier. In FDD, it converts extracted features into a robust normal fault boundary, with soft margins handling noise and kernels handling nonlinear fault patterns.

---

# Topic 4: Model Based Fault Detection

## 一、主题定位

本专题回答的问题是：如何利用系统模型生成 residual，并通过 residual 检测和隔离 fault。

核心范围包括：

1. Model based residual generation。
2. Residual evaluation and threshold。
3. Fault modelling。
4. Parity method in time and frequency domain。
5. Observer based methods: Luenberger, Kalman Filter, UIO, ESO, NDO。
6. CUSUM and ARL。

本专题的逻辑主线：

模型给出系统正常行为预测。真实测量与模型预测之间的差就是 residual。健康时 residual 接近 noise，故障时 residual 带有 fault signature。

## 二、17 页 PPT 结构

### Page 1: Title and Big Question

标题：Model Based Fault Detection

页面内容：

1. Big question: How can a mathematical model be used to detect faults?
2. 关键词：analytical redundancy, residual, observer, parity, CUSUM。

讲解方式：

说明 Topic 4 与 Topic 1 的区别：Topic 1 从数据分布出发，Topic 4 从 dynamics model 出发。

### Page 2: Basic Model Based FDD Architecture

页面内容：

Plant and model compare outputs:

```math
r(t)=y_{meas}(t)-\hat y(t)
```

流程：

u, y → model or observer → residual → evaluation → alarm or isolation。

讲解方式：

强调 analytical redundancy：用数学模型替代硬件冗余。

可能被问：为什么 residual 是核心？

回答：residual 被设计为 healthy 时接近零或白噪声，fault 时偏离零，因此它是正常模型与真实系统不一致的证据。

### Page 3: Fault Models and Locations

页面内容：

1. Actuator fault: input channel affected。
2. Sensor fault: measurement channel affected。
3. Plant fault: dynamics affected。

模型：

```math
\dot x=Ax+B(u+f_a)
```

```math
y=Cx+f_s
```

```math
A\rightarrow A_f
```

讲解方式：

不同 fault location 对 residual pattern 的影响不同，这决定 isolation 策略。

可能被问：additive fault 和 multiplicative fault 的区别？

回答：additive fault 以额外项进入系统或测量；multiplicative fault 改变系统参数或增益，例如 actuator effectiveness loss。

### Page 4: Fault Time Behavior

页面内容：

1. Abrupt fault: step like。
2. Incipient fault: drift or ramp。
3. Intermittent fault: appears and disappears。

讲解方式：

不同 fault time profile 对 detection algorithm 要求不同。Abrupt 需要 fast detection，incipient 需要 small change accumulation，intermittent 需要避免短时误报。

可能被问：CUSUM 更适合哪类故障？

回答：适合 small but persistent change，尤其是均值偏移或 residual distribution shift。

### Page 5: Residual Generation and Evaluation

页面内容：

Residual generation:

```math
r=Hy
```

or

```math
r=y-\hat y
```

Residual evaluation:

```math
J(r)>h \Rightarrow alarm
```

讲解方式：

生成 residual 和评价 residual 是两个不同问题。前者设计 signal，后者设计 statistic and threshold。

可能被问：Residual 越大就一定是 fault 吗？

回答：还可能来自 disturbance, noise, model mismatch。因此 residual design 需要对 disturbance 鲁棒，对 fault 敏感。

### Page 6: Parity Method in Time Domain

页面内容：

1. Use input output model over a time window。
2. Eliminate unknown states。
3. Generate parity relation。
4. Fault violates parity relation。

讲解方式：

Parity method 的核心是：健康系统的输入输出必须满足某些模型约束。故障会破坏这些约束。

可能被问：Parity method 依赖什么？

回答：依赖准确 input output model 和合适的 parity vector。模型误差会直接影响 residual。

### Page 7: Parity Method in Frequency Domain

页面内容：

1. Transfer function representation。
2. Use frequency response or polynomial relation。
3. Generate residual in frequency domain。

讲解方式：

频域 parity 适合线性系统和频域模型清楚的场景。

可能被问：time domain 和 frequency domain parity 如何选？

回答：时域适合状态空间和在线递推；频域适合传递函数和频率特征明显系统。

### Page 8: Luenberger Observer Residual

页面内容：

Observer:

```math
\dot{\hat x}=A\hat x+Bu+L(y-C\hat x)
```

Residual:

```math
r=y-C\hat x
```

Error dynamics:

```math
\dot e=(A-LC)e
```

讲解方式：

健康时 observer error 收敛，residual 小。fault 出现后，measurement 或 dynamics mismatch 进入 residual。

可能被问：Observer 设计需要什么条件？

回答：系统需要 observable 或 detectable，才能通过输出误差修正状态估计。

### Page 9: Kalman Filter

页面内容：

1. For noisy systems。
2. Uses process noise covariance \(Q\) and measurement noise covariance \(R\)。
3. Residual or innovation:

```math
\nu_k=y_k-C\hat x_{k|k-1}
```

讲解方式：

Kalman Filter 是 stochastic observer。innovation 在健康模型下应符合预期统计特性。

可能被问：Q 和 R 如何影响滤波？

回答：大 \(Q\) 表示更不信模型，估计更跟随测量；大 \(R\) 表示更不信测量，估计更平滑。

### Page 10: ESO and NDO

页面内容：

ESO idea:

```math
x_e=\begin{bmatrix}x\\d\end{bmatrix}
```

Assume disturbance or fault is slowly varying。

NDO idea:

1. Estimate disturbance in nonlinear systems。
2. Requires nonlinear model and gain design。

讲解方式：

ESO 把未知扰动或故障当成扩展状态估计。适合慢变化未知输入。

可能被问：ESO 的局限？

回答：通常假设未知输入缓慢变化。快速突变时估计可能滞后，高增益还会放大噪声。

### Page 11: UIO Unknown Input Observer

页面内容：

System:

```math
\dot x=Ax+Bu+Ed
```

UIO goal: decouple unknown input from estimation error。

Condition example:

```math
(I-HC)E=0
```

讲解方式：

UIO 不追踪未知输入，而是通过矩阵条件让未知输入不污染状态估计误差。

可能被问：ESO 和 UIO 的根本差别？

回答：ESO 是 estimation approach，把 unknown input 当状态估计；UIO 是 decoupling approach，使 unknown input 不进入 error dynamics。

### Page 12: CUSUM Logic

页面内容：

Likelihood increment:

```math
s(x_k)=\ln\frac{p(x_k|H_1)}{p(x_k|H_0)}
```

Recursive CUSUM:

```math
g(k)=\max(0,g(k-1)+s(x_k))
```

Alarm:

```math
g(k)>h
```

讲解方式：

CUSUM 累积的是 fault evidence，不是原始信号。正常时证据平均为负，故障后平均为正。

可能被问：max with zero 的作用？

回答：清除过去负证据，只保留最近一段持续支持 fault 的证据。

### Page 13: ARL and Threshold Design

页面内容：

```math
\tau_D=E[k_a-k_f|H_1]
```

```math
\tau_F=E[k_a|H_0]
```

解释：

1. \(\tau_D\): average detection delay, smaller better。
2. \(\tau_F\): average false alarm interval, larger better。
3. Threshold \(h\) balances both。

讲解方式：

ARL 把 threshold tuning 从经验调参变成可量化工程目标。

可能被问：提高 threshold 对 ARL 有什么影响？

回答：通常提高 \(\tau_F\)，减少 false alarm，但也提高 \(\tau_D\)，检测更慢。

### Page 14: Method Selection Summary

页面内容：

1. Parity: reliable model and direct residual。
2. Luenberger: deterministic LTI and observability。
3. Kalman: noisy stochastic system。
4. ESO: slow unknown disturbance。
5. UIO: unknown input direction known and rank condition holds。
6. CUSUM: persistent small changes。

讲解方式：

展示你知道每种方法的适用条件和限制。

### Page 15: Assignment Position 1

内容建议：

1. System model。
2. Fault type。
3. Given matrices or transfer functions。
4. Required residual or detector。

### Page 16: Assignment Position 2

内容建议：

1. Residual generation method。
2. Observer or parity design。
3. CUSUM statistic or threshold。
4. Calculation results。

### Page 17: Assignment Position 3

内容建议：

1. Fault decision。
2. Delay and false alarm discussion。
3. Isolation interpretation。
4. Limitations due to model mismatch or noise。

## 三、考官可能提问与回答

### Q1: Model based FDD 和 data driven FDD 的区别？

回答：Model based FDD 使用系统模型生成 residual；data driven FDD 依赖数据分布、特征和分类器。前者解释性强但依赖模型，后者模型要求低但依赖数据覆盖。

### Q2: Residual 健康时为什么不一定等于零？

回答：因为存在 noise, disturbance, model mismatch, numerical error。健康 residual 通常期望为小幅白噪声或在统计界限内。

### Q3: 如何实现 fault isolation？

回答：可以设计 structured residuals，使不同 fault 激活不同 residual pattern；也可以用 observer bank 或 residual signature matrix。

### Q4: Kalman Filter 为什么适合 noisy system？

回答：它显式建模 process noise 和 measurement noise，用 covariance 权衡模型预测和测量校正。

### Q5: UIO 的主要限制？

回答：需要未知输入方向矩阵已知，且满足 rank 或 decoupling 条件。若条件不成立，无法完全消除未知输入对估计误差的影响。

### Q6: CUSUM 比单点阈值强在哪里？

回答：CUSUM 能累积小但持续的异常证据，因此对 small persistent faults 更敏感。

### Q7: ARL 有什么工程意义？

回答：\(\tau_F\) 描述平均 false alarm 间隔，\(\tau_D\) 描述平均检测延迟。它们用于制定阈值和评价 detector。

### Q8: Model mismatch 会造成什么问题？

回答：模型不准会产生 residual bias，导致 false alarm 或掩盖 fault。需要 robust residual, adaptive model, threshold tuning 或 model validation。

## 四、本专题最终总结

Topic 4 的 PPT 应突出：

1. Residual 是模型预测与真实测量的差。
2. Observer, parity, Kalman, UIO, ESO 都是 residual generation 思路。
3. CUSUM 和 ARL 是 residual evaluation 和 threshold 设计工具。
4. 成功的 model based FDD 要求 fault sensitivity 和 disturbance robustness 同时考虑。

最重要的考试表达：

Model based FDD uses analytical redundancy. A fault is detected when the measured behavior can no longer be explained by the nominal model, and this inconsistency is captured by residuals and evaluated statistically.

---

# Topic 5: Fault Tolerant Control Strategy

## 一、主题定位

本专题回答的问题是：故障发生后，系统如何通过 robust, adaptive or reconfigurable control 保持稳定和可接受性能。

核心范围包括：

1. Fault locations and redundancy。
2. Passive FTC and Active FTC。
3. Robust and adaptive control。
4. Reconfigurable control system architecture。
5. Model matching。
6. Transient elimination and bumpless transfer。
7. Virtual actuator。
8. Virtual sensor。
9. Physical limits: controllability and observability after fault。

本专题的逻辑主线：

Topic 4 负责诊断 fault。Topic 5 使用 fault information 调整 controller, actuator path or sensor path，让 faulty plant 尽量维持稳定、跟踪和性能。

## 二、17 页 PPT 结构

### Page 1: Title and Big Question

标题：Fault Tolerant Control Strategy

页面内容：

1. Big question: What should the controller do after a fault is detected?
2. 关键词：passive FTC, active FTC, reconfiguration, model matching, virtual actuator, virtual sensor。

讲解方式：

说明本专题从 detection 进入 control action。

### Page 2: FTC Architecture and Fault Locations

页面内容：

1. Actuator fault: \(B\rightarrow B_f\), control authority changes。
2. Plant fault: \(A\rightarrow A_f\), dynamics changes。
3. Sensor fault: \(C\rightarrow C_f\), feedback corrupted。

讲解方式：

故障位置决定重构方式。执行器故障需要重新分配控制，传感器故障需要恢复或替代测量。

可能被问：为什么 sensor fault 和 actuator fault 的补偿策略不同？

回答：sensor fault 污染 feedback information；actuator fault 改变 actual input to plant。前者需要清洗或重构测量，后者需要补偿或重分配控制输入。

### Page 3: Redundancy as the Prerequisite

页面内容：

1. Hardware redundancy: extra sensors, actuators, backup channels。
2. Analytical redundancy: model and remaining measurements reconstruct missing signals。
3. No redundancy means limited tolerance。

讲解方式：

强调容错控制有物理上限。控制算法不能创造不存在的 actuator authority 或 measurement information。

可能被问：冗余为什么是 FTC 前提？

回答：故障后仍需要足够 control channel 或 information channel。如果故障导致系统不可控或不可观，就无法完全恢复 nominal behavior。

### Page 4: Passive FTC

页面内容：

Logic:

Design one robust controller for expected fault range。

Advantages:

1. Simple online。
2. No switching。
3. No diagnosis dependency。

Disadvantages:

1. Conservative。
2. Limited to predefined faults。
3. Normal performance may be sacrificed。

讲解方式：

Passive FTC 把 fault 当成 uncertainty 或 disturbance，由固定 controller 承受。

可能被问：Passive FTC 适合什么场景？

回答：故障范围已知、较小、诊断不可靠、系统不希望在线切换时适用。

### Page 5: Active FTC

页面内容：

Logic:

FDD → diagnosis → decision → controller or sensor or actuator reconfiguration。

Advantages:

1. Better performance after severe fault。
2. Can use fault information。
3. Less conservative。

Disadvantages:

1. Relies on diagnosis quality。
2. Switching can cause transient shock。
3. More complex。

讲解方式：

Active FTC 的关键是 fault information 必须足够快、准、稳定。

可能被问：Active FTC 最大风险是什么？

回答：错误诊断或延迟诊断会触发错误重构，可能使 closed loop 变差甚至不稳定。

### Page 6: Robust Control and Adaptive Control

页面内容：

Robust control:

1. Fixed controller。
2. Handles bounded uncertainty。
3. Conservative。

Adaptive control:

1. Parameters update online。
2. Handles changing dynamics。
3. Needs stability safeguards。

讲解方式：

Robust 解决已知范围内的不确定性；adaptive 根据在线信息调参数。

可能被问：Robust 和 adaptive 如何对应 passive 与 active FTC？

回答：Passive FTC 常用 robust control；Active FTC 常结合 adaptive or reconfigurable control。

### Page 7: Reconfigurable Control Systems

页面内容：

1. Controller bank。
2. Gain scheduling。
3. Multiple model control。
4. LPV control。
5. Online synthesized control。

讲解方式：

解释预先设计多个 controller，对不同 operating point 或 fault mode 切换。

可能被问：为什么需要 controller bank？

回答：不同故障模式和工况下最优 controller 不同，一个固定 controller 难以兼顾所有情况。

### Page 8: Model Matching

页面内容：

Nominal closed loop:

```math
A_{des}=A-BK
```

Faulty system:

```math
\dot x=A_fx+B_fu
```

Reconfigured controller:

```math
A_f-B_fK_f=A-BK
```

讲解方式：

Model Matching 的目标是让 faulty closed loop mimic nominal closed loop。

可能被问：Model matching 匹配的是开环还是闭环？

回答：匹配闭环动态，因为 control performance 由 closed loop poles and transient behavior 决定。

### Page 9: Exact and Approximate Model Matching

页面内容：

1. Exact model matching: equation exactly solvable。
2. Approximate model matching: minimize mismatch norm。
3. Feasibility depends on \(B_f\), controllability and actuator authority。

讲解方式：

如果 \(B_f\) rank 下降，可能无法 exact matching，只能做最佳近似。

可能被问：什么时候 model matching 不可行？

回答：故障后控制输入通道不足，\(B_f\) 无法产生所需 closed loop correction，或系统失去 controllability。

### Page 10: Sensor Reconfiguration and Virtual Sensor

页面内容：

Sensor fault corrupts feedback:

```math
y=Cx+f_s
```

Virtual sensor:

1. Use observer and healthy measurements。
2. Reconstruct missing or faulty output。
3. Provide clean feedback to controller。

讲解方式：

Virtual sensor 的任务是让 controller 看到类似 healthy sensor 的信号。

可能被问：Virtual sensor 成功需要什么条件？

回答：剩余传感器和模型必须保留 observability 或足够 analytical redundancy。

### Page 11: Virtual Actuator

页面内容：

Actuator fault changes input mapping。

Virtual actuator idea:

1. Sits between nominal controller and faulty plant。
2. Transforms nominal command。
3. Makes faulty plant appear like nominal plant to controller。

讲解方式：

Virtual actuator 尽量保持原 controller 不变，通过接口层补偿 actuator fault。

可能被问：Virtual actuator 与重新设计 controller 的区别？

回答：Virtual actuator 保留 nominal controller，加入中间补偿模块；直接重设计 controller 会改变控制律本身。

### Page 12: Transient Elimination and Bumpless Transfer

页面内容：

Problem:

Controller switching can cause input jump。

Bumpless transfer logic:

1. Multiple controllers run in parallel。
2. Passive controller tracks active controller output。
3. Before switching, \(u_i\approx u\)。
4. Switching causes small bump。

Formula:

```math
e_u=u-u_i
```

```math
\dot x_{ci}=A_{ci}x_{ci}+B_{ci}y_{ref}+E_{ci}y+L_i(u-u_i)
```

讲解方式：

备用控制器在后台同步状态，避免切换瞬间输出突变。

可能被问：为什么 controller switching 会有 bump？

回答：两个 controller 内部状态不同，即使输入输出相同，计算出的 control input 也可能不同，切换瞬间造成 actuator command jump。

### Page 13: Physical Limits of FTC

页面内容：

1. Need controllability after actuator fault。
2. Need observability after sensor fault。
3. Need enough actuator authority。
4. Need diagnosis fast enough。
5. Need constraints not infeasible。

讲解方式：

这是考官很看重的工程判断。容错控制不能承诺恢复所有故障。

可能被问：严重 actuator fault 下还能不能保持 nominal performance？

回答：若剩余 actuator authority 不足，只能退化性能或进入 safe mode，不能保证 nominal tracking。

### Page 14: Topic Summary

页面内容：

1. Passive FTC: fixed robust controller。
2. Active FTC: diagnosis driven reconfiguration。
3. Model matching: recover nominal closed loop dynamics。
4. Virtual sensor and actuator: hide faults from nominal controller。
5. Bumpless transfer: reduce switching transient。
6. Redundancy sets the upper bound。

讲解方式：

用一句话总结：Topic 5 是从 fault information 到 control reconfiguration 的桥梁。

### Page 15: Assignment Position 1

内容建议：

1. Faulty system model。
2. Nominal controller。
3. Fault scenario。
4. Required reconfiguration。

### Page 16: Assignment Position 2

内容建议：

1. Model matching equation。
2. Calculation of \(K_f\) or compensation。
3. Stability or pole comparison。
4. Simulation result。

### Page 17: Assignment Position 3

内容建议：

1. Performance before and after fault。
2. Transient response。
3. Constraint or actuator authority discussion。
4. Limitations and possible safe mode。

## 三、考官可能提问与回答

### Q1: Passive FTC 和 Active FTC 的差别？

回答：Passive FTC 使用固定 robust controller，提前覆盖预期故障。Active FTC 依赖 FDD 结果，在线切换或重构 controller, sensor, actuator path。

### Q2: FTC 与 FDD 的关系？

回答：FDD 提供 fault detection, isolation, identification。FTC 使用这些信息进行 control accommodation or reconfiguration。

### Q3: Model matching 的目标是什么？

回答：使 faulty closed loop dynamics 尽量匹配 nominal closed loop dynamics，例如 \(A_f-B_fK_f=A-BK\)。

### Q4: Exact model matching 失败怎么办？

回答：可做 approximate model matching，性能降级，重新分配控制，放宽参考，或进入 safe mode。

### Q5: Virtual actuator 解决什么问题？

回答：解决 actuator fault 后 nominal controller 与 faulty plant 不匹配的问题。它在 controller 和 plant 之间转换控制命令。

### Q6: Virtual sensor 解决什么问题？

回答：解决 sensor fault 导致反馈污染的问题。它用模型和剩余健康测量重构可靠输出。

### Q7: Bumpless transfer 为什么重要？

回答：controller 切换时若控制输入突跳，会冲击 actuator 和 plant，可能导致 transient overshoot or instability。

### Q8: 为什么冗余决定 FTC 上限？

回答：剩余 actuator 决定能否控制，剩余 sensor 决定能否估计。无 controllability 或 observability 时，算法无法恢复必要信息或控制能力。

## 四、本专题最终总结

Topic 5 的 PPT 应突出：

1. 从 diagnosis 到 reconfiguration。
2. Passive 与 Active 的设计哲学。
3. Model matching 是核心数学表达。
4. Virtual actuator 和 virtual sensor 是结构性容错接口。
5. Bumpless transfer 处理切换瞬态。
6. 物理可控性和可观测性是理论边界。

最重要的考试表达：

Fault tolerant control is not only detecting faults. It uses diagnosis information and redundancy to reconfigure the control loop so that the faulty system remains stable, safe, and sufficiently performant.

---

# Topic 6: Safe and Secure Control for CPS

## 一、主题定位

本专题回答的问题是：在 cyber physical systems 中，面对 fault, attack, anomaly, constraints and uncertainty，如何检测异常并保持安全控制。

核心范围包括：

1. Attack and anomaly detection。
2. Resilient control for CPS。
3. Safe control synthesis。
4. LQR to constrained optimal control。
5. MPC and receding horizon。
6. Terminal set and terminal cost。
7. Reference tracking and target selection。
8. Offset free MPC。
9. LESO or UIO plus MPC for AFTC。
10. VAE anomaly detection。

本专题的逻辑主线：

CPS 需要同时面对 fault, cyber attack, disturbance and hard constraints。控制目标的优先级是先保证 safety constraints，再保证 stability，再恢复 tracking performance。

## 二、17 页 PPT 结构

### Page 1: Title and Big Question

标题：Safe and Secure Control for CPS

页面内容：

1. Big question: How can CPS remain safe under faults, attacks, disturbances, and constraints?
2. 关键词：anomaly detection, resilient estimation, MPC, terminal set, offset free, VAE。

讲解方式：

说明 Topic 6 是整门课的终点：从检测到约束内安全控制。

### Page 2: CPS Faults, Attacks, and Anomalies

页面内容：

1. Fault: physical component deviation。
2. Attack: malicious manipulation of sensor, actuator, communication or data。
3. Anomaly: observed abnormal behavior。
4. Disturbance: non malicious unknown input。

讲解方式：

强调 anomaly 是观测层概念，原因可能是 fault, attack, disturbance 或工况变化。

可能被问：attack 和 fault 的区别？

回答：fault 通常是非恶意物理或部件异常；attack 是恶意 cyber manipulation。两者都可能表现为 residual or data anomaly。

### Page 3: Safe and Resilient CPS Control Chain

页面内容：

Anomaly detection → resilient state estimation → fault or attack isolation → target selection → constrained MPC → safe operation。

Control priorities:

1. Maintain safety constraints。
2. Maintain stability。
3. Maintain tracking performance。
4. Recover nominal operation if possible。

讲解方式：

解释安全控制中性能不是第一优先级。先不越界，再谈 tracking。

可能被问：diagnosis uncertain 时怎么办？

回答：可以收紧 constraints，降低 reference，切换 safe mode，隔离可疑通道，使用冗余估计。

### Page 4: From LQR to Constrained Optimal Control

页面内容：

LQR cost:

```math
J=\sum x_k^TQx_k+u_k^TRu_k
```

Problem:

1. LQR no hard constraints。
2. Real systems have input and state limits。
3. Need constrained finite horizon optimization。

讲解方式：

LQR 提供 optimal control 背景，但现实系统有 voltage, force, temperature, position constraints。

可能被问：为什么 Riccati LQR 不够？

回答：无约束 LQR 得到 linear feedback，但当输入或状态受限时，最优解可能在约束边界，无法由固定 Riccati gain 直接表达。

### Page 5: Finite Horizon Constrained Optimization

页面内容：

```math
\min_{u_0,...,u_{N-1}}\sum_{k=0}^{N-1}(x_k^TQx_k+u_k^TRu_k)+x_N^TSx_N
```

subject to:

```math
x_{k+1}=Ax_k+Bu_k
```

```math
x_k\in\mathcal X,\quad u_k\in\mathcal U
```

讲解方式：

这是 MPC 的优化核心。线性模型、二次代价、线性约束通常形成 QP。

可能被问：QP 的变量是什么？

回答：可以只优化未来输入序列 \(U\)，也可以同时优化未来状态 \(X\) 和输入 \(U\)。

### Page 6: MPC and Receding Horizon

页面内容：

MPC loop:

1. Measure current state。
2. Solve finite horizon optimization。
3. Apply first input。
4. Shift horizon and repeat。

讲解方式：

MPC 把 open loop optimization 变成 feedback control，因为每个 sampling time 都重新测量和优化。

可能被问：为什么只执行第一个 input？

回答：未来预测会因 disturbance 和 model mismatch 改变。执行第一个 input 后重新测量，可保持 feedback correction。

### Page 7: State Elimination and QP Form

页面内容：

```math
X=\mathcal A x_0+\mathcal B U
```

QP:

```math
\min_U \frac12U^THU+f(x_0)^TU
```

讲解方式：

State elimination 减少优化变量，但 Hessian 可能变 dense。

可能被问：State elimination 的优缺点？

回答：优点是变量少、实现直接；缺点是矩阵稠密、长 horizon 或不稳定系统下数值条件可能差。

### Page 8: MPC Stability Issue

页面内容：

Finite horizon MPC may be unstable without terminal ingredients。

Stabilizing tools:

1. Terminal equality。
2. Terminal set。
3. Terminal cost。

讲解方式：

有限 horizon 可能只看眼前代价，不保证长期稳定，需要 terminal design 补上未来。

可能被问：terminal equality 为什么太严格？

回答：要求有限步内精确到 origin，输入受限时可能 infeasible。

### Page 9: Terminal Set and Terminal Cost

页面内容：

Terminal set:

```math
x_{t+N|t}\in\mathcal X_f
```

Terminal cost:

```math
x_N^TPx_N
```

Local controller:

```math
u=K_fx
```

讲解方式：

Terminal set 是安全降落区。进入后局部 controller 能保持 constraints 并收敛。

可能被问：positive invariant set 是什么？

回答：若 \(x\in\mathcal X_f\)，应用 local controller 后下一步仍在 \(\mathcal X_f\)。这保证 recursive feasibility。

### Page 10: Reference Tracking and Target Selection

页面内容：

Steady state target:

```math
\begin{bmatrix}I-A&-B\\C&0\end{bmatrix}
\begin{bmatrix}x_s\\u_s\end{bmatrix}
=
\begin{bmatrix}0\\y_{spec}\end{bmatrix}
```

Shifted variables:

```math
\tilde x=x-x_s,\quad \tilde u=u-u_s
```

讲解方式：

Tracking 非零参考时，先找可行稳态 \((x_s,u_s)\)，再用 MPC 调节偏差。

可能被问：为什么不能直接惩罚 \(x-x_{spec}\)？

回答：可能没有对应的稳态输入，或输入约束下不可达。Target selection 同时保证 steady state equations 和 constraints。

### Page 11: Offset Free MPC

页面内容：

Augmented disturbance:

```math
x_{k+1}=Ax_k+Bu_k+B_dd_k
```

```math
d_{k+1}=d_k
```

Observer estimates:

```math
\hat x_k,\quad \hat d_k
```

Modified target selector:

```math
\begin{bmatrix}I-A&-B\\C&0\end{bmatrix}
\begin{bmatrix}x_s\\u_s\end{bmatrix}
=
\begin{bmatrix}B_d\hat d_k\\y_{spec}-C_d\hat d_k\end{bmatrix}
```

讲解方式：

Offset free MPC 用扰动估计修正 steady state target，消除模型不匹配或恒定扰动导致的稳态误差。

可能被问：Offset free MPC 的执行逻辑？

回答：Measure, estimate, target selection, dynamic optimization, actuate。实际输入为 \(u_k=u_s+\tilde u^*_{0|t}\)。

### Page 12: LESO plus MPC for AFTC

页面内容：

Faulty model:

```math
x_{k+1}=Ax_k+B(u_k+f_{a,k})
```

```math
y_k=Cx_k+C_{fs}f_{s,k}
```

LESO estimates:

```math
\hat x_k,\quad \hat f_{a,k},\quad \hat f_{s,k}
```

讲解方式：

LESO 提供状态和故障估计。MPC 使用 cleaned state，target selector 补偿 actuator fault。

可能被问：actuator fault 和 sensor fault 在 MPC 中如何处理？

回答：actuator fault 改变物理输入，需要 target selector 调整 baseline input；sensor fault 污染测量，需要 observer 提供 cleaned state estimate。

### Page 13: UIO plus MPC for Resilient Control

页面内容：

Unknown input model:

```math
x_{k+1}=Ax_k+Bu_k+Ef_k
```

UIO decoupling:

```math
TE=0
```

or continuous form:

```math
(I-HC)E=0
```

讲解方式：

UIO 防止 actuator fault 或 attack 进入 state estimation error，再将 clean state 和 reconstructed fault 交给 MPC。

可能被问：LESO 与 UIO 在 resilient MPC 中如何选择？

回答：LESO 适合慢变化故障估计，结构条件较宽但可能滞后；UIO 解耦更快，但需要严格 rank and structure conditions。

### Page 14: VAE Anomaly Detection for CPS

页面内容：

Training:

Healthy data → VAE learns healthy manifold。

Online:

```math
x_{live}\rightarrow encoder\rightarrow z\rightarrow decoder\rightarrow anomaly score
```

Scores:

1. Low reconstruction likelihood。
2. Large latent deviation。
3. Low ELBO。

讲解方式：

VAE 是 generative anomaly detector，适合 unknown faults or cyber physical anomalies。

可能被问：VAE 为什么适合 unknown fault detection？

回答：它只需学习 healthy behavior 的概率结构。未知异常会表现为低 likelihood 或 latent deviation。

### Page 15: Assignment Position 1

内容建议：

1. CPS model or MPC setup。
2. Constraints。
3. Fault, disturbance or attack scenario。
4. Required safe control objective。

### Page 16: Assignment Position 2

内容建议：

1. QP formulation or target selector。
2. Observer, LESO, UIO or VAE anomaly score。
3. Simulation results。
4. Constraint satisfaction plot。

### Page 17: Assignment Position 3

内容建议：

1. Safety result。
2. Tracking result。
3. Fault or attack response。
4. Feasibility and limitation discussion。

## 三、考官可能提问与回答

### Q1: Safe control 和 normal tracking control 的优先级有什么不同？

回答：Safe control 先保证 state and input constraints，再保证 stability，然后才是 tracking performance。安全优先于性能。

### Q2: MPC 为什么适合 CPS safety？

回答：MPC 能显式处理 input and state constraints，并每个采样时刻重新优化，适合在扰动和故障下保持约束安全。

### Q3: Terminal set 如何保证稳定？

回答：MPC 只需把预测末端带入 terminal set，之后 local controller 能在不违反约束的情况下保持在该 set 内并收敛。结合 terminal cost 可证明 cost decrease。

### Q4: Offset free MPC 为什么能消除稳态误差？

回答：它估计 constant disturbance，把 \(\hat d\) 写入 steady state target equations，得到补偿后的 \((x_s,u_s)\)，再调节偏差变量。

### Q5: LESO plus MPC 的关键逻辑？

回答：LESO 估计状态、故障和扰动；MPC 使用 cleaned estimate 做动态优化；target selector 用故障估计修正 steady state input。

### Q6: UIO plus MPC 的优势？

回答：UIO 用结构解耦条件使 unknown input 不污染 state estimate，MPC 可基于 clean state 做安全优化。

### Q7: VAE 与普通 Autoencoder 的区别？

回答：普通 AE 把输入映射为 deterministic latent point。VAE 把输入映射为 latent distribution，并通过 KL regularization 形成连续紧凑 latent space。

### Q8: VAE 的 anomaly score 为什么比单纯 MSE 更强？

回答：它同时看 reconstruction likelihood 和 latent deviation。故障可能重构误差不明显，但 latent state 已偏离 healthy prior。

## 四、本专题最终总结

Topic 6 的 PPT 应突出：

1. CPS 中 fault, attack, anomaly 的关系。
2. Safe control 的首要目标是 constraints。
3. MPC 是 constrained safe synthesis 的核心工具。
4. Terminal set and terminal cost 保证稳定和可行性。
5. Offset free MPC 解决 steady state offset。
6. LESO or UIO plus MPC 实现 resilient FTC。
7. VAE 用于 unknown anomaly detection。

最重要的考试表达：

Safe and secure CPS control integrates anomaly detection, resilient state estimation, constrained target selection, and MPC. The controller should first preserve safety constraints, then maintain stability, and finally recover performance when feasible.

---

# 全部专题的横向校对

## 1. 主题边界校对

1. Topic 1: 统计信号处理和概率分类。包含 Bayes, PDF estimation, LDA, Logistic Regression。阈值、FAR, MAR, MLE, MAP, Parzen, kNN 是支撑内容。
2. Topic 2: 降维和表示学习。包含 PCA, KPCA, FDA, MDS, Isomap。分类器本身留给 Topic 1 和 Topic 3。
3. Topic 3: SVM 和 Perceptron。包含 Hard Margin, Soft Margin, Kernel Trick, Perceptron。
4. Topic 4: 模型故障检测。包含 residual, parity, observer, UIO, KF, ESO, NDO, CUSUM, ARL。
5. Topic 5: 故障容错控制策略。包含 robust, adaptive, reconfigurable control, model matching, transient elimination, virtual actuator, virtual sensor。
6. Topic 6: CPS 安全与韧性控制。包含 attack or anomaly detection, resilient control, safe synthesis, MPC, offset free MPC, LESO or UIO plus MPC, VAE anomaly detection。

## 2. 页数校对

每个 Topic 均为 17 页：

1. 第 1 到 14 页：知识讲解。
2. 第 15 到 17 页：assignment。
3. 每页只保留一个核心目的。
4. 每个 Topic 都有 summary slide。
5. 每个 Topic 都有 Q&A 准备。

## 3. 逻辑完整性校对

整门课可串成一条线：

Sensor data or system model → fault detection → fault isolation and identification → fault tolerant control → constrained safe resilient operation。

Topic 1 到 Topic 3 覆盖 data driven FDD。

Topic 4 覆盖 model based FDD。

Topic 5 覆盖 control reconfiguration。

Topic 6 覆盖 constrained safe and secure CPS control。

## 4. 考官视角总检查

考官最可能看四件事：

1. 你是否知道每个方法解决什么问题。
2. 你是否知道每个公式中的变量和物理意义。
3. 你是否知道方法假设和失败条件。
4. 你是否能把 assignment 结果放回课程主线解释。

## 5. 最终制作建议

每页 PPT 建议采用固定结构：

1. 标题：一句问题。
2. 左侧：公式或流程图。
3. 右侧：三到五个解释点。
4. 底部：一句 takeaway。

例如：

Title: Why is CUSUM better than a single threshold?

Formula: \(g(k)=\max(0,g(k-1)+s(x_k))\)

Explanation: single sample evidence, accumulated evidence, reset mechanism, threshold。

Takeaway: CUSUM detects small persistent changes by accumulating log likelihood evidence。

## 6. 最终答辩策略

1. 先回答概念，再回答公式。
2. 先说明假设，再说明优缺点。
3. 先解释工程意义，再讲数学细节。
4. 不要承诺方法能解决所有故障。
5. 遇到 severe fault, infeasible constraints, loss of observability, loss of controllability 时，要明确说明系统只能 degraded operation or safe mode。

