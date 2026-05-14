# Advanced Machine Learning - Full Algorithm Knowledge Chains

Each algorithm is explained through the same chain:

```text
engineering problem -> inputs/outputs -> core principle -> basic process -> training/inference logic -> evaluation metrics
```

The goal is not to memorize isolated definitions. The goal is to explain each method as a complete reasoning chain from engineering motivation to practical evaluation.

## 0. Whole-Course Knowledge Chain

### Overall Logic

The main idea of Advanced Machine Learning is to convert engineering problems into learning problems, then use neural networks or related algorithms to learn useful patterns from data.

The whole workflow is:

```text
real engineering system
  -> collect or simulate data
  -> define the task type
  -> choose input features and labels
  -> choose a model
  -> choose a loss function
  -> train with an optimizer
  -> evaluate with suitable metrics
  -> analyze errors and limitations
  -> deploy or improve the system
```

A good oral-exam opening is:

> The core of this course is not just remembering individual models. It is understanding how an engineering problem becomes a machine learning workflow. First, I define the input and output. Then I identify whether the task is regression, classification, anomaly detection, time-series prediction, or object detection. After that, I choose the model and loss function, train on the training set, tune on the validation set, evaluate once on the test set, and finally discuss reliability and limitations in a real engineering environment.

### Shared Training Framework

Most supervised learning models can be described as:

```text
y_hat = f_theta(x)
loss = L(y_hat, y)
theta <- theta - alpha * gradient(loss)
```

where:

- `x` is the input data.
- `y` is the true label or target.
- `f_theta` is the model with parameters.
- `theta` represents model parameters such as weights and biases.
- `L` is the loss function.
- `alpha` is the learning rate.

This framework is shared by linear regression, MLPs, CNNs, RNNs, and ResNets. The differences are the model structure, the loss function, and the data format.

## 1. Defining a Machine Learning Problem: From Engineering Task to Model Task

### Problem Definition

Before choosing any model, the first step is to define the task. For every course example, ask four questions:

1. What is the input?
2. What is the output?
3. Do we have labels?
4. What is the cost of a wrong prediction?

### Core Principle

Machine learning learns a mapping:

```text
input x -> output y
```

If paired data `(x, y)` is available, the problem is supervised learning. If only `x` is available, it is unsupervised learning. If an agent acts in an environment and receives rewards, it is reinforcement learning.

This course mainly uses supervised and unsupervised learning.

### Basic Process

```text
Step 1: Define the engineering goal
Step 2: Identify the output type
Step 3: Choose the task type
Step 4: Choose the data representation
Step 5: Choose the model and loss function
Step 6: Train, validate, and test
Step 7: Explain results and risks
```

### Task-Type Chain

| Output Type | Task | Course Example | Common Models | Common Metrics |
|---|---|---|---|---|
| Continuous value | Regression | Motor speed, temperature prediction | Linear regression, MLP, RNN, LSTM | MSE, MAE, RMSE |
| Two classes | Binary classification | Healthy/faulty | Logistic regression, MLP | Precision, recall, F1, ROC-AUC |
| Multiple classes | Multiclass classification | Bearing fault type, MRI four-class classification | MLP, CNN, ResNet | Accuracy, per-class recall, confusion matrix |
| Normal data without full labels | Anomaly detection | Screw defects, machine anomalies | K-means, autoencoder | Reconstruction error, AUC, F1 |
| Multiple objects in an image | Object detection | Warehouse object detection | YOLO, RT-DETR | IoU, mAP |
| Text or engineering explanation | Language modeling | LLM-based engineering monitoring | Transformer LLM | Correctness, verifiability, safety |

### Oral Explanation

> I first define the task from the output. If the output is a continuous value, I use regression. If the output is a class, I use classification. If there are no labels but I need to find unusual samples, I use anomaly detection. If an image requires both class and position, I use object detection. Only after the task is clear can I choose the correct model, loss function, and evaluation metric.

### Common Trap

Do not start by saying "I used a neural network" and then explain the task afterward. The correct order is: problem first, then why the model fits that problem.

## 2. Digital Twin + Linear Regression Algorithm Chain

### Problem Definition

In Week 1, the problem is that we cannot run unlimited experiments on a real motor system. Therefore, we first build a DC motor digital twin, generate simulation data, and then learn the relationship between input voltage and output speed.

```text
Input: voltage or motor state
Output: angular speed
Task: regression
Model: linear regression or the simplest neural network
```

### Core Principle

The digital twin generates data that approximates the real system. Linear regression learns a simple relation from the generated data:

```text
y_hat = w * x + b
```

For the motor example:

```text
speed_hat = w * voltage + b
```

Here, `w` represents how strongly voltage affects speed, and `b` is the bias or offset term.

### Physical Model Chain

The simplified DC motor model used in the course is:

```text
J * dω/dt = K * V - b * ω
dω/dt = (K * V - b * ω) / J
ω_new = ω_old + dω/dt * dt
```

Meaning:

- Voltage `V` creates a driving effect through the motor constant `K`.
- Friction `b * ω` resists motion.
- Inertia `J` controls how hard it is to change speed.
- With time step `dt`, the speed can be updated repeatedly to simulate motor behavior.

### Basic Algorithm Process

