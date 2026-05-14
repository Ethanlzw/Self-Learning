# Fault Tolerant Control

## How to Use This File

- `【核心】` means this point should appear in an oral exam answer.
- `【公式】` means the formula or notation should be remembered.
- `【口述】` gives a simple sentence that can be spoken directly.
- `【易问】` marks likely examiner questions.
- `【边界】` explains what should not be repeated in other topics.

## Course-Wide Knowledge Map

| Topic | Core question | Core methods | Key formulas / symbols | Boundary |
|---|---|---|---|---|
| Topic 1. Fault and Statistical Signal Processing | How do we decide normal vs fault from sensor signals and probabilities? | Features, metrics, Bayes, PDF estimation, LDA, QDA, logistic regression | FAR, MAR, Bayes, MAP, LRT, MLE, MAP, Parzen, kNN | Probability/statistical classifiers only; PCA goes Topic 2, SVM goes Topic 3 |
| Topic 2. Dimensionality Reduction | How do we compress high-dimensional sensor data while keeping fault information? | PCA, KPCA, SPCA, FDA, MDS, Isomap | `Su_i=lambda_i u_i`, SPE/Q, Hotelling T2, FDA scatter ratio | Representation learning only; downstream classification belongs to Topic 1 or 3 |
| Topic 3. SVM and Perceptron | How do we classify faults using maximum-margin decision boundaries? | Hard margin, soft margin, dual form, kernel trick, perceptron | `y_i(beta^T x_i+beta_0)>=1`, slack `xi_i`, kernel `K(x_i,x_j)` | Maximum-margin route only; probability route belongs to Topic 1 |
| Topic 4. Model-Based Fault Detection | How do we use a model to generate residuals and detect changes? | Residuals, parity, observers, KF, ESO/LESO, UIO, NDO, CUSUM, ARL | `r=y-yhat`, UIO rank condition, CUSUM recursion | Diagnosis ends at detection/isolation/estimation; controller redesign is Topic 5/6 |
| Topic 5. Fault Tolerant Control Strategy | After a fault is diagnosed, how do we reconfigure control? | Passive/active FTC, robust/adaptive control, model matching, virtual actuator/sensor, bumpless transfer | `A_f-B_fK_f=A-BK`, `K_f C_f=KC` | FTC strategy and reconfiguration; hard constraints and CPS security belong to Topic 6 |
| Topic 6. Safe/Secure Control for CPS | How do we keep CPS safe under constraints, faults, attacks, and anomalies? | Attack/anomaly detection, MPC, terminal set/cost, offset-free MPC, LESO/UIO + MPC, VAE anomaly detection | QP, `X=Abar x0+Bbar U`, terminal set, ELBO | Safe/resilient constrained control; not just fault detection |

## Course-Wide Main Logic

Fault Tolerant Control is a chain, not a single method:

```text
sensor/model data
-> feature or residual generation
-> fault detection
-> fault isolation
-> fault identification / estimation
-> controller reconfiguration
-> safe and resilient operation under constraints
```

【口述】这门课的主线是：先发现系统是否故障，再判断故障在哪里、大小是多少，最后根据诊断结果重新配置控制器，让系统在故障、约束甚至攻击下仍然安全运行。

## Basic Terms Used Across All Topics

| Term | Meaning | Oral explanation |
|---|---|---|
| Fault | A component or signal deviates from allowed behavior | `fault` 是部件异常，例如 sensor bias 或 actuator loss |
| Failure | The system can no longer perform the required function | `failure` 是 fault 的严重后果 |
| Disturbance | Unknown input or environmental influence | disturbance 不一定是损坏，例如负载变化 |
| Noise | Random measurement or process uncertainty | noise 通常是随机误差 |
| Anomaly | Observed abnormal behavior | anomaly 可能来自 fault、attack、disturbance 或 noise |
| Attack | Malicious manipulation in CPS | attack 是人为恶意注入或篡改 |
| Detection | Whether a fault exists | 判断有没有故障 |
| Isolation | Where or which type of fault | 判断故障位置或类型 |
| Identification | Size/time/direction of fault | 估计故障大小、时间或参数 |
| FTC | Fault diagnosis plus control reconfiguration | 诊断告诉控制器发生了什么，重构决定怎么继续控制 |

---

# Topic 1. Introduction to Fault and Statistical Signal Processing

## Boundary and Objective

【边界】Topic 1 covers signal-based and probability-based FDD: sensor features, detector metrics, Bayes decision theory, PDF estimation, LDA/QDA, and logistic regression. It does not fully explain PCA/FDA/MDS/Isomap, which belong to Topic 2, or SVM, which belongs to Topic 3.

【口述】Topic 1 的目标是把原始传感器信号变成特征，再用概率和统计分类方法判断 normal 或 fault。

## Required Concepts

| Knowledge point | Marked explanation |
|---|---|
| Signal-based FDD | 【核心】Input is sensor signals or features; output is normal/fault or fault type. It is easy to implement but sensitive to noise and operating condition changes. |
| Threshold alarm | 【核心】Alarm if a signal or statistic exceeds a threshold. Simple but can cause false alarms under noise or missed alarms under weak faults. |
| Feature extraction | 【核心】Raw time series is transformed into diagnostic quantities. Time features include mean, variance, RMS, peak-to-peak, skewness, kurtosis, crest factor. Frequency features include FFT, PSD, dominant frequency, band energy, wavelet coefficients. |
| Feature selection | 【核心】Keep features sensitive to faults but not too sensitive to noise. |
| Detection vs isolation | 【核心】Detection is normal vs fault. Isolation is normal vs fault type 1 vs fault type 2. |
| Supervised vs unsupervised | 【核心】Supervised methods use labels; unsupervised methods model normal data or clusters without labels. |
| Confusion matrix | 【核心】TP, TN, FP, FN are used to evaluate detection performance. |
| FAR and MAR | 【公式】`FAR = FP/(FP+TN)`, `MAR = FN/(FN+TP)`. FAR means false alarm during normal operation; MAR means missed fault during faulty operation. |
| Sensitivity and specificity | 【公式】`Sensitivity = TP/(TP+FN) = 1-MAR`; `Specificity = TN/(TN+FP) = 1-FAR`. |
| FAR/MAR trade-off | 【核心】Lower threshold reduces missed alarms but increases false alarms; higher threshold reduces false alarms but increases missed alarms. |
| Accuracy limitation | 【核心】Accuracy can be misleading when faults are rare. A detector that always says "normal" may have high accuracy but no fault detection value. |

