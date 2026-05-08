# Fault Tolerant Control Review: Full English Version by 6 Topics



| Topic | Official subitems | Section in this note | Boundary rule |
|---|---|---|---|
| 1. Introduction to Fault and Statistical Signal Processing | Bayes' theorem and PDF estimation; LDA; logistic regression | Topic 1 | Signal features, thresholds, FAR/MAR, MLE/MAP, Parzen, and kNN are supporting material for statistical signal processing. |
| 2. Dimensionality Reduction Approaches | PCA and variants; FDA; MDS and Isomap | Topic 2 | PCA variants, supervised PCA, kernel PCA, metric learning, and manifold methods belong here. Classifier design after projection belongs to Topic 1 or 3. |
| 3. Support Vector Machines and Perceptron | Hard-margin SVM; soft-margin SVM; kernel trick and perceptron | Topic 3 | Duality, KKT, slack variables, kernels, and perceptron are all part of the SVM/perceptron topic. |
| 4. Model based fault detection | Model-based residual generation; parity in time/frequency domain; observer-based methods (UIO, KF, ESO, NDO); CUSUM and ARL | Topic 4 | Observers are used here for residual generation and diagnosis, not for controller reconfiguration. |
| 5. Fault Tolerant Control Strategy | Robust/adaptive control; reconfigurable control systems; model matching, transient elimination, virtual actuator and sensor | Topic 5 | State feedback, LQR, and observers are included only as the control background needed for FTC strategies. |
| 6. Safe/Secure Control for CPS | Attack/anomaly detection; resilient control for CPS; safe control synthesis | Topic 6 | MPC, terminal sets, offset-free MPC, LESO/UIO plus MPC, and VAE anomaly detection are placed here. |

## Guide: Overall Course Logic

The whole course can be read as one engineering chain:

```text
sensor data / system model
-> fault detection and diagnosis
-> decision: normal, faulty, attacked, or uncertain
-> fault accommodation / controller reconfiguration
-> safe and resilient operation under constraints
```

Diagnosis answers three questions:

- Detection: has something abnormal happened?
- Isolation: where did it happen, such as sensor, actuator, plant component, or cyber channel?
- Identification: how large or what type is the fault?

Fault-tolerant control (FTC) then answers:

- Can the system remain stable?
- Can it still track the reference or keep acceptable performance?
- Can it stay inside hard safety constraints?
- Which redundancy or reconfiguration path is available?

### 0.1 Core Terms

Fault:

- An abnormal deviation in a component, sensor, actuator, plant parameter, or communication channel.
- A fault may not immediately stop the system, but it changes the system behavior.

Failure:

- A more severe event where the system or component cannot perform its required function.

Error:

- The difference between the actual state/output and the intended or nominal value.

Anomaly:

- Any observed behavior that is unusual relative to normal data.
- It may be caused by a fault, attack, disturbance, noise, or operating condition change.

Disturbance:

- External or internal input that affects the system but is not necessarily a fault.

Noise:

- Random measurement or process uncertainty.

Residual:

- A signal designed to be small during normal operation and large when faults or attacks occur.
- Typical form:

```math
r=y-\hat y
```

Threshold:

- A decision boundary for residuals or statistics.
- Low threshold: more sensitive but more false alarms.
- High threshold: fewer false alarms but larger missed alarm probability and delay.

### 0.2 Three Diagnosis Levels

Fault detection:

```text
normal or abnormal?
```

Fault isolation:

```text
which component, channel, or fault class?
```

Fault identification:

```text
what magnitude, shape, or time profile?
```

### 0.3 Two Parts of FTC

Fault diagnosis:

```text
data/model -> residual/statistic/classifier -> alarm and fault information
```

Fault accommodation or reconfiguration:

```text
fault information -> controller/sensor/actuator reconfiguration -> stable and safe operation
```

---

# Topic 1: Introduction to Fault and Statistical Signal Processing

## 1.1 Topic Boundary

Topic 1 covers data-driven and statistical fault detection before advanced dimensionality reduction and SVM.

Curriculum subitems:

- Bayes' theorem and PDF estimation: Sections 1.11-1.23.
- LDA: Section 1.25.
- Logistic regression: Section 1.27.

Supporting lecture material in this topic:

- signal feature extraction;
- supervised vs unsupervised diagnosis;
- detection vs isolation;
- performance metrics such as FAR, MAR, sensitivity, specificity;
- threshold design;
- likelihood ratio and risk-based decision;
- MLE and MAP;
- histogram, Parzen window, and kNN density estimation;
- QDA as the natural comparison to LDA.

## 1.2 Basic Problem in Signal-Based Fault Detection

Signal-based diagnosis starts from measured data:

```math
x=[x_1,x_2,\ldots,x_d]^T
```

Here `x` may contain raw sensor values, time-domain features, frequency-domain features, or features extracted from a sliding window.

The central task is to build a decision rule:

```math
g(x)\rightarrow \{\text{normal},\text{fault class 1},\ldots,\text{fault class K}\}
```

Main assumptions:

- normal data and fault data have different statistical patterns;
- the differences can be captured by features, PDFs, posterior probabilities, or decision boundaries;
- diagnosis quality depends strongly on feature quality and threshold choice.

Signal-based methods are useful when:

- a detailed physical model is unavailable;
- many measurements are available;
- labeled normal/fault data are available or normal data are sufficiently representative;
- fast online monitoring is required.

Limitations:

- training data may not cover future faults;
- sensor noise and operating condition changes can look like faults;
- purely data-driven methods may lack physical interpretability.

## 1.3 Signal Diagnosis Pipeline

Typical workflow:

```text
raw sensor signals
-> preprocessing
-> windowing
-> feature extraction
-> normalization
-> statistical model or classifier
-> alarm / fault class / confidence
```

Preprocessing includes detrending, filtering, synchronization, and removing invalid samples.

Windowing converts time series into samples:

```math
x_k=[y(k-L+1),\ldots,y(k)]
```

Feature extraction compresses the window into informative numbers such as mean, variance, RMS, spectral energy, and correlation.

Normalization is important because many classifiers and density estimators are scale-sensitive:

```math
\tilde x_j=\frac{x_j-\mu_j}{\sigma_j}
```

## 1.4 Time-Domain Features

Mean:

```math
\bar x=\frac1N\sum_{i=1}^N x_i
```

Use:

- detects bias, offset, drift, and slow degradation.

Variance:

```math
s^2=\frac1{N-1}\sum_{i=1}^N(x_i-\bar x)^2
```

Use:

- detects increased vibration, instability, and noise amplification.

Root mean square:

```math
x_{rms}=\sqrt{\frac1N\sum_{i=1}^N x_i^2}
```

Use:

- common for vibration and energy-related monitoring.

Peak-to-peak value:

```math
x_{pp}=\max_i x_i-\min_i x_i
```

Use:

- detects impulsive or intermittent abnormal behavior.

Skewness:

```math
\text{skew}=\frac{E[(x-\mu)^3]}{\sigma^3}
```

Use:

- measures asymmetry of the signal distribution.

Kurtosis:

```math
\text{kurt}=\frac{E[(x-\mu)^4]}{\sigma^4}
```

Use:

- sensitive to impulses and heavy tails.

Crest factor:

```math
CF=\frac{\max_i |x_i|}{x_{rms}}
```

Use:

- useful when faults create sharp peaks.

Autocorrelation:

```math
R(\tau)=E[x(t)x(t-\tau)]
```

Use:

- detects periodicity, memory, and dynamic correlation changes.

Pros of time-domain features:

- simple and fast;
- easy to interpret;
- suitable for online monitoring.

Cons:

- may miss frequency-specific faults;
- sensitive to operating condition changes;
- feature selection is problem-dependent.

## 1.5 Frequency-Domain Features

Frequency-domain diagnosis uses Fourier analysis:

```math
X(\omega)=\sum_{n=0}^{N-1}x(n)e^{-j\omega n}
```

Power spectral density:

```math
P(\omega)=|X(\omega)|^2
```

Typical features:

- dominant frequency;
- energy in a frequency band;
- harmonic amplitudes;
- spectral centroid;
- spectral bandwidth;
- sideband energy.

Use cases:

- rotating machinery;
- motors and pumps;
- periodic faults;
- resonance and vibration monitoring.

Advantages:

- good for periodic and oscillatory faults;
- physically meaningful for many mechanical systems.

Disadvantages:

- weak for purely transient faults unless time-frequency methods are used;
- requires careful sampling and window design;
- nonstationary signals may need short-time Fourier transform or wavelets.

## 1.6 Role of Dimensionality Reduction in Topic 1

Topic 1 may use simple projection or feature selection only as a preprocessing step.

The full theory of PCA, FDA, MDS, and Isomap belongs to Topic 2.

In Topic 1, the important logic is:

```text
raw/high-dimensional signal
-> compact feature vector
-> probability model or classifier
```

## 1.7 Supervised vs Unsupervised Diagnosis

Supervised diagnosis:

- training data have labels;
- labels can be normal/fault or different fault types;
- examples: LDA, logistic regression, SVM, FDA.

Advantages:

- can directly learn class boundaries;
- useful for fault isolation.

Disadvantages:

- needs representative labeled fault data;
- unknown future faults may not be recognized.

Unsupervised diagnosis:

- usually trained on normal data only;
- detects deviation from normal behavior;
- examples: PCA monitoring, density estimation, autoencoder/VAE anomaly detection.

Advantages:

- useful when fault labels are rare;
- can detect novel anomalies.

Disadvantages:

- isolation is harder;
- abnormal operating conditions may cause false alarms.

## 1.8 Detection vs Isolation

Detection:

```math
H_0:\text{normal},\qquad H_1:\text{fault}
```

Isolation:

```math
H_i:\text{fault type }i
```

Detection can use one scalar statistic:

```math
J(x)>J_{th}
```

Isolation usually needs a residual pattern, classifier output, or multiple statistics.

Example:

- high residual in sensor 1 only: sensor fault candidate;
- high residual in multiple correlated sensors: plant or actuator fault candidate;
- residual pattern matching a known class: fault isolation.

## 1.9 Performance Metrics

Confusion matrix for binary diagnosis:

- true positive (TP): fault occurs and alarm is raised;
- false positive (FP): no fault but alarm is raised;
- true negative (TN): no fault and no alarm;
- false negative (FN): fault occurs but no alarm is raised.

False alarm rate:

```math
FAR=\frac{FP}{FP+TN}
```

Missed alarm rate:

```math
MAR=\frac{FN}{FN+TP}
```

Sensitivity or recall:

```math
Sensitivity=\frac{TP}{TP+FN}
```

Specificity:

```math
Specificity=\frac{TN}{TN+FP}
```

Precision:

```math
Precision=\frac{TP}{TP+FP}
```

Accuracy:

```math
Accuracy=\frac{TP+TN}{TP+TN+FP+FN}
```

F1 score:

```math
F1=\frac{2\cdot Precision\cdot Recall}{Precision+Recall}
```

Detection delay:

```math
D=k_{alarm}-k_{fault}
```

Important trade-off:

- lower threshold gives smaller delay but higher FAR;
- higher threshold gives lower FAR but higher MAR and delay.

## 1.10 Threshold Trade-Off

Let the decision statistic be `J(x)`.

Alarm rule:

```math
J(x)>h
```

If `h` is too low:

- high sensitivity;
- many false alarms;
- operators may lose trust in the monitoring system.

If `h` is too high:

