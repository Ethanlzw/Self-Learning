# FTC Topics 2-6 Detailed PPT Blueprints

## Global Slide Design Standard

For each slide:

- **Title:** one concept only.
- **Takeaway:** one sentence under the title.
- **Main visual:** equation, method diagram, comparison table, or lecture figure.
- **Text limit:** 3 short bullets or one compact table.
- **Oral cue:** details that are important but should not clutter the slide.

Recommended visual hierarchy:

```text
Title
Takeaway
Main formula / diagram / figure
3 bullets maximum
Small source footer
```

---

# Topic 2: Dimensionality Reduction Approaches

## Topic Spine

```text
High-dimensional sensor data
-> projection / embedding
-> normal subspace and fault-sensitive residual space
-> monitoring statistics or visualization
-> method selection: PCA, KPCA, FDA, MDS, Isomap
```

Topic 2 is about **representation learning for FDD**. It should not become a classifier lecture.

## Slide 1: Title and Big Question

**Takeaway:** Dimensionality reduction converts many correlated sensor signals into a smaller fault-sensitive representation.

**On-slide content:**

```text
Big question:
How can high-dimensional sensor data be compressed without losing fault information?

Course position:
Topic 1: probability decision
Topic 2: representation / projection
Topic 3: classifier boundary
```

**Visual:** pipeline diagram:

```text
Raw sensor data X
-> lower-dimensional representation z
-> monitoring / visualization / classification
```

**Oral cue:** emphasize that Topic 2 prepares better inputs for later detection and classification.

## Slide 2: Why FDD Needs Dimensionality Reduction

**Takeaway:** Faults often hide in correlated high-dimensional variation, so raw sensor space is hard to interpret.

**On-slide content:**

```text
Problems in raw sensor space:
- many sensors and strong correlation
- noise and redundant variables
- difficult visualization and isolation
- high-dimensional distance becomes unreliable
```

**Visual:** left: high-dimensional sensor matrix; right: 2D/3D embedded clusters.

**Oral cue:** mention that dimensionality reduction is not only for plotting; it also improves monitoring statistics and robustness.

## Slide 3: PCA Core Idea

**Takeaway:** PCA finds directions that preserve maximum variance in normal operating data.

**On-slide formula:**

```text
S = (1/(n-1)) X^T X

S p_i = lambda_i p_i

z = P_p^T x
```

**On-slide explanation:**

```text
P_p: first p principal directions
z: low-dimensional score vector
lambda_i: variance explained by direction p_i
```

**Visual:** ellipse with PC1 and PC2 axes.

**Oral cue:** PCA is normally trained on fault-free data, so the principal subspace represents normal process variation.

## Slide 4: Normal Subspace and Residual Subspace

**Takeaway:** PCA separates normal variation from residual variation where faults may appear.

**On-slide formula:**

```text
Projection:
z = P_p^T x

Reconstruction:
x_hat = P_p z = P_p P_p^T x

Residual:
e = x - x_hat
```

**Visual:** geometric diagram with `x`, `x_hat`, principal subspace, residual vector.

**On-slide bullets:**

```text
- principal subspace: dominant normal variation
- residual subspace: variation not explained by PCA
- faults may increase residual energy
```

**Oral cue:** this slide is the bridge from dimensionality reduction to fault monitoring.

## Slide 5: PCA Monitoring Statistics

**Takeaway:** PCA-based FDD usually monitors both score abnormality and residual abnormality.

**On-slide formula:**

```text
Hotelling statistic:
T^2 = z^T Lambda_p^{-1} z

Residual statistic:
Q = SPE = ||x - x_hat||^2
```

**On-slide comparison table:**

| Statistic | Detects | Space |
|---|---|---|
| `T^2` | abnormal but modeled variation | principal subspace |
| `Q / SPE` | unmodeled variation | residual subspace |

**Visual:** two gauges or two threshold lines: `T^2 > h_T`, `Q > h_Q`.

**Oral cue:** exact threshold formulas can be left for oral explanation.

## Slide 6: Choosing the Number of Principal Components

**Takeaway:** Too few PCs lose normal information; too many PCs hide faults inside the model.

**On-slide formula:**

```text
CPV(p) = (sum_{i=1}^p lambda_i) / (sum_{i=1}^d lambda_i)
```

**On-slide bullets:**

```text
- scree plot: look for elbow
- CPV: retain enough normal variance
- FDD view: keep normal dynamics, leave fault energy visible
```

**Visual:** scree plot or cumulative variance plot.

**Oral cue:** explain that maximum CPV is not always best for fault detection.

## Slide 7: Kernel PCA

**Takeaway:** Kernel PCA applies PCA logic in a nonlinear feature space without explicitly computing that space.

**On-slide formula:**

```text
phi: x -> phi(x)

K_ij = k(x_i, x_j) = phi(x_i)^T phi(x_j)

center K, then eigendecompose K
```

**On-slide bullets:**

