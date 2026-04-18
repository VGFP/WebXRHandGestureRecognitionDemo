# WebXR Hand Gesture Recognition

Real-time hand gesture classification (rock / paper / scissors) running entirely in the browser via WebXR and ONNX Runtime for Web.

Key highlights:
• Utilized ONNX web runtime with WebAssembly (Wasm) as the execution provider for browser compatibility.

• Developed models to detect 3 gestures: Rock, Paper, Scissors.

• Preprocessing: normalization of points and extracting distances from the root joint as features.

• Implemented Support Vector Classifier (SVC) from the scikit-learn library with Radial Basis Function (RBF) kernel for gesture classification.

• Demo is using BabylonJS as the 3D engine to render graphics and to get WebXRHand points for detection.

• Detection is set to run every 30 frames and probability threshold for detection is set to 0.9.


Try on your VR headset: https://vgfp.github.io/WebXRHandGestureRecognitionDemo/

https://github.com/VGFP/WebXRHandGestureRecognitionDemo/assets/45914819/c9c71e4c-af62-43a2-aa8c-d9293460eb14

---

## How It Works

The system captures hand joint positions from WebXR hand tracking, normalizes them into a fixed-length feature vector, and classifies the gesture using a trained Support Vector Machine model running via ONNX Runtime WebAssembly.

### Pipeline

1. WebXR hand tracking captures 25 joint positions (XYZ) for each hand
2. Joint positions are normalized through a multi-step preprocessing pipeline
3. The normalized feature vector is fed into an ONNX SVC model
4. The model outputs class probabilities for rock, paper, and scissors
5. If the highest probability exceeds a 0.9 threshold, that gesture is displayed; otherwise "Unknown" is shown

---

## Machine Learning Model

Two separate **Support Vector Classifier (SVC)** models are used — one for the left hand and one for the right hand.

- **Algorithm**: Support Vector Machine with RBF (Radial Basis Function) kernel
- **Gamma**: Automatically scaled based on feature variance (`scale`)
- **Output**: Class probabilities via Platt scaling
- **Classes**: Rock (0), Paper (1), Scissors (2)
- **Format**: ONNX, served as static files and loaded by the browser at runtime

The models are trained in Python using scikit-learn, exported to ONNX via `skl2onnx`, and deployed as static assets that the browser loads through ONNX Runtime Web.

---

## Data & Training

Training data is collected from WebXR hand tracking — 30 frames of joint positions are recorded per gesture sample for both left and right hands. Three gesture classes are captured: rock, paper, and scissors. The dataset is split 80/20 for training and evaluation.

---

## Normalization Pipeline

Hand joint positions are raw 3D coordinates that vary based on hand position, size, and orientation in space. The normalization pipeline transforms them into a consistent, invariant representation:

| Step                  | Purpose              | Detail                                                      |
|-----------------------|----------------------|-------------------------------------------------------------|
| Translate to origin   | Position invariance  | Subtract wrist position from all joints                     |
| Scale normalization   | Size invariance      | Divide all points by the maximum distance from the wrist    |
| PCA rotation          | Orientation invariance | Rotate points using eigenvectors of the covariance matrix |
| Distance features     | Compact representation | Compute Euclidean distance from wrist to each joint       |
| Min-max normalization | Bounded range        | Scale all features to the [0, 1] range                      |

The result is a 25-element feature vector — one normalized distance value per hand joint — that is invariant to hand position, size, and rotation in 3D space.

### PCA Rotation

Principal Component Analysis rotation aligns the hand point cloud to a canonical orientation:

1. A 3x3 covariance matrix is computed from the 25 joint positions, capturing how joints vary and correlate along each axis
2. Eigendecomposition of the covariance matrix yields eigenvectors that define the principal directions of the hand's spatial structure — the first eigenvector points along the direction of greatest variance (typically the finger extension direction)
3. All points are projected onto this new coordinate frame, so the same gesture at different wrist angles produces the same feature representation

---

## ONNX Runtime Web

The models run entirely client-side in the browser using ONNX Runtime Web with a WebAssembly SIMD backend. Inference is offloaded to a Web Worker via the proxy mode to prevent UI jank. The WASM binary with SIMD support (`ort-wasm-simd.wasm`) is served alongside the model files.

---

## Inference

- Input: a single `[1, 25]` float32 tensor of normalized distance features
- Output: a `[1, 3]` float32 tensor of per-class probabilities
- Threshold: any class probability above 0.9 is accepted; below that, the result is "Unknown"
- Frequency: predictions are throttled (not every frame) to balance responsiveness and CPU load
- Display: results are shown on a 3D GUI panel floating in front of the camera

---

## Tech Stack

| Component          | Technology                     |
|--------------------|--------------------------------|
| 3D Engine          | Babylon.js                     |
| XR Framework       | WebXR (immersive-ar)           |
| Hand Tracking      | WebXR Hand Tracking API        |
| ML Runtime         | ONNX Runtime Web (WASM SIMD)   |
| ML Model           | scikit-learn SVC (RBF kernel)  |
| Training Pipeline  | Python (scikit-learn, skl2onnx)|
| Build              | Webpack + TypeScript           |