- fewer false alarms;
- faults are detected late or missed.

Engineering choice:

- safety-critical systems often prefer lower missed alarm probability;
- systems with expensive shutdowns often require strict false alarm control.

## 1.11 Bayes' Theorem

Bayes' theorem:

```math
P(C_i|x)=\frac{p(x|C_i)P(C_i)}{p(x)}
```

where:

- `C_i` is a class or hypothesis;
- `P(C_i)` is the prior probability;
- `p(x|C_i)` is the likelihood;
- `p(x)` is evidence;
- `P(C_i|x)` is the posterior probability.

For multiple classes:

```math
p(x)=\sum_j p(x|C_j)P(C_j)
```

Interpretation:

- prior: what is believed before seeing data;
- likelihood: how likely the data are under each class;
- posterior: updated belief after seeing data.

In fault diagnosis:

- normal operation may have high prior probability;
- rare but dangerous faults may still require strong risk weighting;
- likelihood describes how well the measured features match each class.

## 1.12 MAP Decision Rule

Maximum a posteriori decision:

```math
\hat C=\arg\max_i P(C_i|x)
```

Using Bayes' theorem:

```math
\hat C=\arg\max_i p(x|C_i)P(C_i)
```

because `p(x)` is common for all classes.

For two classes:

```math
\frac{p(x|C_1)P(C_1)}{p(x|C_0)P(C_0)} > 1
```

means choose `C_1`.

Advantages:

- principled probabilistic decision;
- naturally includes prior probabilities.

Limitations:

- requires reliable class-conditional density estimates;
- wrong priors can bias decisions;
- rare fault classes may be under-selected unless risk is considered.

## 1.13 Continuous Variables and PDF

For continuous variables, probability at a single point is zero. A probability density function satisfies:

```math
P(a\le x\le b)=\int_a^b p(x)dx
```

and:

```math
\int_{-\infty}^{\infty}p(x)dx=1
```

For a vector:

```math
x\in R^d,\qquad p(x)\ge0,\qquad \int p(x)dx=1
```

In classification, the absolute PDF value is less important than relative likelihood under different hypotheses.

## 1.14 Gaussian Assumption

Univariate Gaussian:

```math
p(x|\mu,\sigma^2)=\frac1{\sqrt{2\pi\sigma^2}}\exp\left[-\frac{(x-\mu)^2}{2\sigma^2}\right]
```

Multivariate Gaussian:

```math
p(x|\mu,\Sigma)=\frac1{(2\pi)^{d/2}|\Sigma|^{1/2}}
\exp\left[-\frac12(x-\mu)^T\Sigma^{-1}(x-\mu)\right]
```

Gaussian assumption is useful because:

- mean and covariance fully define the distribution;
- likelihood is easy to compute;
- LDA and QDA have closed-form decision boundaries;
- many aggregated noise effects are approximately Gaussian.

Limitations:

- real fault data may be multimodal;
- outliers and heavy tails break the assumption;
- nonlinear processes may require mixtures, kernels, or nonparametric density estimation.

## 1.15 Likelihood Ratio Test

For hypotheses:

```math
H_0:\theta=\theta_0,\qquad H_1:\theta=\theta_1
```

Likelihood ratio:

```math
\Lambda(x)=\frac{p(x|\theta_1)}{p(x|\theta_0)}
```

Decision rule:

```math
\Lambda(x)>\eta \Rightarrow H_1
```

Log-likelihood ratio:

```math
s(x)=\ln\frac{p(x|\theta_1)}{p(x|\theta_0)}
```

Use in fault detection:

- `H_0`: normal model;
- `H_1`: faulty model;
- positive log-likelihood ratio supports fault;
- negative log-likelihood ratio supports normal operation.

## 1.16 Risk-Based Decision

If different errors have different costs, MAP is not enough.

Expected risk for choosing action `alpha_i`:

```math
R(\alpha_i|x)=\sum_j \lambda(\alpha_i|C_j)P(C_j|x)
```

Choose:

```math
\alpha^*=\arg\min_i R(\alpha_i|x)
```

For fault diagnosis:

- missing a dangerous fault may be much more costly than a false alarm;
- false shutdown may also be expensive;
- risk-based decision explicitly balances safety and economy.

Example:

If the cost of missing a fault is high, the alarm threshold should be lower than the pure MAP threshold.

## 1.17 MLE Parameter Estimation

Given data:

```math
D=\{x_1,\ldots,x_N\}
```

Assume independent samples:

```math
p(D|\theta)=\prod_{i=1}^N p(x_i|\theta)
```

Maximum likelihood estimation:

```math
\hat\theta_{MLE}=\arg\max_\theta p(D|\theta)
```

Usually maximize log-likelihood:

```math
\ell(\theta)=\sum_{i=1}^N\ln p(x_i|\theta)
```

Gaussian mean MLE:

```math
\hat\mu=\frac1N\sum_{i=1}^N x_i
```

Gaussian covariance MLE:

```math
\hat\Sigma=\frac1N\sum_{i=1}^N(x_i-\hat\mu)(x_i-\hat\mu)^T
```

Advantages:

- data-driven and efficient with enough data;
- often has closed-form solutions.

Disadvantages:

- can overfit with small samples;
- does not use prior knowledge;
- sensitive to model mismatch.

## 1.18 MAP Parameter Estimation

Maximum a posteriori estimation:

```math
\hat\theta_{MAP}=\arg\max_\theta p(\theta|D)
```

By Bayes:

```math
p(\theta|D)\propto p(D|\theta)p(\theta)
```

Thus:

```math
\hat\theta_{MAP}=\arg\max_\theta \left[\ln p(D|\theta)+\ln p(\theta)\right]
```

Interpretation:

- likelihood fits data;
- prior regularizes the estimate;
- MAP is useful when data are limited or prior physical knowledge exists.

Relation to regularization:

- Gaussian prior often leads to L2 regularization;
- Laplace prior often leads to L1 regularization.

MLE vs MAP:

- MLE uses only data;
- MAP combines data and prior;
- MLE and MAP become close when data are abundant and the prior is weak.

## 1.19 Recursive Estimation

Recursive estimation updates parameters online:

```math
\hat\theta_k=\hat\theta_{k-1}+K_k\left(x_k-\hat x_k\right)
```

Generic logic:

```text
old estimate + gain * prediction error = new estimate
```

Use cases:

- online adaptation;
- slowly varying operating conditions;
- real-time monitoring when storing all past data is not feasible.

Risk:

- too fast adaptation may absorb faults as normal behavior;
- too slow adaptation may not follow normal condition changes.

## 1.20 Parametric vs Nonparametric Density Estimation

Parametric density estimation:

- assumes a fixed distribution family, such as Gaussian;
- estimates finite-dimensional parameters.

Advantages:

- compact and interpretable;
- less data required;
- efficient online computation.

Disadvantages:

- wrong distribution family causes biased decisions.

Nonparametric density estimation:

- does not impose a fixed parametric form;
- examples: histogram, Parzen window, kNN density.

Advantages:

- flexible and can model irregular distributions.

Disadvantages:

- needs more data;
- suffers from curse of dimensionality;
- bandwidth or neighborhood choice is critical.

## 1.21 Histogram Density Estimation

Divide the space into bins. If bin width is `h`, number of samples in the bin is `n_b`, and total sample number is `N`, then:

```math
\hat p(x)=\frac{n_b}{Nh}
```

In `d` dimensions with bin volume `V_b`:

```math
\hat p(x)=\frac{n_b}{NV_b}
```

Advantages:

- simple;
- easy to visualize;
- useful in one or two dimensions.

Disadvantages:

- discontinuous estimate;
- highly sensitive to bin width and bin location;
- poor in high dimension.

Use cases:

- exploratory analysis;
- rough empirical threshold design;
- visualizing normal/fault feature distributions.

## 1.22 Parzen Window

Parzen density estimation:

```math
\hat p(x)=\frac1N\sum_{i=1}^N\frac1{h^d}K\left(\frac{x-x_i}{h}\right)
```

where:

- `K` is a kernel function;
- `h` is bandwidth;
- `d` is feature dimension.

Gaussian kernel:

```math
K(u)=\frac1{(2\pi)^{d/2}}\exp\left(-\frac12u^Tu\right)
```

Bandwidth effect:

- small `h`: spiky density, overfitting, sensitive to noise;
- large `h`: oversmoothed density, may hide faults.

Advantages:

- flexible smooth density estimate;
- no fixed Gaussian assumption.

Disadvantages:

- expensive for many training samples;
- bandwidth is difficult to choose;
- weak in high dimensions.

Use cases:

- normal data density estimation;
- anomaly detection by low density;
- non-Gaussian feature distributions.

## 1.23 kNN Density Estimation

Let `V_k(x)` be the volume containing the `k` nearest training samples around `x`. Then:

```math
\hat p(x)=\frac{k}{NV_k(x)}
```

Interpretation:

- dense normal regions have small `V_k` and high density;
- sparse abnormal regions have large `V_k` and low density.

Advantages:

- adapts local bandwidth automatically;
- simple conceptually.

Disadvantages:

- distance metric matters;
- expensive online search unless indexed;
- poor under high dimension;
- sensitive to irrelevant features.

Use cases:

- local anomaly detection;
- fault detection when normal data form irregular clusters.

## 1.24 Classification Formulation

For labeled data:

```math
\{(x_i,y_i)\}_{i=1}^N,\qquad y_i\in\{1,\ldots,K\}
```

Goal:

```math
f(x)\rightarrow y
```

Generative classifiers:

- model `p(x|C_i)` and `P(C_i)`;
- use Bayes rule.

Examples:

- LDA;
- QDA;
- naive Bayes.

Discriminative classifiers:

- directly model class boundary or posterior probability.

Examples:

- logistic regression;
- SVM;
- perceptron.

Generative methods are useful when:

- density assumptions are meaningful;
- data are limited;
- probabilistic interpretation is needed.

Discriminative methods are useful when:

- boundary accuracy is more important than density modeling;
- many features are available;
- class-conditional distributions are hard to model.

## 1.25 LDA

Linear Discriminant Analysis assumes:

```math
x|C_i\sim N(\mu_i,\Sigma)
```

All classes share the same covariance `Sigma`.

The discriminant function is:

```math
g_i(x)=x^T\Sigma^{-1}\mu_i-\frac12\mu_i^T\Sigma^{-1}\mu_i+\ln P(C_i)
```

Decision:

```math
\hat C=\arg\max_i g_i(x)
```

For two classes, the boundary is linear:

```math
w^Tx+w_0=0
```

where:

```math
w=\Sigma^{-1}(\mu_1-\mu_0)
```

Fisher view for two classes:

```math
J(w)=\frac{w^TS_Bw}{w^TS_Ww}
```

with:

```math
S_B=(\mu_1-\mu_0)(\mu_1-\mu_0)^T
```

and:

```math
S_W=\Sigma_0+\Sigma_1
```

The optimal direction is proportional to:

```math
w\propto S_W^{-1}(\mu_1-\mu_0)
```

Advantages:

- simple and fast;
- interpretable linear boundary;
- works well when Gaussian equal-covariance assumption is reasonable;
- useful for fault isolation with labeled classes.

Disadvantages:

- linear boundary only;
- shared covariance assumption may be wrong;
- sensitive to outliers and covariance estimation error.

Use cases:

- normal/fault classification with approximately Gaussian features;
- multiple known fault classes with similar covariance;
- baseline classifier before more complex models.

## 1.26 QDA