```text
- handles nonlinear normal operating regions
- projection depends on kernel similarity to training data
- kernel choice and parameters dominate performance
```

**Visual:** nonlinear curved manifold unfolded into a separable embedding.

**Oral cue:** no need to derive centered kernel PCA; emphasize why kernel trick is useful.

## Slide 8: Supervised Dimensionality Reduction and FDA

**Takeaway:** FDA uses labels to find directions that separate fault classes.

**On-slide formula:**

```text
J(w) = (w^T S_B w) / (w^T S_W w)

Goal:
maximize between-class scatter
minimize within-class scatter
```

**On-slide bullets:**

```text
- PCA ignores labels
- FDA uses normal/fault labels
- useful for fault isolation when classes are known
```

**Visual:** PCA direction vs FDA direction on two overlapping classes.

**Oral cue:** FDA can overfit if labels are few or class covariance assumptions are poor.

## Slide 9: MDS

**Takeaway:** MDS preserves pairwise distances when mapping high-dimensional data to low dimension.

**On-slide content:**

```text
Input:
distance matrix D = [d_ij]

Goal:
low-dimensional points z_i with
||z_i - z_j|| approximately d_ij
```

**Optional formula:**

```text
B = -1/2 J D^(2) J
```

**Visual:** distance matrix -> 2D map.

**Oral cue:** MDS is useful when the relation between samples is better expressed by distances than by original coordinates.

## Slide 10: Isomap

**Takeaway:** Isomap preserves manifold geodesic distances instead of direct Euclidean distances.

**On-slide algorithm:**

```text
1. Build k-nearest-neighbor graph
2. Estimate shortest-path geodesic distances
3. Apply MDS to geodesic distance matrix
```

**Visual:** curved manifold with local graph edges and unfolded 2D embedding.

**On-slide bullets:**

```text
- good for nonlinear manifolds
- local neighborhood size k is critical
- sensitive to disconnected graphs and noise
```

**Oral cue:** explain that Euclidean distance cuts through the manifold, while geodesic distance follows it.

## Slide 11: Method Comparison

**Takeaway:** The correct method depends on whether the structure is linear, nonlinear, supervised, or distance-based.

**On-slide table:**

| Method | Uses labels? | Preserves | Strength | Limitation |
|---|---:|---|---|---|
| PCA | No | variance | simple, interpretable | linear |
| KPCA | No | nonlinear variance | nonlinear FDD | kernel tuning |
| FDA | Yes | class separation | fault isolation | needs labels |
| MDS | No | pairwise distance | visualization | distance choice |
| Isomap | No | geodesic distance | manifold structure | sensitive to k |

**Visual:** color-coded method map.

**Oral cue:** do not rank methods universally; rank by data structure.

## Slide 12: Topic 2 Knowledge Map

**Takeaway:** Topic 2 builds the representation that later detectors and classifiers use.

**On-slide map:**

```text
Sensor matrix X
-> PCA / KPCA / FDA / MDS / Isomap
-> low-dimensional z
-> T^2, Q, visualization, or classifier input
```

**On-slide summary:**

```text
PCA: linear normal subspace
KPCA: nonlinear normal subspace
FDA: supervised class separation
MDS/Isomap: distance and manifold visualization
```

**Visual:** final flow chart with Topic 2 highlighted in the course chain.

**Oral cue:** close by connecting Topic 2 to Topic 3: after representation, we need a classifier boundary.

---

# Topic 3: Support Vector Machines and Perceptron

## Topic Spine

```text
Feature vector
-> hyperplane
-> margin
-> hard margin
-> soft margin
-> kernel trick
-> perceptron comparison
```

Topic 3 is about **geometric classification** for normal/fault data.

## Slide 1: Title and Big Question

**Takeaway:** SVM finds a decision boundary that separates classes with the largest possible margin.

**On-slide content:**

```text
Big question:
Among all possible fault/normal boundaries, which one generalizes best?
```

**Visual:** two possible separating lines; highlight maximum-margin line.

**Oral cue:** compare briefly with Logistic Regression: logistic learns probability; SVM learns margin.

## Slide 2: SVM in the FDD Pipeline

**Takeaway:** SVM is applied after signals have been converted into feature vectors.

**On-slide flow:**

```text
Raw signal
-> features / PCA scores
-> SVM classifier
-> normal or fault
```

**On-slide bullets:**

```text
- input: feature vector x
- output: class label y in {-1, +1}
- decision: sign(f(x))
```

**Visual:** FDD pipeline with SVM block emphasized.

**Oral cue:** feature scaling is important because margin depends on geometry.

## Slide 3: Hyperplane and Decision Rule

**Takeaway:** A linear SVM separates data by a hyperplane.

**On-slide formula:**

```text
f(x) = w^T x + b

Decision:
y_hat = sign(w^T x + b)
```

**On-slide explanation:**

```text
w: normal vector to the hyperplane
b: offset
```

**Visual:** 2D points with separating line and normal vector.