```text
Step 1: Build a DC motor simulator
Step 2: Apply different voltages and record speed
Step 3: Add noise to make the data closer to sensor data
Step 4: Create training pairs (voltage, speed)
Step 5: Build a linear model speed_hat = w * voltage + b
Step 6: Use MSE to measure prediction error
Step 7: Use gradient descent to update w and b
Step 8: Test whether the model predicts speed for new voltages
```

### Loss Function and Optimization

Mean squared error:

```text
MSE = mean((y_hat - y_true)^2)
```

Parameter update:

```text
w <- w - alpha * dL/dw
b <- b - alpha * dL/db
```

MSE fits regression because it measures continuous numerical error and is differentiable, so it can be optimized by gradient descent.

### Oral Explanation

> The algorithm chain is: first build a digital twin of the DC motor using physical equations, then use different voltages to generate speed data. After that, assume the voltage-speed relation can be approximated by a line, `speed = w * voltage + b`. During training, MSE measures the difference between predicted and true speed, and gradient descent updates `w` and `b` until the error decreases. This example shows how machine learning can learn system behavior from simulated or experimental data.

### Common Trap

Linear regression is not unrelated to neural networks. It can be seen as a single neuron without a nonlinear activation. It is the starting point for deep learning.

## 3. MLP + Backpropagation Algorithm Chain

### Problem Definition

In Week 2, the problem is that real engineering systems are often nonlinear, and manual tuning is fragile. We need a model that can automatically learn complex nonlinear relationships.

```text
Input: multiple engineering features, such as voltage, current, previous speed
Output: predicted speed or system state
Task: regression or classification
Model: MLP
```

### Core Principle

An MLP is a multilayer neural network. Each layer does two things:

```text
linear transform -> activation
```

In formula form:

```text
z = W * x + b
a = activation(z)
```

By stacking multiple layers, the model can represent complex nonlinear relationships.

### Why Activation Functions Are Needed

Without activation functions, multiple linear layers are still equivalent to one linear layer:

```text
W3 * (W2 * (W1 * x)) = W_total * x
```

Therefore, deep networks need nonlinear activations such as ReLU:

```text
ReLU(x) = max(0, x)
```

### Full Training Logic

An MLP does not directly "know" the answer. It repeatedly performs four steps:

```text
Forward pass -> Compute loss -> Backward pass -> Optimizer step
```

Detailed process:

```text
Step 1: Initialize network weights
Step 2: Feed one batch into the model
Step 3: Perform forward propagation to get y_hat
Step 4: Compute the loss
Step 5: Backpropagate to compute gradients for every parameter
Step 6: Let the optimizer update parameters using gradients
Step 7: Check generalization on the validation set
Step 8: Repeat for multiple epochs until convergence or early stopping
```

### PyTorch Training Template

```text
predictions = model(X_train)
loss = criterion(predictions, y_train)
optimizer.zero_grad()
loss.backward()
optimizer.step()
```

These five lines represent the complete learning loop of a neural network.

### Backpropagation Principle

Backpropagation answers this question:

```text
The final loss is high. Which weights caused it, and by how much should each weight change?
```

The mathematical basis is the chain rule. A simple two-layer example:

```text
h = w1 * x
y_hat = w2 * h
L = (y_hat - y)^2

dL/dw1 = dL/dy_hat * dy_hat/dh * dh/dw1
```

This means the responsibility of the first-layer weight is computed through the entire computational chain after it.

### Optimizer Logic

Basic gradient descent:

```text
theta <- theta - alpha * gradient
```

Momentum adds memory of previous update directions and reduces oscillation. Adam adapts the step size for different parameters and is often the default optimizer in the course practice.

### Train / Validation / Test Chain

```text
Train set: update parameters
Validation set: tune hyperparameters, early stopping, model selection
Test set: final evaluation of generalization
```

### Oral Explanation

> The core idea of an MLP is to use stacked linear transformations and nonlinear activation functions to represent complex relationships. During training, the model first performs a forward pass to make a prediction, then the loss measures the difference between prediction and target. Backpropagation uses the chain rule to compute how much each parameter contributed to the loss, and the optimizer updates the weights based on those gradients. This repeats over many batches and epochs until the training and validation losses become stable.

### Common Trap

`loss.backward()` only computes gradients. It does not update parameters. The actual parameter update happens in `optimizer.step()`. Also, `optimizer.zero_grad()` must be called before each backward pass, otherwise gradients accumulate.

## 4. Logistic Regression + Binary Fault Detection Algorithm Chain

### Problem Definition

The Week 3 problem is: how can we decide whether a machine is faulty?

```text
Input: vibration signal or features extracted from vibration signal
Output: healthy or broken
Task: binary classification
Model: logistic regression or binary MLP
```

### Core Principle

Linear regression outputs arbitrary continuous values, so it is not suitable for probabilities. Logistic regression first computes a linear score, then applies sigmoid to map it into the range 0 to 1.

```text
z = w · x + b
y_hat = sigmoid(z) = 1 / (1 + exp(-z))
```

`y_hat` is interpreted as the probability of fault.

### Classification Decision Logic

```text
if y_hat >= threshold:
    predict broken
else:
    predict healthy
```

The default threshold can be 0.5, but in engineering the threshold should be adjusted according to the cost of false alarms and missed faults.

### Loss Function Logic

Binary cross entropy:

```text
BCE = -[y * log(y_hat) + (1 - y) * log(1 - y_hat)]
```

Its logic is:

- If the true label is fault `y=1`, we want `y_hat` close to 1.
- If the true label is healthy `y=0`, we want `y_hat` close to 0.
- If the model is very confident but wrong, the loss becomes very large.

### Full Fault Detection Workflow

```text
Step 1: Collect or simulate healthy and faulty vibration signals
Step 2: Split each signal into windows
Step 3: Extract time-domain or frequency-domain features
Step 4: Create labels healthy=0, fault=1
Step 5: Split into train/test sets
Step 6: Standardize the features
Step 7: Train a binary model with sigmoid output
Step 8: Output fault probability
Step 9: Convert probability into class using a threshold
Step 10: Evaluate using confusion matrix, precision, recall, F1, and ROC-AUC
```

### FFT Feature Chain

Vibration signals often use FFT because faults usually appear as specific frequency components.

```text
time signal
  -> windowing
  -> FFT
  -> frequency spectrum
  -> extract features
  -> classifier
```

Common features:

- RMS: overall vibration energy.
- Kurtosis: impulsiveness and spikiness.
- Crest factor: peak magnitude relative to overall energy.
- Peak frequency: dominant frequency peak.
- Band energy ratio: energy proportion in a fault-related frequency band.

Nyquist theorem:

```text
f_max = f_sample / 2
```

The sampling frequency must be at least twice the highest frequency of interest, otherwise aliasing can occur.

### Evaluation Metric Chain

Confusion matrix:

| Actual / Predicted | Predict Healthy | Predict Fault |
|---|---|---|
| Actually Healthy | TN | FP |
| Actually Faulty | FN | TP |

Metrics:

```text
Precision = TP / (TP + FP)
Recall = TP / (TP + FN)
F1 = 2 * Precision * Recall / (Precision + Recall)
```

In industrial fault detection, FN, or missed fault, is often the most dangerous case. Therefore, recall is often more important than accuracy.

### Oral Explanation

> The binary fault detection workflow is: first extract features from machine vibration signals, such as RMS, kurtosis, and FFT-based frequency energy. Then logistic regression or a small neural network outputs the probability of fault. Sigmoid converts the model score into a probability between 0 and 1, and a threshold converts the probability into an alarm decision. The model is trained with binary cross entropy because this is a probability classification problem. During evaluation, accuracy is not enough. I must inspect the confusion matrix, especially recall, because missing a real fault is usually much more costly than a false alarm.

### Common Trap

The threshold is not always fixed at 0.5. If the cost of missing a fault is very high, the threshold should be lowered to increase recall, even if false alarms increase.

## 5. Softmax Multiclass Classification + Bearing Diagnosis Algorithm Chain

### Problem Definition

In Week 4, the problem changes from "is it faulty?" to "which fault is it?"

```text
Input: bearing vibration signal or extracted features
Output: Normal / Ball fault / Inner race fault / Outer race fault
Task: multiclass classification
Model: multi-output MLP
```

### Core Principle

A multiclass model does not output one probability. It outputs one logit for each class:

```text
z = [z1, z2, ..., zK]
```

Softmax converts logits into a probability distribution:

```text
softmax(z_i) = exp(z_i) / sum_j exp(z_j)
```

All class probabilities sum to 1, and the predicted class is usually the class with the highest probability.

### CrossEntropyLoss Logic

The multiclass loss is categorical cross entropy:

```text
CE = -sum_i y_i * log(y_hat_i)
```

If the true class is class `k`, this becomes:

```text
CE = -log(y_hat_k)
```

In other words, the model is penalized when it assigns low probability to the correct class.

Key PyTorch point:

```text
model output: raw logits
target: integer class index
loss: nn.CrossEntropyLoss(logits, target)
```

Do not manually apply softmax, and do not manually one-hot encode the target.

### Full Bearing Diagnosis Workflow

```text
Step 1: Load CWRU bearing acceleration data
Step 2: Organize classes: Normal, Ball, Inner Race, Outer Race
Step 3: Split signals into windows
Step 4: Apply a Hann window to reduce spectral leakage
Step 5: Compute FFT
Step 6: Extract time-domain statistics or FFT bins
Step 7: Standardize inputs
Step 8: Train a multiclass neural network
Step 9: Output the probability of each fault class
Step 10: Analyze errors using confusion matrix and per-class recall
```

### Fault Frequency Logic

Different bearing fault locations produce different spectral patterns:

- Normal: mainly shaft rotation frequency and background noise.
- Outer race fault: the outer race is stationary, so impacts are more stable and often produce a clear BPFO peak.
- Inner race fault: the inner race rotates, so the fault enters and leaves the load zone, often producing BPFI and sidebands.
- Ball fault: the ball defect contacts both races, so the spectrum is more complex.

Therefore, FFT is not just a mathematical transformation. It reveals periodic mechanical impacts caused by faults.

### Dropout in Multiclass Classification

When a multiclass network has many parameters, it can memorize the training data. Dropout randomly disables some neurons during training:

```text
training: dropout active
evaluation: dropout disabled
```

This forces the model to learn more distributed and robust representations, reducing overfitting.

### Oral Explanation

> The logic of multiclass bearing diagnosis is to transform vibration signals into features that reflect fault mechanisms, especially frequency-domain features, because different fault locations create different characteristic frequencies. The model outputs one logit for each class, and softmax can interpret them as probabilities. Training uses CrossEntropyLoss, which penalizes the model when it assigns low probability to the true class. During evaluation, I inspect the multiclass confusion matrix because it shows which fault types are confused with each other.