Quadratic Discriminant Analysis assumes:

```math
x|C_i\sim N(\mu_i,\Sigma_i)
```

Each class has its own covariance.

Discriminant:

```math
g_i(x)=-\frac12\ln|\Sigma_i|-\frac12(x-\mu_i)^T\Sigma_i^{-1}(x-\mu_i)+\ln P(C_i)
```

Boundary is quadratic because covariance differs across classes.

Advantages:

- more flexible than LDA;
- can model classes with different covariance shapes.

Disadvantages:

- requires more samples to estimate covariances;
- easier to overfit;
- covariance matrices may become singular in high dimensions.

Use cases:

- fault classes have clearly different variance/correlation structure;
- enough labeled data exist.

## 1.27 Logistic Regression

Binary logistic regression models posterior probability:

```math
P(y=1|x)=\sigma(w^Tx+b)
```

where:

```math
\sigma(z)=\frac1{1+e^{-z}}
```

Decision:

```math
P(y=1|x)>0.5 \Rightarrow y=1
```

Equivalently:

```math
w^Tx+b>0
```

Loss function:

```math
L(w,b)=-\sum_i\left[y_i\ln p_i+(1-y_i)\ln(1-p_i)\right]
```

where:

```math
p_i=\sigma(w^Tx_i+b)
```

Multiclass softmax:

```math
P(y=k|x)=\frac{\exp(w_k^Tx+b_k)}{\sum_j\exp(w_j^Tx+b_j)}
```

Advantages:

- directly estimates posterior probability;
- easy to regularize;
- interpretable coefficients;
- efficient for high-dimensional data.

Disadvantages:

- linear boundary unless features are transformed;
- sensitive to strong class overlap;
- probability calibration depends on data quality.

Use cases:

- probabilistic fault/no-fault decision;
- threshold tuning based on risk;
- baseline classifier for labeled FDD.

---

# Topic 2: Dimensionality Reduction Approaches

## 2.1 Topic Boundary

Topic 2 answers:

```text
How should high-dimensional measurements be represented in a lower-dimensional fault-sensitive space?
```

Curriculum subitems:

- PCA and variants: Sections 2.3-2.14.
- FDA: Section 2.15.
- MDS and Isomap: Sections 2.16-2.19.

Boundary:

- Topic 2 focuses on representation, projection, subspaces, and manifolds.
- Classification after projection belongs to Topic 1 or Topic 3.
- Model-based residual generation belongs to Topic 4.

## 2.2 Why FDD Needs Dimensionality Reduction

Industrial systems often have many sensors:

```math
x\in R^d,\qquad d\text{ large}
```

Problems:

- sensor variables are correlated;
- noise exists in many channels;
- high dimensional density estimation is hard;
- visualization is difficult;
- classifiers may overfit.

Dimensionality reduction builds:

```math
z=W^Tx,\qquad z\in R^p,\quad p\ll d
```

Goals:

- remove redundancy;
- denoise;
- preserve normal process structure;
- preserve fault-sensitive information;
- make monitoring statistics easier.

## 2.3 PCA Core Idea

Principal Component Analysis finds orthogonal directions of maximum variance in the training data.

Given centered data matrix:

```math
X=[x_1,\ldots,x_N]\in R^{d\times N}
```

Covariance:

```math
S=\frac1NXX^T
```

Eigenvalue problem:

```math
Su_i=\lambda_i u_i
```

with:

```math
\lambda_1\ge\lambda_2\ge\cdots\ge\lambda_d
```

Projection onto first `p` principal components:

```math
z=U_p^Tx
```

Reconstruction:

```math
\hat x=U_pU_p^Tx
```

Residual:

```math
\tilde x=x-\hat x
```

Interpretation:

- large-variance directions define the normal subspace;
- residual subspace captures deviations not explained by normal variation.

## 2.4 Normal Subspace and Residual Subspace

Let:

```math
U=[U_p\ U_r]
```

where:

- `U_p` contains retained principal components;
- `U_r` contains discarded components.

Projection matrix:

```math
P=U_pU_p^T
```

Residual projection:

```math
I-P
```

Normal component:

```math
\hat x=Px
```

Residual component:

```math
\tilde x=(I-P)x
```

Fault detection logic:

- normal data lie close to the learned normal subspace;
- faults may increase residual energy;
- some faults may also change score values inside the normal subspace.

## 2.5 PCA Monitoring Statistics

Squared prediction error:

```math
SPE=Q=\|x-\hat x\|^2=\|(I-U_pU_p^T)x\|^2
```

Hotelling statistic:

```math
T^2=z^T\Lambda_p^{-1}z
```

where:

- `z=U_p^Tx`;
- `Lambda_p` contains retained eigenvalues.

Interpretation:

- `SPE/Q` detects variation outside the normal subspace;
- `T^2` detects unusual variation inside the normal subspace.

Use both because:

- some faults create new abnormal directions;
- some faults move the process along normal directions but to abnormal magnitudes.

## 2.6 Direct PCA Algorithm

Algorithm:

1. collect fault-free training data;
2. center and scale data;
3. compute covariance `S=XX^T/N`;
4. solve `Su_i=lambda_i u_i`;
5. choose first `p` eigenvectors;
6. online compute score and residual statistics.

Advantages:

- simple;
- interpretable;
- efficient when sensor dimension is moderate.

Disadvantages:

- covariance matrix is `d x d`;
- expensive when `d` is very large;
- linear only.

## 2.7 Dual PCA

When `d` is large and sample number `N` is smaller, use:

```math
X^TXv_i=\sigma_i^2v_i
```

Then:

```math
u_i=\frac{Xv_i}{\sigma_i}
```

Dual PCA solves an `N x N` eigenproblem instead of `d x d`.

Advantages:

- useful when sensors or features are many;
- computationally cheaper for `N<d`.

Disadvantages:

- reconstruction still lives in original dimension;
- memory can still be large for many samples.

## 2.8 Choosing the Number of Principal Components

Cumulative variance ratio:

```math
\frac{\sum_{i=1}^p\lambda_i}{\sum_{i=1}^d\lambda_i}
```

Common choice:

- retain 85 percent, 90 percent, or 95 percent variance.

Scree plot:

- choose `p` at the elbow of eigenvalue decay.

Cross-validation:

- choose `p` that gives best detection or isolation performance.

Important warning:

- maximum variance is not always maximum fault sensitivity;
- a low-variance direction may contain important fault information.

## 2.9 PCA Advantages, Disadvantages, and Use Cases

Advantages:

- reduces dimension and noise;
- handles correlated sensors;
- has clear geometric interpretation;
- works without fault labels;
- useful for process monitoring.

Disadvantages:

- linear method;
- unsupervised and does not use fault labels;
- sensitive to scaling;
- normal operating variation with large variance may dominate;
- nonlinear normal manifolds are not handled well.

Use cases:

- normal operation data are abundant;
- sensor variables are strongly correlated;
- early anomaly detection is needed;
- labels for faults are limited.

Not ideal when:

- nonlinear manifolds dominate;
- fault information is in small-variance directions;
- operation modes are strongly multimodal.

## 2.10 Kernel PCA

Kernel PCA maps data into feature space:

```math
\Phi:x\rightarrow \Phi(x)
```

and performs PCA there.

Kernel:

```math
K(x_i,x_j)=\Phi(x_i)^T\Phi(x_j)
```

Common kernels:

Linear:

```math
K(x_i,x_j)=x_i^Tx_j
```

Polynomial:

```math
K(x_i,x_j)=(\gamma x_i^Tx_j+c)^d
```

RBF:

```math
K(x_i,x_j)=\exp(-\gamma\|x_i-x_j\|^2)
```

Advantages:

- handles nonlinear normal manifolds;
- keeps the PCA logic through the kernel trick.

Disadvantages:

- kernel and hyperparameters are critical;
- feature-space reconstruction is difficult;
- computational cost grows with sample number;
- interpretation is weaker.

Use cases:

- nonlinear process monitoring;
- normal data lie on curved manifold;
- linear PCA gives many false alarms.

## 2.11 Supervised PCA

PCA is unsupervised, so it may ignore low-variance but fault-relevant directions.

Supervised PCA uses response or label information `Y`.

Typical workflow:

1. compute dependence between each sensor and `Y`;
2. select fault-sensitive sensors;
3. run PCA on selected sensors;
4. use PCs for classification or regression.

Correlation score:

```math
s_j=\frac{X_j^TY}{\sqrt{X_j^TX_j}}
```

Keep sensors satisfying:

```math
\lvert s_j\rvert>\delta
```

Advantages:

- keeps features related to faults;
- improves fault isolation compared with unsupervised PCA.

Disadvantages:

- needs labels or fault-related response;
- may overfit labels;
- selected features depend on training fault types.

## 2.12 HSIC-SPCA

Hilbert-Schmidt Independence Criterion measures dependence between variables using kernels.

A common empirical form:

```math
HSIC=\frac1{(n-1)^2}Tr(KHLH)
```

where:

- `K` is kernel matrix for inputs;
- `L` or `B` is kernel matrix for labels/responses;
- `H=I-\frac1n11^T` is centering matrix.

HSIC-SPCA chooses projection `U` to maximize dependence with labels:

```math
\max_U Tr(U^TXHBHX^TU)
```

Solution:

- top eigenvectors of `XHBHX^T`.

Advantages:

- captures nonlinear dependence through kernels;
- more fault-sensitive than ordinary PCA.

Disadvantages:

- needs labels;
- kernel choice matters;
- more complex and less interpretable.

## 2.13 Kernel SPCA

Kernel SPCA combines:

- kernel representation for nonlinear input structure;
- supervised dependence measure for fault relevance.

Use cases:

- nonlinear processes;
- labeled fault data available;
- fault-sensitive directions are not captured by linear PCA.

Limitations:

- high computational cost;
- kernel and regularization tuning;
- risk of overfitting with limited labeled fault data.

## 2.14 Metric Learning and SDR

Mahalanobis distance:

```math
d_A(x_i,x_j)=\sqrt{(x_i-x_j)^TA(x_i-x_j)}
```

where `A` is positive semidefinite.

If:

```math
A=UU^T
```

then:

```math
d_A(x_i,x_j)=\|U^Tx_i-U^Tx_j\|
```

Thus metric learning can be interpreted as learning a projection.

Sufficient dimension reduction (SDR) aims to find `U` such that:

```math
p(y|x)=p(y|U^Tx)
```

Interpretation:

- all information about output or fault label is preserved in low-dimensional projection.

Use cases:

- fault-sensitive representation learning;
- classification with fewer dimensions;
- preserving label-relevant information.

## 2.15 FDA

Fisher Discriminant Analysis is supervised dimensionality reduction.

For two classes, projection:

```math
z=w^Tx
```

Goal:

- maximize between-class separation;
- minimize within-class scatter.

Objective:

```math
J(w)=\frac{w^TS_Bw}{w^TS_Ww}
```

where:

```math
S_B=(\mu_1-\mu_0)(\mu_1-\mu_0)^T
```

and:

```math
S_W=\Sigma_0+\Sigma_1
```

Solution:

```math
S_Bw=\lambda S_Ww
```

For two classes:

```math
w\propto S_W^{-1}(\mu_1-\mu_0)
```

Multiclass FDA uses:

```math
S_B=\sum_k n_k(\mu_k-\mu)(\mu_k-\mu)^T
```

and:

```math
S_W=\sum_k\sum_{i\in C_k}(x_i-\mu_k)(x_i-\mu_k)^T
```

