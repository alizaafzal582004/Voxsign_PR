# VoxaSign

### AI-Powered Real-Time Sign Language → Text → Speech

[![Python](https://img.shields.io/badge/Python-3.10-blue?style=for-the-badge&logo=python)](https://python.org)
[![TensorFlow.js](https://img.shields.io/badge/TensorFlow.js-WebGL-orange?style=for-the-badge&logo=tensorflow)](https://www.tensorflow.org/js)
[![MediaPipe](https://img.shields.io/badge/MediaPipe-Vision-green?style=for-the-badge&logo=google)](https://mediapipe.dev)
[![Edge AI](https://img.shields.io/badge/Edge--AI-Privacy--First-black?style=for-the-badge)]()
[![Deployment](https://img.shields.io/badge/Deployment-Live--Ready-brightgreen?style=for-the-badge)]()

**Bridging communication gaps with inclusive AI 🤝**
Real-time ASL gesture translation — runs entirely in your browser. No servers. No latency. No data leaves your device.

---

## 🌐 Live Demo

**[https://voxa-sign.netlify.app/](https://voxa-sign.netlify.app/)**

---

## 📌 Project Overview

**VoxaSign** is a browser-based Edge-AI system that translates **American Sign Language (ASL) hand gestures (A–Z)** into text and speech in real time.

It evolved from a **Python desktop application** (Tkinter + OpenCV + gTTS) into a **fully browser-based system** using TensorFlow.js and MediaPipe, delivering:

- ⚡ Real-time predictions at 30+ FPS
- 🔒 Complete privacy — all inference on-device
- 🌐 Works on desktop and mobile browsers
- 📡 Zero server dependency

---

## ⚠️ The Problem

> 🌍 Over **70 million people** rely on sign language globally.

Communication barriers exist across healthcare, education, and banking — environments where sign language interpreters are often unavailable.

💡 **VoxaSign solves this** with a hardware-agnostic, browser-based AI translator that anyone can use instantly.

---

## ✨ Features

| Feature | Description |
|---------|-------------|
| 🎥 Live Camera Mode | Real-time gesture recognition via webcam |
| 🖼️ Photo Analysis Mode | Upload a static image for analysis |
| ✋ Manual Detection | Click "ADD TO WORD" to commit a letter |
| ⚡ Auto Detection | Hold a sign for 1.5s — letter commits automatically |
| 🔊 Text-to-Speech | Web Speech API speaks your word aloud |
| ⌫ Backspace | Remove the last letter with one click |
| 📊 Session Analytics | Live letter count, word count, accuracy stats |
| 📱 Camera Switch | Toggle front/rear camera on mobile |
| 💾 Mode Persistence | Manual/Auto preference saved across sessions |
| 🔒 Privacy-First | Zero data transmitted — 100% on-device |

---

## 🧠 Technical Architecture

### Inference Pipeline

```
Webcam / Image Upload
       ↓
MediaPipe HandLandmarker
(detects 21 3D hand landmarks)
       ↓
Wrist-Relative Normalization
([lm.x - wrist.x, lm.y - wrist.y, lm.z - wrist.z] × 21 = 63 values)
       ↓
Max-Value Normalization (scale-invariant)
       ↓
TensorFlow.js DNN — web_model/model.json
(softmax over 26 ASL letter classes)
       ↓
Confidence Threshold Gate (≥ 75%)
       ↓
Predicted Letter + Confidence Score
       ↓
Word Buffer → Web Speech API (TTS)
```

### Model Details

- **Input:** 63-element feature vector (21 landmarks × 3 normalized coordinates)
- **Architecture:** Dense Neural Network with ReLU activations
- **Output:** 26-class softmax (ASL letters A–Z)
- **Training:** TensorFlow/Keras (Python), converted to TF.js for browser deployment
- **Confidence threshold:** 75% (web) / 85% (Python desktop)

---

## 📂 Project Structure

```
VoxaSign/
├── web_model/                  ← TensorFlow.js converted model
│   ├── model.json              ← Model topology
│   └── group1-shard1of1.bin   ← Binary weight file
│
├── index.html                  ← Landing page (marketing + alphabet grid)
├── studio.html                 ← Main gesture studio application
├── script.js                   ← Core inference engine (ES Module)
├── style.css                   ← Global styles
│
├── signspeak_ui.py             ← Legacy Python desktop app (Tkinter)
├── convert_model.py            ← Keras → TF.js model conversion script
├── signspeak_model.keras       ← Trained Keras model
├── hand_landmarker.task        ← MediaPipe hand detection model
│
├── demo.mp4                    ← Demo video (shown on landing page)
├── img.png                     ← ASL alphabet reference chart
├── .gitignore
└── README.md
```

---

## 🚀 Quick Start — Web Studio

> ⚠️ **Important:** A local HTTP server is required. Opening `index.html` directly as a `file://` URL will fail because MediaPipe loads the `.task` model via HTTP fetch and ES Modules require HTTP.

### 1. Clone Repository

```bash
git clone https://github.com/alizaafzal582004/VoxaSign.git
cd VoxaSign
```

### 2. Start a Local Server

**Option A — Python (built-in):**
```bash
python -m http.server 8000
```

**Option B — VS Code Live Server:**
- Install the [Live Server](https://marketplace.visualstudio.com/items?itemName=ritwickdey.LiveServer) extension
- Click **"Go Live"** in the status bar

**Option C — Node.js:**
```bash
npx serve .
```

### 3. Open in Browser

```
http://localhost:8000              ← Landing page
http://localhost:8000/studio.html  ← Jump directly to studio
```

---

## 💻 Python Desktop App (Legacy)

The original desktop application built with Tkinter, OpenCV, and gTTS.

### Prerequisites

```bash
python -m venv venv

# Windows:
venv\Scripts\activate

# macOS / Linux:
source venv/bin/activate
```

### Install Dependencies

```bash
pip install tensorflow mediapipe opencv-python Pillow gTTS pygame numpy
```

### Run

```bash
python signspeak_ui.py
```

> **Required files** (must be in the same directory):
> - `signspeak_model.keras`
> - `hand_landmarker.task`
> - `label_map.npy`

---

## 🔧 Model Conversion (Keras → TensorFlow.js)

Only needed if you retrain the model:

```bash
pip install tensorflowjs tf_keras
python convert_model.py
```

**Output:** `web_model/model.json` + `web_model/group1-shard1of1.bin`

---

## ⚙️ Engineering Highlights

- **Wrist-Relative Normalization** — Coordinates are offset from the wrist landmark, making predictions invariant to hand position on screen
- **Max-Value Scaling** — Feature vector scaled by max absolute value, making predictions invariant to hand size and camera distance
- **Confidence Thresholding** — 75% gate reduces false positives and jitter
- **Memory Safety** — `tf.dispose()` called after every inference to prevent browser memory leaks
- **Auto-Mode Cooldown** — 500ms cooldown between same-letter additions prevents accidental flooding
- **GPU Acceleration** — TF.js uses WebGL backend for smooth real-time inference
- **ES Module Architecture** — `script.js` uses `type="module"` for scoped, modern JavaScript

---

## 🛠️ Tech Stack

### Browser / Frontend
| Library | Role |
|---------|------|
| TensorFlow.js | Run DNN in browser via WebGL |
| MediaPipe Tasks Vision | 21-point 3D hand landmark detection |
| Web Speech API | Text-to-speech (browser-native) |
| Google Fonts (Lora + Nunito) | Typography |

### Python (Training & Desktop)
| Library | Role |
|---------|------|
| TensorFlow / Keras | Model training & inference |
| tensorflowjs | Keras → TF.js conversion |
| MediaPipe (Python) | Hand detection in desktop app |
| OpenCV | Webcam capture + visualization |
| Tkinter | Desktop GUI |
| gTTS + Pygame | Text-to-speech for Python |
| NumPy | Array operations |

---

## 📋 Known Limitations

- **Static signs only** — Letters J and Z require motion; accuracy may vary
- **Single hand** — Two-hand signs are not currently supported
- **ASL only** — BSL, ISL, and other sign languages require separate models
- **No training script** — The model training pipeline is not included in this repo

---

## 🤝 Contributing

Contributions are welcome! Some ideas:

- Add a `requirements.txt` for easier Python setup
- Include training script and dataset documentation
- Add `signs/` directory with A–Z PNG images for the alphabet grid
- Implement motion-based sign detection for J and Z
- Add multi-language sign language support

---

## 📜 License

This project is open-source. See repository for license details.

---

*VoxaSign — Empowering Silence through Artificial Intelligence.*