### Common Trap

Do not only look at overall accuracy. If the recall of a minority or safety-critical fault class is low, the model may still be unacceptable in engineering practice.

## 6. CNN Image Classification Algorithm Chain

### Problem Definition

The Week 5 problem is: how can a model recognize defects or classes from images?

```text
Input: image
Output: class label
Task: image classification
Model: CNN
```

### Why CNNs Are Needed

If an image is flattened into a long vector and fed directly into an MLP, two problems occur:

1. Spatial structure is lost.
2. The number of parameters becomes very large, increasing overfitting risk.

The key improvement of CNNs is:

```text
local receptive field + parameter sharing
```

This means a convolution filter looks at local image regions, and the same filter is reused across the entire image.

### Convolution Algorithm Logic

A convolution kernel slides over the image and computes a weighted sum of local pixels:

```text
local patch -> filter -> feature response
```

A typical CNN layer sequence is:

```text
Convolution -> ReLU -> Pooling
```

Early layers learn edges and textures. Deeper layers learn more abstract shapes and class-level features.

### Output Size Formula

```text
output_size = floor((n + 2p - f) / s) + 1
```

where:

- `n` is the input size.
- `p` is padding.
- `f` is filter size.
- `s` is stride.

### Full CNN Classification Workflow

```text
Step 1: Organize images into class folders
Step 2: Use ImageFolder to load images and labels
Step 3: Apply transforms: resize -> tensor -> normalize
Step 4: Use DataLoader to create batches
Step 5: CNN extracts local visual features
Step 6: Pooling reduces spatial size
Step 7: Flatten and pass through fully connected layers
Step 8: Output class logits
Step 9: Train using CrossEntropyLoss
Step 10: Evaluate using confusion matrix, precision, recall, and F1
```

### Internal CNN Knowledge Chain

```text
raw pixels
  -> edges
  -> textures
  -> parts
  -> object-level features
  -> class logits
```

This chain shows that CNNs do not directly "understand" an image. They gradually combine pixels into higher-level features.

### Oral Explanation

> CNNs are suitable for images because they preserve spatial structure. A convolution filter slides across the image and detects local patterns such as edges, textures, or defect shapes. Because of parameter sharing, the same feature detector can work at different image locations, so a CNN has fewer parameters than an MLP. After several layers of convolution, ReLU, and pooling, the model obtains high-level visual features, then a fully connected classifier outputs class logits and the network is trained with CrossEntropyLoss.

### Common Trap

Training data can use random data augmentation, but validation and test data usually should not use random augmentation. Otherwise, evaluation becomes unstable.

## 7. Transfer Learning + Advanced Vision Algorithm Chain

### Problem Definition

In Week 6, the problem is that industrial or medical image datasets are often small. Training a CNN from scratch can overfit. Transfer learning uses visual ability learned from large datasets.

```text
Input: small task-specific image dataset
Output: classes for the current task
Model: pre-trained CNN, for example ResNet-18
Method: transfer learning
```

### Core Principle

A pre-trained CNN trained on a large dataset such as ImageNet learns general visual features:

```text
edges -> textures -> shapes -> object parts
```

Many low-level and mid-level features are useful across tasks. Transfer learning keeps those learned features and replaces or fine-tunes the final classification part.

### ResNet Logic

One reason deep networks are difficult to train is that gradients may not flow well to early layers. ResNet uses shortcut connections:

```text
output = F(x) + x
```

The model learns what should be changed relative to the input, rather than producing the full output from scratch. This makes deeper networks easier to train.

### Full Transfer Learning Workflow

```text
Step 1: Load a pre-trained model, such as ResNet-18
Step 2: Replace the final classifier so the output matches the number of task classes
Step 3: Freeze the backbone and train only the new classification head
Step 4: Validate performance
Step 5: Optionally unfreeze part of the backbone and fine-tune with a smaller learning rate
Step 6: Add data augmentation and a learning rate scheduler
Step 7: Compare SimpleCNN and ResNet results
```

### Data Augmentation Logic

Data augmentation creates realistic variation without changing the label:

```text
original image
  -> random crop / flip / rotation / color jitter
  -> same label
```

Its purpose is to prevent the model from only memorizing the fixed appearance of training images.

### Learning Rate Scheduler Logic

At the beginning of training, a larger learning rate can help the model improve quickly. Later, a smaller learning rate can help fine convergence.

Common strategies:

- StepLR: reduce learning rate every fixed number of epochs.
- CosineAnnealingLR: adjust learning rate smoothly using a cosine schedule.
- ReduceLROnPlateau: reduce learning rate when validation performance stops improving.

### Oral Explanation

> The logic of transfer learning is that a pre-trained model has already learned general visual features from a large image dataset, so we do not need to start from zero on a small dataset. In practice, I load ResNet-18, replace the final classification layer, first freeze the backbone and train the classifier head, and then optionally unfreeze some layers and fine-tune with a smaller learning rate. This uses existing features while adapting the model to the current task. Data augmentation and learning rate scheduling further improve generalization.

### Common Trap

Transfer learning is not always automatically better. If the pre-training domain and the target domain are very different, for example natural images versus MRI images, there is a domain gap. The result must still be verified using the test set and confusion matrix.

## 8. K-means + Unsupervised Anomaly Detection Algorithm Chain

### Problem Definition

