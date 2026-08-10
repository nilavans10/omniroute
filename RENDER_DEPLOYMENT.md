# Deploying OmniRoute to Render

This guide walks you through deploying OmniRoute as a live Web Service on Render using the existing Docker configuration.

## Prerequisites

1. A **Render account** (sign up at [render.com](https://render.com)).
2. A **GitHub** or **GitLab** repository containing this project.
3. If you want to use the automated blueprint deployment, make sure the project is pushed to your Git repository.

---

## Method 1: Deploy using Render Blueprints (Recommended)

Render Blueprints let you define your infrastructure as code using the `render.yaml` file we created.

1. Push this project to your GitHub/GitLab repository.
2. Log in to the [Render Dashboard](https://dashboard.render.com).
3. Click **New +** at the top right and select **Blueprint**.
4. Connect your repository containing the project.
5. Render will automatically read `render.yaml` and configure:
   - A Web Service named `omniroute`.
   - The correct Docker build commands and the `runner-base` target.
   - Environment variables (`PORT`, `NODE_ENV`, `DATA_DIR`, `OMNIROUTE_MEMORY_MB`).
   - A **10 GB Persistent Disk** mounted at `/app/data` to persist your SQLite database.
6. Click **Approve** to deploy.

---

## Method 2: Manual Deployment via Render Dashboard

If you prefer to configure the service manually through the Render UI, follow these steps:

### 1. Create a Web Service
1. Log in to [Render Dashboard](https://dashboard.render.com).
2. Click **New +** and select **Web Service**.
3. Connect your Git repository.

### 2. Configure Build & Runtime
In the service creation form, set the following values:
* **Name**: `omniroute`
* **Region**: Choose the region closest to your users.
* **Instance Type**: Select **Starter** or higher (at least 1 GB RAM is required; the Free plan does not support persistent disks and lacks sufficient memory).
* **Runtime**: `Docker`
* **Docker Command**: (Leave empty; Render will use the default `CMD` in the Dockerfile).

### 3. Configure Docker Target (Optional)
If your repository is nested, or if you want to use the web-cookie provider support:
* Under **Advanced**, look for **Docker Build Target**:
  - Set it to `runner-base` for standard lightweight deployment.
  - Set it to `runner-web` if you need Playwright/Chromium browser engines (for cookie-based Gemini Web/Claude Turnstile providers).
* If your Dockerfile is in a subdirectory (e.g. `OmniRoute-release-v3.8.50`), set:
  - **Docker Build Context**: `OmniRoute-release-v3.8.50`
  - **Dockerfile Path**: `OmniRoute-release-v3.8.50/Dockerfile`

### 4. Add Environment Variables
Under the **Environment** section, add:
* `PORT`: `20128`
* `NODE_ENV`: `production`
* `DATA_DIR`: `/app/data`
* `OMNIROUTE_MEMORY_MB`: `1024`

### 5. Attach a Persistent Disk (Critical for SQLite)
Since OmniRoute uses a SQLite database, Render's default ephemeral storage will erase database state (users, routing rules) on every restart.
1. Scroll down to the **Disks** section under **Advanced**.
2. Click **Add Disk**.
3. Configure the disk:
   * **Name**: `omniroute-data`
   * **Mount Path**: `/app/data`
   * **Size**: `10 GB` (or as preferred)
4. Click **Create Web Service**.

---

## Accessing Your Service

Once deployed successfully, Render will provide a public URL (e.g., `https://omniroute.onrender.com`). You can visit this URL to access the OmniRoute dashboard.