## Bayes Decision Theory

| Knowledge point | Marked explanation |
|---|---|
| Bayes theorem | 【公式】`P(F|x)=f(x|F)P(F)/f(x)`. Posterior equals likelihood times prior divided by evidence. |
| MAP rule | 【公式】Decide fault if `P(F|x)>P(N|x)`, or equivalently `f(x|F)P(F)>f(x|N)P(N)`. |
| Likelihood ratio test | 【公式】`LR(x)=f(x|F)/f(x|N)`. Decide fault if `LR(x)` is above a threshold. |
| Gaussian assumption | 【公式】`x|N ~ N(mu_N,sigma_N^2)`, `x|F ~ N(mu_F,sigma_F^2)`. The threshold separates the two PDFs; FAR and MAR are tail areas. |
| Bayes risk | 【核心】If false alarm and missed alarm have different costs, choose the action with minimum expected loss, not simply maximum posterior probability. |
| Loss matrix | 【公式】Conditional risk: `R(alpha_i|x)=sum_j lambda_ij P(omega_j|x)`. |

【口述】Bayes 方法的核心是：不是只看当前信号大不大，而是看这个信号在 normal 和 fault 两种状态下分别有多可能，再结合先验概率和错误代价做决定。

## PDF and Parameter Estimation

| Knowledge point | Marked explanation |
|---|---|
| Parametric estimation | 【核心】Assume a distribution family, such as Gaussian, then estimate parameters. Efficient but wrong assumptions cause bias. |
| MLE | 【公式】`theta_MLE = argmax_theta P(D|theta)`. It trusts data only. For Gaussian, the mean and variance estimates come from samples. |
| MAP estimation | 【公式】`theta_MAP = argmax_theta P(D|theta)P(theta)`. It combines data and prior. More stable for small data; approaches MLE when data is large. |
| Recursive mean | 【公式】`mu_hat_n = mu_hat_{n-1} + (1/n)(x_n-mu_hat_{n-1})`. This introduces the innovation idea used later by observers and Kalman filtering. |
| Non-parametric density | 【核心】Does not assume Gaussian. Useful for multimodal normal data but suffers in high dimension. |
| General density estimate | 【公式】`p_hat(x)=k/(N V)`, where `k` is sample count in a local region, `N` is total samples, and `V` is local volume. |
| Histogram | 【核心】Fixed bins; simple, but depends heavily on bin width and grid. |
| Parzen window | 【公式】`p_hat(x)=1/N sum_i 1/h^d K((x-x_i)/h)`. Fixed bandwidth `h`; smooth PDF estimate. |
| kNN density | 【核心】Fixed neighbor count `k`, adaptive volume `V_k(x)`. Dense areas use small volume; sparse areas use large volume. |

## LDA, QDA, and Logistic Regression

| Method | Marked explanation |
|---|---|
| LDA | 【核心】Generative Gaussian classifier with shared covariance. Boundary is linear. Suitable when classes have similar covariance. |
| LDA discriminant | 【公式】`delta_k(x)=x^T Sigma^{-1} mu_k - 1/2 mu_k^T Sigma^{-1} mu_k + log pi_k`. |
| QDA | 【核心】Generative Gaussian classifier with class-specific covariance. Boundary is quadratic. More flexible but needs more data. |
| QDA discriminant | 【公式】`delta_k(x)= -1/2 log|Sigma_k| - 1/2(x-mu_k)^T Sigma_k^{-1}(x-mu_k) + log pi_k`. |
| Logistic regression | 【核心】Directly models `P(y=1|x)`. It is discriminative, interpretable, fast online, and outputs a fault probability. |
| Sigmoid | 【公式】`h_theta(x)=1/(1+exp(-theta^T x))`. |
| Cross entropy | 【核心】Training minimizes classification probability loss rather than fitting full class PDFs. |

## Topic 1 Key Emphasis

- 【必讲】FAR/MAR definitions and their trade-off.
- 【必讲】Bayes theorem, MAP decision, likelihood ratio, and risk-based decision.
- 【必讲】MLE vs MAP: MLE uses only data; MAP uses data plus prior.
- 【必讲】Parametric vs non-parametric PDF estimation.
- 【必讲】LDA vs QDA vs logistic regression: assumptions, decision boundary, and use case.
- 【可略讲】Detailed derivation of Parzen convergence and full Bayes risk proof.

## Likely Examiner Q&A

| Question | Short answer |
|---|---|
| Why is accuracy not enough for fault detection? | Because faults are rare. A detector can get high accuracy by always predicting normal but still miss all faults. FAR and MAR are more meaningful. |
| What is the difference between MLE and MAP? | MLE maximizes likelihood from data only. MAP maximizes likelihood times prior, so it is more stable with small samples. |
| Why use logistic regression if Bayes classifier is optimal? | Bayes classifier needs true distributions. Logistic regression directly estimates posterior probability from data and is easier to implement. |

## 30-Second Oral Template

Topic 1 is about statistical signal-based fault detection. We first extract time or frequency features from sensor data, then evaluate detectors using FAR, MAR, sensitivity and specificity. The decision theory is Bayes: compare posterior probabilities or likelihood ratios, and include risk if missed alarms and false alarms have different costs. To estimate PDFs, we can use parametric methods like Gaussian MLE/MAP or non-parametric methods like histogram, Parzen window and kNN. Finally, LDA, QDA and logistic regression turn these ideas into practical classifiers.

---

# Topic 2. Dimensionality Reduction Approaches

## Boundary and Objective

