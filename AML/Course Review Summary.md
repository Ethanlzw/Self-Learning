# Advanced Machine Learning Course Review Summary (English Version)

## 0. Exam And Course Requirements

### Exam Format

- Course: Advanced Machine Learning, 5 ECTS, Master level.
- Exam: oral examination based on an individually handed-in mini-project. The mini-project is included in the assessment.
- Lecture note requirement: mini-project hand-in, about 5 report pages, including code and data; oral exam about 20 minutes, discussing the mini-project plus AML theory questions.
- Revision priority: do not only know how to run notebooks. Be ready to explain why each step is used, what the model input/output is, what the loss and metrics mean, and what limitations remain.

### Course Scope

- Neural networks for regression and classification.
- Deep learning for computer vision: classification and detection.
- Deep learning for time series.
- Learning techniques: splitting, normalization, tuning, regularization, learning-rate scheduling, transfer learning.
- Anomaly detection and predictive maintenance.
- Engineering cases: motor digital twin, bearing fault diagnosis, industrial image inspection, visual anomaly detection, warehouse object detection, and MRI image classification.

### Expected Competences

You should be able to explain:

- How to prepare a dataset: cleaning, splitting, normalization, augmentation, batching.
- How to train MLPs, CNNs, RNNs, LSTMs, autoencoders, pretrained ResNets, and YOLO/RT-DETR detectors.
- How to identify the task type: regression, binary classification, multi-class classification, unsupervised anomaly detection, object detection, and time-series forecasting.
- How learning works: forward pass, loss function, backpropagation, optimizer, learning rate.
- How to evaluate models: MSE/MAE/RMSE/R2, accuracy, precision, recall, F1, ROC-AUC, confusion matrix, IoU, mAP.
- How to apply ML to mechatronic systems: sensor noise, asymmetric fault cost, data leakage, train/test separation, deployment constraints.

## 1. Course Knowledge Map

| Week | Topic | Main Algorithms/Models | Key Question |
|---|---|---|---|
| W1 | Digital twin and linear regression | DC motor simulator, linear regression, MSE, gradient descent | How can simulation generate training data? |
| W2 | Neural network learning | MLP, backpropagation, Adam, train/val/test split | How does a neural network tune parameters automatically? |
| W3 | Binary classification | Logistic regression, sigmoid, BCE, FFT features, ROC-AUC | How do we detect whether a machine is faulty? |
| W4 | Multi-class fault diagnosis | Softmax, CrossEntropyLoss, FFT bins, CWRU bearing data | How do we identify the fault type? |
| W5 | CNN fundamentals | Convolution, pooling, ImageFolder, SimpleCNN | Why are CNNs better than MLPs for images? |
| W6 | Advanced vision and transfer learning | Augmentation, StepLR, ResNet-18, fine-tuning | How do we use pretrained models for small visual datasets? |
| W7 | Unsupervised learning and anomaly detection | K-means, density/anomaly detection, convolutional autoencoder | How do we detect defects without defect labels? |
| W8 | Time series and sequence models | Sliding windows, FFN, RNN, LSTM, recursive forecast | How do we use history to predict the future? |
| W9 | Object detection | R-CNN, Fast/Faster R-CNN, YOLO, SSD, DETR/RT-DETR, mAP | How do we predict both class and location? |
| Project | MRI classification | SimpleCNN, ResNet-18 transfer learning, confusion matrix | Exam focus: how do we explain the full project pipeline? |

## 2. Core Machine Learning Concepts

### Supervised, Unsupervised, Reinforcement Learning

- Supervised learning: data is `(x, y)`, and the model learns a mapping from input to output. Regression, binary classification, multi-class classification, image classification, and object detection in this course are supervised tasks.
- Unsupervised learning: only `x` is available. Goals include clustering, anomaly detection, compression, generation, denoising, and discovering structure.
- Reinforcement learning: an agent chooses actions in states and receives rewards. It is introduced conceptually, but not a main implementation topic in this course.

### Regression And Classification

- Regression: output is continuous, such as motor speed or temperature. Common loss: MSE.
- Binary classification: output is a class or positive-class probability, such as healthy/faulty. Common setup: sigmoid + binary cross entropy.
- Multi-class classification: output is one of several classes, such as bearing fault type or MRI class. Common setup: logits + CrossEntropyLoss.
- Multi-output tasks: segmentation, depth estimation, and pose estimation were introduced conceptually.

### Parameters And Hyperparameters

- Parameters: weights and biases updated during training.
- Hyperparameters: settings chosen by the engineer, such as learning rate, batch size, epochs, hidden units, dropout, augmentation, and scheduler.
- Key point: learning rate is usually the first hyperparameter to tune. Too high can diverge; too low can train slowly or get stuck.

### Data Splitting

- Train set: updates model parameters.
- Validation set: tunes hyperparameters, early stopping, model selection.
- Test set: used only once at the end for unbiased evaluation.
- Time-series data should not be randomly shuffled because future information can leak into training.
- Medical image datasets with multiple slices per patient should ideally use patient-level splitting. In the MRI project, filenames do not contain reliable patient/session IDs, so the notebook correctly reports this limitation.

## 3. W1: Digital Twin, Linear Regression, MSE, Gradient Descent

### Core Idea

W1 uses a DC motor digital twin to show that a real system can be simulated first, then the simulated or measured data can train a model.

Simplified motor equation:

```text
J * dω/dt = K * V - b * ω
dω/dt = (K * V - b * ω) / J
ω_new = ω_old + dω/dt * dt
```

Meaning:

- `J`: moment of inertia; larger values resist acceleration.
- `K`: motor constant; converts voltage to torque.
- `b`: viscous friction; higher speed creates more drag.
- `V`: input voltage.
- `ω`: output angular velocity.

### Linear Regression Logic

The simplest model:

```text
speed = w * voltage + b
```

The goal is to find `w` and `b` that minimize prediction error. The loss is MSE:

```text
MSE = mean((y_pred - y_true)^2)
```

Advantages:

- Differentiable, suitable for gradient descent.
- Strongly penalizes large errors.
- Natural for continuous regression.

Limitations:

- Sensitive to outliers.
- Usually not appropriate for classification compared with cross entropy.

### Gradient Descent

Update rule:

```text
w := w - α * ∂J/∂w
b := b - α * ∂J/∂b
```

Interpretation:

- The gradient points in the steepest loss-increasing direction.
- Subtracting the gradient moves downhill.
- `α` is the learning rate.

### Notebook Key Code

```python
class DCMotor:
    def __init__(self, J=0.1, b=0.5, K=2.0):
        self.J = J
        self.b = b
        self.K = K
        self.omega = 0.0

    def update(self, voltage, dt):
        torque_electrical = self.K * voltage
        torque_friction = self.b * self.omega
        torque_net = torque_electrical - torque_friction
        angular_acceleration = torque_net / self.J
        self.omega += angular_acceleration * dt
        return self.omega
```

Dataset generation logic:

```python
dataset = {"time": [], "voltage": [], "speed_true": [], "speed_measured": []}

for i in range(steps):
    if i % int(0.5 / dt) == 0:
        current_voltage = np.random.uniform(-12, 12)

    motor.update(current_voltage, dt)
    dataset["time"].append(i * dt)
    dataset["voltage"].append(current_voltage)
    dataset["speed_true"].append(motor.omega)
    dataset["speed_measured"].append(motor.measure_speed())
```

Exam explanation:

- A digital twin provides controllable, repeatable, low-cost experiments.
- Noise is not a bug; it is part of real sensors. A model trained only on perfect data may fail after deployment.
- W1 data is reused in W2 to train a neural network.

## 4. W2: Neural Networks, Backpropagation, Training Loop

### Neural Network Blocks

One neuron computes:

```text
z = w · x + b
a = activation(z)
```

A multi-layer network combines many linear transformations and nonlinear activations. Without activations, deep linear layers collapse into one linear model.

Common activations:

- ReLU: `max(0, x)`, default hidden-layer choice; fast and reduces vanishing-gradient issues.
- Sigmoid: output `(0, 1)`, useful for binary probabilities and gates.
- Tanh: output `(-1, 1)`, zero-centered and often used inside RNN/LSTM state updates.
- Softmax: converts multi-class logits into probabilities summing to 1.

### Backpropagation

Backpropagation is the chain rule applied backward from the loss to every parameter.

Simple single-sample example:

```text
y_hat = w * x
L = (y_hat - y)^2
∂L/∂w = 2 * (y_hat - y) * x
w_new = w - lr * ∂L/∂w
```

### Four Parts Of A PyTorch Pipeline

| Component | Purpose |
|---|---|
| `nn.Module` | Defines architecture |
| loss/criterion | Defines what “wrong” means |
| optimizer | Updates parameters using gradients |
| data/tensor | Provides training inputs and labels |

### Standard Training Loop

```python
for epoch in range(epochs):
    optimizer.zero_grad()
    predictions = model(X_train)
    loss = criterion(predictions, Y_train)
    loss.backward()
    optimizer.step()
```

Important details:

- `optimizer.zero_grad()` is required because PyTorch accumulates gradients.
- `loss.backward()` computes parameter gradients.
- `optimizer.step()` applies the update.
- Evaluation should use `model.eval()` and `torch.no_grad()`.

### W2 Notebook Model

Input is `[speed_measured(t), voltage(t)]`; target is `speed_measured(t+H)`.

```python
class MotorNet(nn.Module):
    def __init__(self):
        super().__init__()
        self.fc1 = nn.Linear(2, 32)
        self.fc2 = nn.Linear(32, 16)
        self.fc3 = nn.Linear(16, 1)
        self.relu = nn.ReLU()

    def forward(self, x):
        x = self.relu(self.fc1(x))
        x = self.relu(self.fc2(x))
        return self.fc3(x)
```

Loss and evaluation:

- Regression uses `nn.MSELoss()`.
- Test metrics include MSE, RMSE, MAE, and R2.
- Low training error but high test error indicates overfitting.
- High training error indicates underfitting or failed optimization.

## 5. W3: Binary Classification, Logistic Regression, FFT, Fault Metrics

### From Linear Regression To Logistic Regression

Linear regression outputs any real number, so it is not suitable as a probability. Logistic regression applies sigmoid:

```text
z = w · x + b
p(y=1|x) = sigmoid(z) = 1 / (1 + exp(-z))
```

Typical threshold:

```text
if p >= 0.5 -> class 1
else -> class 0
```

### Binary Cross Entropy

```text
BCE = -[y log(p) + (1-y) log(1-p)]
```

Why it is used:

- Heavily penalizes confident wrong predictions.
- Matches probabilistic interpretation.
- Better suited to classification than MSE.

### Industrial Maintenance Metrics

Confusion matrix:

- TP: true fault predicted as fault.
- TN: true healthy predicted as healthy.
- FP: healthy predicted as fault, false alarm.
- FN: fault predicted as healthy, missed fault.

Metrics:

- Precision = `TP / (TP + FP)`: alarm quality.
- Recall = `TP / (TP + FN)`: fraction of real faults caught.
- F1 = `2PR / (P + R)`: balance of precision and recall.
- ROC-AUC: separability across thresholds.

