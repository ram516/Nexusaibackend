# Deployment Guide - NexusAI

**Production Deployment Instructions**

---

## Local Development

### Setup
```bash
git clone https://github.com/ram516/Nexusaibackend.git
cd Nexusaibackend
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

### Run
```bash
python run.py
# Server: http://localhost:8000
# Docs: http://localhost:8000/docs
```

---

## Production Deployment

### Option 1: Gunicorn + Nginx

#### Install Gunicorn
```bash
pip install gunicorn
```

#### Systemd Service
Create `/etc/systemd/system/nexusai.service`:

```ini
[Unit]
Description=NexusAI Backend
After=network.target

[Service]
Type=notify
User=www-data
WorkingDirectory=/opt/nexusai
ExecStart=/usr/bin/python3 -m gunicorn \
    -w 4 \
    -k uvicorn.workers.UvicornWorker \
    --bind 0.0.0.0:8000 \
    app.main:app
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

#### Enable Service
```bash
sudo systemctl daemon-reload
sudo systemctl enable nexusai
sudo systemctl start nexusai
```

#### Nginx Configuration
Create `/etc/nginx/sites-available/nexusai`:

```nginx
upstream nexusai_backend {
    server localhost:8000 weight=1;
    server localhost:8001 weight=1;
    server localhost:8002 weight=1;
    server localhost:8003 weight=1;
}

server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://nexusai_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # CORS headers
        add_header 'Access-Control-Allow-Origin' '*';
        add_header 'Access-Control-Allow-Methods' 'POST, GET, DELETE, OPTIONS';
        add_header 'Access-Control-Allow-Headers' 'Content-Type';
        add_header 'Access-Control-Max-Age' '86400';
    }
}
```

#### Enable Nginx Site
```bash
sudo ln -s /etc/nginx/sites-available/nexusai /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

### Option 2: Docker

#### Dockerfile
```dockerfile
FROM python:3.10-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    libsm6 \
    libxext6 \
    libxrender-dev \
    && rm -rf /var/lib/apt/lists/*

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["python", "run.py"]
```

#### Build & Run
```bash
docker build -t nexusai-backend .
docker run -d -p 8000:8000 nexusai-backend
```

#### Docker Compose
```yaml
version: '3.8'

services:
  backend:
    build: .
    ports:
      - "8000:8000"
    environment:
      - WORKERS=4
    restart: always

  nginx:
    image: nginx:latest
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - backend
```

---

## Environment Variables

Create `.env` file:

```bash
DATABASE_URL=sqlite:///drivers.db
CORS_ORIGINS=https://app.example.com
WORKERS=4
LOG_LEVEL=INFO
ENABLE_METRICS=true
```

---

## SSL/TLS Configuration

### Let's Encrypt with Certbot
```bash
sudo apt-get install certbot python3-certbot-nginx
sudo certbot certonly --nginx -d api.example.com
```

### Nginx HTTPS Configuration
```nginx
server {
    listen 443 ssl http2;
    server_name api.example.com;

    ssl_certificate /etc/letsencrypt/live/api.example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.example.com/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;

    # ... rest of configuration
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name api.example.com;
    return 301 https://$server_name$request_uri;
}
```

---

## Monitoring

### Health Check Endpoint
```bash
curl http://localhost:8000/
# Expected: {"message": "Smart Vehicle AI Backend Running"}
```

### Systemd Status
```bash
sudo systemctl status nexusai
sudo journalctl -u nexusai -f  # Live logs
```

### Nginx Status
```bash
sudo systemctl status nginx
```

---

## Database Backup

```bash
# Backup
cp drivers.db drivers.db.backup.$(date +%s)

# Or with compression
gzip -c drivers.db > drivers.db.$(date +%Y%m%d).gz
```

---

## Performance Tuning

### Gunicorn Workers
```
workers = (2 × cpu_count) + 1
# For 4-core CPU: workers = 9
# But for CPU-bound AI: use cpu_count + 1 = 5
```

### Timeout Configuration
```bash
gunicorn --timeout 30 ...  # 30 seconds per request
```

---

**Version**: 1.0.0 | **Last Updated**: 2026-06-29