In Week 7, the problem is: how can we detect anomalies when enough defect labels are not available?

```text
Input: unlabeled samples or mostly normal samples
Output: normal or anomaly
Task: unsupervised learning / anomaly detection
Model: K-means, density estimation, autoencoder
```

### Core Logic of Unsupervised Learning

Supervised learning learns:

```text
x -> y
```

Unsupervised learning only has:

```text
x
```

The goal is to discover structure in the data, such as clusters, compressed representations, outliers, or latent patterns.

### K-means Principle

K-means assumes the data can be grouped into `K` clusters, and each cluster has a center. The algorithm repeats two operations:

```text
assign points -> update centroids
```

### Full K-means Workflow

```text
Step 1: Choose K
Step 2: Randomly initialize K centroids
Step 3: Assign each sample to the nearest centroid
Step 4: Recompute each centroid as the mean of its assigned samples
Step 5: Repeat Step 3 and Step 4
Step 6: Stop when assignments stop changing or cost stops decreasing
```

Cost function:

```text
J = (1/m) * sum_i ||x_i - mu_{c_i}||^2
```

### K-means for Anomaly Detection

If a sample is far from its nearest cluster center, it may be atypical:

```text
distance_to_nearest_centroid > threshold -> anomaly
```

### Density Estimation Logic

Another anomaly detection approach is to estimate the probability of normal data:

```text
p(x) high -> normal
p(x) low -> anomaly
```

A low-probability sample is unlikely under the normal data distribution.

### Oral Explanation

> The logic of K-means is to first choose K random centers, then repeatedly assign samples to the nearest center and update each center as the mean of its cluster. The final clusters represent typical data patterns. For anomaly detection, if a sample is far from all normal clusters, it does not look like normal data and can be marked as an anomaly. This method is useful when defect labels are limited but normal data is available.

### Common Trap

K and the anomaly threshold are not perfectly determined automatically. K can be selected using the elbow method or engineering requirements, and the anomaly threshold must be chosen using validation data and the cost of false alarms versus missed faults.

## 9. Autoencoder Anomaly Detection Algorithm Chain

### Problem Definition

In Week 7 visual anomaly detection, a common situation is that normal samples are common but defect samples are rare or unknown. Therefore, the model can learn only what normal looks like.

```text
Input: normal images
Training target: reconstruct the input
Anomaly decision: reconstruction error too high or not
Model: convolutional autoencoder
```

### Core Principle

An autoencoder has two parts:

```text
encoder: x -> z
decoder: z -> x_hat
```

`z` is a compressed latent representation. During training, the reconstructed output `x_hat` should be as close as possible to the original input `x`.

### Why It Can Detect Anomalies

If the autoencoder is trained only on normal samples, it becomes good at reconstructing normal patterns. Anomalous regions were not learned well, so reconstruction becomes worse.

```text
normal image -> low reconstruction error
anomaly image -> high reconstruction error
```

### Full Workflow

```text
Step 1: Collect normal-only training images
Step 2: Resize, normalize, and batch the images
Step 3: Encoder compresses the image into latent code
Step 4: Decoder reconstructs the image from latent code
Step 5: Train using reconstruction loss
Step 6: Compute the normal reconstruction-error distribution on validation data
Step 7: Set an anomaly threshold
Step 8: Compute reconstruction error for test images
Step 9: If error > threshold, classify as anomaly
Step 10: Visualize the error map to localize the defect
```

### Loss Function

Common reconstruction error:

```text
MSE_reconstruction = mean((x - x_hat)^2)
```

The error can also be computed per pixel to create a defect localization map.

### ResNet Embeddings + K-means Alternative

The course also uses a fast baseline:

```text
image
  -> pre-trained ResNet feature embedding
  -> K-means clusters of normal features
  -> distance to nearest cluster
  -> anomaly score
```

This method does not reconstruct the image. It decides anomaly based on distance in feature space.

### Oral Explanation

> The core idea of autoencoder anomaly detection is to train the model only on normal samples, so it learns to compress and reconstruct normal images. At test time, if an image is normal, the reconstruction error is low. If it contains a defect, the model has not learned that pattern, so the reconstruction error becomes high. Therefore, reconstruction error can be used as an anomaly score, and a threshold is set using validation data or engineering cost.

### Common Trap

An autoencoder may sometimes reconstruct anomalies well, especially if the model capacity is too high. Therefore, do not trust training loss alone. Check defect test images, error maps, and threshold performance.

## 10. Time Series Forecasting + RNN/LSTM Algorithm Chain

### Problem Definition

The Week 8 problem is: how can we predict future values from historical sequences, for example predicting the next temperature from previous temperature values?

```text
Input: historical sequence
Output: future value
Task: time series forecasting
Model: FFN, RNN, LSTM
```

### Difference Between Time Series and Ordinary Data

Ordinary supervised learning often assumes independent samples. Time series has temporal order:

```text
x_t depends on x_{t-1}, x_{t-2}, ...
```

Therefore, we cannot randomly shuffle time before splitting train and test data. Doing so can leak future information into the training process.

### Time Series Structure

Before modeling, check whether the sequence contains:

- Trend: long-term increase or decrease.
- Seasonality: periodic patterns.
- Autocorrelation: current values related to past values.
- Non-stationarity: statistical properties change over time.
- Structural break: the underlying rule changes suddenly.
- Missing values: missing observations.