Key point: in industrial fault detection, false negatives often cost far more than false positives, so recall is often prioritized.

### W3 Notebook Algorithm

1. Generate healthy and faulty vibration signals.
2. Use Hann window + FFT to reveal frequency peaks.
3. Extract time/frequency features.
4. Standardize features.
5. Train a binary neural network.
6. Evaluate with confusion matrix, classification report, and ROC-AUC.

Key code:

```python
def generate_vibration_signal(is_faulty=False, duration=DURATION, fs=FS):
    t = np.linspace(0, duration, int(fs * duration), endpoint=False)
    sig = np.sin(2 * np.pi * F_SHAFT * t)
    sig += 0.3 * np.random.normal(size=len(t))
    if is_faulty:
        sig += 0.5 * np.sin(2 * np.pi * F_FAULT * t)
    return t, sig
```

```python
def compute_fft_spectrum(vib, fs=FS):
    N = len(vib)
    window = scipy_signal.windows.hann(N)
    vib_windowed = vib * window
    fft_values = fft(vib_windowed)
    freqs = fftfreq(N, 1 / fs)
    pos_mask = freqs > 0
    return freqs[pos_mask], np.abs(fft_values[pos_mask]) / N
```

```python
class FaultDetector(nn.Module):
    def __init__(self, n_features=7):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(n_features, 16),
            nn.ReLU(),
            nn.Linear(16, 8),
            nn.ReLU(),
            nn.Linear(8, 1),
            nn.Sigmoid(),
        )

    def forward(self, x):
        return self.net(x)
```

## 6. W4: Multi-Class Classification, Softmax, CrossEntropyLoss, Bearing Diagnosis

### Binary To Multi-Class

| Task | Output Layer | Loss |
|---|---|---|
| Regression | One or more linear outputs | MSE/MAE |
| Binary classification | One sigmoid probability | BCE |
| Multi-class classification | K logits | CrossEntropyLoss |

Softmax:

```text
softmax(z_i) = exp(z_i) / sum_j exp(z_j)
```

Cross entropy:

```text
L = -sum_i y_i log(p_i)
```

PyTorch details:

- `nn.CrossEntropyLoss()` expects raw logits, not manual softmax probabilities.
- Labels should be integer class indices such as `0, 1, 2, 3`; manual one-hot encoding is not needed.

### One-Hot Encoding

Class IDs have no numeric order. `Normal=0` and `Ball=1` does not mean Ball is twice Normal. One-hot encoding represents which class is correct without imposing a false scale.

### Dropout

Dropout randomly silences neurons during training, forcing the network not to rely on one path and reducing overfitting.

```text
model.train() -> dropout active
model.eval()  -> dropout disabled
```

### FFT And Bearing Faults

The time domain shows what happened; the frequency domain helps identify the mechanism.

- Normal: mainly shaft rotation frequency and broadband noise.
- Outer race fault: stable peak near BPFO.
- Inner race fault: BPFI and sidebands because the defect moves in and out of the load zone.
- Ball fault: more complex pattern around BSF.

Nyquist theorem:

```text
sampling frequency >= 2 * highest frequency of interest
```

### W4 Notebook Workflow

1. Load drive-end accelerometer signals from CWRU `.mat` files.
2. Downsample and slice fixed-length windows.
3. Compute Hann-windowed FFT.
4. Verify BPFO/BPFI/BSF frequencies.
5. Extract 10 time-domain features or 102 FFT-bin features.
6. Train a four-class network.
7. Compare confusion matrices for time-domain features and FFT features.

Core model:

```python
class BearingDiagnosticNet(nn.Module):
    def __init__(self, input_size=10, hidden1=64, hidden2=64, num_classes=4):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(input_size, hidden1),
            nn.ReLU(),
            nn.Linear(hidden1, hidden2),
            nn.ReLU(),
            nn.Linear(hidden2, num_classes),
        )

    def forward(self, x):
        return self.net(x)

criterion = nn.CrossEntropyLoss()
optimizer = optim.Adam(model.parameters(), lr=0.001)
```

FFT features:

```python
def extract_fft_features(win, n_bins=N_BINS, fs=FS):
    freqs, mag = compute_fft_spectrum(win, fs=fs)
    return mag[:n_bins]
```

Exam emphasis:

- Engineering diagnosis should inspect per-class recall, not only accuracy.
- FFT-bin features often preserve more diagnostic frequency information than a few hand-crafted statistics.
- Confusion matrices identify which fault classes are confused and guide feature improvement.

## 7. W5: CNN Fundamentals And Industrial Defect Classification

### Why Not A Plain MLP For Images

An MLP flattens the image and loses local spatial structure. The same crack in different image positions becomes a different input pattern. CNNs use shared filters that slide over the image, making local pattern detection far more efficient.

### Convolution

A kernel, such as `3x3`, slides over the image and computes local weighted sums, producing a feature map.

Output size:

```text
output = floor((n + 2p - f) / s) + 1
```

- `n`: input size.
- `p`: padding.
- `f`: kernel/filter size.
- `s`: stride.

### Pooling

- Max pooling: keeps strongest local response and reduces spatial size.
- Average pooling: averages local values and smooths the representation.

### CNN Advantages

- Parameter sharing: the same filter can detect the same pattern anywhere.
- Sparse local connections: fewer parameters than an MLP.
- Translation equivariance: when the object shifts, feature activations shift too.

### W5 Notebook Workflow

1. `ImageFolder` builds labels from subfolder names.
2. `transforms` resize, convert to grayscale, tensor, normalize.
3. `DataLoader` feeds batches.
4. A three-block CNN is trained.
5. CrossEntropyLoss handles binary image classification.
6. The confusion matrix is used to analyze dangerous “defective predicted OK” errors.