**Oral cue:** in high-dimensional feature space, the line becomes a hyperplane.

## Slide 4: Margin and Support Vectors

**Takeaway:** The support vectors are the points that determine the SVM boundary.

**On-slide formula:**

```text
Margin width = 2 / ||w||

Support vector condition:
y_i(w^T x_i + b) = 1
```

**On-slide bullets:**

```text
- closest points define the margin
- moving non-support points often does not change the boundary
- large margin usually improves generalization
```

**Visual:** margin bands with support vectors circled.

**Oral cue:** this is the most important geometric intuition in SVM.

## Slide 5: Hard-Margin SVM

**Takeaway:** Hard-margin SVM assumes the data are perfectly linearly separable.

**On-slide optimization:**

```text
min_{w,b}  1/2 ||w||^2

subject to:
y_i(w^T x_i + b) >= 1
```

**On-slide bullets:**

```text
- maximizes margin
- no training error allowed
- sensitive to outliers
```

**Visual:** clean separable normal/fault points.

**Oral cue:** emphasize that hard margin is the idealized case.

## Slide 6: Why Hard Margin Fails in Real Fault Data

**Takeaway:** Real sensor data contain noise, overlap, and outliers, so strict separation is often impossible.

**On-slide content:**

```text
Real FDD data:
- measurement noise
- operating-point variation
- overlapping normal and fault features
- mislabeled or transitional samples
```

**Visual:** overlapping scatter plot where no perfect line exists.

**Oral cue:** this motivates soft-margin SVM.

## Slide 7: Soft-Margin SVM

**Takeaway:** Soft-margin SVM allows margin violations but penalizes them.

**On-slide formula:**

```text
min_{w,b,xi}  1/2 ||w||^2 + C sum_i xi_i

subject to:
y_i(w^T x_i + b) >= 1 - xi_i
xi_i >= 0
```

**On-slide explanation:**

```text
xi_i: violation amount
C: penalty for violations
```

**Visual:** points inside margin and misclassified points labeled by `xi`.

**Oral cue:** link `C` to FAR/MAR trade-off in FDD.

## Slide 8: Role of C

**Takeaway:** `C` controls the trade-off between a wide margin and training errors.

**On-slide comparison:**

| `C` | Boundary | Behavior |
|---|---|---|
| small `C` | wider margin | allows more violations |
| large `C` | narrower margin | fits training data more strictly |

**Visual:** side-by-side boundaries for low `C` and high `C`.

**Oral cue:** large `C` can overfit noise; small `C` can underfit real faults.

## Slide 9: Dual Form and KKT Intuition

**Takeaway:** The dual form shows why only support vectors matter.

**On-slide formula:**

```text
maximize:
sum_i alpha_i - 1/2 sum_i sum_j alpha_i alpha_j y_i y_j x_i^T x_j

subject to:
0 <= alpha_i <= C
sum_i alpha_i y_i = 0
```

**On-slide explanation:**

```text
alpha_i > 0  -> support vector
alpha_i = 0  -> not boundary-critical
```

**Visual:** support vectors highlighted; non-support points faded.

**Oral cue:** do not derive KKT fully; state the interpretation.

## Slide 10: Kernel Trick

**Takeaway:** Kernels let SVM draw nonlinear boundaries using inner products in feature space.

**On-slide formula:**

```text
K(x_i, x_j) = phi(x_i)^T phi(x_j)

Decision:
f(x) = sum_i alpha_i y_i K(x_i, x) + b
```

**On-slide bullets:**

```text
- no explicit high-dimensional mapping required
- nonlinear boundary in original space
- kernel choice is a modeling assumption
```

**Visual:** nonlinear data -> feature space -> linear separation.

**Oral cue:** connect to KPCA: both use kernel trick, but for different goals.

## Slide 11: Common Kernels

**Takeaway:** Different kernels encode different assumptions about class geometry.

**On-slide table:**

| Kernel | Formula | Use |
|---|---|---|
| Linear | `x_i^T x_j` | linear separation |
| Polynomial | `(x_i^T x_j + c)^d` | curved polynomial boundary |
| RBF | `exp(-gamma ||x_i-x_j||^2)` | local nonlinear boundary |
| Sigmoid | `tanh(k x_i^T x_j + c)` | neural-like boundary |

**Visual:** boundary examples for linear vs RBF.

**Oral cue:** `gamma` too high can create overly complex boundaries.

## Slide 12: Perceptron

**Takeaway:** Perceptron is a simple linear classifier that updates only when a sample is misclassified.

**On-slide formula:**

```text
If y_i(w^T x_i + b) <= 0:

w <- w + eta y_i x_i
b <- b + eta y_i
```

**On-slide bullets:**

```text
- learns a separating hyperplane
- does not maximize margin
- converges only if data are linearly separable
```

**Visual:** step-by-step boundary movement after mistakes.

**Oral cue:** use perceptron as a contrast to SVM, not as the main method.

