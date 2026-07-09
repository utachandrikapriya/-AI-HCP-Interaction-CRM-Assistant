# Live Deployment Guide: HCP CRM Assistant

This guide explains how to deploy the HCP CRM Interaction Assistant live for production.

---

## 1. Deploy the Backend (FastAPI) on **Render** (Free Tier)
Render is a cloud platform that allows hosting FastAPI services with automatic deployments from your GitHub repository.

### Step A: Prepare the Backend Code
1. Ensure your backend code is in the repository.
2. Render needs to know how to start the FastAPI server. The startup command is:
   ```bash
   uvicorn app.main:app --host 0.0.0.0 --port $PORT
   ```

### Step B: Create a Web Service on Render
1. Go to [Render](https://render.com/) and sign up.
2. Click **New +** and select **Web Service**.
3. Connect your GitHub repository.
4. Set the following configuration details:
   - **Name**: `hcp-crm-backend`
   - **Runtime**: `Python`
   - **Build Command**: `pip install -r backend/requirements.txt`
   - **Start Command**: `uvicorn app.main:app --host 0.0.0.0 --port $PORT`
   - **Cwd (Root Directory)**: `backend`
5. Click **Deploy Web Service**. Render will assign a live URL (e.g. `https://hcp-crm-backend.onrender.com`).

### Step C: Persistent Database (Optional but Recommended)
SQLite databases are ephemeral on free servers (they reset whenever the server restarts). For persistent storage:
1. Spin up a free PostgreSQL database on [Neon.tech](https://neon.tech/) or [Supabase](https://supabase.com/).
2. Copy the database connection URL.
3. On Render, go to your Web Service's **Environment** tab and add:
   - Key: `DATABASE_URL`
   - Value: `postgresql://username:password@host/dbname`
4. If you have API keys (Groq, Gemini, OpenAI), add them to the environment variables as well (e.g., `GROQ_API_KEY`, `GEMINI_API_KEY`).

---

## 2. Deploy the Frontend (React) on **Vercel** (Free Tier)
Vercel is the premier platform for static hosting, offering automatic builds and global CDN distribution.

### Step A: Configure Production API URL
Ensure your React app knows where to reach your live backend.
1. Open `frontend/src/App.jsx`.
2. Locate the backend URL definition. In production, it should point to your Render backend:
   ```javascript
   const BACKEND_URL = import.meta.env.VITE_API_URL || "http://localhost:8000";
   ```
3. Update the fetch statements to use this configuration.

### Step B: Deploy on Vercel
1. Go to [Vercel](https://vercel.com/) and sign up.
2. Click **Add New** -> **Project**.
3. Import your GitHub repository.
4. Set the project settings:
   - **Framework Preset**: `Vite`
   - **Root Directory**: `frontend`
5. Open **Environment Variables** and add:
   - Key: `VITE_API_URL`
   - Value: `https://hcp-crm-backend.onrender.com` (your live Render backend URL)
6. Click **Deploy**. Vercel will build your static files and give you a public URL (e.g., `https://hcp-crm-assistant.vercel.app`).

---

## 3. Alternative: Docker Deployment (Single Server VPS)
If you own a virtual private server (like AWS, DigitalOcean, or Linode), you can deploy both using Docker Compose.

### `docker-compose.yml` (Root Directory)
```yaml
version: '3.8'

services:
  backend:
    build: ./backend
    ports:
      - "8000:8000"
    environment:
      - GROQ_API_KEY=${GROQ_API_KEY}
      - GEMINI_API_KEY=${GEMINI_API_KEY}
    volumes:
      - crm-db:/app/data

  frontend:
    build: ./frontend
    ports:
      - "80:80"
    depends_on:
      - backend

volumes:
  crm-db:
```