Key code:

```python
transform = transforms.Compose([
    transforms.Resize((IMG_SIZE, IMG_SIZE)),
    transforms.Grayscale(num_output_channels=1),
    transforms.ToTensor(),
    transforms.Normalize(mean=(0.5,), std=(0.5,)),
])

train_dataset = ImageFolder(TRAIN_DIR, transform=transform)
train_loader = DataLoader(train_dataset, batch_size=BATCH_SIZE, shuffle=True)
```

```python
class DefectCNN(nn.Module):
    def __init__(self):
        super().__init__()
        self.features = nn.Sequential(
            nn.Conv2d(1, 8, 3, padding=1), nn.ReLU(), nn.MaxPool2d(2),
            nn.Conv2d(8, 16, 3, padding=1), nn.ReLU(), nn.MaxPool2d(2),
            nn.Conv2d(16, 32, 3, padding=1), nn.ReLU(), nn.MaxPool2d(2),
        )
        self.classifier = nn.Sequential(
            nn.Flatten(),
            nn.Linear(32 * 8 * 8, 64),
            nn.ReLU(),
            nn.Linear(64, 2),
        )

    def forward(self, x):
        return self.classifier(self.features(x))
```

## 8. W6: Advanced Vision, Augmentation, Scheduling, Transfer Learning

### Classic CNN Architectures

- LeNet-5: early digit-recognition CNN with convolution, pooling, and fully connected layers.
- AlexNet: key ImageNet-era deep CNN.
- VGG: repeated `3x3` convolutions; simple but parameter-heavy.
- ResNet: residual connections help train deep networks.
- Inception: multi-scale convolution branches, often using `1x1` convolutions for dimensionality reduction.

### ResNet Residual Connection

Core idea:

```text
output = F(x) + x
```

The model learns a residual `F(x)`. If a block is not needed, it can learn near-zero instead of damaging the signal.

### Data Augmentation

Randomly modifies training images to improve generalization.

Common methods:

- Geometric: RandomResizedCrop, HorizontalFlip, VerticalFlip, Rotation, Affine.
- Photometric: ColorJitter, GaussianBlur, RandomGrayscale.

Important: validation/test transforms must be deterministic, usually resize and normalize only.

### Learning-Rate Scheduling

- `StepLR`: multiplies learning rate by gamma after fixed intervals.
- `ReduceLROnPlateau`: reduces learning rate when validation performance stalls.
- `CosineAnnealingLR`: smoothly decays learning rate; common in fine-tuning.

### Transfer Learning

Transfer learning reuses features learned on a large dataset, such as ImageNet, for a new task.

Two modes:

- Feature extraction: freeze backbone, train only the classifier head.
- Fine-tuning: unfreeze some or all layers and continue training with a smaller learning rate.

W6 notebook key code:

```python
model_adv = models.resnet18(weights=models.ResNet18_Weights.DEFAULT)
model_adv.fc = nn.Linear(model_adv.fc.in_features, NUM_CLASSES)

for param in model_adv.parameters():
    param.requires_grad = False
for param in model_adv.fc.parameters():
    param.requires_grad = True
```

Two-phase training:

```python
# Phase 1: train classifier head only
optimizer_ph1 = optim.Adam(filter(lambda p: p.requires_grad, model_adv.parameters()), lr=LR_PHASE1)

# Phase 2: unfreeze all layers and fine-tune with lower LR
for param in model_adv.parameters():
    param.requires_grad = True
optimizer_ph2 = optim.Adam(model_adv.parameters(), lr=LR_PHASE2)
```

Advantages:

- More stable with limited data.
- Faster than training from scratch.
- Often outperforms a small SimpleCNN.

Limitations:

- Source domain and target domain may differ.
- Model is larger and more expensive to deploy.
- Leakage or unrealistic augmentation can inflate results.

## 9. W7: Unsupervised Learning, K-Means, Anomaly Detection, Autoencoder

### Unsupervised Learning Goals

- Clustering: discover groups.
- Anomaly detection: find samples far from normal structure.
- Generation: learn a distribution and produce new samples.
- Compression/denoising: learn compact representations.

### K-Means

Algorithm:

1. Choose number of clusters `K`.
2. Randomly initialize K centroids.
3. Assign each point to the nearest centroid.
4. Move each centroid to the mean of its assigned points.
5. Repeat until convergence.

Objective:

```text
J = mean(||x_i - μ_c(i)||^2)
```

Pros:

- Simple, fast, interpretable.

Cons:

- Requires choosing K.
- Sensitive to initialization, scale, and outliers.
- Prefers spherical clusters.

### Anomaly Detection

Core idea:

```text
If p(x_test) is low, or x_test is far from the normal distribution, mark it anomalous.
```

Appropriate when:

- Normal samples are common and abnormal samples are rare.
- Defect types are unknown or changing.
- Use cases include industrial inspection, machine monitoring, and fraud detection.

### Autoencoder

An autoencoder has an encoder and decoder:

```text
x -> encoder -> z -> decoder -> x_hat
```

Training objective:

```text
loss = MSE(x_hat, x)
```

Anomaly logic:

- Train only on normal images.
- Normal images reconstruct well and have low error.
- Anomalous images reconstruct poorly and have high error.
- Set a threshold using reconstruction errors from normal validation samples.

W7 notebook key code:

```python
class ConvAutoencoder(nn.Module):
    def __init__(self):
        super().__init__()
        self.encoder = Encoder()
        self.decoder = Decoder()

    def forward(self, x):
        z = self.encoder(x)
        return self.decoder(z)

criterion = nn.MSELoss()
```