## Slide 13: SVM vs Logistic Regression vs Perceptron

**Takeaway:** These classifiers differ in what they optimize.

**On-slide table:**

| Method | Optimizes | Output | Boundary |
|---|---|---|---|
| Logistic Regression | probability / cross-entropy | posterior probability | linear unless features expand |
| SVM | margin | class label / score | linear or kernel nonlinear |
| Perceptron | mistake correction | class label | linear |

**Visual:** three classifier cards.

**Oral cue:** this slide helps connect Topic 3 back to Topic 1.

## Slide 14: Topic 3 Knowledge Map

**Takeaway:** SVM is the maximum-margin answer to fault classification.

**On-slide map:**

```text
Features
-> hyperplane
-> margin
-> support vectors
-> soft margin
-> kernel SVM
-> fault decision
```

**Final sentence:**

```text
Hard margin is the ideal case; soft margin and kernels make SVM usable for real fault data.
```

**Visual:** method map with hard margin, soft margin, kernel, perceptron branches.

**Oral cue:** close by saying Topic 4 switches from data geometry to system-model residuals.

---

# Topic 4: Model-Based Fault Detection

## Topic Spine

```text
System model
-> predicted output/state
-> residual
-> residual evaluation
-> observer/parity/KF/UIO/CUSUM
-> alarm and fault information
```

Topic 4 is about **model-based residual generation and sequential decision**.

## Slide 1: Title and Big Question

**Takeaway:** A model reveals faults by comparing expected behavior with measured behavior.

**On-slide content:**

```text
Big question:
If we know how the system should behave, how can deviations reveal faults?
```

**Visual:** measured output vs predicted output.

**Oral cue:** distinguish Topic 4 from Topic 1: probability vs dynamics model.

## Slide 2: Model-Based FDD Architecture

**Takeaway:** Model-based FDD converts model mismatch into a residual signal.

**On-slide flow:**

```text
u, y
-> model / observer
-> y_hat or x_hat
-> residual r
-> threshold / CUSUM
-> alarm and isolation
```

**Formula:**

```text
r(t) = y(t) - y_hat(t)
```

**Visual:** block diagram with plant and observer/model.

**Oral cue:** residual is the central object of Topic 4.

## Slide 3: Fault Locations and Fault Models

**Takeaway:** Fault location determines how residuals should be generated and interpreted.

**On-slide model:**

```text
x_dot = A x + B u + E_d d + E_f f
y     = C x + D_f f_s + v
```

**On-slide fault locations:**

```text
- actuator fault: input channel changes
- sensor fault: measurement channel changes
- plant fault: dynamics or parameters change
```

**Visual:** control loop with actuator, plant, sensor fault labels.

**Oral cue:** bias, drift, abrupt, incipient, and intermittent faults can be explained verbally.

## Slide 4: Residual Generation and Evaluation

**Takeaway:** A good residual is small under normal operation and sensitive to faults.

**On-slide content:**

```text
Residual generator:
r = H(q) y + G(q) u

Decision statistic:
J(r) = ||r|| or r^T R^{-1} r

Alarm:
J(r) > h
```

**Visual:** residual time plot crossing threshold.

**Oral cue:** due to noise and modeling error, residual is not exactly zero under normal operation.

## Slide 5: Fault Isolation by Residual Pattern

**Takeaway:** Fault isolation uses different residual responses to different fault locations.

**On-slide table:**

| Residual | Sensor fault | Actuator fault | Plant fault |
|---|---:|---:|---:|
| `r1` | high | low | medium |
| `r2` | low | high | medium |
| `r3` | medium | medium | high |

**Visual:** residual signature matrix.

**Oral cue:** isolation requires structured residuals, not only one alarm signal.

## Slide 6: Parity Method in Time Domain

**Takeaway:** Parity equations check consistency between measured data and the system model.

**On-slide idea:**

```text
Model consistency:
Y = O x0 + T U

Choose W such that:
W O = 0

Residual:
r = W (Y - T U)
```

**Visual:** stacked input-output data window -> parity residual.

**Oral cue:** explain that state `x0` is eliminated, leaving an input-output consistency test.

## Slide 7: Parity Method in Frequency Domain

**Takeaway:** Frequency-domain parity checks whether measured input-output behavior matches the transfer model.

**On-slide formula:**

```text
y(s) = G(s) u(s)

r(s) = Q(s) [ y(s) - G(s)u(s) ]
```

**On-slide bullets:**

```text
- useful for transfer-function models
- residual filter Q(s) shapes sensitivity
- model mismatch can cause false alarms
```

**Visual:** frequency-domain residual filter diagram.

**Oral cue:** keep derivation oral; show the idea only.

## Slide 8: Observer-Based Residual

**Takeaway:** Observers estimate hidden states, then residuals compare measured and estimated outputs.

**On-slide formula:**

```text
x_hat_dot = A x_hat + B u + L(y - C x_hat)

y_hat = C x_hat

r = y - y_hat
```