【边界】Topic 2 answers how to represent high-dimensional sensor data in a lower-dimensional space. It covers PCA and variants, FDA, MDS, and Isomap. Classification after projection is only mentioned as downstream use.

【口述】Topic 2 的目标是把高维、相关、含噪的传感器数据压缩成低维变量，让 normal 和 fault 更容易被监测、分离和可视化。

## Why Dimensionality Reduction Matters in FDD

| Knowledge point | Marked explanation |
|---|---|
| High-dimensional sensors | 【核心】Industrial systems have many correlated sensors; direct monitoring in original space is noisy and hard to visualize. |
| Latent variables | 【核心】Low-dimensional variables summarize the main operating behavior. |
| Redundancy removal | 【核心】Many sensors contain overlapping information. Dimensionality reduction removes repeated directions. |
| Fault-sensitive representation | 【核心】A fault may not make one signal large; it may change relationships among variables. |
| Visualization | 【核心】2D/3D embeddings help show normal and fault clusters. |

## PCA and PCA-Based FDD

| Knowledge point | Marked explanation |
|---|---|
| PCA objective | 【核心】Find directions with maximum variance in fault-free data. |
| Data matrix | 【公式】For centered data `X in R^{d x N}`, covariance is `S=(1/N)XX^T`. |
| Eigenvalue problem | 【公式】`S u_i = lambda_i u_i`, with `lambda_1 >= lambda_2 >= ...`. |
| Projection | 【公式】Training data: `Y=U_p^T X`; single sample: `y=U_p^T x`. |
| Reconstruction | 【公式】`x_hat=U_p U_p^T x`; residual `r=x-x_hat`. |
| SPE/Q statistic | 【公式】`SPE = Q = ||x-x_hat||^2`. Large SPE means the sample cannot be explained by the normal PCA subspace. |
| Hotelling T2 | 【核心】Measures abnormal variation inside the principal subspace. SPE monitors residual subspace; T2 monitors score subspace. |
| Principal subspace | 【核心】The first `p` components describe normal variation. |
| Residual subspace | 【核心】Remaining directions often reveal faults because faults break normal correlation. |
| Choosing `p` | 【核心】Use cumulative variance, scree plot, cross-validation, or monitoring performance. Too few components lose normal behavior; too many include noise or fault-insensitive directions. |
| Direct vs dual PCA | 【核心】Direct PCA uses `XX^T`; dual PCA uses `X^T X` and is useful when sensors are many but samples are fewer. |

【口述】PCA 的 FDD 思路是：用正常数据学习一个正常子空间。新样本如果能被这个子空间很好重构，就更像 normal；如果重构误差很大，就可能是 fault。

## PCA Variants

| Method | Marked explanation |
|---|---|
| Kernel PCA | 【核心】Handles nonlinear normal manifolds by doing PCA in feature space through kernels. |
| KPCA kernel | 【公式】`K(x_i,x_j)=Phi(x_i)^T Phi(x_j)`. This is the correct kernel form. |
| KPCA use case | 【核心】Useful when normal data lie on a nonlinear manifold. Sensitive to kernel choice and parameters. |
| Supervised PCA | 【核心】PCA only maximizes variance; SPCA keeps directions related to labels or response variables. |
| HSIC-SPCA | 【核心】Uses dependence between projected data and labels to find fault-sensitive directions. It is advanced and can be mentioned briefly. |
| Metric learning | 【核心】Learns a distance metric so same-class samples are close and different-class samples are far. |
| Sufficient dimension reduction | 【核心】Seeks a projection that preserves `p(y|x)`, meaning the low-dimensional data keep information relevant to the output. |

## FDA, MDS, and Isomap

| Method | Marked explanation |
|---|---|
| FDA | 【核心】Supervised dimensionality reduction. It maximizes between-class scatter and minimizes within-class scatter. |
| FDA objective | 【公式】`J(w)=w^T S_B w / (w^T S_W w)`. `S_B` means class separation; `S_W` means within-class spread. |
| FDA use case | 【核心】Useful for labelled fault isolation when we want normal and fault classes separated after projection. |
| PCA vs FDA | 【核心】PCA is unsupervised and preserves variance; FDA is supervised and preserves class separability. |
| MDS | 【核心】Preserves pairwise distance or similarity structure in a low-dimensional embedding. |
| Classical MDS | 【核心】Use squared distance matrix and double centering, then eigen-decomposition. With Euclidean distances, classical MDS is closely related to PCA. |
| Isomap | 【核心】Nonlinear manifold method. It replaces Euclidean distances with graph-based geodesic distances before MDS. |
| Isomap steps | 【核心】Build kNN graph, compute shortest paths, treat them as geodesic distances, then run MDS. |
| Isomap limitation | 【核心】Sensitive to neighbor number, noise, and disconnected graphs. |

## Topic 2 Key Emphasis

- 【必讲】PCA: maximum variance, principal subspace, residual subspace.
- 【必讲】PCA FDD: scores, reconstruction, SPE/Q, T2.
- 【必讲】KPCA: kernel trick for nonlinear normal subspaces.
- 【必讲】FDA: supervised class separation through scatter matrices.
- 【必讲】MDS: preserve distance structure.
- 【必讲】Isomap: kNN graph plus shortest path approximates geodesic distance.
- 【可略讲】HSIC-SPCA, metric learning, SDR derivation.

## Likely Examiner Q&A

| Question | Short answer |
|---|---|
| Why can PCA detect faults without labels? | It learns the normal correlation structure from fault-free data. Faults often break this structure and create large residuals. |
| What is the difference between SPE and T2? | SPE measures reconstruction error in residual subspace; T2 measures unusual score variation inside the principal subspace. |
| PCA vs FDA? | PCA is unsupervised and maximizes variance. FDA is supervised and maximizes class separation. |
| Why does Isomap use shortest paths? | Shortest paths on a local graph approximate geodesic distances on a nonlinear manifold. |

## 30-Second Oral Template