Advantages:

- directly uses labels;
- good for fault isolation;
- produces low-dimensional discriminative features.

Disadvantages:

- assumes classes are separable by linear projection;
- within-class scatter matrix may be singular;
- sensitive to label quality.

Use cases:

- labeled normal and fault classes;
- fault isolation more important than anomaly detection;
- dimensionality reduction before classification.

## 2.16 MDS

Multidimensional Scaling embeds data into low-dimensional coordinates while preserving pairwise distances or similarities.

Given distance matrix `D`, classical MDS constructs a centered inner-product matrix:

```math
B=-\frac12HD^{(2)}H
```

where:

```math
H=I-\frac1n11^T
```

Eigen-decomposition:

```math
B=V\Lambda V^T
```

Low-dimensional embedding:

```math
Y=\Lambda_p^{1/2}V_p^T
```

Classical MDS objective:

```math
\min_Y \|X^TX-Y^TY\|_F^2
```

Relation to PCA:

- for Euclidean distances and centered data, classical MDS is equivalent to PCA.

Advantages:

- works from distance or dissimilarity matrix;
- useful for visualization;
- does not require original coordinates if distances are known.

Disadvantages:

- sensitive to distance definition;
- computationally expensive for many samples;
- mainly preserves global pairwise structure, not necessarily class separability.

## 2.17 Generalized / Kernel MDS

Generalized MDS can start from non-Euclidean dissimilarities.

Kernel MDS uses a positive semidefinite kernel matrix as inner products.

Logic:

```text
dissimilarity or kernel matrix
-> centered Gram matrix
-> eigen-decomposition
-> low-dimensional embedding
```

Use cases:

- only pairwise distances are available;
- domain-specific similarity is more meaningful than raw coordinates;
- visualization of fault clusters.

Limitations:

- new sample extension is not as direct as PCA;
- distance/kernel choice dominates results.

## 2.18 Isomap

Isomap is nonlinear manifold learning.

Main idea:

- Euclidean distance is wrong on a curved manifold;
- geodesic distance along the manifold is more meaningful.

Algorithm:

1. build k-nearest-neighbor graph;
2. edge weight is Euclidean distance between neighbors;
3. compute shortest path distances on the graph;
4. apply classical MDS to geodesic distance matrix.

Geodesic distance approximation:

```math
d_G(i,j)=\text{shortest path distance between }i\text{ and }j
```

Advantages:

- can unfold nonlinear manifolds;
- useful for nonlinear process visualization;
- may separate fault clusters hidden in original space.

Disadvantages:

- sensitive to neighbor number `k`;
- graph disconnection breaks the method;
- noise can create wrong shortcuts;
- out-of-sample extension is not straightforward.

Use cases:

- nonlinear normal operating manifold;
- visualization of fault evolution;
- exploratory fault cluster analysis.

## 2.19 Kernel Isomap

Kernel Isomap interprets the Isomap embedding through kernel/Gram matrix construction from geodesic distances.

Use cases:

- nonlinear manifold monitoring;
- combining graph-based geodesic distance with kernel methods.

Limitations:

- inherits sensitivity to graph construction;
- computationally expensive for large datasets.

---

# Topic 3: Support Vector Machines and Perceptron

## 3.1 Topic Boundary

Topic 3 studies geometric maximum-margin classification.

Curriculum subitems:

- Hard Margin SVM: Sections 3.5-3.7.
- Soft Margin SVM: Sections 3.8-3.11.
- Kernel Trick and Perceptron: Sections 3.12 and 3.14.

Boundary:

- Topic 1 uses probabilistic classifiers such as LDA and logistic regression.
- Topic 3 uses margin, constrained optimization, kernels, and perceptron learning.

## 3.2 Role of SVM in FDD

FDD pipeline:

```text
sensor signals
-> features or dimensionality reduction
-> feature vector x
-> SVM classifier
-> normal / fault / fault type
```

SVM is useful when:

- feature dimension is high;
- samples are limited;
- the boundary is clear after feature extraction;
- strong generalization is needed.

## 3.3 Basic Concepts

Hyperplane:

```math
\beta^Tx+\beta_0=0
```

Decision function:

```math
f(x)=\beta^Tx+\beta_0
```

Class labels:

```math
y_i\in\{-1,+1\}
```

Correct classification:

```math
y_i(\beta^Tx_i+\beta_0)>0
```

Functional margin:

```math
\gamma_i=y_i(\beta^Tx_i+\beta_0)
```

Geometric distance to hyperplane:

```math
d_i=\frac{y_i(\beta^Tx_i+\beta_0)}{\|\beta\|}
```

Support vectors:

- training samples closest to the decision boundary;
- they determine the final SVM boundary.

## 3.4 SVM Advantages and Disadvantages

Advantages:

- strong high-dimensional performance;
- decision boundary depends only on support vectors;
- kernel trick handles nonlinear boundaries;
- maximum-margin principle often improves generalization.

Disadvantages:

- large datasets can be slow;
- kernel and parameter tuning are important;
- native SVM does not output calibrated probabilities;
- performance drops with severe class overlap or label noise.

Use cases:

- fault classification with high-dimensional features;
- condition monitoring with limited labeled data;
- nonlinear class boundaries through RBF kernels.

## 3.5 Hard Margin SVM

Assumption:

- data are linearly separable.

Constraints:

```math
y_i(\beta^Tx_i+\beta_0)\ge1
```

Margin:

```math
M=\frac1{\|\beta\|}
```

Maximizing margin is equivalent to:

```math
\min_{\beta,\beta_0}\frac12\|\beta\|^2
```

subject to:

```math
y_i(\beta^Tx_i+\beta_0)\ge1,\quad i=1,\ldots,N
```

Advantages:

- clean geometric interpretation;
- maximum margin gives good generalization if assumptions hold.

Disadvantages:

- fails if data are not separable;
- extremely sensitive to outliers;
- not suitable for noisy fault data.

Use cases:

- ideal separable teaching case;
- low-noise engineered features.

## 3.6 Lagrangian and KKT

Lagrangian:

```math
\mathcal L(\beta,\beta_0,\alpha)=\frac12\|\beta\|^2
-\sum_{i=1}^N\alpha_i[y_i(\beta^Tx_i+\beta_0)-1]
```

Dual variables:

```math
\alpha_i\ge0
```

KKT conditions:

Stationarity:

```math
\frac{\partial\mathcal L}{\partial\beta}=0,\qquad
\frac{\partial\mathcal L}{\partial\beta_0}=0
```

Primal feasibility:

```math
y_i(\beta^Tx_i+\beta_0)\ge1
```

Dual feasibility:

```math
\alpha_i\ge0
```

Complementary slackness:

```math
\alpha_i[y_i(\beta^Tx_i+\beta_0)-1]=0
```

Interpretation:

- if `alpha_i=0`, the point does not affect the boundary;
- if `alpha_i>0`, the point lies on the margin and is a support vector.

## 3.7 Hard Margin Dual

From stationarity:

```math
\beta=\sum_i\alpha_i y_i x_i
```

and:

```math
\sum_i\alpha_i y_i=0
```

Dual problem:

```math
\max_\alpha \sum_i\alpha_i-\frac12\sum_i\sum_j\alpha_i\alpha_jy_iy_jx_i^Tx_j
```

subject to:

```math
\alpha_i\ge0,\qquad \sum_i\alpha_i y_i=0
```

Decision function:

```math
f(x)=\sum_i\alpha_i y_i x_i^Tx+\beta_0
```

Only support vectors have nonzero `alpha_i`.

## 3.8 Soft Margin SVM

Real fault data are often noisy and not perfectly separable.

Introduce slack variables:

```math
\xi_i\ge0
```

Constraint:

```math
y_i(\beta^Tx_i+\beta_0)\ge1-\xi_i
```

Optimization:

```math
\min_{\beta,\beta_0,\xi}\frac12\|\beta\|^2+C\sum_i\xi_i
```

subject to:

```math
y_i(\beta^Tx_i+\beta_0)\ge1-\xi_i,\qquad \xi_i\ge0
```

Interpretation:

- first term prefers large margin;
- second term penalizes margin violations and misclassification;
- `C` controls the trade-off.

## 3.9 Slack Variable Interpretation

For a sample:

- `xi_i=0`: correctly classified and outside or on the margin;
- `0<xi_i<1`: inside the margin but still correctly classified;
- `xi_i=1`: exactly on the decision boundary;
- `xi_i>1`: misclassified.

Engineering interpretation:

- slack variables allow noisy or overlapping fault classes;
- they prevent one outlier from destroying the boundary.

## 3.10 C Parameter

Small `C`:

- weak penalty for violations;
- wider margin;
- more training errors allowed;
- often better generalization.

Large `C`:

- strong penalty for violations;
- narrower margin;
- fewer training errors;
- higher overfitting risk.

For FDD:

- if false alarms are too frequent, `C` and kernel parameters may need tuning;
- if faults are missed, the boundary may be too conservative or features may be weak.

## 3.11 Soft Margin Dual

Dual objective is similar to hard margin:

```math
\max_\alpha \sum_i\alpha_i-\frac12\sum_i\sum_j\alpha_i\alpha_jy_iy_jx_i^Tx_j
```

subject to:

```math
0\le\alpha_i\le C,\qquad \sum_i\alpha_i y_i=0
```

Difference from hard margin:

- `alpha_i` has upper bound `C`;
- points with `alpha_i=C` often violate the margin or are misclassified.

## 3.12 Kernel Trick

Nonlinear SVM maps data into feature space:

```math
x\rightarrow \Phi(x)
```

The dual only needs dot products:

```math
\Phi(x_i)^T\Phi(x_j)
```

Kernel trick:

```math
K(x_i,x_j)=\Phi(x_i)^T\Phi(x_j)
```

Decision function:

```math
f(x)=\sum_i\alpha_i y_i K(x_i,x)+\beta_0
```

Common kernels:

Linear:

```math
K(x_i,x_j)=x_i^Tx_j
```

Polynomial:

```math
K(x_i,x_j)=(\gamma x_i^Tx_j+c)^d
```

RBF:

```math
K(x_i,x_j)=\exp(-\gamma\|x_i-x_j\|^2)
```

Sigmoid:

```math
K(x_i,x_j)=\tanh(\alpha x_i^Tx_j+c)
```

RBF parameter:

- large `gamma`: complex boundary, possible overfitting;
- small `gamma`: smoother boundary, possible underfitting.

Use cases:

- nonlinear fault class boundaries;
- high-dimensional feature spaces without explicit mapping.

## 3.13 Hard vs Soft Margin Summary

Hard margin:

- assumes perfect linear separability;
- no training error allowed;
- sensitive to outliers;
- mainly theoretical baseline.

Soft margin:

- allows violations through slack variables;
- practical for noisy data;
- controlled by `C`;
- standard choice in real FDD applications.

## 3.14 Perceptron

Perceptron is a linear classifier:

```math
\hat y=\text{sign}(w^Tx+b)
```

Update on misclassified sample:

```math
w\leftarrow w+\eta y_i x_i
```

```math
b\leftarrow b+\eta y_i
```

Convergence:

- if data are linearly separable, perceptron converges to a separating hyperplane;
- if data are not separable, it may not converge.

Perceptron vs SVM:

- perceptron finds a separating boundary;
- SVM finds the maximum-margin boundary;
- SVM is usually more robust and has better generalization.

Use cases:

- simple linear baseline;
- conceptual bridge to margin classifiers;
- online linear classification.

---

# Topic 4: Model-Based Fault Detection

## 4.1 Topic Boundary

Topic 4 uses system models to generate residuals or state estimates, then applies statistical decision logic.

Curriculum subitems:

- Model-based residual generation: Sections 4.2-4.3.
- Parity method in frequency/time domain: Section 4.4.
- Observer-based methods (UIO, KF, ESO, NDO): Sections 4.5-4.10.
- CUSUM and Average Run Length: Sections 4.12-4.16.

Controller reconfiguration belongs to Topic 5 and Topic 6.

## 4.2 Basic Model-Based FDD Model

Continuous-time LTI model:

```math
\dot x=Ax+Bu+Ed
```

```math
y=Cx
```

With faults:

```math
\dot x=Ax+Bu+F_af_a+Ed
```

```math
y=Cx+F_sf_s+v
```

where:

- `x` is state;
- `u` is known input;
- `y` is measured output;
- `d` is unknown disturbance;
- `f_a` is actuator or process fault;
- `f_s` is sensor fault;
- `v` is measurement noise.

Discrete model:

```math
x_{k+1}=Ax_k+Bu_k+F_af_{a,k}+Ed_k
```

```math
y_k=Cx_k+F_sf_{s,k}+v_k
```

Goal:

```text
generate residual r that is small under normal operation and large under faults
```

## 4.3 Residual Design Goals

Residual:

```math
r=y-\hat y
```

Good residual should be:

- sensitive to faults;
- robust to noise and disturbances;
- easy to threshold;
- useful for isolation;
- stable and causal for online use.

Main trade-off:

- high sensitivity may increase false alarms;
- high robustness may hide small faults.

Residual evaluation:

```math
J(r)>J_{th}\Rightarrow alarm
```

Common statistics:

```math
J=r^Tr
```

or:

```math
J=r^TR^{-1}r
```

where `R` is residual covariance.

## 4.4 Parity Method

Parity method constructs input-output relations that must hold for the nominal model.

Discrete-time system:

```math
x_{k+1}=Ax_k+Bu_k
```

```math
y_k=Cx_k
```

Stack outputs over a time window:

```math
Y_k=O_s x_k+H_s U_k
```

where `O_s` is the observability matrix.

Choose parity vector or matrix `W` such that:

```math
WO_s=0
```

Then:

```math
r_k=W(Y_k-H_sU_k)
```

Nominally:

```math
r_k=0
```

Faults break the parity relation and make residual nonzero.

Frequency-domain parity:

- uses transfer functions;
- constructs residual filters that cancel nominal input-output behavior;
- faults appear as nonzero residual components.

Advantages:

- direct input-output method;
- no explicit state observer needed;
- useful when a reliable transfer model is available.

Disadvantages:

- sensitive to model mismatch;
- noise and unmodeled dynamics create residuals;
- window size and filter design affect delay.

Use cases:

- LTI systems with reliable models;
- input-output data available;
- fault detection rather than detailed state estimation.

## 4.5 Observer-Based Residual Generation

Luenberger observer:

```math
\dot{\hat x}=A\hat x+Bu+L(y-C\hat x)
```

Output estimate:

```math
\hat y=C\hat x
```

Residual:

```math
r=y-\hat y
```

Estimation error:

```math
e=x-\hat x
```

Error dynamics:

```math
\dot e=(A-LC)e
```

Choose `L` so that `A-LC` is stable.

With faults:

```math
\dot e=(A-LC)e+F_af_a+Ed-LF_sf_s
```

Faults influence residual through the observer error.

Advantages:

- uses dynamic model;
- can filter noise;
- residual has physical interpretation.

Disadvantages:

- requires observability or detectability;
- model mismatch produces false residual;
- observer gain tuning affects noise sensitivity and delay.

## 4.6 Kalman Filter

Stochastic model:

```math
x_{k+1}=Ax_k+Bu_k+w_k
```

```math
y_k=Cx_k+v_k
```

with:

```math
w_k\sim N(0,Q),\qquad v_k\sim N(0,R)
```

Prediction:

```math
\hat x_{k|k-1}=A\hat x_{k-1|k-1}+Bu_{k-1}
```

Innovation:

```math
\nu_k=y_k-C\hat x_{k|k-1}
```

Kalman gain:

```math
K_k=P_{k|k-1}C^T(CP_{k|k-1}C^T+R)^{-1}
```

Update:

```math
\hat x_{k|k}=\hat x_{k|k-1}+K_k\nu_k
```

Innovation residual:

```math
r_k=\nu_k
```

Advantages:

- optimal for linear Gaussian systems;
- explicitly handles process and measurement noise;
- innovation covariance supports statistical thresholds.

Disadvantages:

- depends on correct `Q` and `R`;
- Gaussian and linear assumptions may fail;
- faults may be partially absorbed by the estimator.

Use cases:

- noisy sensor systems;
- model-based residual generation with uncertainty.

## 4.7 ESO / LESO

Extended State Observer treats unknown disturbance or fault as an additional state.

Example:

```math
\dot x=Ax+Bu+Bd
```

Assume:

```math
\dot d\approx0
```

Augmented state:

```math
x_a=\begin{bmatrix}x\\d\end{bmatrix}
```

Augmented dynamics:

```math
\dot x_a=A_ax_a+B_au
```

Observer:

```math
\dot{\hat x}_a=A_a\hat x_a+B_au+L(y-C_a\hat x_a)
```

Estimated disturbance:

```math
\hat d
```

Residual or compensation:

```math
r_d=\hat d
```

or use `hat d` to compensate the actuator fault.

LESO:

- Linear Extended State Observer;
- often used when the augmented model is linear or locally linearized.

Advantages:

- estimates lumped disturbance or actuator fault;
- useful for active fault-tolerant control;
- does not require exact fault model.

Disadvantages:

- assumes disturbance/fault is slowly varying;
- fast faults create estimation lag;
- high observer gain amplifies noise.

Use cases:

- slowly varying actuator faults;
- disturbance rejection;
- LESO plus MPC resilient control.

## 4.8 UIO

Unknown Input Observer aims to estimate state while decoupling unknown input.

System:

```math
\dot x=Ax+Bu+Ed
```

```math
y=Cx
```

Observer structure:

```math
\dot z=Fz+TBu+Ky
```

```math
\hat x=z+Hy
```

Decoupling condition:

```math
(I-HC)E=0
```

Existence condition:

```math
rank(E)=rank(CE)
```

One choice:

```math
H=E(CE)^\dagger
```

```math
T=I-HC
```

```math
F=(I-HC)A-K_1C
```

```math
K=K_1+FH
```

Choose `K_1` so that `F` is stable.

Interpretation:

- UIO does not necessarily estimate the unknown input directly;
- it designs the estimation error so unknown input does not corrupt state estimation.

Advantages:

- gives clean state estimate under unknown inputs;
- useful for sensor cleaning and fault isolation;
- can support algebraic fault reconstruction.

Disadvantages:

- rank condition is restrictive;
- requires accurate model and known unknown-input direction;
- not always feasible.

Use cases:

- actuator fault isolation;
- resilient MPC with clean state estimate;
- systems where unknown input enters through known matrix `E`.

## 4.9 NDO

NDO means Nonlinear Disturbance Observer.

It is an observer-based method for nonlinear systems:

```math
\dot x=f(x,u)+E(x)d
```

```math
y=h(x)
```

Goal:

```math
\hat d\rightarrow d
```

Generic observer idea:

```math
\dot{\hat x}=f(\hat x,u)+E(\hat x)\hat d+L_x(y-\hat y)
```

```math
\dot{\hat d}=L_d(y-\hat y)
```

```math
\hat y=h(\hat x)
```

Residual choices:

```math
r_y=y-\hat y
```

or:

```math
r_d=\hat d
```

NDO vs ESO vs UIO:

- ESO/LESO: treats disturbance as an extended state, often slowly varying.
- NDO: uses nonlinear model structure to estimate lumped disturbance/fault.
- UIO: decouples unknown input from state estimation error through structural conditions.

Advantages:

- suitable for nonlinear dynamics;
- can estimate lumped disturbances and faults;
- useful for compensation.

Disadvantages:

- depends on nonlinear model quality;
- gain design can be difficult;
- noise sensitivity and convergence proof depend on assumptions.

Use cases:

- nonlinear plants;
- disturbance/fault estimation;
- nonlinear active fault-tolerant control.

## 4.10 Observability and Detectability

Observability:

```math
\mathcal O=\begin{bmatrix}C\\CA\\\vdots\\CA^{n-1}\end{bmatrix}
```

System is observable if:

```math
rank(\mathcal O)=n
```

Detectability:

- unobservable modes may exist;
- but all unobservable modes must be stable.

Observer design requires:

- observability for arbitrary pole placement;
- detectability for stable estimation error.

In UIO:

- after decoupling, the effective pair must be detectable.

## 4.11 Statistical Decision and Hypothesis Testing

Residual-based detection can be formulated as:

```math
H_0:\text{normal}
```

```math
H_1:\text{fault}
```

Test statistic:

```math
J_k=g(r_k)
```

Alarm:

```math
J_k>h
```

If residual is Gaussian under normal operation:

```math
r_k\sim N(0,\Sigma_r)
```

then:

```math
J_k=r_k^T\Sigma_r^{-1}r_k
```

may follow a chi-square distribution under suitable assumptions.

## 4.12 CUSUM Offline Derivation

Cumulative Sum detects changes as early as possible.

Log-likelihood ratio:

```math
s(x_k)=\ln\frac{f_{\theta_1}(x_k)}{f_{\theta_0}(x_k)}
```

If change time `n_c` were known:

```math
S_{n_c}^k=\sum_{i=n_c}^k s(x_i)
```

Unknown change time:

```math
g(k)=\max_{1\le n_c\le k}\sum_{i=n_c}^k s(x_i)
```

Alarm:

```math
g(k)>h
```

Interpretation:

- before fault, log-likelihood ratio tends to drift negative;
- after fault, it tends to drift positive;
- CUSUM searches for the most likely change time.

## 4.13 Recursive CUSUM

Recursive form:

```math
g(k)=\max(0,g(k-1)+s(x_k))
```

with:

```math
g(0)=0
```

Alarm:

```math
k_a=\inf\{k:g(k)>h\}
```

The `max(0,.)` reset means:

- negative evidence is discarded;
- accumulation starts again when data begin to support a fault.

Change-time estimate:

```math
\hat n_c=k_a-N(k_a)
```

where `N(k)` counts the length of the current positive accumulation period.

## 4.14 Gaussian CUSUM

Assume:

```math
H_0:x_k\sim N(\mu_0,\sigma^2)
```

```math
H_1:x_k\sim N(\mu_1,\sigma^2)
```

Log-likelihood increment:

```math
s(x_k)=\frac{\mu_1-\mu_0}{\sigma^2}
\left(x_k-\frac{\mu_1+\mu_0}{2}\right)
```

Under `H_1`:

```math
E[s(x_k)|H_1]=\frac{(\mu_1-\mu_0)^2}{2\sigma^2}>0
```

Under `H_0`:

```math
E[s(x_k)|H_0]=-\frac{(\mu_1-\mu_0)^2}{2\sigma^2}<0
```

This drift difference is why CUSUM detects persistent mean shifts.

Use cases:

- small but persistent faults;
- sensor bias;
- slow degradation;
- change detection in residual statistics.