**Visual:** Luenberger observer block diagram.

**On-slide bullets:**

```text
- L controls estimation error dynamics
- residual reacts to model inconsistency
- unknown disturbances can corrupt residuals
```

**Oral cue:** observer design details belong to Q&A unless asked.

## Slide 9: Kalman Filter Residual

**Takeaway:** Kalman filters generate statistically meaningful residuals for noisy systems.

**On-slide formula:**

```text
Innovation:
nu_k = y_k - C x_hat_{k|k-1}

Normalized statistic:
J_k = nu_k^T S_k^{-1} nu_k
```

**Visual:** innovation sequence with threshold.

**On-slide bullets:**

```text
- handles process and measurement noise
- innovation should be statistically small under normal operation
- faults change innovation mean or variance
```

**Oral cue:** covariance prediction/update equations can be skipped.

## Slide 10: ESO and NDO

**Takeaway:** ESO and NDO estimate unknown disturbances or fault effects as augmented states.

**On-slide idea:**

```text
Extended model:
x_aug = [x; d]

Estimate:
d_hat -> disturbance/fault compensation or diagnosis
```

**On-slide comparison:**

| Method | Main idea | Use |
|---|---|---|
| ESO | estimate total disturbance | active disturbance rejection |
| NDO | estimate nonlinear disturbance | nonlinear systems |

**Visual:** unknown input entering plant and estimated by observer.

**Oral cue:** gain tuning and convergence proof are non-key for presentation.

## Slide 11: Unknown Input Observer

**Takeaway:** UIO is designed to estimate states while decoupling unknown inputs.

**On-slide model:**

```text
x_dot = A x + B u + E d
y = C x
```

**On-slide objective:**

```text
state estimation insensitive to d
residual sensitive to selected faults
```

**Key condition prompt:**

```text
rank(C E) = rank(E)
```

**Visual:** unknown disturbance path blocked from estimation error.

**Oral cue:** mention that UIO existence is restrictive.

## Slide 12: CUSUM for Sequential Detection

**Takeaway:** CUSUM accumulates weak evidence over time to detect persistent changes.

**On-slide formula:**

```text
g_k = max(0, g_{k-1} + s_k)

Alarm if:
g_k > h
```

**Gaussian mean-shift score:**

```text
s_k = log p(y_k | theta_1) / p(y_k | theta_0)
```

**Visual:** CUSUM statistic rising after a change point.

**Oral cue:** CUSUM is stronger than single-sample threshold for small persistent faults.

## Slide 13: ARL and Threshold Design

**Takeaway:** ARL connects threshold design to false alarm rate and detection delay.

**On-slide definitions:**

```text
ARL_0: average run length before false alarm under normal condition
ARL_1: average detection delay after fault
```

**On-slide trade-off:**

```text
Higher threshold h:
ARL_0 increases, false alarms decrease
ARL_1 increases, detection becomes slower
```

**Visual:** threshold vs ARL trade-off curve.

**Oral cue:** exact ARL formulas can be discussed only if asked.

## Slide 14: Topic 4 Method Map

**Takeaway:** Model-based FDD is a family of residual generators plus residual decision algorithms.

**On-slide table:**

| Method | Residual source | Strength | Limitation |
|---|---|---|---|
| Parity | input-output consistency | no state estimator needed | model order/window sensitive |
| Observer | state estimate | intuitive and modular | disturbance sensitive |
| KF | innovation | handles noise | needs noise model |
| UIO | decoupled estimate | robust to unknown input | existence conditions |
| CUSUM | accumulated evidence | detects small changes | threshold/ARL design |

**Visual:** final flow:

```text
model -> residual -> statistic -> threshold/CUSUM -> alarm/isolation
```

**Oral cue:** close by saying Topic 5 uses diagnosis information to reconfigure control.

---

# Topic 5: Fault Tolerant Control Strategy

## Topic Spine

```text
Fault diagnosis
-> redundancy
-> passive or active FTC
-> model matching / virtual components
-> bumpless reconfiguration
-> stable acceptable performance
```

Topic 5 is about **what control does after fault diagnosis**.

## Slide 1: Title and Big Question

**Takeaway:** FTC aims to keep the system stable and useful after faults occur.

**On-slide content:**

```text
Big question:
After a fault is detected, how can the controller maintain acceptable performance?
```

**Visual:** healthy controller path vs faulty/reconfigured path.

**Oral cue:** distinguish FTC from FDD: FDD diagnoses; FTC acts.

## Slide 2: FTC Architecture

**Takeaway:** Active FTC closes the loop from fault diagnosis to controller reconfiguration.

**On-slide flow:**

```text
Plant measurements
-> FDD module
-> fault information
-> reconfiguration mechanism
-> controller / actuator / sensor adjustment
```

**Visual:** block diagram with FDD and reconfigurable controller.

**Oral cue:** diagnosis delay and wrong isolation limit FTC performance.

