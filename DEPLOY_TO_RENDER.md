# Deploying Multi-Organ AI to Render (Render.com)

This guide walks you through deploying both the **FastAPI Backend** and **React Vite Frontend** to [Render](https://render.com).

---

## 📑 Deployment Options Summary

| Method | Description | Best For |
|---|---|---|
| **Option 1: Render Blueprint (`render.yaml`)** ⭐ | Automatic 1-click deployment of Backend Web Service + Frontend Static Site | Recommended for quick setup |
| **Option 2: Unified Docker Service** | Single free Render Web Service running both API and Frontend together | Simplest setup, zero CORS configuration |
| **Option 3: Manual Dashboard Setup** | Manually creating the Web Service and Static Site via the Render UI | Full visual control |

---

## ⚠️ Important: Handling Trained Model Weights on Render

Trained TensorFlow model files (`.keras`) are often 50MB–110MB. GitHub blocks files over 100MB by default.

### Recommended Cloud Method: Automatic Model Download (Zero Git Bloat)
Our backend code has been enhanced to automatically download model weights on startup if not present locally!

1. Upload your `multi_organ_disease_model.keras` file to **Google Drive**, **GitHub Releases**, **Hugging Face**, or **Dropbox**.
2. Get the direct download URL.
3. In Render's Environment Variables, set:
   - `RESNET50_MODEL_URL`: `https://...your-direct-download-url...`
   - `MOBILENETV2_MODEL_URL`: *(optional, for secondary model)*

*(If your file is under 100MB and already in `backend/models/multi_organ_disease_model.keras`, you can also commit it directly to Git).*

---

## 🚀 Step 1: Push Project to GitHub

In your local command prompt or PowerShell:

```powershell
cd "d:\project 123\multi-organ-ai"

# Initialize git repository
git init

# Add all files
git add .

# Commit
git commit -m "Initial commit for Multi-Organ AI application"

# Rename branch to main
git branch -M main

# Add your GitHub repository remote
# (Create a new private or public repository on https://github.com/new first)
git remote add origin https://github.com/<YOUR_GITHUB_USERNAME>/<YOUR_REPOSITORY_NAME>.git

# Push code
git push -u origin main
```

---

## ⚡ Option 1: 1-Click Deployment with Render Blueprint (Recommended)

The repository includes a pre-configured [render.yaml](file:///d:/project%20123/multi-organ-ai/render.yaml) blueprint file.

1. Log in to [Render Dashboard](https://dashboard.render.com).
2. Click the **"New +"** button at the top right and select **"Blueprint"**.
3. Connect your GitHub account and select your repository.
4. Render will automatically detect `render.yaml` and display the blueprint plan:
   - `multi-organ-ai-backend` (Python Web Service)
   - `multi-organ-ai-frontend` (Static Site)
5. Under **Environment Variables**:
   - If using remote weights, fill in `RESNET50_MODEL_URL` with your direct model download URL.
6. Click **"Apply"**.
7. Render will build and deploy both services automatically!
   - Frontend will automatically be wired to the backend URL via `VITE_API_URL`.

---

## 🐳 Option 2: Single-Service Unified Docker Deployment

If you want to host both the backend and frontend within **one single free service**:

1. In Render Dashboard, click **"New +"** → **"Web Service"**.
2. Select your repository.
3. Choose **Docker** as the Runtime (Render automatically detects our root [Dockerfile](file:///d:/project%20123/multi-organ-ai/Dockerfile)).
4. Set:
   - **Instance Type**: `Free`
   - **Environment Variables**:
     - `PORT`: `8000`
     - `ALLOWED_ORIGINS`: `*`
     - `RESNET50_MODEL_URL`: *(Optional URL to download model weights)*
5. Click **"Create Web Service"**.
6. Render builds the React frontend, packages it with FastAPI, and serves the entire app from a single URL!

---

## 🛠️ Option 3: Manual Deployment via Render Dashboard

If you prefer to configure each service manually in the UI:

### Part A: Deploy Backend Web Service
1. In Render, click **"New +"** → **"Web Service"**.
2. Select your repository.
3. Fill in the fields:
   - **Name**: `multi-organ-backend`
   - **Region**: `Oregon` (or closest to you)
   - **Root Directory**: `backend`
   - **Runtime**: `Python`
   - **Build Command**: `pip install --upgrade pip && pip install -r requirements.txt`
   - **Start Command**: `uvicorn app.main:app --host 0.0.0.0 --port $PORT`
   - **Plan**: `Free`
4. Expand **"Advanced"** → **Add Environment Variables**:
   - `PYTHON_VERSION`: `3.11.9`
   - `ALLOWED_ORIGINS`: `*`
   - `MODEL_DIR`: `./models`
   - `RESNET50_MODEL_URL`: *(Your model download URL, if not committed to Git)*
5. Click **"Create Web Service"**.
6. Copy the URL assigned by Render (e.g., `https://multi-organ-backend.onrender.com`).

### Part B: Deploy Frontend Static Site
1. In Render, click **"New +"** → **"Static Site"**.
2. Select the same repository.
3. Fill in the fields:
   - **Name**: `multi-organ-frontend`
   - **Root Directory**: `frontend`
   - **Build Command**: `npm install && npm run build`
   - **Publish Directory**: `dist`
4. Expand **"Advanced"** → **Add Environment Variable**:
   - `VITE_API_URL`: `https://multi-organ-backend.onrender.com` *(paste your backend URL from Part A)*
5. Under **"Redirects/Rewrites"**, add a rewrite rule:
   - **Type**: `Rewrite`
   - **Source**: `/*`
   - **Destination**: `/index.html`
   *(This ensures client-side routing like `/analyze` works on reload)*
6. Click **"Create Static Site"**.

---

## ✅ Verifying Your Deployment

1. **Verify Backend Health**:
   Visit `https://<your-backend-name>.onrender.com/api/health`
   You should see:
   ```json
   {
     "status": "ok",
     "resnet50_loaded": true,
     "mobilenetv2_loaded": false,
     "message": "ResNet50 model is ready for predictions."
   }
   ```
2. **Open the Web App**:
   Visit your frontend URL (`https://<your-frontend-name>.onrender.com`).
   - The top navigation bar should display the green **Connected** status indicator.
   - Upload a sample scan and run AI classification with Grad-CAM!

---

## 💡 Troubleshooting on Render

- **Cold Starts (Free Tier)**: Render free tier services sleep after 15 minutes of inactivity. The first request after sleep may take ~30–50 seconds to wake up.
- **Model Load Memory**: If the free tier instance (512MB RAM) runs low on memory during TensorFlow load, use `RESNET50_MODEL_URL` so weights are downloaded cleanly into disk cache.
- **CORS Issues**: Ensure `ALLOWED_ORIGINS` is set to `*` on the backend, or explicitly includes your frontend Render URL.
