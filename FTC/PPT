# FTC Topics 2-6 PPT Framework Templates

## Overall Deck Rule

Each topic deck should follow the same presentation logic:

```text
Big Question
-> Topic Boundary
-> Core Problem
-> Main Method
-> Key Formula / Algorithm
-> Engineering Meaning
-> Method Comparison
-> Limitations
-> Final Knowledge Map
```

Main ideas should be shown on slides. Secondary derivations, parameter details, implementation notes, and edge cases should be handled orally or in Q&A.

Recommended length for each topic: **12-15 slides**.

---

# Topic 2: Dimensionality Reduction Approaches

## Core Story

```text
High-dimensional sensor data
-> low-dimensional representation
-> fault-sensitive subspace
-> monitoring / visualization / isolation
```

Topic 2 is about **representation**, not classifier design. Classification after projection belongs mainly to Topic 1 or Topic 3.

## Slide Template

| Slide | Title | Main Display | Oral / Prompt Only |
|---|---|---|---|
| 1 | Big Question | How can high-dimensional sensor data be represented for FDD? | Sensor data are high-dimensional, correlated, noisy. |
| 2 | Topic Boundary | Representation, projection, subspace, manifold | Classifier design is not the main topic. |
| 3 | Why Dimensionality Reduction? | High dimension -> redundancy, noise, visualization difficulty | Curse of dimensionality. |
| 4 | PCA Core Idea | Find directions of maximum variance | PCA learns normal operating variation. |
| 5 | PCA Projection | `X -> scores T = X P` | Eigenvectors and covariance details can be oral. |
| 6 | Normal Subspace vs Residual Subspace | Principal subspace / residual subspace diagram | Faults may appear as residual energy. |
| 7 | PCA Monitoring Statistics | `T^2` and `Q / SPE` | `T^2` detects abnormal scores; `Q` detects residual abnormality. |
| 8 | Choosing Number of PCs | CPV / scree plot | Threshold selection details oral. |
| 9 | PCA Variants | Direct PCA, dual PCA, kernel PCA | KPCA handles nonlinear structure. |
| 10 | FDA | Maximize between-class separation, minimize within-class spread | FDA needs labels. |
| 11 | MDS | Preserve pairwise distances | Good for global distance structure. |
| 12 | Isomap | Preserve geodesic distances on manifold | k-neighbor graph sensitivity oral. |
| 13 | Method Comparison | PCA / KPCA / FDA / MDS / Isomap table | Strength and limitation of each. |
| 14 | Final Knowledge Map | Data -> representation -> monitoring/classification | Low-dimensional representation supports later FDD. |

## Must Show

- PCA goal and projection.
- Normal subspace vs residual subspace.
- `T^2` and `Q/SPE`.
- FDA as supervised dimensionality reduction.
- MDS vs Isomap distinction.
- Method comparison table.

## Oral Only

- Full eigenvalue derivation.
- Kernel PCA details.
- Isomap graph construction proof.
- Exact threshold formulas.

---

# Topic 3: Support Vector Machines and Perceptron

## Core Story

```text
Feature vectors
-> separating hyperplane
-> maximum margin
-> soft margin for noisy data
-> kernel trick for nonlinear faults
-> perceptron as basic linear learner
```

Topic 3 is about **classification boundary geometry**.

## Slide Template

| Slide | Title | Main Display | Oral / Prompt Only |
|---|---|---|---|
| 1 | Big Question | How can normal and fault data be separated with maximum margin? | SVM is a geometric classifier. |
| 2 | SVM in FDD Pipeline | Features -> SVM -> normal/fault decision | Features may come from Topic 1 or 2. |
| 3 | Hyperplane | `w^T x + b = 0` | Sign gives class decision. |
| 4 | Margin | Margin diagram and `2 / ||w||` | Larger margin improves generalization. |
| 5 | Support Vectors | Points on margin boundaries | Only critical samples define boundary. |
| 6 | Hard-Margin SVM | `min 1/2||w||^2` subject to class constraints | Linearly separable assumption. |
| 7 | Why Hard Margin Fails | Overlap, noise, mislabeled data | Real FDD data are rarely perfectly separable. |
| 8 | Soft Margin | Slack variable `xi` | Violations become allowed but penalized. |
| 9 | Role of C | High C vs low C comparison | High C fits data more; low C allows wider margin. |
| 10 | Dual and KKT Intuition | Support vectors have nonzero multipliers | Full derivation oral. |
| 11 | Kernel Trick | `K(x_i,x_j)=phi(x_i)^T phi(x_j)` | Avoid explicit high-dimensional mapping. |
| 12 | RBF Kernel | Nonlinear fault boundary | Gamma controls locality. |
| 13 | Perceptron | Misclassification update rule | Converges only if linearly separable. |
| 14 | Method Summary | Hard SVM / Soft SVM / Kernel SVM / Perceptron | When to use each. |