## Slide 3: Fault Locations and Redundancy

**Takeaway:** FTC is possible only if the system has enough redundancy to compensate for the fault.

**On-slide content:**

```text
Fault locations:
- sensor fault
- actuator fault
- plant/component fault

Redundancy:
- hardware redundancy
- analytical redundancy
- actuator authority / control allocation
```

**Visual:** control loop with fault locations and redundant paths.

**Oral cue:** if no alternative information or actuation exists, control cannot recover fully.

## Slide 4: Passive FTC

**Takeaway:** Passive FTC uses one robust controller designed to tolerate a predefined fault set.

**On-slide formula prompt:**

```text
Controller K designed for:
plant uncertainty + expected fault set
```

**On-slide bullets:**

```text
- no online fault diagnosis required
- fast and simple
- conservative performance
- limited to anticipated faults
```

**Visual:** fixed controller covering a fault envelope.

**Oral cue:** passive FTC is robust control applied to fault scenarios.

## Slide 5: Active FTC

**Takeaway:** Active FTC changes the control structure after fault diagnosis.

**On-slide flow:**

```text
detect -> isolate -> identify -> reconfigure
```

**On-slide bullets:**

```text
- less conservative than passive FTC
- depends on reliable FDD
- reconfiguration can cause transients
```

**Visual:** switch from nominal controller to reconfigured controller.

**Oral cue:** active FTC is powerful but only as good as diagnosis and reconfiguration design.

## Slide 6: Robust vs Adaptive Control

**Takeaway:** Robust control prepares for uncertainty; adaptive control updates when parameters change.

**On-slide table:**

| Strategy | Main idea | Strength | Limitation |
|---|---|---|---|
| Robust control | fixed controller for uncertainty set | reliable, fast | conservative |
| Adaptive control | parameters updated online | handles changing dynamics | convergence and stability concerns |

**Visual:** uncertainty set vs online parameter update.

**Oral cue:** keep this as background, not the main FTC method.

## Slide 7: Reconfigurable Control Systems

**Takeaway:** Reconfiguration changes the controller, command path, sensor path, or actuator allocation.

**On-slide content:**

```text
Reconfiguration modes:
- controller switching
- controller retuning
- control allocation
- virtual actuator
- virtual sensor
```

**Visual:** central reconfiguration supervisor connected to controller, actuator, sensor.

**Oral cue:** reconfiguration can be structural or parametric.

## Slide 8: Model Matching

**Takeaway:** Model matching tries to make the faulty closed-loop system behave like the nominal one.

**On-slide objective:**

```text
Nominal closed-loop behavior:
M_nom(s)

Faulty reconfigured behavior:
M_f(s, K_f)

Goal:
M_f(s, K_f) approximately M_nom(s)
```

**Visual:** nominal model and faulty plant both pointing to same desired response.

**Oral cue:** this is a key active FTC strategy in the lecture.

## Slide 9: Exact vs Approximate Model Matching

**Takeaway:** Exact matching is ideal but often impossible, so approximate matching is used.

**On-slide comparison:**

| Type | Goal | Condition | Result |
|---|---|---|---|
| Exact model matching | `M_f = M_nom` | strict solvability | perfect recovery |
| Approximate model matching | minimize mismatch | least squares / pseudo-inverse | best possible recovery |

**Generic optimization:**

```text
min_{K_f} || M_nom - M_f(K_f) ||^2
```

**Visual:** response curves: nominal, faulty, reconfigured.

**Oral cue:** mention kernel/range conditions only as an exam cue.

## Slide 10: Virtual Sensor

**Takeaway:** A virtual sensor reconstructs the nominal measurement when a real sensor is faulty.

**On-slide flow:**

```text
faulty measurement y_f
-> virtual sensor
-> reconstructed y_hat_nom
-> nominal controller
```

**On-slide bullets:**

```text
- handles sensor fault
- preserves nominal controller interface
- requires analytical redundancy or observer model
```

**Visual:** faulty sensor bypass with virtual sensor block.

**Oral cue:** virtual sensor solves an information problem.

## Slide 11: Virtual Actuator

**Takeaway:** A virtual actuator transforms nominal control commands into feasible commands for a faulty actuator system.

**On-slide flow:**

```text
nominal controller command u
-> virtual actuator
-> reconfigured command u_f
-> faulty actuator/plant
```

**On-slide bullets:**

```text
- handles actuator fault
- hides actuator fault from nominal controller
- limited by remaining actuator authority
```

**Visual:** nominal controller connected to faulty actuator through virtual actuator block.

**Oral cue:** virtual actuator solves a command delivery problem.

## Slide 12: Transient Elimination and Bumpless Transfer

**Takeaway:** Controller switching must avoid large transients that destabilize or stress the plant.

**On-slide content:**

```text
Problem:
switching controller states can create control jumps

Goal:
u_old(t_s) approximately u_new(t_s)
```

**Visual:** control signal with bump vs bumpless transfer.