Topic 2 is about transforming high-dimensional sensor data into a low-dimensional representation. PCA learns the main normal subspace from fault-free data; online faults can be detected by SPE or Hotelling T2. KPCA extends PCA to nonlinear manifolds using kernels. FDA is supervised and tries to separate normal and fault classes. MDS preserves pairwise distances, while Isomap uses kNN graph shortest paths to approximate geodesic distances before embedding.

---

# Topic 3. Support Vector Machines and Perceptron

## Boundary and Objective

【边界】Topic 3 covers maximum-margin classification: hard margin SVM, soft margin SVM, dual form, kernel trick, and perceptron. It does not cover Bayes/LDA/logistic regression except as contrast.

【口述】Topic 3 的目标是用一个间隔最大的分类边界，把 normal 和 fault 或不同 fault classes 分开。

## SVM Core Concepts

| Knowledge point | Marked explanation |
|---|---|
| Hyperplane | 【公式】`beta^T x + beta_0 = 0`. This is the decision boundary. |
| Decision function | 【公式】`f(x)=beta^T x + beta_0`; sign decides class. |
| Support vectors | 【核心】Training samples closest to the boundary. They determine the SVM boundary. |
| Margin | 【核心】Distance from boundary to nearest samples. SVM maximizes this margin for better generalization. |
| SVM in FDD | 【核心】Features from sensor signals are classified as normal/fault or fault type. |
| SVM vs probability classifiers | 【核心】LDA/QDA/logistic regression are probability-based; SVM is optimization and margin-based. |

## Hard Margin SVM

| Knowledge point | Marked explanation |
|---|---|
| Use case | 【核心】Linearly separable data with little noise. |
| Optimization | 【公式】`min_{beta,beta_0} 1/2 ||beta||^2`, subject to `y_i(beta^T x_i+beta_0)>=1`. |
| Meaning | 【核心】Maximizing margin is equivalent to minimizing `||beta||^2`. |
| Limitation | 【核心】Very sensitive to outliers and overlapping fault/normal data. |
| Lagrangian and KKT | 【核心】Used to derive dual form and identify support vectors. |
| Hard dual | 【公式】Maximize `sum_i alpha_i - 1/2 sum_i sum_j alpha_i alpha_j y_i y_j x_i^T x_j`, subject to `alpha_i>=0`, `sum_i alpha_i y_i=0`. |
| Weight vector | 【公式】`beta=sum_i alpha_i y_i x_i`. Only samples with `alpha_i>0` contribute. |

## Soft Margin SVM

| Knowledge point | Marked explanation |
|---|---|
| Why soft margin | 【核心】Real fault data have noise, overlap, and outliers; strict separation may be impossible. |
| Slack variable | 【公式】`xi_i>=0`; constraint becomes `y_i(beta^T x_i+beta_0)>=1-xi_i`. |
| Objective | 【公式】`min 1/2||beta||^2 + C sum_i xi_i`. |
| `C` parameter | 【核心】Controls trade-off between wide margin and training errors. Small `C` tolerates errors; large `C` penalizes errors strongly. |
| Slack interpretation | 【核心】`xi_i=0` correct outside margin; `0<xi_i<1` correct but inside margin; `xi_i=1` on boundary; `xi_i>1` misclassified. |
| Soft dual | 【公式】Same dual objective as hard margin, but `0<=alpha_i<=C`. |

## Kernel Trick and Perceptron

| Knowledge point | Marked explanation |
|---|---|
| Kernel trick | 【核心】Replace dot product by `K(x_i,x_j)=Phi(x_i)^T Phi(x_j)` without explicitly computing `Phi(x)`. |
| Linear kernel | 【核心】Good when boundary is almost linear. |
| Polynomial kernel | 【核心】Captures polynomial interactions between features. |
| RBF kernel | 【核心】Common for nonlinear fault boundaries. `gamma` large gives complex boundary; `gamma` small gives smoother boundary. |
| Sigmoid kernel | 【核心】Related to neural network activation, less common in practice. |
| Perceptron | 【核心】Simple linear classifier. If a sample is misclassified, update `w <- w + eta y_i x_i`. |
| Perceptron vs SVM | 【核心】Perceptron only finds a separating hyperplane; SVM finds the maximum-margin hyperplane. |

## Topic 3 Key Emphasis

- 【必讲】Hyperplane, margin, support vectors.
- 【必讲】Hard margin constraints and why they maximize margin.
- 【必讲】Soft margin slack variables and `C` trade-off.
- 【必讲】Dual form enables kernel trick.
- 【必讲】Kernel function must be `K(x_i,x_j)=Phi(x_i)^T Phi(x_j)`.
- 【必讲】Perceptron is a baseline linear classifier, not maximum-margin.
- 【可略讲】Full KKT derivation unless asked.

## Likely Examiner Q&A

| Question | Short answer |
|---|---|
| Why does SVM use support vectors only? | Only points on or inside the margin have nonzero dual variables, so they determine the boundary. |
| What happens if `C` is too large? | The model strongly penalizes training errors, which can reduce margin and overfit noise. |
| Why use kernels? | Kernels allow nonlinear separation by computing inner products in a high-dimensional feature space implicitly. |

## 30-Second Oral Template

Topic 3 studies SVM and perceptron for fault classification. SVM finds a hyperplane with maximum margin, determined by support vectors. Hard margin assumes perfectly separable data, while soft margin introduces slack variables for noise and overlap. The parameter C balances margin width and training error. In dual form, only dot products appear, so we can replace them with kernels such as RBF to handle nonlinear fault boundaries. Perceptron is a simpler linear classifier but does not maximize margin.

---

# Topic 4. Model-Based Fault Detection

## Boundary and Objective

【边界】Topic 4 covers model-based diagnosis: residual generation, parity methods, observers, Kalman filter, ESO/LESO, UIO, NDO, hypothesis testing, CUSUM, and ARL. It stops at diagnosis; it does not redesign the controller.

【口述】Topic 4 的目标是用系统模型预测系统应该怎样表现，再把预测和实际测量的差作为 residual，用 residual 判断故障。

## Residual Generation