```python
def compute_anomaly_scores(model, loader, device):
    model.eval()
    all_scores, all_labels = [], []
    with torch.no_grad():
        for images, labels in loader:
            images = images.to(device)
            xhat = model(images)
            mse = ((xhat - images) ** 2).mean(dim=(1, 2, 3))
            all_scores.append(mse.cpu().numpy())
            all_labels.append(labels.numpy())
    return np.concatenate(all_scores), np.concatenate(all_labels)
```

Thresholding:

```python
threshold = np.percentile(val_scores, THRESHOLD_PCT)
preds = (test_scores > threshold).astype(int)
```

## 10. W8: Time Series, RNN, LSTM

### Time-Series Concepts

- Trend: long-term increase or decrease.
- Seasonality: repeating pattern.
- Autocorrelation: relationship between current and previous values.
- Stationary/non-stationary: whether statistical properties change over time.
- Missing values: need interpolation, forward fill, or other strategies.

### Sliding Windows

Use the past `seq_length` points to predict the next point:

```python
for i in range(len(data_scaled) - seq_length):
    X.append(data_scaled[i : i + seq_length])
    y.append(data_scaled[i + seq_length])
```

### FFN, RNN, LSTM Comparison

| Model | Logic | Advantage | Limitation |
|---|---|---|---|
| FFN | Flatten window into features | Simple, fast | Does not explicitly model order |
| RNN | Updates hidden state over time | Has memory for short sequences | Long dependencies suffer |
| LSTM | Hidden state + cell state + gates | Better long-term memory | More parameters, slower |

### RNN

```text
h_t = f(U x_t + W h_{t-1})
y_t = g(V h_t)
```

RNNs share parameters over time and can process sequences.

### LSTM

LSTM controls information flow through gates:

- Forget gate: what old information to discard.
- Input gate: what new information to write.
- Output gate: what memory to expose for prediction.
- Cell state: long-term memory path.

W8 notebook key code:

```python
class SimpleRNN(nn.Module):
    def __init__(self, input_dim, hidden_dim, num_layers, output_dim):
        super().__init__()
        self.rnn = nn.RNN(input_dim, hidden_dim, num_layers, batch_first=True)
        self.fc = nn.Linear(hidden_dim, output_dim)

    def forward(self, x):
        out, hn = self.rnn(x)
        out = out[:, -1, :]
        return self.fc(out)
```

```python
class LSTMModel(nn.Module):
    def __init__(self, input_dim, hidden_dim, num_layers, output_dim):
        super().__init__()
        self.lstm = nn.LSTM(input_dim, hidden_dim, num_layers, batch_first=True)
        self.fc = nn.Linear(hidden_dim, output_dim)

    def forward(self, x):
        out, (hn, cn) = self.lstm(x)
        out = out[:, -1, :]
        return self.fc(out)
```

Recursive forecasting:

```python
def recursive_forecast(model, initial_seq, steps):
    model.eval()
    curr_seq = initial_seq.clone().unsqueeze(0)
    predictions = []
    with torch.no_grad():
        for _ in range(steps):
            pred = model(curr_seq)
            predictions.append(pred.item())
            curr_seq = torch.cat((curr_seq[:, 1:, :], pred.view(1, 1, 1)), dim=1)
    return np.array(predictions)
```

Key risk: recursive multi-step prediction accumulates error because later inputs include the model’s own previous predictions.

## 11. W9: Object Detection, YOLO, RT-DETR, mAP

### Image Classification vs Object Detection

- Image classification: what class is the whole image?
- Object detection: what objects are present, and where are they?

Detection output:

```text
class + bounding box
box = (x, y, w, h) or (x1, y1, x2, y2)
```

### R-CNN Family

- R-CNN: generate region proposals, then classify each proposal with a CNN. Accurate but slow.
- Fast R-CNN: run CNN once on the image, then use RoI pooling.
- Faster R-CNN: learns proposals with a Region Proposal Network for more end-to-end detection.

### YOLO

YOLO treats detection as one forward-pass regression problem:

```text
image -> grid/cells -> boxes + confidence + class scores
```

Advantages:

- Fast and suitable for real-time applications.
- Uses global image context.

Limitations:

- Small or densely overlapping objects can be difficult.
- Earlier YOLO versions relied on NMS and anchor design; modern versions improve this.

### DETR / RT-DETR

- DETR: transformer encoder-decoder predicts a set of objects directly, removing anchors and NMS.
- RT-DETR: optimized for real-time detection with efficient CNN and hybrid encoder.

### Detection Metrics

- IoU: intersection over union between predicted and ground-truth boxes.
- Precision: how many detections are real.
- Recall: how many real objects are found.
- AP: area under precision-recall curve for one class.
- mAP: mean AP over classes.
- mAP@0.5: IoU threshold 0.5, relatively lenient.
- mAP@0.5:0.95: average over multiple IoU thresholds, stricter and more standard.

W9 notebook key code:

```python
ultra_yaml = {
    "path": DATASET_DIR,
    "train": "train/images",
    "val": "valid/images",
    "nc": NUM_CLASSES,
    "names": CLASSES,
}
```

```python
yolo26_model = YOLO("yolo26n.pt")
yolo26_model.train(
    data=DATA_YAML,
    epochs=5,
    imgsz=320,
    batch=16,
    lr0=5e-4,
    optimizer="Adam",
    freeze=10,
)
```

```python
rtdetr_model = YOLO("rtdetr-l.pt")
rtdetr_model.train(
    data=DATA_YAML,
    epochs=MAX_EPOCHS_RTDETR,
    imgsz=320,
    batch=8,
    lr0=1e-4,
    optimizer="AdamW",
    freeze=12,
)
```

