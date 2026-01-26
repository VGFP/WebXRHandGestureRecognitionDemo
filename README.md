# WebXRHandGestureRecognitionDemo
Simple demo showing hand gesture recognition for WebXR using ONNX web runtime with WASM backend

Key highlights:
• Utilized ONNX web runtime with WebAssembly (Wasm) as the execution provider for browser compatibility.

• Developed models to detect 3 gestures: Rock, Paper, Scissors.

• Preprocessing: normalization of points and extracting distances from the root joint as features.

• Implemented Support Vector Classifier (SVC) from the scikit-learn library with Radial Basis Function (RBF) kernel for gesture classification.

• Demo is using BabylonJS as the 3D engine to render graphics and to get WebXRHand points for detection.

• Detection is set to run every 30 frames and probability threshold for detection is set to 0.9.


Try on your VR headset: https://vgfp.github.io/WebXRHandGestureRecognitionDemo/

https://github.com/VGFP/WebXRHandGestureRecognitionDemo/assets/45914819/c9c71e4c-af62-43a2-aa8c-d9293460eb14