| Knowledge point | Marked explanation |
|---|---|
| Model-based FDD | 【核心】Use a model such as `x_dot=Ax+Bu+Ed`, `y=Cx` to predict output and compare with measurement. |
| Residual | 【公式】`r=y-y_hat`. Normal operation gives small residual; fault gives large or structured residual. |
| Good residual | 【核心】Sensitive to faults, robust to noise/disturbance, useful for isolation, and easy to threshold. |
| Threshold design | 【核心】Low threshold gives false alarms; high threshold gives missed alarms or detection delay. |
| Fault sensitivity | 【核心】Residual should react strongly to the fault of interest. |
| Disturbance robustness | 【核心】Residual should not react strongly to unknown inputs or noise. |

## Parity and Observer-Based Methods

| Method | Marked explanation |
|---|---|
| Parity method | 【核心】Construct an input-output relation that nominal data should satisfy. Fault breaks this relation and creates residual. |
| Time-domain parity | 【核心】Uses sampled input-output data and model equations over a window. |
| Frequency-domain parity | 【核心】Uses transfer functions or frequency-domain relations. |
| Observer residual | 【公式】Observer: `xhat_dot=A xhat + Bu + L(y-C xhat)`; residual `r=y-C xhat`. |
| Error dynamics | 【公式】`e_dot=(A-LC)e` without fault/disturbance. Choose `L` so error converges. |
| Observability | 【核心】If the state can be reconstructed from outputs, observer design is possible. |
| Detectability | 【核心】Even if not fully observable, observer can work if unobservable modes are stable. |
| Kalman filter | 【核心】Observer for noisy systems. It balances model prediction and measurement using noise covariance. |
| ESO/LESO | 【核心】Extended state observer treats unknown disturbance/fault as an extra state. Good for lumped disturbance estimation. |
| NDO | 【核心】Nonlinear disturbance observer estimates disturbances/faults in nonlinear systems. Depends on model quality and gain choice. |

## Unknown Input Observer

| Knowledge point | Marked explanation |
|---|---|
| UIO goal | 【核心】Estimate state while decoupling unknown input `d`, so the observer error is not polluted by `d`. |
| System | 【公式】`x_dot=Ax+Bu+Ed`, `y=Cx`. |
| Decoupling condition | 【公式】`(I-HC)E=0`. |
| Rank condition | 【公式】A UIO exists if `rank(E)=rank(CE)` plus suitable detectability of the transformed system. |
| Interpretation | 【口述】UIO 不是简单估计未知输入，而是设计结构让未知输入对状态估计误差不起作用。 |
| Limitation | 【核心】If the unknown input does not show through the measured output enough, rank condition fails. |

## Statistical Change Detection

| Knowledge point | Marked explanation |
|---|---|
| Hypothesis testing | 【核心】`H0` means normal; `H1` means fault. A statistic compresses data for decision. |
| Log-likelihood ratio | 【公式】`s(x)=log(f_theta1(x)/f_theta0(x))`. Positive supports fault; negative supports normal. |
| CUSUM purpose | 【核心】Detect a change as quickly as possible while controlling false alarms. |
| Recursive CUSUM | 【公式】`g(k)=max(0,g(k-1)+s(x_k))`. Alarm if `g(k)>h`. |
| Why max with zero | 【核心】Normal negative drift is reset; after fault, positive drift accumulates. |
| Change-time estimate | 【核心】The length of the current positive run estimates when the change started. |
| Gaussian mean shift | 【核心】If mean changes from `mu0` to `mu1`, the log-likelihood ratio becomes a linear function of `x_k`. |
| ARL | 【核心】Average run length. Under `H0`, large ARL means rare false alarms; under `H1`, small ARL means fast detection. |

## Topic 4 Key Emphasis

- 【必讲】Residual `r=y-y_hat` and residual design goals.
- 【必讲】Parity method vs observer method.
- 【必讲】Luenberger observer/Kalman filter/ESO/UIO roles.
- 【必讲】UIO decoupling and rank condition.
- 【必讲】CUSUM recursion and why it detects changes.
- 【必讲】ARL trade-off between false alarm and detection delay.
- 【可略讲】Full algebraic construction of parity matrices and NDO proof details.

## Likely Examiner Q&A

| Question | Short answer |
|---|---|
| Why use model-based FDD instead of data-driven FDD? | If a reliable physical model exists, residuals can detect faults early and explain which part of the system deviates. |
| What makes a residual good? | It should be sensitive to faults but robust to noise, disturbances, and model uncertainty. |
| Why is UIO useful? | It can estimate the state while rejecting unknown inputs, so residuals can focus on faults instead of disturbances. |
| Why is CUSUM better than a single threshold? | It accumulates weak evidence over time, so it can detect small persistent changes earlier. |

## 30-Second Oral Template

Topic 4 is model-based fault detection. We use a model to predict the system output and compare it with measurement; the difference is the residual. Residuals can be generated by parity equations or observers such as Luenberger observers, Kalman filters, ESO/LESO, NDO, or UIO. UIO is important because it can decouple unknown inputs under a rank condition. After generating residuals, statistical tests such as likelihood ratio and CUSUM decide whether a change occurred, while ARL measures false alarm and detection delay performance.

---

# Topic 5. Fault Tolerant Control Strategy

## Boundary and Objective

【边界】Topic 5 starts after diagnosis. It covers robust/adaptive control, passive/active FTC, reconfigurable control, model matching, transient elimination, virtual actuator, virtual sensor, and bumpless transfer. Safe MPC and CPS attack/anomaly response are Topic 6.

【口述】Topic 5 的目标是：故障发生并被诊断后，怎样重新配置控制器或信号路径，让系统仍然稳定并保持可接受性能。

## FTC Architecture and Fault Types