Speed benchmark:

```python
def benchmark_fps(model, img_path, n_warmup=5, n_runs=50):
    for _ in range(n_warmup):
        model.predict(img_path, conf=0.4, verbose=False)
    times = []
    for _ in range(n_runs):
        t0 = time.perf_counter()
        model.predict(img_path, conf=0.4, verbose=False)
        times.append(time.perf_counter() - t0)
    return np.mean(times) * 1000, 1.0 / np.mean(times)
```

## 12. MRI MiniProject (Exam Focus)

### Project Task

Project file: `MiniProject_version_2.0.ipynb`.

Goal: four-class Alzheimer MRI image classification:

- `VeryMildDemented`
- `MildDemented`
- `ModerateDemented`
- `NonDemented`

Workflow:

```text
Data inspection -> Data pipeline -> SimpleCNN baseline -> Training utilities ->
SimpleCNN training/evaluation -> ResNet-18 transfer learning -> Final evaluation ->
Visual predictions -> Single external image prediction
```

Dataset structure:

| Split | Images Per Class | Total |
|---|---:|---:|
| Train | 8000 | 32000 |
| Val | 1000 | 4000 |
| Test | 1000 | 4000 |

For CPU-friendly execution, the notebook uses balanced subsets by default:

- `MAX_TRAIN_SAMPLES_PER_CLASS = 500`
- `MAX_EVAL_SAMPLES_PER_CLASS = 250`

Exam explanation: this is a computational compromise. With a GPU, image size, epochs, and samples per class can be increased.

### Questions You Must Be Able To Explain

1. Why inspect the dataset first?
2. Why can MRI images be treated as grayscale?
3. Why is SimpleCNN used as a baseline?
4. Why does ResNet-18 need three input channels?
5. Why use ImageNet normalization?
6. Why split train/validation/test?
7. Why use data augmentation?
8. Why does CrossEntropyLoss not need manual softmax?
9. How do we interpret confusion-matrix errors?
10. Why is this not a clinical diagnostic system?

### Step 0: Setup And Reproducibility

Key code:

```python
SEED = 42
DATA_ROOT = Path("Alzheimer MRI Dataset")
TRAIN_DIR = DATA_ROOT / "train"
VAL_DIR = DATA_ROOT / "val"
TEST_DIR = DATA_ROOT / "test"

IMG_SIZE_BASE = 64
IMG_SIZE_ADV = 64
BATCH_SIZE = 64

def set_seed(seed=SEED):
    random.seed(seed)
    np.random.seed(seed)
    torch.manual_seed(seed)
    if torch.cuda.is_available():
        torch.cuda.manual_seed_all(seed)
```

Explanation:

- Fixed random seed improves reproducibility of sampling, splitting, and initialization.
- `get_device()` selects CUDA/MPS/CPU.
- `validate_dataset_dirs()` catches path errors before training.

### Step 1: Data Inspection

Checks performed:

- Image counts per split and class.
- Image size, PIL mode, RGB channel differences.
- Class balance.
- Whether patient-level splitting can be audited.

Important limitation:

```python
def infer_patient_id(filename):
    match = re.search(r"(patient|subject|sub)[-_]?(\\d+)", filename, flags=re.IGNORECASE)
    return match.group(0).lower() if match else None
```

Filenames such as `MRI-ND-00001.jpg` do not contain reliable patient/session IDs, so patient-level leakage cannot be verified. This is an important oral-exam limitation: if slices from the same patient appear in multiple splits, test accuracy may be inflated.

### Step 2: Preprocessing And DataLoader

SimpleCNN uses single-channel grayscale:

```python
base_transform = make_transform(IMG_SIZE_BASE, augment=False, num_channels=1)
```

ResNet-18 uses three channels:

```python
adv_train_transform = make_transform(
    IMG_SIZE_ADV,
    augment=True,
    num_channels=3,
    mean=IMAGENET_MEAN,
    std=IMAGENET_STD,
)
```

Reasoning:

- MRI images are essentially grayscale, so SimpleCNN can use one channel.
- ResNet-18 pretrained on ImageNet expects RGB-like three-channel input, so grayscale images are converted/repeated to three channels and normalized using ImageNet mean/std.

Core transform:

```python
def make_transform(img_size, augment=False, num_channels=1, mean=None, std=None):
    preprocessing = [transforms.Grayscale(num_output_channels=num_channels)]
    if augment:
        preprocessing.extend([
            transforms.RandomResizedCrop(img_size, scale=(0.90, 1.00)),
            transforms.RandomRotation(8),
            transforms.RandomAffine(degrees=0, translate=(0.02, 0.02), scale=(0.95, 1.05)),
        ])
    else:
        preprocessing.append(transforms.Resize((img_size, img_size)))
    preprocessing.extend([
        transforms.ToTensor(),
        transforms.Normalize(mean=mean.tolist(), std=std.tolist()),
    ])
    return transforms.Compose(preprocessing)
```

Augmentation logic:

- Training can use random crop/rotation/affine to improve generalization.
- Validation/test must be deterministic: resize and normalize only.
- Medical image augmentation should be conservative to avoid destroying diagnostic structure.

Balanced subset:

```python
def make_balanced_subset(dataset, max_per_class=None, shuffle=False):
    if max_per_class is None:
        return dataset
    selected_indices = []
    targets = np.array(dataset.targets)
    for class_idx in range(len(dataset.classes)):
        class_indices = np.flatnonzero(targets == class_idx)
        selected_indices.extend(class_indices[:max_per_class].tolist())
    return Subset(dataset, selected_indices)
```