## Must Show

- Hyperplane and margin.
- Hard-margin objective.
- Soft-margin slack variables.
- Role of `C`.
- Kernel trick.
- Perceptron update intuition.

## Oral Only

- Full dual derivation.
- KKT proof.
- Perceptron convergence proof.
- Detailed kernel parameter tuning.

---

# Topic 4: Model-Based Fault Detection

## Core Story

```text
System model
-> expected output/state
-> residual
-> threshold or statistical decision
-> fault detection / isolation
```

Topic 4 differs from Topics 1-3 because it starts from a **dynamic model**, not only data distribution.

## Slide Template

| Slide | Title | Main Display | Oral / Prompt Only |
|---|---|---|---|
| 1 | Big Question | How can a model reveal a fault? | Model gives expected behavior. |
| 2 | Model-Based FDD Architecture | Input -> plant/model/observer -> residual -> decision | Distinguish from data-driven FDD. |
| 3 | Fault Locations | Sensor, actuator, plant/component fault | Link to isolation. |
| 4 | Fault Types | Bias, drift, abrupt, incipient, intermittent | Examples oral. |
| 5 | Residual Generation | `r = y - y_hat` | Healthy residual small; faulty residual large. |
| 6 | Residual Evaluation | Residual statistic vs threshold | Threshold trade-off links back to Topic 1. |
| 7 | Parity Method: Time Domain | Consistency equation | State elimination details oral. |
| 8 | Parity Method: Frequency Domain | Transfer-function consistency | Frequency-domain implementation oral. |
| 9 | Observer-Based Residual | Luenberger observer diagram | Estimate state, compare output. |
| 10 | Kalman Filter Residual | Innovation sequence | For noisy systems. |
| 11 | ESO and NDO | Estimate disturbance/fault effect | Robust estimation idea. |
| 12 | UIO | Decouple unknown inputs | Main limitation: existence conditions. |
| 13 | CUSUM and ARL | Sequential detection statistic | Detect small persistent changes. |
| 14 | Method Selection Summary | Parity / Observer / KF / UIO / CUSUM table | Strengths and limitations. |

## Must Show

- Residual concept.
- Fault locations.
- Parity method idea.
- Observer-based residual.
- Kalman innovation.
- UIO unknown input decoupling.
- CUSUM and ARL.

## Oral Only

- Full parity derivation.
- Kalman covariance recursion.
- ESO/NDO gain design.
- UIO solvability proof.
- ARL derivation.

---

# Topic 5: Fault Tolerant Control Strategy

## Core Story

```text
Fault diagnosis
-> fault information
-> redundancy
-> controller reconfiguration
-> stability and acceptable performance
```

Topic 5 starts where Topic 4 stops. Topic 4 detects and diagnoses. Topic 5 uses that information to keep the system controlled.

## Slide Template