| Knowledge point | Marked explanation |
|---|---|
| FTC objective | 【核心】Maintain stability and acceptable performance when faults occur. |
| Control reconfiguration | 【核心】Use diagnosis results to switch, tune, or redesign the controller. |
| Actuator fault | 【核心】Input channel changes, often modeled as `B -> B_f` or `u+f_a`. |
| Plant/component fault | 【核心】System dynamics change, often modeled as `A -> A_f`. |
| Sensor fault | 【核心】Measurement path changes, often modeled as `C -> C_f` or sensor bias. |
| Redundancy | 【核心】FTC needs remaining actuator authority or sensor information. Algorithms cannot create missing physical channels. |
| Hardware redundancy | 【核心】Backup actuators/sensors. |
| Analytical redundancy | 【核心】Use models and other measurements to reconstruct missing information. |

## Passive FTC, Active FTC, Robust and Adaptive Control

| Method | Marked explanation |
|---|---|
| Passive FTC | 【核心】Fixed robust controller handles a predefined fault/uncertainty range without online diagnosis-based redesign. |
| Passive advantage | 【核心】Simple and low real-time risk. |
| Passive limitation | 【核心】Conservative and cannot handle unexpected severe faults well. |
| Active FTC | 【核心】Uses FDD results to switch or recompute controller. |
| Active advantage | 【核心】Can handle specific and severe faults better. |
| Active limitation | 【核心】Depends on diagnosis accuracy; wrong diagnosis can cause wrong reconfiguration. |
| Robust control | 【核心】Fixed controller designed for uncertainty range. Nominal performance may be conservative. |
| Adaptive control | 【核心】Controller parameters update as plant parameters change. Better for gradual changes; difficult for abrupt severe faults. |

## Reconfigurable Control and Model Matching

| Knowledge point | Marked explanation |
|---|---|
| Reconfigurable control system | 【核心】A supervisor selects or computes controllers based on fault diagnosis. |
| Pre-computed controllers | 【核心】Gain scheduling, multiple model, LPV, controller bank. |
| Online synthesized controllers | 【核心】Adaptive control, eigenstructure assignment, MPC-based redesign. |
| Nominal model | 【公式】`x_dot=Ax+Bu`, `u=-Kx`, nominal closed-loop `A_des=A-BK`. |
| Faulty model | 【公式】`x_dot=A_f x+B_f u`. |
| Model matching goal | 【公式】`A_f-B_f K_f = A-BK`. |
| Meaning | 【口述】故障后选择新的 `K_f`，让 faulty closed-loop 尽量像 healthy closed-loop。 |
| Exact model matching | 【核心】Requires strict solvability: the required correction must lie in the image/range of `B_f`. |
| Approximate model matching | 【公式】`min_{K_f} ||(A_f-B_fK_f)-(A-BK)||_F`. Use least squares or pseudo-inverse when exact matching is impossible. |

## Sensor Reconfiguration, Virtual Actuator/Sensor

| Knowledge point | Marked explanation |
|---|---|
| Sensor reconfiguration goal | 【公式】For output feedback, require `K_f C_f = K C`. |
| Exact sensor condition | 【公式】`Ker(C_f) subseteq Ker(C)`. If faulty sensors lose information needed by the nominal controller, exact reconfiguration is impossible. |
| Virtual actuator | 【核心】An interface between nominal controller and faulty plant. It transforms nominal control commands into commands the faulty plant can execute. |
| Virtual sensor | 【核心】An interface between faulty measurements and nominal controller. It reconstructs the measurement the controller expects. |
| Oral distinction | 【口述】Virtual actuator solves "command cannot be applied correctly"; virtual sensor solves "measurement cannot be read correctly". |

## Transient Elimination and Bumpless Transfer

| Knowledge point | Marked explanation |
|---|---|
| Switching transient | 【核心】Controller switching can create sudden input jumps because controllers have different gains or internal states. |
| Input jump | 【公式】A typical jump can be written as `Delta u=(K_f C_f-KC)x` in output-feedback form. |
| Why dangerous | 【核心】Sudden jumps can saturate or damage actuators and excite unstable transients. |
| Bumpless transfer | 【核心】Make inactive controllers track the active controller output before switching, so switching causes little or no input jump. |
| Controller bank | 【核心】Multiple controllers run in parallel; supervisor selects the active one. |
| Tracking feedback | 【核心】Use `u-u_i` feedback so inactive controller `i` tracks the active output. |
| Stability of tracking | 【核心】Choose tracking gain so the inactive controller's tracking dynamics are stable. |
| Tuning trade-off | 【核心】Fast tracking reduces mismatch but amplifies noise; slow tracking is smoother but may not be ready at switching. |
| Soft switching | 【核心】Blend controller outputs using weights instead of hard switching. |

## Topic 5 Key Emphasis

- 【必讲】Passive vs active FTC.
- 【必讲】Robust vs adaptive control.
- 【必讲】Fault locations: actuator, plant, sensor.
- 【必讲】Redundancy is the physical limit of FTC.
- 【必讲】Model matching equation and exact vs approximate matching.
- 【必讲】Sensor reconfiguration condition.
- 【必讲】Virtual actuator vs virtual sensor.
- 【必讲】Bumpless transfer and why transient elimination matters.
- 【可略讲】Full dynamic controller derivation unless asked.

## Likely Examiner Q&A

| Question | Short answer |
|---|---|
| Why can FTC fail even with a good algorithm? | If the fault removes controllability or observability, there is not enough physical redundancy left. |
| Passive vs active FTC? | Passive uses a fixed robust controller. Active uses FDD results to switch or redesign the controller. |
| What is model matching? | Choose a faulty-controller gain so the faulty closed-loop matrix matches or approximates the nominal closed-loop matrix. |
| Why is bumpless transfer needed? | Direct controller switching can create large input jumps; bumpless transfer makes inactive controllers track the active output before switching. |

## 30-Second Oral Template

Topic 5 is about what to do after a fault has been detected. Passive FTC uses one robust controller for a predefined fault range, while active FTC uses diagnosis results to reconfigure the controller. The key method is model matching: choose a new gain so the faulty closed-loop behaves like the nominal one. Exact matching may be impossible if actuator or sensor faults remove necessary control or measurement directions, so approximate matching and redundancy are important. Virtual actuators and sensors interface the nominal controller with a faulty plant, and bumpless transfer prevents large transients during controller switching.