**On-slide bullets:**

```text
- initialize new controller state
- smooth command transition
- protect actuator and plant
```

**Oral cue:** detailed transient elimination equations can stay in Q&A.

## Slide 13: Physical Limits of FTC

**Takeaway:** FTC cannot recover performance beyond physical redundancy and actuator limits.

**On-slide content:**

```text
FTC limitations:
- no redundancy -> no recovery path
- actuator saturation
- diagnosis delay
- wrong fault isolation
- model mismatch
- severe plant damage
```

**Visual:** performance envelope shrinking after fault.

**Oral cue:** professional answer: FTC maintains acceptable performance, not magic perfect recovery.

## Slide 14: Topic 5 Knowledge Map

**Takeaway:** Topic 5 turns fault information into control reconfiguration.

**On-slide map:**

```text
FDD result
-> fault type and location
-> redundancy check
-> passive or active FTC
-> model matching / virtual component
-> stable acceptable control
```

**Final comparison:**

| Method | Main role |
|---|---|
| Passive FTC | tolerate expected faults |
| Active FTC | reconfigure after diagnosis |
| Model matching | recover nominal behavior |
| Virtual sensor | repair information path |
| Virtual actuator | repair command path |
| Bumpless transfer | avoid switching shock |

**Oral cue:** close by connecting to Topic 6: constraints and security make FTC a safe CPS problem.

---

# Topic 6: Safe and Secure Control for CPS

## Topic Spine

```text
CPS faults / attacks / anomalies
-> detection and estimation
-> constraints and safety envelope
-> MPC / resilient MPC
-> anomaly detection with VAE
-> safe operation
```

Topic 6 is the course endpoint: **fault tolerance plus safety, constraints, and cyber-physical security**.

## Slide 1: Title and Big Question

**Takeaway:** Safe CPS control prioritizes constraint satisfaction and resilience under faults and attacks.

**On-slide content:**

```text
Big question:
How can a cyber-physical system remain safe when sensors, actuators, or communication channels are faulty or attacked?
```

**Visual:** CPS loop with physical process, controller, network, sensors, actuators.

**Oral cue:** safety is more important than perfect tracking.

## Slide 2: Faults, Attacks, and Anomalies

**Takeaway:** CPS diagnosis must distinguish abnormal behavior, physical faults, and malicious attacks.

**On-slide table:**

| Event | Meaning | Example |
|---|---|---|
| fault | component abnormality | sensor bias |
| anomaly | unusual behavior | unexpected trajectory |
| attack | malicious manipulation | false data injection |
| disturbance | non-malicious external effect | load change |

**Visual:** abnormal event taxonomy.

**Oral cue:** an anomaly is an observation; the cause may be fault, attack, or disturbance.

## Slide 3: Safe and Resilient CPS Control Chain

**Takeaway:** Safe CPS control links detection to constrained corrective action.

**On-slide flow:**

```text
monitor
-> detect anomaly
-> isolate/estimate
-> update state/disturbance estimate
-> solve constrained control problem
-> apply safe input
```

**Visual:** closed-loop resilient control architecture.

**Oral cue:** this slide connects all six topics.

## Slide 4: From LQR to Constrained Control

**Takeaway:** LQR is elegant but assumes an unconstrained world; real CPS must respect hard limits.

**On-slide formula:**

```text
LQR:
min sum (x_k^T Q x_k + u_k^T R u_k)

u_k = -K x_k
```

**On-slide limitations:**

```text
Real systems:
u_min <= u_k <= u_max
x_min <= x_k <= x_max
```

**Visual:** unconstrained trajectory violating safety bounds.

**Oral cue:** this motivates MPC.

## Slide 5: Finite-Horizon Constrained Optimization

**Takeaway:** MPC optimizes future behavior while explicitly enforcing constraints.

**On-slide optimization:**

```text
min_{u_0,...,u_{N-1}}
sum_{k=0}^{N-1} (x_k^T Q x_k + u_k^T R u_k) + x_N^T P x_N

subject to:
x_{k+1} = A x_k + B u_k
x_k in X
u_k in U
```

**Visual:** prediction horizon with constrained state/input trajectory.

**Oral cue:** do not derive QP matrices; show the problem structure.

## Slide 6: Receding Horizon MPC

**Takeaway:** MPC repeatedly solves the finite-horizon problem and applies only the first input.

**On-slide algorithm:**

```text
1. measure or estimate current state x_k
2. solve constrained optimization
3. apply first input u_k
4. shift horizon and repeat
```

**Visual:** moving prediction horizon.

**Oral cue:** this online feedback loop is why MPC can react to faults and disturbances.

## Slide 7: QP Form and State Elimination

**Takeaway:** Linear MPC becomes a quadratic program that can be solved efficiently online.

**On-slide compact form:**

```text
min_U  1/2 U^T H U + f^T U

subject to:
G U <= g
```

**On-slide explanation:**