Explanation:

- Controls samples per class and keeps class balance.
- Reduces CPU training time.
- Prevents a majority class from dominating if the dataset is imbalanced.

### Step 3: SimpleCNN Baseline

Architecture:

```python
class SimpleCNN(nn.Module):
    def __init__(self, num_classes=NUM_CLASSES):
        super().__init__()
        self.features = nn.Sequential(
            nn.Conv2d(1, 16, kernel_size=3, padding=1), nn.ReLU(), nn.MaxPool2d(2),
            nn.Conv2d(16, 32, kernel_size=3, padding=1), nn.ReLU(), nn.MaxPool2d(2),
            nn.Conv2d(32, 64, kernel_size=3, padding=1), nn.ReLU(), nn.MaxPool2d(2),
        )
        self.classifier = nn.Sequential(
            nn.Flatten(),
            nn.Linear(64 * 8 * 8, 256),
            nn.ReLU(),
            nn.Linear(256, num_classes),
        )

    def forward(self, x):
        return self.classifier(self.features(x))
```

Shape logic:

```text
Input: 1 x 64 x 64
Conv+Pool 1 -> 16 x 32 x 32
Conv+Pool 2 -> 32 x 16 x 16
Conv+Pool 3 -> 64 x 8 x 8
Flatten -> 64*8*8 = 4096
Linear -> 256 -> 4 logits
```

Why it is a baseline:

- Simple and explainable.
- Trained from scratch without external pretrained features.
- Provides a comparison point for transfer learning.

Limitations:

- Limited feature capacity for subtle class differences.
- No batch norm/dropout regularization.
- `64x64` may lose MRI detail.

### Step 4: Training Utilities

The project wraps training and validation into reusable functions:

```python
def run_epoch(model, loader, criterion, device, optimizer=None):
    is_train = optimizer is not None
    model.train(is_train)
    total_loss, correct, total = 0.0, 0, 0

    with torch.set_grad_enabled(is_train):
        for imgs, labels in loader:
            imgs, labels = imgs.to(device), labels.to(device)
            logits = model(imgs)
            loss = criterion(logits, labels)

            if is_train:
                optimizer.zero_grad()
                loss.backward()
                optimizer.step()

            total_loss += loss.item() * labels.size(0)
            correct += (logits.argmax(dim=1) == labels).sum().item()
            total += labels.size(0)

    return total_loss / total, correct / total
```

Explanation:

- With an optimizer it trains; without one it validates.
- `torch.set_grad_enabled(is_train)` avoids gradients during validation.
- `logits.argmax(dim=1)` gives predicted class.
- CrossEntropyLoss expects logits and integer labels.

### Step 5: SimpleCNN Training

```python
model_cnn = SimpleCNN().to(DEVICE)
criterion = nn.CrossEntropyLoss()
optimizer_cnn = optim.Adam(model_cnn.parameters(), lr=LR_BASE)
scheduler_cnn = optim.lr_scheduler.StepLR(optimizer_cnn, step_size=LR_STEP_SIZE, gamma=LR_GAMMA)
```

Oral explanation:

- `Adam` adapts update sizes per parameter.
- `StepLR` periodically reduces learning rate for more stable late training.
- Compare train loss/accuracy and validation loss/accuracy to detect underfitting or overfitting.

### Step 6: ResNet-18 Transfer Learning

Core code:

```python
def set_trainable(model, trainable):
    for param in model.parameters():
        param.requires_grad = trainable

def build_resnet18(num_classes=NUM_CLASSES, freeze_backbone=True):
    weights = models.ResNet18_Weights.IMAGENET1K_V1
    model = models.resnet18(weights=weights)
    model.fc = nn.Linear(model.fc.in_features, num_classes)

    if freeze_backbone:
        set_trainable(model, False)
        for param in model.fc.parameters():
            param.requires_grad = True

    return model.to(DEVICE)
```

Two phases:

```python
# Phase 1: freeze backbone, train classifier head
optimizer_ph1 = optim.Adam((p for p in model_adv.parameters() if p.requires_grad), lr=LR_PHASE1)

# Phase 2: unfreeze all layers, fine-tune with lower LR
set_trainable(model_adv, True)
optimizer_ph2 = optim.Adam(model_adv.parameters(), lr=LR_PHASE2)
```

Why:

- Phase 1 prevents a randomly initialized head from disrupting pretrained features.
- Phase 2 adapts generic visual features to the MRI task with a small learning rate.
- Transfer learning is often better than training from scratch when data or compute is limited.

### Step 7: Final Evaluation

```python
preds_adv, labels_test_adv = collect_predictions(model_adv, adv_loaders["test"])
acc_adv = (preds_adv == labels_test_adv).mean() * 100
cm_adv = confusion_matrix(labels_test_adv, preds_adv)
print(classification_report(labels_test_adv, preds_adv, target_names=CLASS_NAMES, digits=3))
```

What to explain:

- Accuracy: overall correctness, but hides class-specific errors.
- Classification report: precision, recall, F1, support per class.
- Confusion matrix: which classes are confused.
- In medical settings, clinically important recall can matter more than overall accuracy.

### Step 8: Visual Validation And Single-Image Prediction

Visual predictions:

```python
with torch.no_grad():
    logits = model(imgs_device)
    probs = torch.softmax(logits, dim=1)
    preds = logits.argmax(dim=1).cpu()
```

Single-image top-k:

```python
def predict_image(image_path, model, transform, top_k=3):
    image = Image.open(image_path).convert("RGB")
    input_tensor = transform(image).unsqueeze(0).to(DEVICE)
    model.eval()
    with torch.no_grad():
        probs = torch.softmax(model(input_tensor), dim=1)[0]
        top_probs, top_indices = probs.topk(top_k)
    return image, top_probs.cpu(), top_indices.cpu()
```