---

# Topic 6. Safe/Secure Control for Cyber-Physical Systems

## Boundary and Objective

【边界】Topic 6 covers CPS faults, attacks, anomalies, resilient control, safe control synthesis, constrained MPC, terminal set/cost, offset-free MPC, LESO/UIO + MPC, and VAE anomaly detection. Topic 6 is broader than detection: it asks how the system remains safe under constraints and abnormal conditions.

【口述】Topic 6 的目标是：在 CPS 中，即使存在故障、攻击、异常、扰动和硬约束，也要让系统继续满足安全约束并保持可接受控制性能。

## CPS Fault, Attack, and Anomaly

| Knowledge point | Marked explanation |
|---|---|
| CPS | 【核心】A system where computation, communication, and physical dynamics interact. |
| Fault | 【核心】Component degradation or abnormal component behavior. |
| Attack | 【核心】Malicious manipulation of sensors, actuators, communication, or computation. |
| Anomaly | 【核心】Observed abnormal behavior; cause can be fault, attack, disturbance, or noise. |
| Topic 4 vs Topic 6 | 【边界】Topic 4 focuses on detecting faults with models. Topic 6 focuses on safe and resilient response under CPS constraints. |
| Detection routes | 【核心】Model-based residual detection, data-driven anomaly detection, and sequential change detection can all support CPS security. |

## Safe Control and MPC

| Knowledge point | Marked explanation |
|---|---|
| Why LQR is not enough | 【核心】LQR does not explicitly handle hard state and input constraints. Real CPS has limits on voltage, current, valve opening, torque, pressure, temperature, position, speed. |
| Safe control constraints | 【公式】`x_k in X`, `u_k in U`. |
| Finite-horizon constrained problem | 【公式】Minimize `sum_{k=0}^{N-1}(x_k^TQx_k+u_k^TRu_k)+x_N^TPx_N`, subject to dynamics and constraints. |
| MPC idea | 【核心】At each step: measure, optimize over a horizon, apply only the first input, then repeat. This gives feedback. |
| Open-loop limitation | 【核心】A one-shot input sequence cannot react to disturbances or faults after the plan is computed. |
| Prediction stacking | 【公式】`X = Abar x_0 + Bbar U`. |
| Standard QP | 【公式】`min_U 1/2 U^T H U + f(x_0)^T U`, subject to `G U <= W(x_0)`. |
| State elimination | 【核心】Eliminates predicted states and optimizes over input sequence only; efficient in variables but can produce dense matrices. |

## MPC Stability and Tracking

| Knowledge point | Marked explanation |
|---|---|
| Finite horizon problem | 【核心】Short horizon can be short-sighted and may not guarantee stability. |
| Terminal equality | 【公式】`x_{t+N|t}=0`. Strong stability condition but often infeasible or aggressive. |
| Terminal set | 【公式】`x_{t+N|t} in X_f`. It means the predicted terminal state enters a safe invariant region. |
| Terminal cost | 【公式】`x_N^T P x_N`. It approximates infinite-horizon cost beyond the prediction horizon. |
| Positive invariance | 【核心】If state enters terminal set, local controller keeps it inside constraints. |
| Recursive feasibility | 【核心】If the MPC problem is feasible now, it remains feasible at the next step under the designed terminal ingredients. |
| Reference tracking | 【核心】For nonzero target, compute steady-state pair `(x_s,u_s)` and control shifted variables `x-x_s`, `u-u_s`. |
| Target selector | 【核心】Computes feasible steady state that matches output target and constraints, especially in overactuated systems. |
| Offset-free MPC | 【核心】Estimate persistent disturbances and modify target selector so steady-state error disappears. |
| Disturbance augmented model | 【公式】`x_{k+1}=Ax_k+Bu_k+B_d d_k`, `d_{k+1}=d_k`. |

## Resilient Control with LESO/UIO and MPC

| Knowledge point | Marked explanation |
|---|---|
| AFTC with MPC | 【核心】Diagnosis/estimation feeds MPC so the controller compensates for faults while respecting constraints. |
| Actuator fault | 【核心】Physical input channel changes. The target selector or controller must compensate baseline input. |
| Sensor fault | 【核心】Measurement lies. The controller should use cleaned state estimates rather than corrupted measurements. |
| LESO role | 【核心】Estimate state and lumped fault/disturbance dynamically. Useful for adaptive compensation. |
| UIO role | 【核心】Decouple unknown inputs from state estimation and reconstruct actuator fault algebraically if conditions hold. |
| Real-time AFTC loop | 【核心】Measure -> estimate state/fault -> select target -> solve MPC -> apply corrected input. |
| Resilient objective | 【核心】Continue tracking and satisfy constraints despite faults or attacks. |

## Data-Driven Anomaly Detection with VAE

| Knowledge point | Marked explanation |
|---|---|
| VAE purpose | 【核心】Learn healthy data manifold and detect abnormal samples by reconstruction or likelihood/latent deviation. |
| Encoder | 【核心】Maps input `x` to latent distribution parameters `mu` and `log sigma^2`. |
| Reparameterization | 【公式】`z=mu+sigma * epsilon`, `epsilon~N(0,I)`. This allows gradient training. |
| Decoder | 【核心】Reconstructs `x_hat` from latent `z`. |
| ELBO | 【公式】`ELBO = E_q[log p_theta(x|z)] - KL(q_phi(z|x)||p(z))`. |
| Reconstruction term | 【核心】Measures how well healthy data can be reconstructed. |
| KL term | 【核心】Regularizes latent distribution toward normal prior `N(0,I)`. |
| Anomaly score | 【核心】High reconstruction error, low likelihood, or unusual latent position indicates anomaly. |
| Limitation | 【核心】VAE is useful for unknown anomalies but may smooth signals, collapse posterior, or fail if training data are not clean. |

## Topic 6 Key Emphasis