## 4.15 Alarm Time Factors

Alarm time depends on:

- threshold `h`;
- signal-to-noise ratio;
- mismatch between assumed and actual fault model;
- sampling period;
- residual filtering;
- pre-fault residual drift.

Lower `h`:

- faster alarms;
- more false alarms.

Higher `h`:

- fewer false alarms;
- longer detection delay.

## 4.16 ARL

Average Run Length under fault:

```math
\tau_D=E[k_a-k_f|H_1]
```

This is average detection delay. Smaller is better.

Average Run Length under normal operation:

```math
\tau_F=E[k_a|H_0]
```

This is average time between false alarms. Larger is better.

Threshold design:

1. choose acceptable false alarm ARL `tau_F`;
2. tune threshold `h` to satisfy it;
3. check detection delay `tau_D`;
4. adjust residual design or fault model if delay is too large.

## 4.17 Topic 4 Method Selection

Parity method:

- best when input-output model is reliable;
- direct and simple;
- less suitable with large model mismatch.

Luenberger observer:

- good for deterministic LTI systems;
- requires observability or detectability.

Kalman filter:

- good for noisy systems;
- needs covariance models.

ESO/LESO:

- good for slowly varying unknown disturbance or fault;
- can support compensation;
- may lag for fast changes.

UIO:

- good when unknown input direction is known and rank condition holds;
- provides clean state estimate;
- restrictive feasibility condition.

NDO:

- good for nonlinear disturbance/fault estimation;
- model quality and gain design are critical.

CUSUM:

- good for persistent small changes;
- needs normal/fault statistical models.

ARL:

- used to tune threshold based on false alarm and delay requirements.

---

# Topic 5: Fault Tolerant Control Strategy

## 5.1 Topic Boundary

Topic 5 studies how to maintain stability and acceptable performance after faults by robust, adaptive, or reconfigurable control.

Curriculum subitems:

- Introduction to robust/adaptive control: Section 5.6.
- Reconfigurable control systems: Section 5.10.
- FTC methods: model matching, transient elimination, virtual actuator and sensor: Sections 5.11-5.21.

Boundary:

- Topic 4 diagnoses faults.
- Topic 5 uses diagnosis results to reconfigure control.
- Topic 6 adds CPS security, hard constraints, MPC, and safe/resilient synthesis.

## 5.2 Fault Locations

Actuator fault:

```math
B\rightarrow B_f
```

or:

```math
u_{actual}=u+f_a
```

Plant/component fault:

```math
A\rightarrow A_f
```

Sensor fault:

```math
C\rightarrow C_f
```

or:

```math
y=Cx+f_s
```

Impact:

- actuator faults reduce or distort control authority;
- plant faults change dynamics;
- sensor faults corrupt feedback information.

## 5.3 Redundancy

FTC requires redundancy.

Hardware redundancy:

- extra sensors;
- extra actuators;
- backup channels.

Analytical redundancy:

- use model and remaining measurements to reconstruct missing or corrupted signals;
- observers and virtual sensors are examples.

Key principle:

- if a fault destroys controllability or observability with no remaining redundancy, no controller can fully recover nominal performance.

## 5.4 Passive FTC

Passive FTC uses one fixed robust controller designed to tolerate a set of faults and uncertainties.

Logic:

```text
design one controller robust enough for all expected faults
```

Advantages:

- simple online implementation;
- no fault diagnosis required for switching;
- low real-time complexity;
- avoids wrong switching due to misdiagnosis.

Disadvantages:

- conservative;
- handles only predefined fault range;
- performance may be poor in normal operation;
- cannot adapt to severe unexpected faults.

Use cases:

- faults are bounded and known;
- diagnosis is unreliable or unavailable;
- real-time switching risk must be avoided.

## 5.5 Active FTC

Active FTC uses fault diagnosis and reconfiguration.

Logic:

```text
FDD -> decision -> controller/sensor/actuator reconfiguration
```

Advantages:

- less conservative than passive FTC;
- can handle larger and more specific faults;
- can recover better performance if diagnosis is accurate.

Disadvantages:

- depends on diagnosis accuracy and speed;
- wrong isolation can choose wrong controller;
- switching transients must be managed.

Use cases:

- reliable FDD available;
- multiple fault modes;
- performance recovery is important.

## 5.6 Robust vs Adaptive Control

Robust control:

- assumes uncertainty belongs to a known set;
- designs controller to maintain stability for all allowed uncertainties.

Advantages:

- strong guarantees;
- no online parameter estimation required.

Disadvantages:

- conservative;
- uncertainty set must be known.

Adaptive control:

- estimates unknown parameters online;
- updates controller according to estimates.

Advantages:

- can handle slowly varying parameters;
- less conservative when adaptation works.

Disadvantages:

- needs persistent excitation or good adaptation conditions;
- may react badly to abrupt faults;
- stability proof is more complex.

FTC relation:

- passive FTC is often robust-control based;
- active FTC can use adaptive estimation and reconfiguration.

## 5.7 State-Space Control Foundation

Nominal system:

```math
\dot x=Ax+Bu
```

State feedback:

```math
u=-Kx+Nr
```

Closed loop:

```math
\dot x=(A-BK)x+BNr
```

Goal:

- choose `K` so that `A-BK` is stable and has desired performance.

Reference feedforward `N` is used to improve steady-state tracking.

This foundation is needed because FTC often tries to modify `K`, `N`, input mapping, or measured state.

## 5.8 LQR

LQR minimizes:

```math
J=\int_0^\infty (x^TQx+u^TRu)dt
```

where:

- `Q` penalizes state deviation;
- `R` penalizes control effort.

Optimal feedback:

```math
u=-Kx
```

with:

```math
K=R^{-1}B^TP
```

where `P` solves the algebraic Riccati equation:

```math
A^TP+PA-PBR^{-1}B^TP+Q=0
```

Advantages:

- systematic trade-off between performance and effort;
- produces stabilizing controller under standard assumptions.

Limitations for FTC:

- no hard constraints;
- assumes nominal model unless redesigned;
- actuator faults may invalidate `B`.

## 5.9 Observer and Separation Principle

If state is not measured, use observer:

```math
\dot{\hat x}=A\hat x+Bu+L(y-C\hat x)
```

Controller:

```math
u=-K\hat x
```

Separation principle:

- for nominal linear systems, controller poles and observer poles can be designed separately.

Closed-loop estimation error:

```math
\dot e=(A-LC)e
```

FTC relevance:

- sensor faults corrupt `y`, so observer may fail;
- virtual sensors and UIO can provide clean estimates;
- observer dynamics matter during controller switching.

## 5.10 Reconfigurable Control Systems

A reconfigurable control system changes controller structure, parameters, or signal paths after fault diagnosis.

Architecture:

```text
plant
-> sensors
-> FDD
-> decision system
-> controller reconfiguration
-> actuator command
```

Reconfiguration options:

- switch to a predesigned controller;
- recompute feedback gain;
- redistribute actuator commands;
- replace faulty sensor by virtual sensor;
- insert virtual actuator;
- change reference or performance objective.

Challenges:

- diagnosis delay;
- diagnosis uncertainty;
- switching transient;
- actuator saturation;
- loss of controllability or observability.

## 5.11 Model Matching

Goal:

- make the faulty closed-loop system match the nominal closed-loop system.

Nominal closed loop:

```math
A_c=A-BK
```

Faulty plant:

```math
\dot x=A_fx+B_fu
```

Choose `K_f` so that:

```math
A_f-B_fK_f=A-BK
```

This is the model matching equation.

Interpretation:

- if exact matching is possible, the faulty system behaves like the nominal closed loop;
- if not, choose the closest achievable behavior.

## 5.12 Exact Model Matching

Exact model matching condition:

```math
B_fK_f=A_f-A+BK
```

A solution exists if the right-hand side lies in the column space of `B_f`.

If `B_f` loses rank due to actuator fault:

- exact matching may be impossible;
- some control directions are lost.

Advantages:

- restores nominal dynamics exactly if feasible;
- clear design objective.

Disadvantages:

- strong algebraic feasibility condition;
- exact recovery impossible after severe actuator loss.

Use cases:

- actuator effectiveness loss but enough control authority remains;
- fault model is accurately identified.

## 5.13 Approximate Model Matching

If exact model matching is impossible, solve least squares:

```math
\min_{K_f}\|(A_f-B_fK_f)-(A-BK)\|_F
```

Equivalent:

```math
\min_{K_f}\|B_fK_f-(A_f-A+BK)\|_F
```

Pseudo-inverse solution:

```math
K_f=B_f^\dagger(A_f-A+BK)
```

Advantages:

- always gives a best approximation;
- useful under rank loss.

Disadvantages:

- cannot fully recover nominal dynamics;
- residual mismatch may reduce performance or stability margin.

## 5.14 Plant and Actuator Fault Reconfiguration

Actuator effectiveness fault:

```math
B_f=B\Gamma
```

where `Gamma` is diagonal effectiveness matrix.

If some actuators lose effectiveness:

- remaining actuators may compensate if enough authority remains;
- control allocation may redistribute input demand.

Plant fault:

```math
A\rightarrow A_f
```

Reconfiguration may require:

- new feedback gain;
- model matching;
- gain scheduling;
- adaptive parameter update.

Key limitation:

- if the faulty pair `(A_f,B_f)` is not stabilizable, no state feedback can stabilize all unstable modes.

## 5.15 Sensor Reconfiguration

Nominal control:

```math
u=-Ky
```

or output feedback based on:

```math
y=Cx
```

Faulty sensor matrix:

```math
y_f=C_fx
```

Need a new gain `K_f` such that:

```math
K_fC_f=KC
```

One least-squares choice:

```math
K_f=KCC_f^\dagger
```

Exact matching condition:

```math
Ker(C_f)\subseteq Ker(C)
```

Interpretation:

- any state direction invisible to faulty sensors must also be irrelevant to the nominal measured signal;
- otherwise the lost information cannot be reconstructed exactly.

Use cases:

- sensor loss with remaining redundant measurements;
- virtual sensor construction.

## 5.16 Virtual Actuator and Virtual Sensor

Virtual actuator:

- inserted between nominal controller and faulty plant;
- transforms nominal command into a command that the faulty actuator system can implement.

Logic:

```text
nominal controller output u
-> virtual actuator
-> faulty plant input u_f
```

Goal:

- keep the nominal controller unchanged;
- hide actuator fault from controller as much as possible.

Virtual sensor:

- inserted between faulty sensors and nominal controller;
- reconstructs the signal expected by the nominal controller.

Logic:

```text
faulty measurement y_f
-> virtual sensor
-> reconstructed y
-> nominal controller
```

Advantages:

- modular design;
- nominal controller can be reused;
- useful in reconfigurable FTC.

Disadvantages:

- needs sufficient redundancy;
- reconstruction errors affect closed-loop performance.

## 5.17 Switching Transient Problem

When switching controllers:

```math
u_1=C_{c1}x_{c1}+D_{c1}y
```

```math
u_2=C_{c2}x_{c2}+D_{c2}y
```

Controller internal states may not be consistent.

Direct switching can cause input jump:

```math
\Delta u=u_2-u_1
```

Consequences:

- actuator shock;
- saturation;
- excitation of unstable dynamics;
- performance degradation.

Thus transient elimination or bumpless transfer is needed.

## 5.18 Bumpless Transfer Principle

Core idea:

- inactive controllers keep tracking the active controller output;
- when switched in, their outputs are already close.