### Sliding Window Modeling

Convert the time series into supervised learning samples:

```text
input  = [x_{t-window+1}, ..., x_t]
target = x_{t+1}
```

This is the key bridge between time series and supervised learning.

### FFN Workflow

```text
window of past values -> flatten vector -> FFN -> next value
```

An FFN can use a fixed window, but it does not have internal memory.

### RNN Principle

An RNN updates a hidden state at every time step:

```text
h_t = f(U x_t + W h_{t-1} + b)
y_t = g(V h_t + c)
```

`h_t` is a compressed memory of the past.

Full logic:

```text
x_1 -> h_1
x_2 + h_1 -> h_2
x_3 + h_2 -> h_3
...
h_t -> prediction
```

### RNN Problem

When an RNN is unrolled through time, it becomes similar to a very deep network. In long sequences, gradients may shrink step by step, so early information cannot strongly influence later outputs. This is the vanishing gradient problem.

### LSTM Principle

LSTM adds a cell state and gate mechanisms:

- Forget gate: decides how much old information to keep.
- Input gate: decides how much new information to write.
- Output gate: decides how much memory to expose as output.

Logic chain:

```text
previous memory
  -> forget irrelevant part
  -> add useful new information
  -> expose relevant memory for prediction
```

### Full Time-Series Forecasting Workflow

```text
Step 1: Sort data by time
Step 2: Handle missing values and outliers
Step 3: Fit the scaler only on the training set
Step 4: Generate input-target pairs using sliding windows
Step 5: Split train/validation/test by time
Step 6: Train FFN, RNN, or LSTM
Step 7: Evaluate using MAE, RMSE, and MSE
Step 8: For multi-step prediction, remember that errors can recursively accumulate
```

### Evaluation Metrics

```text
error = forecast - actual
MSE = mean(error^2)
RMSE = sqrt(MSE)
MAE = mean(abs(error))
MAPE = mean(abs(error / actual))
```

### Oral Explanation

> The key point in time-series forecasting is that samples are not completely independent. First, I split the data in time order, then use a sliding window to convert the past sequence into input and the next value into the target. An FFN can process a fixed window but has no internal memory. An RNN keeps a hidden state to carry past information forward. An LSTM improves this by using forget, input, and output gates to control long-term memory and reduce the vanishing gradient problem.

### Common Trap

The scaler must be fitted only on the training set and then applied to validation and test sets. Otherwise, statistics from the test set leak into the training pipeline.

## 11. Object Detection Algorithm Chain

### Problem Definition

The Week 9 problem is: an image may contain several objects, and we need to know both what they are and where they are.

```text
Input: image
Output: class label + bounding box
Task: object detection
Model: R-CNN family, YOLO, SSD, DETR, RT-DETR
```

### Difference Between Classification and Detection

Image classification:

```text
image -> class
```

Object detection:

```text
image -> [(class_1, box_1), (class_2, box_2), ...]
```

Object detection must solve two problems:

1. What object is it?
2. Where is it?

### Bounding Box Representation

Common formats:

```text
(x1, y1, x2, y2)
```

or:

```text
(x_center, y_center, width, height)
```

### R-CNN Family Logic

R-CNN:

```text
image
  -> region proposals
  -> crop each region
  -> CNN feature extraction
  -> classifier + bounding box regressor
```

The problem is that it is slow, because every candidate region must be processed.

Fast R-CNN:

```text
image
  -> CNN feature map once
  -> RoI pooling for proposals
  -> classification + box regression
```

Faster R-CNN:

```text
image
  -> shared CNN feature map
  -> Region Proposal Network
  -> detector head
```

The improvement is that region proposal generation is integrated into the network.

### YOLO Logic

YOLO treats object detection as a single forward-pass regression problem:

```text
image
  -> grid / feature maps
  -> boxes + objectness + class scores
  -> final detections
```

It does not generate separate proposals first, so it is fast and suitable for real-time applications.

### DETR / RT-DETR Logic

DETR uses the transformer idea:

```text
image features
  -> transformer encoder-decoder
  -> fixed set of object queries
  -> boxes + classes
```

It directly predicts a set of objects and reduces dependence on hand-designed anchors and NMS. RT-DETR is a more engineering-oriented version for real-time detection.

### IoU and mAP Evaluation Chain

IoU:

```text
IoU = area(pred_box ∩ true_box) / area(pred_box ∪ true_box)
```

A detection is correct only if the class is correct and the predicted box overlaps sufficiently with the ground truth box.

mAP logic:

```text
predictions
  -> match with ground truth using IoU threshold
  -> compute precision-recall curve per class
  -> compute AP per class
  -> average AP over classes = mAP
```

Common metrics:

- `mAP@0.5`: a detection is accepted if IoU is at least 0.5. This is relatively loose.
- `mAP@0.5:0.95`: average over multiple IoU thresholds. This is stricter.

### Full Object Detection Workflow

```text
Step 1: Collect images
Step 2: Label bounding boxes using CVAT, Label Studio, or Roboflow
Step 3: Split train/validation/test
Step 4: Choose YOLO or RT-DETR
Step 5: Train the detector
Step 6: Adjust confidence threshold
Step 7: Evaluate using IoU, precision, recall, and mAP
Step 8: Run inference on images or video
Step 9: Analyze missed detections, false detections, and poor localization
```

### Oral Explanation

