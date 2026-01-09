# GeoSentinel 

**A mine-safety and geotechnical monitoring platform** combining a FastAPI backend (ML models, video/frame analysis, chatbot) with a Vite + React frontend.

**Youtube link :** https://youtu.be/0eWDi7hRyVU?si=BX7edHwNt_ZLY78h  

---

## 🔧 Tech stack

- Backend: Python, FastAPI, Uvicorn
- ML / Vision: OpenCV, CatBoost, custom GenAI integration (Google Generative AI)
- Frontend: React (Vite + TypeScript)
- Misc: ffmpeg (for video conversion), dotenv for environment variables

---

## 🚀 Quick overview

- The backend exposes endpoints for chat, video/frame predictions, slope stability prediction, and real-time weather data.
- The frontend (in `Frontend/`) is a Vite React app that talks to the backend.

---

## 📋 Prerequisites

- Python 3.11+ (recommended)
- Node.js 18+ (for frontend)
- ffmpeg (system installed and available on PATH) — required for `/predict_vedio` video conversions
- Git (optional)

---

## 🧩 Environment variables

The code expects several environment variables (stored in `.env` in `Backend/` or root):

- GOOGLE_API_KEY, GOOGLE_API_KEY1, GOOGLE_API_KEY2 (used by internal Generative AI code)
- OWM_API_KEY (OpenWeatherMap API key for realtime weather)
- TAVILY_API_KEY (used by websearch / tools)
- OUTER_SURFACE_API_KEY (used by single-frame / video models)
- AGRO_API_KEY (used by some tools)
- Optional frontend: `VITE_BACKEND_URL` to point the frontend to a different backend URL

Important: there is an existing `.env` in the repo root/Backend; these are sensitive credentials — rotate them if they are real secrets and **never** commit secret keys to public repos. Use `.env` locally and keep it in `.gitignore`.

Create a `.env` file with values like:

```
GOOGLE_API_KEY=your_key_here
GOOGLE_API_KEY1=your_key_here
OWM_API_KEY=your_key_here
TAVILY_API_KEY=your_key_here
OUTER_SURFACE_API_KEY=your_key_here
AGRO_API_KEY=your_key_here
```

---

## Backend — setup & run (development)

1. Open a terminal in the repository root.

2. Create and activate a virtual environment (Windows example):

```powershell
py -3 -m venv .venv
.\.venv\Scripts\Activate.ps1   # PowerShell
# or .\.venv\Scripts\activate.bat for cmd
```

3. Install dependencies (recommended to use the backend requirements):

```powershell
pip install -r Backend/requirements.txt
# or to install everything (ML-heavy): pip install -r requirements.txt
```

4. Ensure `ffmpeg` is installed and on your PATH. On Windows you can use Chocolatey (if available):

```powershell
choco install ffmpeg
```

—or download from https://ffmpeg.org and add to PATH.

5. Create a `.env` file in `Backend/` (see env section above).

6. Run the FastAPI server (from repo root):

```powershell
uvicorn Backend.main:app --reload --host 0.0.0.0 --port 8000
```

7. Verify: open http://127.0.0.1:8000 → should show a JSON message.
- FastAPI interactive docs: http://127.0.0.1:8000/docs
- Redoc: http://127.0.0.1:8000/redoc

---

## Frontend — setup & run

1. Change to the frontend folder and install node deps:

```bash
cd Frontend
npm install
# or pnpm install / bun install
```

2. (Optional) Set the backend URL for development by creating `Frontend/.env` with:

```
VITE_BACKEND_URL=http://127.0.0.1:8000
```

3. Start the dev server:

```bash
npm run dev
```

4. Open the URL printed by Vite (usually http://localhost:5173) and interact with the app.

5. Build for production:

```bash
npm run build
```

---

## 🔗 API endpoints & examples

Base: `http://127.0.0.1:8000`

- GET `/` — health check
  - Example: curl http://127.0.0.1:8000

- POST `/chat` — chatbot
  - JSON body: `{ "user_query": "Is the slope stable?", "user_id": "user123" }`
  - Example:

```bash
curl -X POST "http://127.0.0.1:8000/chat" -H "Content-Type: application/json" -d '{"user_query":"Hi","user_id":"u1"}'
```

- POST `/predict_vedio` — upload a video for analysis (note: endpoint name uses `vedio` as in code)
  - Form file field: `file`

```bash
curl -F "file=@/path/to/video.mp4" http://127.0.0.1:8000/predict_vedio
```

- POST `/predict_frame` — upload an image for frame anomaly detection

```bash
curl -F "file=@/path/to/image.jpg" http://127.0.0.1:8000/predict_frame
```

- GET `/realtimedata?lat=<lat>&lon=<lon>` — weather and 7-day rainfall forecast

```bash
curl "http://127.0.0.1:8000/realtimedata?lat=21.15&lon=79.10"
```

- POST `/predict_slope` — slope stability (JSON payload with numeric fields)
  - Use the `SlopePredictionRequest` structure (fields like `height`, `cohesion`, `friction_angle`, etc.)

- POST `/curl` — simple rotating test endpoint

---

## 📝 Notes & gotchas

- The video endpoint uses `ffmpeg` to convert WebM → MP4. If ffmpeg is missing, uploads may fail with a conversion error.
- Many internal modules load `.env` and will raise an error if expected keys are missing (e.g., `GOOGLE_API_KEY`, `OWM_API_KEY`, `TAVILY_API_KEY`).
- Some functions point to hardcoded `.env` paths (e.g., in ML models). If you move your `.env`, update those paths or ensure the environment variables are exported system-wide.
- ML models (CatBoost) and other heavy libs are included in the top-level `requirements.txt`. Installing the full set may be large.

---

## 📦 Deployment hints

- The project contains `Backend/vercel.json` indicating deployment to Vercel (Python builder) is supported. For production consider using:
  - Vercel: for serverless FastAPI endpoints (check config)
  - Docker + Gunicorn + Uvicorn workers: `gunicorn -k uvicorn.workers.UvicornWorker Backend.main:app -w 4`
- Frontend can be built and deployed to any static hosting (Vercel, Netlify, Cloudflare Pages).

---

## ✅ Troubleshooting checklist

- 500 / 400 errors → check backend logs for missing env vars or thrown exceptions
- Video upload failing → confirm `ffmpeg` is in PATH
- Chatbot errors → verify Google API keys are set and valid
- Frontend can't reach backend → set `VITE_BACKEND_URL` or make sure CORS origins in `Backend/main.py` include your frontend host

---

## 🙏 Contributing

Contributions welcome — open an issue or a PR. Please:
- Avoid committing secrets
- Add tests for new features where possible
- Document high-level changes in this README

---

## 📄 License & authors

Add your project license and author information here.

---

If you'd like, I can also:
- Add a sample `.env.example` to the repo (without secrets)
- Add a quick-start script to automate venv + install + run steps

Happy to add either — tell me which one you prefer. 🔧