For controller `i`:

```math
\dot x_{ci}=A_{ci}x_{ci}+B_{ci}y_{ref}+E_{ci}y+L_i(u-u_i)
```

```math
u_i=C_{ci}x_{ci}+D_{ci}y_{ref}+F_{ci}y
```

Tracking error:

```math
e_i=u-u_i
```

Choose `L_i` so that `e_i` converges to zero.

Advantages:

- reduces input jump;
- allows smoother controller switching;
- protects actuators.

Disadvantages:

- extra dynamics and tuning;
- noise may be amplified if tracking is too fast.

## 5.19 Bumpless Transfer Stability

Inactive controller tracking dynamics depend on:

```math
A_{ci}-L_iC_{ci}
```

Stability requirement:

```math
A_{ci}-L_iC_{ci}\text{ is Hurwitz}
```

If `D_ci` is invertible, one special choice may decouple reference tracking:

```math
L_i=B_{ci}D_{ci}^{-1}
```

Otherwise:

- use pole placement;
- choose tracking speed based on noise and switching requirements.

## 5.20 Bumpless Transfer Tuning

Fast tracking poles:

- inactive controller output follows active output closely;
- smaller bump at switching;
- more noise amplification.

Slow tracking poles:

- smoother internal dynamics;
- less noise amplification;
- inactive controller may not be ready at switching.

Engineering compromise:

- tracking should be fast enough for expected fault switching;
- but not so fast that measurement noise dominates.

## 5.21 Multi-Controller and Soft Switching

Controller bank:

```text
controller 1: nominal
controller 2: actuator fault mode
controller 3: sensor fault mode
...
```

Hard switching:

```math
u=u_i
```

Soft switching:

```math
u=\sum_i\omega_i u_i
```

where:

```math
\sum_i\omega_i=1,\qquad \omega_i\ge0
```

Advantages of soft switching:

- smoother transition;
- can handle diagnosis uncertainty.

Disadvantages:

- stability analysis is harder;
- mixing controllers may violate constraints or degrade performance.

---

# Topic 6: Safe and Secure Control for CPS

## 6.1 Topic Boundary

Topic 6 studies safe and secure control for cyber-physical systems under faults, attacks, anomalies, uncertainty, and hard constraints.

Curriculum subitems:

- Attack/Anomaly Detection: Sections 6.2 and 6.20-6.28.
- Resilient Control for CPS: Sections 6.16-6.19.
- Safe Control Synthesis: Sections 6.3-6.15.

Boundary:

- Topic 5 covers general FTC reconfiguration strategy.
- Topic 6 emphasizes constraints, safety envelopes, CPS attacks, and resilient MPC.

## 6.2 Fault, Attack, and Anomaly in CPS

Fault:

- natural component degradation or malfunction;
- examples: sensor bias, actuator loss, friction increase.

Attack:

- malicious or intentional action;
- examples: false data injection, actuator command manipulation, replay attack, denial of service.

Anomaly:

- observed behavior that deviates from normal patterns;
- cause may be fault, attack, disturbance, or operating change.

Topic 6 is not only about alarm generation. The control objective is:

```text
Even under anomaly, keep states and inputs inside safe constraints.
```

## 6.2.1 Three Routes for Attack/Anomaly Detection

Model-based attack/anomaly detection:

```math
r_k=y_k-\hat y_k
```

Alarm:

```math
\|r_k\|>J_{th}
```

Use cases:

- false data injection;
- actuator manipulation;
- replay attacks detectable through model inconsistency.

Advantages:

- physically interpretable;
- can integrate with observer and controller.

Disadvantages:

- model mismatch causes false alarms;
- stealthy attacks may be designed to hide in residual space.

Data-driven anomaly detection:

```math
p(x_k|\text{normal})<\epsilon
```

or anomaly score:

```math
a(x_k)>h
```

Methods:

- PCA;
- kNN density;
- Parzen density;
- autoencoder;
- VAE.

Advantages:

- can detect unknown fault/attack patterns;
- needs mainly normal data.

Disadvantages:

- difficult to isolate cause;
- distribution shift can produce false alarms.

Sequential/change detection:

```math
g(k)=\max(0,g(k-1)+s(x_k))
```

Use cases:

- small persistent attack;
- slow sensor bias;
- stealthy drift.

## 6.2.2 Difference and Overlap Between Attack, Fault, and Disturbance

Disturbance:

- usually unintentional;
- handled by robustness, disturbance rejection, or offset-free control.

Fault:

- component behavior changes;
- handled by diagnosis, isolation, and accommodation.

Attack:

- data or actuation may be intentionally manipulated;
- requires trust assessment, secure estimation, isolation of compromised channels, and safe fallback.

Overlap:

- measured anomaly may look similar for all three;
- the same residual may detect them, but control response should differ.

## 6.3 Why LQR Is Not Enough

LQR assumes:

- no hard state constraints;
- no hard input constraints;
- nominal model;
- full and trustworthy state estimate.

Real CPS has constraints:

```math
x_k\in\mathcal X
```

```math
u_k\in\mathcal U
```

Examples:

- voltage limits;
- torque limits;
- pressure limits;
- temperature limits;
- safety distance constraints.

Therefore safe control synthesis needs constrained optimization, especially MPC.

## 6.4 Constrained Finite-Horizon LQ Problem

Problem:

```math
\min_{x_0,\ldots,x_N,u_0,\ldots,u_{N-1}}
\sum_{k=0}^{N-1}(x_k^TQx_k+u_k^TRu_k)+x_N^TSx_N
```

subject to:

```math
x_{k+1}=Ax_k+Bu_k
```

```math
x_k\in X_k,\qquad u_k\in U_k
```

and given initial state:

```math
x_0=\bar x
```

With constraints, standard Riccati solution is not enough. The problem becomes a quadratic program.

## 6.5 Open-Loop Optimal Control

Open-loop solution computes:

```math
U^*=[u_0^*,u_1^*,\ldots,u_{N-1}^*]
```

and applies the whole sequence.

Problem:

- disturbances occur;
- model mismatch exists;
- state estimate changes;
- attacks or faults may happen.

Thus open-loop plan becomes outdated.

## 6.6 Receding Horizon Control / MPC

MPC procedure:

```text
measure or estimate current state
-> solve finite-horizon constrained optimization
-> apply only first input
-> move horizon forward
-> repeat
```

Control law:

```math
u_t=u_{0|t}^*
```

Advantages:

- handles constraints explicitly;
- uses feedback through repeated re-optimization;
- can include references, disturbances, and fault estimates.

Disadvantages:

- online optimization cost;
- feasibility must be maintained;
- stability requires terminal design or sufficiently long horizon.

## 6.7 State Prediction Stacking

System:

```math
x_{k+1}=Ax_k+Bu_k
```

Predictions:

```math
x_1=Ax_0+Bu_0
```

```math
x_2=A^2x_0+ABu_0+Bu_1
```

Stacked form:

```math
X=\mathcal A x_0+\mathcal B U
```

where:

```math
X=[x_1^T,\ldots,x_N^T]^T
```

```math
U=[u_0^T,\ldots,u_{N-1}^T]^T
```

This converts dynamic constraints into a compact algebraic prediction.

## 6.8 MPC Cost to QP

Stacked cost:

```math
J=X^T\bar QX+U^T\bar RU
```

Substitute:

```math
X=\mathcal A x_0+\mathcal B U
```

Then:

```math
J=U^T(\mathcal B^T\bar Q\mathcal B+\bar R)U
+2x_0^T\mathcal A^T\bar Q\mathcal B U+\text{const}
```

Standard QP:

```math
\min_U \frac12U^THU+f(x_0)^TU
```

subject to:

```math
GU\le W(x_0)
```

where:

```math
H=2(\mathcal B^T\bar Q\mathcal B+\bar R)
```

`H` can often be precomputed for fixed model and horizon.

## 6.9 Constraint Translation

State constraints:

```math
X\in\mathcal X
```

Input constraints:

```math
U\in\mathcal U
```

Using stacked prediction:

```math
G_x(\mathcal A x_0+\mathcal B U)\le h_x
```

which becomes:

```math
G_x\mathcal B U\le h_x-G_x\mathcal A x_0
```

Input constraints:

```math
G_uU\le h_u
```

Together:

```math
GU\le W(x_0)
```

This is the core of constrained safe synthesis.

## 6.10 State Elimination Pros and Cons

State elimination means optimizing only over `U` after substituting predicted states.

Advantages:

- fewer decision variables;
- simple QP form;
- easy to implement for small and medium horizons.

Disadvantages:

- Hessian can become dense;
- numerical conditioning may worsen for unstable `A`;
- long horizon increases cost.

Alternative:

- keep both states and inputs as decision variables;
- exploit sparse dynamics constraints.

## 6.11 MPC Stability Problem

Finite-horizon MPC can be short-sighted.

Without terminal ingredients:

- optimizer may choose actions that look good within horizon;
- behavior after horizon may be unstable;
- recursive feasibility may fail.

Common stability tools:

- terminal equality constraint;
- terminal set;
- terminal cost;
- local terminal controller.

## 6.12 Terminal Equality

Terminal equality:

```math
x_{t+N|t}=0
```

Advantages:

- simple stability proof;
- forces trajectory to reach equilibrium.

Disadvantages:

- often infeasible for short horizons;
- may require aggressive control;
- not suitable for nonzero reference tracking unless shifted coordinates are used.

## 6.13 Terminal Set and Terminal Cost

Terminal constraint:

```math
x_{t+N|t}\in X_f
```

Terminal cost:

```math
V_f(x)=x^TPx
```

Local controller:

```math
u=K_fx
```

Requirements:

Positive invariance:

```math
x\in X_f\Rightarrow (A+BK_f)x\in X_f
```

Constraint satisfaction:

```math
x\in X_f\Rightarrow K_fx\in U
```

Lyapunov decrease:

```math
(A+BK_f)^TP(A+BK_f)-P+Q+K_f^TRK_f\le0
```

Benefits:

- recursive feasibility;
- asymptotic stability;
- less conservative than terminal equality.

## 6.14 Reference Tracking MPC

For nonzero reference, compute steady-state pair `(x_s,u_s)`.

Equilibrium:

```math
x_s=Ax_s+Bu_s
```

Output target:

```math
y_s=Cx_s=y_{spec}
```

Linear system:

```math
\begin{bmatrix}I-A&-B\\C&0\end{bmatrix}
\begin{bmatrix}x_s\\u_s\end{bmatrix}
=
\begin{bmatrix}0\\y_{spec}\end{bmatrix}
```

Use shifted variables:

```math
\tilde x=x-x_s
```

```math
\tilde u=u-u_s
```

Then regulate shifted system to zero.

## 6.15 Target Selector

If the steady-state equations have multiple solutions or constraints exist, solve a target selection QP:

```math
\min_{x_s,u_s}\|x_s-x_{spec}\|_{Q_s}^2+\|u_s-u_{spec}\|_{R_s}^2
```

subject to:

```math
x_s=Ax_s+Bu_s
```

```math
C x_s=y_{spec}
```

```math
x_s\in X,\qquad u_s\in U
```

Use cases:

- overactuated systems;
- constrained reference tracking;
- offset-free MPC with disturbance estimate.

## 6.16 Offset-Free MPC

To remove steady-state offset, augment constant disturbance:

```math
x_{k+1}=Ax_k+Bu_k+B_dd_k
```

```math
d_{k+1}=d_k
```