> Object detection is different from image classification because it predicts both object category and location. The R-CNN family first finds candidate regions and then classifies them, and later versions integrate proposal generation into the network. YOLO treats detection as a single regression problem, so one forward pass outputs both boxes and classes, which makes it fast. DETR uses a transformer to directly predict a set of objects. Evaluation must use IoU to check whether boxes match, and mAP to summarize detection performance across classes and thresholds.

### Common Trap

For object detection, accuracy alone is not enough. A model can predict the correct class but produce a badly placed box, which still fails the detection task.

## 12. LLM Transformer Algorithm Chain

### Problem Definition

The Week 10 problem is: how do Large Language Models process text, and how can they support engineering analysis?

```text
Input: text prompt or engineering sensor description
Output: next tokens / explanation / decision suggestion
Model: Transformer-based LLM
```

### Core Training Objective of LLMs

The basic objective of an LLM is next-token prediction:

```text
given previous tokens -> predict next token
```

By repeating this task on massive text data, the model learns language structure, factual patterns, code patterns, and reasoning-like patterns.

### Tokenization Chain

An LLM does not directly process raw text. It uses:

```text
text
  -> tokens
  -> token IDs
  -> embeddings
```

Tokens can be words, subwords, or character pieces. Embeddings convert token IDs into high-dimensional vectors so the model can process semantic relationships in vector space.

### Transformer Core Logic

RNNs process text sequentially, which is difficult to parallelize and weaker for long-range dependencies. Transformers use self-attention, allowing each token to attend directly to other tokens in the context.

Three roles in self-attention:

- Query: what the current token is looking for.
- Key: what other tokens offer as an index.
- Value: the actual information carried by other tokens.

Logic chain:

```text
token embeddings
  -> add positional information
  -> self-attention computes relationships
  -> feed-forward network transforms features
  -> stacked transformer blocks
  -> next-token probability distribution
```

### Multi-Head Attention

One attention head can look at the context from one perspective. Multi-head attention learns multiple relationships at the same time, such as:

- grammatical relations;
- semantic relations;
- reference relations;
- conditional and logical relations.

### Positional Encoding

Transformers view the whole sequence at once and do not naturally know token order, so positional information must be added. Otherwise, "man bites dog" and "dog bites man" would contain similar tokens but have very different meanings.

### Training and Alignment Workflow

```text
Step 1: Self-supervised pretraining
        Predict the next token on large-scale text to obtain a base model

Step 2: Supervised fine-tuning
        Train on high-quality question-answer or instruction examples

Step 3: RLHF or similar preference optimization
        Align responses with human preferences for usefulness and safety
```

### Engineering Application Workflow

The engineering monitoring application in the course can be abstracted as:

```text
sensor data
  -> format as prompt
  -> LLM reads context and thresholds
  -> LLM gives explanation or warning
  -> system logs result
  -> human verifies important decisions
```

### Limitation Logic

The LLM training objective is to generate statistically plausible tokens, not to guarantee factual truth. Therefore, hallucination can occur.

In engineering applications, we must:

- provide clear context and thresholds;
- verify important conclusions;
- avoid using an LLM as the only decision source in a safety-critical system;
- log outputs for traceability.

### Oral Explanation

> The core of an LLM is using a Transformer for next-token prediction. Text is first split into tokens, then converted into embeddings. Self-attention lets each token decide which other tokens in the context are relevant, and multi-head attention models grammar, semantics, and logic from multiple perspectives. Pretraining teaches broad language patterns, while fine-tuning and human feedback make the model better at following instructions. In engineering, an LLM can help explain sensor data and generate reports, but because it can hallucinate, important decisions must be verified.

### Common Trap

Do not say that an LLM "understands the real world" or "has intention." A more accurate statement is: it learns language and patterns from large-scale data and can generate useful responses, but the output requires factual verification.

## 13. MRI Mini-project Complete Project Algorithm Chain

### Problem Definition

The mini-project is MRI brain neurological class classification.

```text
Input: MRI image
Output: dementia class
Task: multiclass image classification
Model: SimpleCNN baseline + ResNet-18 transfer learning
```

### Project Knowledge Chain

```text
medical image classification problem
  -> dataset exploration
  -> preprocessing and DataLoader
  -> baseline CNN
  -> transfer learning with ResNet-18
  -> training with CrossEntropyLoss
  -> evaluation with confusion matrix
  -> error analysis and limitations
```

### Data Workflow

```text
Step 1: Check folder classes
Step 2: Count samples in each class
Step 3: Inspect example MRI images
Step 4: Resize to model input size
Step 5: Convert to tensor
Step 6: Normalize
Step 7: Use DataLoader to create batches
```

### Baseline: SimpleCNN

The role of SimpleCNN is not necessarily to get the best result. It creates a baseline:

```text
MRI image
  -> convolution layers
  -> ReLU
  -> pooling
  -> fully connected classifier
  -> class logits
```

The baseline helps answer:

- Is the task learnable?
- How well does a simple model perform?
- Does a more advanced model truly improve the result?

### Transfer Learning: ResNet-18

ResNet-18 logic:

```text
pre-trained visual features
  -> replace final layer
  -> train on MRI classes
  -> optionally fine-tune
```

Reasons for using it:

- MRI dataset size is limited, so training from scratch can overfit.
- ResNet has learned general edge, texture, and shape features.
- Residual connections make deeper models easier to train.

### Loss Function and Output