```text
U = [u_0, ..., u_{N-1}]
H: cost matrix
G U <= g: state and input constraints
```

**Visual:** original MPC problem -> QP solver block.

**Oral cue:** state elimination details are implementation-level.

## Slide 8: MPC Stability Problem

**Takeaway:** Finite-horizon optimization alone does not automatically guarantee closed-loop stability.

**On-slide content:**

```text
Potential problems:
- horizon too short
- optimizer is locally greedy
- terminal behavior unspecified
- constraints may destroy feasibility
```

**Visual:** predicted path looks good short-term but fails later.

**Oral cue:** this motivates terminal equality, terminal set, and terminal cost.

## Slide 9: Terminal Cost and Terminal Set

**Takeaway:** Terminal ingredients make MPC stable and recursively feasible.

**On-slide formula:**

```text
Terminal constraint:
x_N in X_f

Terminal cost:
V_f(x_N) = x_N^T P x_N
```

**On-slide bullets:**

```text
X_f: safe invariant terminal region
P: approximates infinite-horizon tail cost
local controller keeps state inside X_f
```

**Visual:** terminal set as safe landing region.

**Oral cue:** positive invariance and Lyapunov decrease can be explained orally.

## Slide 10: Reference Tracking and Target Selection

**Takeaway:** Safe MPC should track a feasible target, not an impossible reference.

**On-slide idea:**

```text
Target selection:
(x_s, u_s) chosen so that
x_s = A x_s + B u_s
y_s approximately r
constraints satisfied
```

**Visual:** desired reference outside constraints; feasible target inside constraints.

**On-slide bullets:**

```text
- prevents infeasible tracking commands
- separates steady-state target from dynamic correction
- important under actuator limits
```

**Oral cue:** this prepares offset-free MPC.

## Slide 11: Offset-Free MPC

**Takeaway:** Offset-free MPC estimates disturbances so the controller can remove steady-state error.

**On-slide augmented model:**

```text
x_{k+1} = A x_k + B u_k + B_d d_k
d_{k+1} = d_k
y_k = C x_k + C_d d_k
```

**On-slide execution loop:**

```text
measure -> estimate disturbance -> select target -> optimize -> actuate
```

**Visual:** disturbance estimator feeding MPC target selection.

**Oral cue:** the disturbance estimate shifts the steady-state baseline.

## Slide 12: LESO / UIO plus MPC

**Takeaway:** Resilient MPC can use observers to estimate unknown inputs or attacks before optimizing control.

**On-slide flow:**

```text
sensor data
-> LESO / UIO
-> disturbance or attack estimate
-> offset-free / resilient MPC
-> safe control input
```

**On-slide comparison:**

| Observer | Main role |
|---|---|
| LESO | estimate total disturbance |
| UIO | decouple unknown input and estimate state |

**Visual:** observer-enhanced MPC architecture.

**Oral cue:** this connects Topic 4 observers to Topic 6 resilient control.

## Slide 13: VAE Anomaly Detection

**Takeaway:** VAE detects anomalies by learning a probabilistic model of normal behavior.

**On-slide architecture:**

```text
x -> encoder q_phi(z|x)
z -> decoder p_theta(x|z)
reconstruction / likelihood -> anomaly score
```

**On-slide formula:**

```text
ELBO = E_q[ log p_theta(x|z) ] - D_KL(q_phi(z|x) || p(z))

Anomaly score:
a(x) = reconstruction error or -ELBO
```

**Visual:** encoder-latent-decoder diagram with anomaly score output.

**Oral cue:** ELBO derivation and reparameterization trick are backup/Q&A material.

## Slide 14: Topic 6 Knowledge Map

**Takeaway:** Safe CPS control combines anomaly detection, estimation, constraints, and resilient control.

**On-slide map:**

```text
CPS data
-> anomaly / attack / fault detection
-> observer or VAE estimate
-> constrained MPC
-> terminal safety and offset-free tracking
-> resilient operation
```

**On-slide final table:**

| Component | Role |
|---|---|
| anomaly detection | identify abnormal behavior |
| MPC | enforce state/input constraints |
| terminal set/cost | stability and recursive feasibility |
| offset-free MPC | remove steady-state error |
| LESO/UIO | estimate unknown disturbance/attack |
| VAE | probabilistic anomaly score |

**Oral cue:** final statement: Topic 6 is where FTC becomes safe and secure CPS control.

---

# Cross-Topic Presentation Map

Use this as a final slide or a recurring small footer in all decks.

```text
Topic 1: Statistical decision from signal data
Topic 2: Fault-sensitive low-dimensional representation
Topic 3: Maximum-margin classification
Topic 4: Model-based residual generation and diagnosis
Topic 5: Controller reconfiguration after faults
Topic 6: Safe and resilient CPS control
```

Full course chain:

```text
Signal data / system model
-> features, representations, or residuals
-> diagnosis
-> classification or sequential detection
-> control reconfiguration
-> safe and resilient operation
```

