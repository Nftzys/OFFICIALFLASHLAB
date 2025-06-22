# OFFICIALFLASHLAB

This project consists of a FastAPI backend and a Next.js frontend.  It was
initially used for local development and can now be deployed on a VPS with
Nginx acting as a reverse proxy.

## Repository layout

- `flashlabofficial/face_match_app/` – FastAPI application
- `flashlabofficial/frontend/` – Next.js frontend
- `deployment/` – sample systemd and Nginx configuration
- `requirements.txt` – Python dependencies for the backend

## Deployment guide

These steps assume an Ubuntu/Debian VPS.  Adapt paths and package manager
commands as needed.

### 1. Install system packages

```bash
sudo apt update
sudo apt install -y python3-venv python3-pip nodejs npm nginx
```

### 2. Clone the repository

```bash
sudo mkdir -p /opt/flashlab
sudo chown $USER /opt/flashlab
git clone <REPO_URL> /opt/flashlab
cd /opt/flashlab
```

### 3. Backend setup

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 4. Frontend build

```bash
cd flashlabofficial/frontend
npm install
npm run build
cd ../..
```

### 5. Configure systemd services

Copy the provided service files:

```bash
sudo cp deployment/backend.service /etc/systemd/system/flashlab-backend.service
sudo cp deployment/frontend.service /etc/systemd/system/flashlab-frontend.service
sudo systemctl daemon-reload
sudo systemctl enable --now flashlab-backend.service flashlab-frontend.service
```

### 6. Configure Nginx

```bash
sudo cp deployment/nginx.conf /etc/nginx/sites-available/flashlab.conf
sudo ln -s /etc/nginx/sites-available/flashlab.conf /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

Edit `deployment/nginx.conf` and replace `example.com` with your domain or IP
address.  SSL can be enabled with Certbot once everything works.

The frontend is served on port 3000 and the API on port 8000; Nginx proxies
requests so that `/api/` reaches the backend and all other requests go to the
frontend.

You should now have the application available via `http://your-domain/` with the
API reachable under `http://your-domain/api/`.