```math
y_k=Cx_k+C_dd_k
```

Observer estimates:

```math
\hat x_k,\qquad \hat d_k
```

Target selector becomes:

```math
\begin{bmatrix}I-A&-B\\C&0\end{bmatrix}
\begin{bmatrix}x_s\\u_s\end{bmatrix}
=
\begin{bmatrix}B_d\hat d_k\\y_{spec}-C_d\hat d_k\end{bmatrix}
```

Advantages:

- removes steady-state tracking error caused by constant disturbances or model mismatch;
- integrates naturally with MPC.

Disadvantages:

- augmented observability is required;
- disturbance model must match actual offset source;
- estimator tuning affects performance.

## 6.17 LESO + MPC AFTC

Faulty system example:

```math
x_{k+1}=Ax_k+B(u_k+f_{a,k})
```

```math
y_k=Cx_k+C_{fs}f_{s,k}
```

LESO estimates:

```math
\hat x_k,\qquad \hat f_{a,k},\qquad \hat f_{s,k}
```

Sensor fault:

- measurement is corrupted;
- use cleaned state estimate `hat x_k` instead of raw measurement.

Actuator fault:

- physical actuation is changed;
- target selector compensates for estimated actuator fault.

Steady-state condition with actuator fault:

```math
(I-A)x_s-Bu_s=B\hat f_a
```

Logic:

```text
LESO estimates fault
-> MPC uses cleaned state
-> target selector compensates actuator offset
-> constrained MPC keeps safety
```

Advantages:

- combines estimation and constrained control;
- handles simultaneous sensor and actuator faults.

Disadvantages:

- LESO fault estimate may lag;
- high gain may amplify noise;
- severe actuator loss may still make constraints infeasible.

## 6.18 UIO + MPC AFTC

UIO-based resilient MPC uses UIO to prevent actuator fault from corrupting the state estimate.

Augmented model may include actuator fault as unknown input:

```math
x_{k+1}=Ax_k+Bu_k+Ef_k
```

UIO design:

```math
TE=0
```

or continuous-time equivalent:

```math
(I-HC)E=0
```

Then:

- state estimate is decoupled from actuator fault;
- fault can be reconstructed algebraically if conditions hold;
- MPC uses clean state and estimated fault for target correction.

LESO vs UIO:

- LESO dynamically estimates fault and is useful for slow variations;
- UIO structurally decouples unknown input and can give faster clean estimation;
- UIO has stricter rank/structure conditions.

## 6.19 Safe/Resilient Control Logic

Safe resilient CPS control integrates:

```text
anomaly/fault/attack detection
-> secure or resilient state estimation
-> fault/attack isolation
-> constrained target selection
-> safe MPC
```

Control priorities:

1. maintain safety constraints;
2. maintain stability;
3. maintain tracking or performance;
4. recover nominal operation if possible.

If diagnosis is uncertain:

- use conservative constraints;
- reduce reference demand;
- switch to safe mode;
- isolate suspected channels;
- use redundancy.

## 6.20 VAE Generative Anomaly Detection

VAE is trained on healthy data to learn the normal data distribution or manifold.

Online anomaly logic:

```text
new measurement x
-> encoder
-> latent z
-> decoder reconstruction
-> anomaly score
```

Anomaly if:

- reconstruction error is large;
- latent distribution deviates from prior;
- likelihood or ELBO is low.

Use cases:

- unknown fault detection;
- cyber-physical anomaly detection;
- high-dimensional sensor data with nonlinear normal structure.

## 6.21 Autoencoder and PCA Relation

Autoencoder:

```math
z=f_\phi(x)
```

```math
\hat x=g_\theta(z)
```

Training objective:

```math
\min \|x-\hat x\|^2
```

Linear autoencoder with squared loss is closely related to PCA.

Standard autoencoder limitations:

- latent space may have holes;
- reconstruction can be poor for some normal points;
- anomaly threshold may be unstable;
- no explicit probabilistic model.

VAE addresses this by regularizing latent distribution.

## 6.22 VAE Probabilistic Setup

Latent prior:

```math
p(z)=N(0,I)
```

Decoder likelihood:

```math
p_\theta(x|z)
```

Generative model:

```math
p_\theta(x,z)=p_\theta(x|z)p(z)
```

Posterior:

```math
p_\theta(z|x)=\frac{p_\theta(x|z)p(z)}{p_\theta(x)}
```

But:

```math
p_\theta(x)=\int p_\theta(x|z)p(z)dz
```

is usually intractable.

Use encoder approximation:

```math
q_\phi(z|x)\approx p_\theta(z|x)
```

Typical encoder output:

```math
q_\phi(z|x)=N(\mu_\phi(x),diag(\sigma_\phi^2(x)))
```

## 6.23 KL Divergence

KL divergence:

```math
D_{KL}(q\|p)=\int q(z)\ln\frac{q(z)}{p(z)}dz
```

Properties:

```math
D_{KL}(q\|p)\ge0
```

and:

```math
D_{KL}(q\|p)=0\iff q=p
```

But KL is not symmetric:

```math
D_{KL}(q\|p)\ne D_{KL}(p\|q)
```

In VAE:

- KL term pulls approximate posterior toward the prior;
- this makes latent space continuous and complete.

## 6.24 ELBO Derivation

Start from:

```math
\log p_\theta(x)
```

Insert `q_phi(z|x)`:

```math
\log p_\theta(x)=
E_{q_\phi}\left[\log p_\theta(x)\right]
```

Use:

```math
p_\theta(x)=\frac{p_\theta(x,z)}{p_\theta(z|x)}
```

Then:

```math
\log p_\theta(x)
=E_{q_\phi}\left[
\log\frac{p_\theta(x,z)}{q_\phi(z|x)}
\right]
+D_{KL}(q_\phi(z|x)\|p_\theta(z|x))
```

Therefore:

```math
\log p_\theta(x)=ELBO+D_{KL}(q_\phi(z|x)\|p_\theta(z|x))
```

Since KL is nonnegative:

```math
ELBO\le \log p_\theta(x)
```

ELBO:

```math
L(\theta,\phi;x)
=E_{q_\phi(z|x)}[\log p_\theta(x|z)]
-D_{KL}(q_\phi(z|x)\|p(z))
```

Interpretation:

- reconstruction term: fit data;
- KL term: regularize latent posterior toward prior.

## 6.25 Reparameterization Trick

Sampling:

```math
z\sim N(\mu,\sigma^2)
```

is rewritten as:

```math
z=\mu+\sigma\odot\epsilon
```

where:

```math
\epsilon\sim N(0,I)
```

This separates randomness from network parameters and allows backpropagation through `mu` and `sigma`.

## 6.26 Gaussian KL Closed Form

If:

```math
q_\phi(z|x)=N(\mu,diag(\sigma^2))
```

and:

```math
p(z)=N(0,I)
```

then:

```math
D_{KL}(q_\phi(z|x)\|p(z))
=\frac12\sum_j(\mu_j^2+\sigma_j^2-\ln\sigma_j^2-1)
```

In the ELBO loss, this term is often added as:

```math
-D_{KL}(q_\phi(z|x)\|p(z))
```

or minimized as positive KL plus reconstruction loss.

## 6.27 VAE Anomaly Score

Reconstruction error:

```math
a_1(x)=\|x-\hat x\|^2
```

Latent deviation:

```math
a_2(x)=\|\mu_\phi(x)\|^2
```

Negative ELBO:

```math
a_3(x)=-ELBO(x)
```

Combined score:

```math
a(x)=\alpha\|x-\hat x\|^2+\beta\|\mu_\phi(x)\|^2
```

Alarm:

```math
a(x)>h
```

Threshold can be chosen from healthy validation data, e.g. high percentile.

## 6.28 VAE Advantages and Disadvantages

Advantages:

- learns nonlinear normal manifold;
- can detect unknown anomalies without fault labels;
- probabilistic latent space is more regular than standard autoencoder;
- useful for high-dimensional CPS sensor data.

Disadvantages:

- training is more complex than PCA;
- posterior collapse may occur;
- reconstruction can be overly smooth;
- threshold choice still matters;
- explanation and isolation are harder than model-based residuals.

Use cases:

- high-dimensional sensor anomaly detection;
- complex nonlinear normal operation;
- unknown cyber-physical attacks or faults;
- early warning before explicit fault labels exist.

---

# Appendix A: Boundary Summary for the Six Topics

## Topic 1 vs Topic 2

Topic 1: statistical detection and classification using features, PDFs, Bayes, LDA, logistic regression.

Topic 2: how to construct lower-dimensional representations such as PCA, FDA, MDS, and Isomap.

## Topic 1 vs Topic 3

Topic 1: probabilistic or statistical classifiers.

Topic 3: maximum-margin and geometric classifiers, especially SVM and perceptron.

## Topic 1 vs Topic 4

Topic 1: data/signal-driven detection.

Topic 4: model-based residuals, observers, parity, and sequential detection.

## Topic 2 vs Topic 3

Topic 2: representation learning or projection.

Topic 3: classification boundary after features are available.

## Topic 4 vs Topic 5

Topic 4 stops at diagnosis:

```text
residual -> observer -> CUSUM -> alarm/fault information
```

Topic 5 uses the diagnosis result:

```text
fault information -> controller reconfiguration
```

## Topic 5 vs Topic 6

Topic 5: general FTC strategy, model matching, virtual actuator/sensor, and bumpless switching.

Topic 6: CPS safety/security, constraints, resilient MPC, attack/anomaly detection, and safe synthesis.

---

# Appendix B: Oral Exam Answer Template

For any method, answer in this order:

1. Define the method.
2. Explain what problem it solves in FTC or FDD.
3. State assumptions.
4. Give the key formula.
5. Explain the engineering meaning of the formula.
6. Discuss advantages.
7. Discuss disadvantages.
8. Give suitable use cases.
9. State boundary with other topics.
10. Finish with a fault detection or control reconfiguration example.

---

# Appendix C: Minimal Must-Know Checklist

Topic 1:

- FAR, MAR, sensitivity, specificity.
- Bayes theorem, MAP, LRT, risk-based decision.
- MLE vs MAP.
- Histogram, Parzen, kNN.
- LDA, QDA, logistic regression assumptions and boundary shapes.

Topic 2:

- PCA normal subspace and residual subspace.
- SPE/Q and Hotelling `T^2`.
- KPCA kernel trick.
- SPCA/FDA supervised fault-sensitive projection.
- MDS preserves distances; Isomap uses geodesic distances.

Topic 3:

- Hard-margin primal and dual.
- Soft-margin slack variables and `C`.
- KKT and support vectors.
- Kernel trick and common kernels.
- Perceptron vs SVM.

Topic 4:

- Residual generation.
- Observer-based methods: UIO, KF, ESO, NDO.
- UIO rank condition and decoupling.
- CUSUM recursive formula.
- Gaussian CUSUM increment.
- ARL threshold trade-off.

Topic 5:

- Passive vs active FTC.
- Redundancy.
- Model matching equation.
- Exact vs approximate model matching.
- Sensor reconfiguration kernel condition.
- Virtual actuator and virtual sensor.
- Bumpless transfer tracking law.

Topic 6:

- Constraints and why LQR is insufficient.
- MPC QP formulation.
- Terminal equality, terminal set, and terminal cost.
- Reference tracking and target selector.
- Offset-free MPC.
- LESO/UIO plus MPC AFTC.
- VAE ELBO and anomaly score.
