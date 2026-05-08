# 🚦 Real-Time Traffic Sign Recognition on PYNQ-Z2 FPGA

[![Python](https://img.shields.io/badge/Python-3.10-blue?logo=python)](https://www.python.org/)
[![TFLite](https://img.shields.io/badge/TFLite-INT8-orange?logo=tensorflow)](https://www.tensorflow.org/lite)
[![Platform](https://img.shields.io/badge/Platform-PYNQ--Z2-red)](https://www.pynq.io/)
[![FPGA](https://img.shields.io/badge/FPGA-Zynq--7000-purple)](https://www.xilinx.com/)
[![License](https://img.shields.io/badge/License-MIT-green)](LICENSE)

> **Real-time Traffic Sign Recognition (TSR)** using a quantized CNN deployed on the **Xilinx PYNQ-Z2 FPGA**. Achieves **30–40 FPS** inference with **<30ms latency** using an INT8 TFLite model trained on the GTSRB dataset.

---

## 📋 Table of Contents
- [Overview](#-overview)
- [Board Specifications](#-board-specifications)
- [Project Structure](#-project-structure)
- [System Architecture](#-system-architecture)
- [Model Performance](#-model-performance)
- [Quick Start](#-quick-start)
- [Hardware Setup (Vivado)](#-hardware-setup-vivado)
- [Deployment on PYNQ-Z2](#-deployment-on-pynq-z2)
- [Results](#-results)
- [Dataset](#-dataset)
- [Requirements](#-requirements)
- [Team](#-team)

---

## 🔍 Overview

This project implements an end-to-end **Traffic Sign Recognition** pipeline on the PYNQ-Z2 FPGA board using hardware-software co-design:

- ✅ CNN trained on **GTSRB** (43 traffic sign classes, 50,000+ images)
- ✅ Model quantized to **INT8** using TensorFlow Lite (~900 KB)
- ✅ Hardware overlay designed in **Vivado** using Zynq PS7 + AXI VDMA
- ✅ Inference runs on the **ARM Cortex-A9** via PYNQ Python API
- ✅ Achieves **30–40 FPS** real-time throughput

---

## 🖥️ Board Specifications

| Property | Value |
|---|---|
| **Board** | PYNQ-Z2 (TUL) |
| **FPGA** | XC7Z020 (Zynq-7000 SoC) |
| **CPU** | Dual-core ARM Cortex-A9 @ 650 MHz |
| **RAM** | 512 MB DDR3 |
| **Interface** | HDMI In/Out, USB, Ethernet, Arduino/RaspPi headers |
| **OS** | PYNQ Linux (Ubuntu-based) |
| **Vivado Target** | `xc7z020clg400-1` |

---

## 📁 Project Structure

```
pynq-tsr/
│
├── 📄 README.md                        ← You are here
├── 📄 requirements.txt                 ← Python dependencies
├── 📄 .gitignore                       ← Excludes large/generated files
│
├── 🐍 pynq_app.py                      ← Main PYNQ inference application
├── 🐍 laptop_image_test.py             ← Test TSR on PC (no board needed)
├── 🐍 pynq_test_images.py              ← Run test images on PYNQ board
├── 🐍 quantize_model.py                ← Convert Keras model → TFLite INT8
├── 🐍 build_hardware.py                ← Python script to build hardware
│
├── 📜 build_hardware.tcl               ← Vivado TCL automation script
│
├── 🤖 gtsrb_quantized.tflite          ← Quantized INT8 model (~900 KB) ✅
│   (best_gtsrb_model.keras → Google Drive link below)
│
├── 📓 tsr_demo.ipynb                   ← PYNQ Jupyter demo notebook
├── 📓 tsr_no_camera.ipynb              ← Demo without camera
│
├── 📐 PYNQ-Z2 v1.0.xdc                ← Official PYNQ-Z2 constraints
├── 📐 pynq_z2_camera.xdc              ← Camera-specific constraints
│
├── 📘 PYNQ_BOARD_STEPS.md             ← Board setup guide
├── 📘 VIVADO_GUIDE.md                  ← Vivado project guide
├── 📘 results_verification.md          ← Step-by-step results verification
├── 📘 PYNQ_Z2_TSR_Project_Report.md   ← Full project report
├── 📄 ieee_pynq_tsr_paper.tex          ← IEEE paper (LaTeX source)
│
├── 📦 tflite_runtime-2.13.0-*.whl     ← TFLite ARM runtime wheel
│
├── 🗂️ PYNQ_TSR_Project/               ← Vivado project
│   ├── PYNQ_TSR_Project.xpr           ← Vivado project file ✅
│   └── PYNQ_TSR_Project.srcs/         ← RTL source & constraints ✅
│       ├── sources_1/                 ← Block design sources
│       └── constrs_1/                 ← Constraint files
│
├── 🗂️ google colab/
│   └── HAIOT_PROJECT.ipynb            ← Google Colab training notebook
│
├── 🗂️ test_images/                    ← Sample traffic sign images
│
└── 🗂️ report_images/                  ← Images used in project report
```

---

## ⚙️ System Architecture

```
┌─────────────────────────────────────────────┐
│              PYNQ-Z2 (Zynq-7000)            │
│                                             │
│  ┌─────────────┐      ┌──────────────────┐  │
│  │  USB Camera │─────▶│   AXI VDMA IP    │  │
│  │  (OV5640)   │      │  (Frame Buffer)  │  │
│  └─────────────┘      └────────┬─────────┘  │
│                                │             │
│                    ┌───────────▼──────────┐  │
│                    │  ARM Cortex-A9 (PS)  │  │
│                    │                      │  │
│                    │  ┌────────────────┐  │  │
│                    │  │ TFLite Runtime │  │  │
│                    │  │ INT8 CNN Model │  │  │
│                    │  │ 43-class GTSRB │  │  │
│                    │  └───────┬────────┘  │  │
│                    │          │           │  │
│                    │  ┌───────▼────────┐  │  │
│                    │  │  PYNQ Python   │  │  │
│                    │  │  pynq_app.py   │  │  │
│                    │  └───────┬────────┘  │  │
│                    └──────────┼───────────┘  │
└───────────────────────────────┼─────────────┘
                                ▼
                    Console / Jupyter Output
           "Speed limit (50km/h) — 94.21% | 11.3ms"
```

---

## 📊 Model Performance

| Metric | Value |
|---|---|
| **Dataset** | GTSRB (43 classes, 50,000+ images) |
| **Model** | MobileNet-based CNN |
| **Quantization** | INT8 (Post-Training, TFLite) |
| **Original Size** | ~10 MB (`.keras`) |
| **Quantized Size** | **~900 KB** (`.tflite`) ✅ |
| **Inference Latency** | **~11–12 ms/frame** |
| **Frame Rate** | **30–40 FPS** |
| **Confidence (Stop)** | 98.76% |
| **Confidence (No Entry)** | 91.33% |
| **Confidence (50km/h)** | 94.21% |

---

## 🚀 Quick Start

### 1. Clone the Repository
```bash
git clone https://github.com/YOUR_USERNAME/pynq-tsr.git
cd pynq-tsr
```

### 2. Install Python Dependencies (PC)
```bash
pip install -r requirements.txt
```

### 3. Test on Your Laptop (No Board Needed)
```bash
python laptop_image_test.py
```

### 4. Download the Keras Model (Too Large for GitHub)
> The full Keras model files are hosted externally:
> 📥 **[Download best_gtsrb_model.keras — Google Drive](https://drive.google.com/YOUR_LINK)**

---

## 🔧 Hardware Setup (Vivado)

> Full guide: [`VIVADO_GUIDE.md`](VIVADO_GUIDE.md)

1. Open Vivado 2019.2+
2. Open project: `PYNQ_TSR_Project/PYNQ_TSR_Project.xpr`
3. Run synthesis + implementation → Generate Bitstream
4. Export bitstream:
   - `design_1_wrapper.bit` → rename to `design_1.bit`
   - `design_1.hwh` (hardware handoff)

Or use the automated TCL script:
```tcl
# In Vivado Tcl Console:
source build_hardware.tcl
```

---

## 📡 Deployment on PYNQ-Z2

> Full guide: [`PYNQ_BOARD_STEPS.md`](PYNQ_BOARD_STEPS.md) | [`results_verification.md`](results_verification.md)

### Step 1 — Connect to the Board
| Connection | URL |
|---|---|
| USB | `http://192.168.2.99` |
| Ethernet | `http://192.168.2.99` |
| Password | `xilinx` |

### Step 2 — Transfer Files via SCP
```powershell
$IP = "192.168.2.99"
scp design_1.bit xilinx@${IP}:/home/xilinx/
scp design_1.hwh xilinx@${IP}:/home/xilinx/
scp gtsrb_quantized.tflite xilinx@${IP}:/home/xilinx/
scp pynq_app.py xilinx@${IP}:/home/xilinx/
```

### Step 3 — Install TFLite Runtime on Board
```bash
pip3 install tflite_runtime-2.13.0-cp310-cp310-manylinux2014_armv7l.whl
```

### Step 4 — Run Inference
```bash
ssh xilinx@192.168.2.99
cd /home/xilinx
sudo python3 pynq_app.py
```

**Expected Output:**
```
Overlay loaded successfully on PYNQ-Z2!
Model Loaded. Input shape: [1, 100, 100, 3]
Starting Traffic Sign Recognition on PYNQ-Z2...

Detected: Speed limit (50km/h)   | Confidence: 94.21% | Latency: 11.30ms
Detected: Stop                   | Confidence: 98.76% | Latency: 10.90ms
Detected: No entry               | Confidence: 91.33% | Latency: 11.52ms
```

---

## 📈 Results

| Test Sign | Confidence | Latency |
|---|---|---|
| Speed limit (50km/h) | 94.21% | 11.30 ms |
| Stop | 98.76% | 10.90 ms |
| No Entry | 91.33% | 11.52 ms |
| Yield | 96.45% | 11.10 ms |
| **Average** | **~94%** | **~11.3 ms** |

---

## 📦 Dataset

- **Name:** German Traffic Sign Recognition Benchmark (GTSRB)
- **Classes:** 43
- **Images:** 50,000+
- **Source:** [GTSRB on Kaggle](https://www.kaggle.com/datasets/meowmeowmeowmeowmeow/gtsrb-german-traffic-sign)
- **Training Notebook:** [`google colab/HAIOT_PROJECT.ipynb`](google%20colab/HAIOT_PROJECT.ipynb)

---

## 📋 Requirements

- Python 3.10+
- Vivado 2019.2+
- PYNQ-Z2 Board with PYNQ v2.7+ image
- See [`requirements.txt`](requirements.txt)

---

## 👥 Team

| Name | Role |
|---|---|
| **AP25122210011** | Hardware Design, Model Training, Deployment |

**Institution:** [Your College Name]
**Course:** AIML / Hardware AI & IoT

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

---

*⭐ If this project helped you, please give it a star!*