Explanation:

- `unsqueeze(0)` adds batch dimension.
- `softmax` converts logits to probabilities.
- Top-3 predictions better communicate uncertainty than a single class.

### MRI Project Strengths And Limitations

Strengths:

- Clear train/validation/test split.
- Baseline and transfer-learning comparison.
- Dataset inspection, confusion matrix, and per-class report.
- Engineering-reasonable transfer-learning pipeline.
- Modular code that is easy to reproduce.

Limitations:

- Patient-level split cannot be verified, creating possible slice-level leakage risk.
- MRI classification has medical implications; an image classifier is not a clinical diagnostic system.
- Resizing to `64x64` may remove fine detail.
- ImageNet pretraining creates a domain gap because ImageNet is natural images, not MRI.
- Epochs and subsampling are classroom/CPU-friendly settings, not necessarily optimal.

### MRI Oral Exam Questions

**Q1: Why train SimpleCNN before ResNet-18?**  
A: SimpleCNN is a from-scratch baseline. It gives an explainable lower benchmark. ResNet-18 uses pretrained ImageNet features through transfer learning. Comparing them shows whether the advanced method genuinely improves performance.

**Q2: Why does ResNet-18 use three channels if MRI is grayscale?**  
A: ResNet-18 pretrained weights expect RGB three-channel input. The project converts/repeats grayscale into three channels and uses ImageNet normalization to match the pretrained setting.

**Q3: Why not only report accuracy?**  
A: Accuracy hides class-level failures. In medical and fault-detection tasks, some misses are more important, so confusion matrix, per-class recall, precision, and F1 are needed.

**Q4: Why no softmax before CrossEntropyLoss?**  
A: PyTorch `nn.CrossEntropyLoss` already combines `log_softmax` and negative log likelihood. The model should output raw logits; manual softmax can make training numerically less stable.

**Q5: What is the most important project limitation?**  
A: There is no reliable patient ID in filenames, so patient-level leakage cannot be audited. If slices from the same patient appear in multiple splits, test performance may be inflated. Also, this is an image classifier, not a clinical diagnosis system.

**Q6: Why should augmentation be conservative?**  
A: Medical image geometry and intensity can be diagnostically relevant. Too-strong augmentation may destroy or alter pathology. The project uses mild crop, rotation, and affine transforms to simulate acquisition variation without changing the medical structure too aggressively.

## 13. Algorithm Comparison Table

| Algorithm/Model | Task | Core Logic | Strengths | Weaknesses |
|---|---|---|---|---|
| Linear Regression | Regression | Learn a linear relationship | Simple, interpretable | Only linear |
| MLP | Regression/classification | Fully connected layers + activations | General function approximator | Poor spatial inductive bias for images |
| Logistic Regression | Binary classification | Linear output + sigmoid | Clear probability interpretation | Simple decision boundary |
| CNN | Image classification | Local filters extract features | Parameter sharing, good for images | Needs enough images or regularization |
| ResNet | Image classification | Residual connections | Deep, stable, strong transfer | More parameters |
| Transfer Learning | Small visual datasets | Reuse pretrained features | Fast, often generalizes well | Domain gap risk |
| K-means | Clustering | Nearest centroid + mean update | Simple and fast | Must choose K, scale-sensitive |
| Autoencoder | Anomaly detection | Normal samples reconstruct well | Does not need anomaly labels | Threshold-sensitive |
| RNN | Sequence prediction | Hidden state remembers past | Good for short sequences | Long dependencies are hard |
| LSTM | Sequence prediction | Gates + cell state | Better long-term memory | More complex and slower |
| YOLO | Object detection | One-stage box/class regression | Real-time speed | Small/dense objects harder |
| RT-DETR | Object detection | Transformer queries predict objects | Global reasoning, no NMS | Heavier model |

## 14. Loss Function And Output Layer Cheat Sheet

| Task | Output Layer | Label Format | Loss Function |
|---|---|---|---|
| Regression | Linear | float | `MSELoss`, `L1Loss` |
| Binary classification | Sigmoid probability | 0/1 float | `BCELoss` |
| Binary classification, stable version | Raw logits | 0/1 float | `BCEWithLogitsLoss` |
| Multi-class classification | Raw logits | integer class index | `CrossEntropyLoss` |
| Autoencoder | Sigmoid/linear reconstructed image | image tensor | `MSELoss` |
| Object detection | boxes + class scores | bounding box labels | box loss + class loss + objectness/assignment |

## 15. Common Exam Mistakes

- Using the test set for tuning.
- Applying manual softmax before `CrossEntropyLoss`.
- Forgetting `optimizer.zero_grad()`.
- Forgetting `model.eval()` and `torch.no_grad()` during evaluation.
- Using normalization that does not match the pretrained model.
- Randomly shuffling time series and leaking future information.
- Reporting only accuracy, without recall/F1/confusion matrix.
- Ignoring patient-level leakage risk in the MRI project.
- Treating high confidence as guaranteed correctness.
- Using overly strong augmentation that changes medical or defect semantics.

## 16. Oral Exam Final Checklist

You should be able to explain:

- Where the data comes from, what the classes are, and how train/validation/test are split.
- The input and output shapes of each model.
- Why the task uses that specific loss function.
- What each line of the training loop does.
- How each metric is computed and what error type it represents.
- Which classes are confused and why.
- Whether the project improves over the baseline and why.
- What extra validation is needed before deployment in a real industrial or medical setting.

