# Multi-Organ Disease Classification Web Application with Grad-CAM

A full-stack medical image classification web application integrating deep transfer learning (ResNet50 and MobileNetV2) with Explainable Artificial Intelligence (Grad-CAM) to classify and localize findings across **Brain MRI**, **Kidney CT**, and **Chest X-Ray** scans.

---

## 🌟 Key Features

- **Genuine Deep Learning Inference**: FastAPI backend loads and runs actual trained Keras/TensorFlow models (`multi_organ_disease_model.keras` and `mobilenetv2_disease_model.keras`).
- **Explainable AI (Grad-CAM)**: True Gradient-weighted Class Activation Mapping computed from `conv5_block3_out` (ResNet50 residual block) using TensorFlow `GradientTape`. Overlays JET heatmaps on input scans with adjustable alpha transparency.
- **Multi-Model Support**:
  - **ResNet50 Transfer Learning**: 3 primary organ classes (`BRAIN`, `KIDNEY`, `LUNG`) with **96.67% test accuracy** and integrated Grad-CAM interpretability.
  - **MobileNetV2 Sub-Type Classifier**: 6 granular categories (`BRAIN_NORMAL`, `BRAIN_TUMOR`, `KIDNEY_NORMAL`, `KIDNEY_TUMOR`, `LUNG_NORMAL`, `LUNG_PNEUMONIA`) with **88.0% test accuracy**.
- **Modern Medical Dashboard**:
  - Interactive file drag-and-drop supporting JPEG, PNG, BMP, TIFF, and WebP (up to 10MB).
  - Class probability distribution with interactive Recharts graphs.
  - Side-by-side original scan vs Grad-CAM heatmap visualization with color-scale legend.
  - Ephemeral session analysis history with thumbnail previews.
  - Model evaluation page with verified confusion matrix and per-class metrics.
- **Medical Research Disclaimer**: Prominent notices emphasizing research and educational use.

---

## 🏗️ System Architecture

```
multi-organ-ai/
├── backend/
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py              # FastAPI app, CORS, routes & static endpoints
│   │   ├── schemas.py           # Pydantic request/response models
│   │   ├── model_loader.py      # ResNet50 & MobileNetV2 model loading & metadata
│   │   ├── preprocessing.py    # Image validation, resizing to 224x224 RGB
│   │   ├── gradcam.py           # TensorFlow GradientTape Grad-CAM generator
│   │   ├── prediction.py        # Pipeline orchestrator (preprocess -> infer -> gradcam)
│   │   └── history.py           # In-memory ephemeral session history
│   ├── models/
│   │   └── README.md            # Model file placement guide
│   ├── requirements.txt         # FastAPI, TensorFlow, Pillow, OpenCV, etc.
│   ├── .env                     # Backend configuration (CORS, HOST, PORT)
│   └── .env.example
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   │   ├── Navbar.tsx
│   │   │   ├── Footer.tsx
│   │   │   ├── DisclaimerBanner.tsx
│   │   │   ├── ModelSelector.tsx
│   │   │   ├── ImageUploader.tsx
│   │   │   ├── ProbabilityChart.tsx
│   │   │   ├── GradCamViewer.tsx
│   │   │   ├── PredictionResults.tsx
│   │   │   └── AnalysisHistory.tsx
│   │   ├── pages/
│   │   │   ├── LandingPage.tsx
│   │   │   ├── AnalysisDashboard.tsx
│   │   │   └── ModelPerformancePage.tsx
│   │   ├── services/
│   │   │   └── api.ts           # API service layer with XHR progress support
│   │   ├── hooks/
│   │   │   └── useModels.ts
│   │   ├── types/
│   │   │   └── index.ts
│   │   ├── App.tsx
│   │   ├── main.tsx
│   │   └── index.css
│   ├── package.json
│   ├── vite.config.ts           # Vite config with /api proxy to FastAPI
│   ├── tailwind.config.js
│   ├── tsconfig.json
│   └── index.html
└── README.md
```

---

## 📋 Prerequisites

- **Python**: 3.8 to 3.12 (or 3.14 via `py` launcher)
- **Node.js**: 18+ and `npm`

---

## 🚀 Quick Start Guide

### 1. Place Trained Model Weights

Download your trained `.keras` files from Google Colab / Google Drive and place them in the `backend/models/` directory:

- Primary ResNet50 Model:
  `multi-organ-ai/backend/models/multi_organ_disease_model.keras`
- Optional MobileNetV2 Model:
  `multi-organ-ai/backend/models/mobilenetv2_disease_model.keras`

*(See `backend/models/README.md` for exact export code from Colab)*

---

### 2. Backend Setup & Startup

Open a terminal in the `backend/` directory:

```powershell
cd "d:\project 123\multi-organ-ai\backend"

# (Optional) Create virtual environment
py -m venv venv
.\venv\Scripts\activate

# Install dependencies
py -m pip install -r requirements.txt

# Start the FastAPI server
py -m uvicorn app.main:app --reload --port 8000
```

Verify backend health at:
- **API Health**: [http://localhost:8000/api/health](http://localhost:8000/api/health)
- **Interactive Swagger Docs**: [http://localhost:8000/docs](http://localhost:8000/docs)
- **Model Catalog**: [http://localhost:8000/api/models](http://localhost:8000/api/models)

---

### 3. Frontend Setup & Startup

Open a second terminal in the `frontend/` directory:

```powershell
cd "d:\project 123\multi-organ-ai\frontend"

# Install npm packages
npm install

# Start Vite development server
npm run dev
```

Open your browser at:
👉 **[http://localhost:5173](http://localhost:5173)**

---

## 📡 REST API Reference

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/health` | Health check & model load status |
| `GET` | `/api/models` | List available models, classes & capabilities |
| `POST` | `/api/predict` | Multipart upload for classification & Grad-CAM |
| `GET` | `/api/history` | Retrieve current session prediction history |
| `DELETE`| `/api/history` | Clear session prediction history |
| `GET` | `/docs` | Interactive OpenAPI / Swagger UI |

---

## 🔬 Model Performance Benchmarks

### ResNet50 Transfer Learning (Primary Architecture)
Evaluated on a held-out test split of 60 scans (20 per organ):

- **Overall Test Accuracy**: **96.67%** (58 / 60 correct)
- **Macro Average F1**: **0.97**
- **Organ Breakdown**:
  - **BRAIN (MRI)**: Precision: 0.95 | Recall: 0.95 | F1: 0.95
  - **KIDNEY (CT)**: Precision: 0.95 | Recall: 0.95 | F1: 0.95
  - **LUNG (X-Ray)**: Precision: 1.00 | Recall: 1.00 | F1: 1.00
- **Grad-CAM Conv Layer**: `conv5_block3_out` (2048 feature maps)

### MobileNetV2 (Sub-Type Architecture)
Evaluated across 6 granular pathological and normal categories:

- **Overall Test Accuracy**: **88.0%**
- **Classes**: `BRAIN_NORMAL`, `BRAIN_TUMOR`, `KIDNEY_NORMAL`, `KIDNEY_TUMOR`, `LUNG_NORMAL`, `LUNG_PNEUMONIA`

---

## ⚠️ Medical & Legal Disclaimer

This application is developed strictly for **academic, educational, and computational research demonstrations**. It is **not** certified as a Medical Device, does not provide clinical diagnostic clearance, and must **never** be used as a substitute for professional radiology consultation, diagnostic evaluations, or clinical decision-making. Grad-CAM visual heatmaps depict deep neural network feature associations and do not define verified pathology borders.