The four-class task uses CrossEntropyLoss:

```text
model output: four logits
target: class index
loss: CrossEntropyLoss
```

Prediction:

```text
predicted_class = argmax(logits)
```

### Evaluation Chain

```text
test images
  -> model predictions
  -> confusion matrix
  -> per-class precision / recall / F1
  -> inspect wrong predictions
```

In medical imaging, overall accuracy is not enough. Per-class recall must be inspected, especially for minority classes or clinically important classes.

### Key Limitations

1. Data leakage risk  
   If different MRI slices from the same patient appear in both train and test sets, the result can be overly optimistic.

2. Class imbalance  
   If one class has few samples, the model may prefer majority classes.

3. Domain gap  
   ResNet is pre-trained on natural images, while MRI images are medical grayscale images with a different distribution.

4. Medical reliability  
   The model output should not be treated as a direct diagnosis. It can only support analysis.

### Oral Explanation

> My mini-project is MRI image multiclass classification. The full workflow is to first explore the dataset and class distribution, then build preprocessing and DataLoader. I train a SimpleCNN as a baseline to understand how CNNs extract local features from MRI images. Then I use ResNet-18 for transfer learning because the dataset is limited and a pre-trained model provides more stable visual features. I use CrossEntropyLoss because it is a multiclass problem. For evaluation, I look at the confusion matrix and recall for each class, not only accuracy. Finally, I discuss limitations such as data leakage, class imbalance, and reliability in medical applications.

### Common Trap

Do not only say "I used ResNet and it worked better." Explain why a SimpleCNN baseline is needed, why transfer learning fits small image datasets, and why medical imaging requires confusion matrix analysis and data leakage discussion.

## 14. Connections Between Course Algorithms

### Model Evolution from Simple to Complex

```text
Linear regression
  -> one neuron
  -> MLP
  -> CNN for images
  -> ResNet transfer learning
  -> object detection models
```

This line shows that model structures become more complex, but the training framework remains:

```text
prediction -> loss -> gradient -> update
```

### Choosing Algorithms by Task Type

```text
continuous output
  -> regression
  -> MSE / MAE

binary label
  -> sigmoid classifier
  -> BCE / recall / F1

multiple labels
  -> softmax classifier
  -> CrossEntropyLoss / confusion matrix

image label
  -> CNN / ResNet
  -> CrossEntropyLoss

no anomaly labels
  -> K-means / autoencoder
  -> anomaly score + threshold

ordered sequence
  -> RNN / LSTM
  -> forecast metrics

class + location
  -> YOLO / RT-DETR
  -> IoU / mAP

text reasoning
  -> Transformer LLM
  -> verification required
```

### Choosing Evaluation Metrics by Engineering Reliability

Whether a model is good depends on task cost:

- Regression: is the numerical error acceptable?
- Fault detection: are missed faults sufficiently rare?
- Multiclass diagnosis: can every class be recognized?
- Medical imaging: is recall reliable for minority and severe classes?
- Object detection: are the boxes accurately localized?
- LLM: is the output verifiable, and can it produce unsafe or incorrect suggestions?

## 15. Final Oral Exam Template

When the examiner asks about any algorithm, answer in this order:

```text
1. What problem does this algorithm solve?
2. What are the inputs and outputs?
3. What is the core assumption or principle?
4. What is the basic process from input to output?
5. What loss is used during training?
6. How is the result evaluated?
7. What are the engineering limitations?
```

Example final answer:

> For an AML problem, I first define the input and output, then identify the task type. If it is regression, I use MSE to optimize continuous output. If it is classification, I use sigmoid or softmax with cross entropy. If it is an image task, I use CNNs or transfer learning. If there are no anomaly labels, I use an autoencoder or clustering to compute an anomaly score. If it is a sequence, I use sliding windows and RNN/LSTM. If it is object detection, I predict both class and bounding box and evaluate with IoU and mAP. The common idea behind all these algorithms is to learn model parameters from data through loss and gradient descent, then evaluate using metrics that match the engineering cost.

## 16. Coverage Check

This version covers the course content by algorithm chain:

- Lecture 01: machine learning tasks, supervised/unsupervised/reinforcement learning, digital twin, linear regression, MSE, gradient descent.
- Lecture 02: MLP, activation functions, loss landscape, backpropagation, Adam, data split, hyperparameters.
- Lecture 03: logistic regression, sigmoid, BCE, threshold, FFT features, precision, recall, F1, ROC-AUC.
- Lecture 04: multiclass classification, softmax, CrossEntropyLoss, dropout, CWRU bearing fault diagnosis.
- Lecture 05: CNN, convolution, padding, stride, pooling, image classification workflow.
- Lecture 06: LeNet, AlexNet, VGG, ResNet, Inception, augmentation, scheduler, transfer learning.
- Lecture 07: unsupervised learning, K-means, density estimation, autoencoder, visual anomaly detection.
- Lecture 08: time series, sliding window, FFN, RNN, LSTM, forecast metrics.
- Lecture 09: object detection, R-CNN, Fast/Faster R-CNN, YOLO, SSD, DETR, RT-DETR, IoU, mAP.
- Lecture 10: LLM, tokenization, embedding, Transformer, self-attention, pretraining, SFT/RLHF, hallucination.
- Mini-project: MRI image multiclass classification, SimpleCNN, ResNet-18, CrossEntropyLoss, confusion matrix, project limitations.