- 【必讲】Fault, attack, anomaly differences.
- 【必讲】Safe control means explicit state/input constraints.
- 【必讲】MPC: measure, optimize, apply first input, repeat.
- 【必讲】QP formulation and prediction stacking.
- 【必讲】Terminal set/cost for recursive feasibility and stability.
- 【必讲】Offset-free MPC uses disturbance estimation to remove steady-state error.
- 【必讲】Actuator fault needs compensation; sensor fault needs isolation/clean estimation.
- 【必讲】LESO/UIO + MPC provides resilient AFTC structure.
- 【可略讲】Full ELBO derivation and advanced VAE variants unless asked.

## Likely Examiner Q&A

| Question | Short answer |
|---|---|
| Why is LQR insufficient for safe CPS control? | LQR does not explicitly handle hard state and input constraints, while CPS safety requires constraints to be respected. |
| What makes MPC suitable for safe control? | MPC solves a constrained optimization problem repeatedly with updated measurements, so it handles constraints and disturbances better than open-loop control. |
| Why use terminal set and terminal cost? | They prevent finite-horizon MPC from being short-sighted and help guarantee recursive feasibility and stability. |
| What is the difference between actuator and sensor fault in AFTC? | Actuator fault changes the physical input channel and needs compensation; sensor fault corrupts measurement and needs isolation or cleaned state estimation. |
| What does VAE contribute? | It learns the normal data manifold and can flag unknown anomalies by reconstruction or likelihood scores. |

## 30-Second Oral Template

Topic 6 is about safe and secure control for CPS. The key point is that detection alone is not enough; after a fault, attack, or anomaly, the controller must still satisfy state and input constraints. MPC is the main safe control tool because it solves a constrained optimization problem at every step and applies only the first input. Terminal set and terminal cost help guarantee stability and recursive feasibility. Offset-free MPC removes steady-state error by estimating disturbances. In resilient AFTC, LESO or UIO estimates faults and cleaned states, then MPC compensates actuator faults or isolates sensor faults while respecting constraints. VAE can be used as a data-driven anomaly detector for unknown abnormal behavior.

---

# Six Topic Boundaries to Avoid Repetition

| Boundary | What to say |
|---|---|
| Topic 1 vs Topic 2 | Topic 1 uses statistical/probability classifiers. Topic 2 builds lower-dimensional features and subspaces. |
| Topic 1 vs Topic 3 | Topic 1 is Bayes/LDA/QDA/logistic probability route. Topic 3 is maximum-margin SVM route. |
| Topic 1 vs Topic 4 | Topic 1 is data/signal distribution based. Topic 4 is system model and residual based. |
| Topic 2 vs Topic 3 | Topic 2 answers how to represent data. Topic 3 answers how to classify data with a maximum-margin boundary. |
| Topic 4 vs Topic 5 | Topic 4 diagnoses the fault. Topic 5 reconfigures control after diagnosis. |
| Topic 5 vs Topic 6 | Topic 5 is FTC strategy and reconfiguration. Topic 6 is safe/resilient CPS control under constraints, faults, attacks, and anomalies. |

# Final Minimal Must-Remember List

## Topic 1

- Bayes: `posterior proportional likelihood x prior`.
- FAR is false alarm during normal; MAR is missed alarm during fault.
- MLE uses data only; MAP uses data plus prior.
- LDA is shared covariance and linear boundary; QDA is class-specific covariance and quadratic boundary.
- Logistic regression directly estimates fault probability.

## Topic 2

- PCA learns normal subspace; SPE/Q monitors residual; T2 monitors score space.
- KPCA uses `K(x,y)=Phi(x)^T Phi(y)` for nonlinear data.
- FDA maximizes class separation, not variance.
- MDS preserves distances.
- Isomap uses kNN graph shortest paths as geodesic distances.

## Topic 3

- SVM maximizes margin.
- Hard margin assumes separable data.
- Soft margin uses slack variables for noisy/overlapping data.
- `C` balances margin and training error.
- Kernel trick enables nonlinear boundary.
- Perceptron is a simpler linear classifier without maximum margin.

## Topic 4

- Residual is measured output minus predicted output.
- Residual should be fault-sensitive and disturbance-robust.
- Observers generate estimates; UIO decouples unknown inputs.
- CUSUM accumulates log-likelihood evidence over time.
- ARL measures false alarm interval and detection delay.

## Topic 5

- Passive FTC uses fixed robust control; active FTC uses diagnosis-based reconfiguration.
- FTC success depends on remaining controllability/observability.
- Model matching aims for `A_f-B_fK_f=A-BK`.
- Sensor reconfiguration uses `K_f C_f=KC`.
- Virtual actuator adapts commands; virtual sensor adapts measurements.
- Bumpless transfer avoids input jumps during controller switching.

## Topic 6

- CPS safety requires explicit `x in X`, `u in U` constraints.
- MPC repeatedly solves constrained QP and applies the first input.
- Terminal set/cost support recursive feasibility and stability.
- Offset-free MPC estimates persistent disturbances and corrects steady-state target.
- Actuator fault needs compensation; sensor fault needs cleaned estimation.
- LESO/UIO + MPC gives resilient AFTC.
- VAE detects anomalies by healthy-manifold reconstruction/likelihood.

# Coverage Checklist

- [x] Official curriculum contains 6 topics, and this file contains exactly 6 topics.
- [x] Topic 1 includes Bayes theorem, PDF estimation, LDA, and logistic regression.
- [x] Topic 2 includes PCA and variants, FDA, MDS, and Isomap.
- [x] Topic 3 includes hard margin SVM, soft margin SVM, kernel trick, and perceptron.
- [x] Topic 4 includes residual generation, parity method, observer methods, UIO/KF/ESO/NDO, CUSUM, and ARL.
- [x] Topic 5 includes robust/adaptive control, reconfigurable control, model matching, transient elimination, virtual actuator, and virtual sensor.
- [x] Topic 6 includes attack/anomaly detection, resilient control, safe control synthesis, MPC, offset-free MPC, LESO/UIO, and VAE anomaly detection.
- [x] Repeated concepts are assigned clear boundaries.
- [x] Key formulas are kept with consistent notation inside each topic.
- [x] Original source files are not modified.