| Slide | Title | Main Display | Oral / Prompt Only |
|---|---|---|---|
| 1 | Big Question | What should the controller do after a fault? | Diagnosis is useful only if control can respond. |
| 2 | FTC Architecture | FDD block -> reconfigurable controller -> plant | Show information flow. |
| 3 | Fault Locations and Redundancy | Sensor / actuator / plant + redundancy | Redundancy is prerequisite. |
| 4 | Passive FTC | Fixed robust controller | No online reconfiguration. |
| 5 | Active FTC | Detect -> isolate -> estimate -> reconfigure | Depends on FDD quality. |
| 6 | Robust vs Adaptive Control | Robust handles uncertainty; adaptive updates parameters | Keep this as comparison. |
| 7 | Reconfigurable Control Systems | Switch, retune, redistribute, restructure | Main active FTC concept. |
| 8 | Control Background | State feedback / observer / separation principle | Only supporting background. |
| 9 | Model Matching | Faulty closed-loop mimics nominal model | Main reconfiguration strategy. |
| 10 | Exact vs Approximate Matching | Exact if solvable; approximate if not | Least-squares idea oral. |
| 11 | Virtual Sensor | Reconstruct faulty sensor measurement | Sensor fault accommodation. |
| 12 | Virtual Actuator | Hide actuator fault from nominal controller | Actuator fault accommodation. |
| 13 | Transient Elimination | Bumpless transfer diagram | Avoid switching shock. |
| 14 | FTC Limits and Summary | Redundancy, saturation, diagnosis delay, model mismatch | FTC has physical limits. |

## Must Show

- Passive vs active FTC.
- FTC architecture.
- Redundancy.
- Model matching.
- Virtual sensor.
- Virtual actuator.
- Bumpless transfer.
- Physical limits.

## Oral Only

- Full state-space background.
- Exact model matching algebra.
- Least-squares implementation.
- Observer design details.
- Detailed transient elimination equations.

---

# Topic 6: Safe/Secure Control for CPS

## Core Story

```text
Faults / attacks / anomalies
-> detection and diagnosis
-> safety constraints
-> resilient MPC / safe synthesis
-> safe operation under uncertainty
```

Topic 6 is the endpoint of the course: it combines diagnosis, constraints, attacks, and resilient control.

## Slide Template

| Slide | Title | Main Display | Oral / Prompt Only |
|---|---|---|---|
| 1 | Big Question | How can CPS remain safe under faults and attacks? | Safety has priority over tracking. |
| 2 | CPS Faults, Attacks, Anomalies | Sensor attack, actuator attack, communication attack | Distinguish fault vs attack vs anomaly. |
| 3 | Safe and Resilient Control Chain | Detect -> diagnose -> isolate -> control action | Connect to whole course. |
| 4 | From LQR to Constrained Control | LQR assumes no hard constraints | Real systems have limits. |
| 5 | Finite-Horizon Optimization | Cost + dynamics + constraints | MPC foundation. |
| 6 | MPC and Receding Horizon | Solve -> apply first input -> repeat | Show loop diagram. |
| 7 | QP Form | State elimination / quadratic program | Implementation detail oral. |
| 8 | MPC Stability Issue | Finite horizon may be unstable | Need terminal ingredients. |
| 9 | Terminal Cost and Terminal Set | Stability and recursive feasibility | Key safe MPC idea. |
| 10 | Reference Tracking | Feasible target selection | Avoid impossible references. |
| 11 | Offset-Free MPC | Remove steady-state error | Disturbance model oral. |
| 12 | LESO / UIO + MPC | Estimate unknown input, then compensate | Resilient control structure. |
| 13 | VAE Anomaly Detection | Reconstruction / latent probability anomaly score | Generative anomaly detection. |
| 14 | Final Safety Map | Attack detection + resilient MPC + constraints | Safety envelope is final objective. |

## Must Show

- CPS fault/attack/anomaly distinction.
- MPC receding horizon.
- State/input constraints.
- Terminal cost and terminal set.
- Offset-free MPC.
- LESO/UIO plus MPC.
- VAE anomaly detection as supporting detection.

## Oral Only

- Full QP derivation.
- Riccati equation details.
- Terminal set computation.
- VAE ELBO derivation.
- Reparameterization trick.
- Detailed cybersecurity taxonomy.

---

# Cross-Topic Consistency Map

Use this map at the end of each deck, with the current topic highlighted:

```text
Topic 1: Statistical decision from signal data
Topic 2: Low-dimensional fault-sensitive representation
Topic 3: Maximum-margin classification
Topic 4: Model-based residual generation and diagnosis
Topic 5: Fault accommodation and controller reconfiguration
Topic 6: Safe and resilient CPS control
```

Full course chain:

```text
Sensor data / system model
-> features, representations, or residuals
-> detection and diagnosis
-> classification or sequential decision
-> controller reconfiguration
-> safe and resilient operation
```

