# Deploying DTRS SYSTEM on Vercel

This guide provides step-by-step instructions for deploying the **DTRS SYSTEM • Compound Delay & Segment Engine** (Next.js 14 Frontend + FastAPI Python Backend) to [Vercel](https://vercel.com).

---

## 1. Architecture Overview

The repository is structured as a full-stack application:
- **`frontend/`**: Next.js 14 React client with TailwindCSS and interactive simulation views.
- **`backend/`**: FastAPI Python server running the 5-Stage Compound Delay Engine, master timetable cache, and simulation sandboxes.
- **`api/index.py`**: Vercel Serverless entrypoint connecting directly to the FastAPI app.
- **`vercel.json`**: Root configuration routing `/api/*` requests to the Python backend and all other routes to Next.js.

---

## 2. Deployment Methods

### Option A: Standard Full-Stack Git Deployment (Recommended)

This deploys both the frontend and backend together using the root [`vercel.json`](./vercel.json).

#### Step 1: Push Code to GitHub
Ensure all your latest files are committed and pushed to your GitHub repository:
```bash
git add .
git commit -m "Prepare repository for Vercel deployment"
git push origin main
```

#### Step 2: Import Project in Vercel
1. Log in to your [Vercel Dashboard](https://vercel.com/dashboard).
2. Click **Add New...** $\rightarrow$ **Project**.
3. Select your GitHub repository (`DTRS-Administrator`).
4. In the **Configure Project** screen:
   - **Project Name**: `dtrs-administrator` (or your choice).
   - **Framework Preset**: `Next.js`.
   - **Root Directory**: Leave as `./` (repository root).

#### Step 3: Configure Environment Variables
Expand the **Environment Variables** section and add the following keys:

| Variable Name | Required | Example / Description |
| :--- | :---: | :--- |
| `RAILRADAR_API_KEY` | Optional | Live GPS tracking API key from RailRadar. |
| `RAILRADAR_API_URL` | Optional | Default: `https://api.railradar.in/v1` |
| `POSTGRESQL` | Optional | Neon Serverless PostgreSQL connection string (e.g., `postgresql://user:pass@ep-xyz.aws.neon.tech/WIN?sslmode=require`). If omitted, uses local `WIN.db`. |
| `BACKEND_URL` | Optional | Set to `http://127.0.0.1:8000` for local or your production API URL if hosting backend separately. |

> **Note**: If `RAILRADAR_API_KEY` is not provided, the system automatically runs the built-in deterministic simulation engine without throwing errors.

#### Step 4: Deploy
Click **Deploy**. Vercel will:
1. Build the Next.js frontend in `frontend/`.
2. Bundle the Python runtime using `requirements.txt` and `backend/AppServer.py`.
3. Provide your live production URL (e.g., `https://dtrs-administrator.vercel.app`).

---

### Option B: Deploying Using Vercel CLI

If you have the [Vercel CLI](https://vercel.com/docs/cli) installed:

1. Open your terminal in the repository root:
   ```bash
   cd c:\Users\AMRITYA\Desktop\DTRS-Administrator
   ```
2. Log in to Vercel:
   ```bash
   vercel login
   ```
3. Deploy to preview:
   ```bash
   vercel
   ```
4. Deploy to production:
   ```bash
   vercel --prod
   ```

---

### Option C: Microservices / Split Deployment (Frontend on Vercel, Backend on Railway/Render)

If you prefer hosting the Python FastAPI backend on a dedicated Python platform (e.g. Railway, Render, Fly.io, or AWS EC2) and keeping Next.js on Vercel:

1. **Deploy Backend**:
   - Point your host to the `backend/` folder.
   - Start command: `uvicorn AppServer:app --host 0.0.0.0 --port $PORT`
   - Copy the live backend URL (e.g., `https://dtrs-backend.up.railway.app`).

2. **Deploy Frontend on Vercel**:
   - In Vercel, set **Root Directory** to `frontend`.
   - Under **Environment Variables**, set:
     ```env
     BACKEND_URL=https://dtrs-backend.up.railway.app
     ```
   - Next.js will automatically proxy all `/api/*` calls from the browser to your deployed backend.

---

## 3. Serverless Environment Specifics

### Read-Only Filesystem & Simulation Sandbox
Vercel Serverless Functions have a read-only filesystem except for the `/tmp` directory.
- **`AppServer.py`** automatically checks `os.getenv('VERCEL')`.
- When running on Vercel, the simulation database sandbox (`WIN_SIMULATION.db`) is copied to `/tmp/WIN_SIMULATION.db`.
- This ensures simulation push (`POST /api/simulation/push`) and reset (`POST /api/simulation/reset`) operations succeed without read-only filesystem errors.

### Cold Starts & Dataset Caching
- Base timetable schedules and corridor segments (~10,620 trains, 1,843 segments) load in memory upon function initialization.
- Subsequent calls within the same warm lambda instance execute with sub-millisecond latency.

---

## 4. Verification After Deployment

Once deployed, verify that the application is operating correctly by testing these endpoints:

| Endpoint | Expected Status | Description |
| :--- | :---: | :--- |
| `https://your-app.vercel.app/` | `200 OK` | Main DTRS web dashboard |
| `https://your-app.vercel.app/api/simulation/status` | `200 OK` | Master vs Simulation DB status |
| `https://your-app.vercel.app/api/search?q=12001` | `200 OK` | Train timetable search |
| `https://your-app.vercel.app/api/train_info?train_no=12001` | `200 OK` | Train schedule & corridor info |
| `https://your-app.vercel.app/api/predict?train_no=12001` | `200 OK` | 5-Stage compound delay calculation |
| `https://your-app.vercel.app/api/railradar/config` | `200 OK` | API key status verification |

---

## 5. Troubleshooting

- **404 on API endpoints**:
  Ensure root [`vercel.json`](./vercel.json) is present in your repository root with the `"rewrites"` block directing `/api/(.*)` to the backend service.
- **Build fails on Node.js**:
  Ensure Node.js version is set to 18.x or 20.x in **Vercel Project Settings $\rightarrow$ General $\rightarrow$ Node.js Version**.
- **Missing Python Packages**:
  Verify [`requirements.txt`](./requirements.txt) exists at the repository root and inside `backend/`.

  Using OG mail
  [EMAIL_ADDRESS]
  