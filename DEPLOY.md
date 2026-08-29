# Aasha Production Deployment Guide

This guide details the steps to deploy the Aasha monorepo ecosystem to cloud providers.

---

## 1. Database Setup (Supabase)

Supabase provides a hosted PostgreSQL instance with standard connection interfaces.

### Provisioning Steps:
1. Sign in to [Supabase](https://supabase.com/).
2. Create a new project named `aasha-prod`.
3. Set a secure database password.
4. Once provisioned, navigate to **Project Settings > Database**.
5. Copy the **Connection String** under **URI** (select node-postgres or pooler format). It looks like:
   `postgresql://postgres.[YOUR_PROJECT_ID]:[YOUR_PASSWORD]@aws-0-[REGION].pooler.supabase.com:6543/postgres?sslmode=require`

---

## 2. Backend Deployment (Railway)

Railway is used to host the Fastify API server and background transformed chapter queue workers.

### Configuration Steps:
1. Sign in to [Railway](https://railway.app/).
2. Click **New Project** > **Deploy from GitHub repository** and select your repository.
3. Railway will detect the monorepo. Configure the service settings:
   - **Build Command**: `npm run build:server`
   - **Start Command**: `npm run db:migrate && npm run start:server`
4. Under **Variables** (Environment variables), add the following:
   - `DATABASE_URL`: *[Your Supabase Database Connection URI copied in Step 1]*
   - `GEMINI_API_KEY`: *[Your Google Gemini API Key]*
   - `PORT`: `3000` *(automatically injected by Railway, but good to define fallback)*
5. Railway will automatically deploy the backend, run the schema migrations on Supabase, and expose a public URL (e.g. `https://aasha-backend.up.railway.app`). Copy this URL.

---

## 3. Frontend Deployment (Vercel)

Vercel is used to host the React + Vite + Tailwind facilitator dashboard.

### Configuration Steps:
1. Sign in to [Vercel](https://vercel.com/).
2. Click **Add New** > **Project** and import your repository.
3. In the project setup panel:
   - **Framework Preset**: `Vite`
   - **Root Directory**: Keep as project root `.` (or set to `dashboard` if building standalone, but keeping root `.` allows Vercel to resolve monorepo packages like `@aasha/shared-types`).
   - If using root `.`:
     - **Build Command**: `npm run build:dashboard`
     - **Output Directory**: `dashboard/dist`
4. Under **Environment Variables**, add:
   - `VITE_API_URL`: `https://[YOUR_RAILWAY_BACKEND_URL]/api/v1` *(The backend URL copied in Step 2)*
5. Click **Deploy**. Vercel will build and host the dashboard. SPA client-side routes will be handled gracefully via the `vercel.json` rewrite file we created.

---

## 4. Verification Check

Once both services are active:
1. Visit your Vercel URL (e.g. `https://aasha-dashboard.vercel.app/status`).
2. Log in using the developer bypass mode or standard credentials.
3. Test uploading a chapter PDF or editing a chapter JSON.
