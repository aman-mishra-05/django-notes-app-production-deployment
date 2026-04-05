# 📝 Django Notes App — Production Deployment

A production-grade deployment of a full-stack Notes Application using **Docker**, **MySQL**, **Nginx**, and **AWS EC2**.

> DevOps & Deployment by [Aman Mishra](https://github.com/aman-mishra-05)

---

## 🏗️ Architecture Overview

```
User Browser
     │
     ▼
 [ Nginx ]             ← Reverse Proxy (Port 80)
     │
     ▼
[ Django + Gunicorn ]  ← App Server (Port 8000)
     │
     ▼
   [ MySQL ]           ← Database (Port 3306)
```

All three services run as **separate Docker containers**, orchestrated via **Docker Compose**, and deployed on **AWS EC2**.

---

## 🛠️ DevOps Stack

| Tool | Purpose |
|------|---------|
| Docker | Containerization |
| Docker Compose | Multi-container orchestration |
| Nginx | Reverse proxy |
| MySQL | Production database |
| Whitenoise | Static file serving |
| AWS EC2 | Cloud hosting |
| Git + GitHub | Version control |

---

## 📁 Project Structure

```
django-notes-app/
├── Dockerfile              # Django app container
├── docker-compose.yml      # Orchestrates all 3 containers
├── .dockerignore           # Excludes heavy folders from build context
├── .gitignore              # Excludes .env, build, node_modules
├── requirements.txt        # Python dependencies (handled inside Docker)
├── nginx/
│   ├── Dockerfile          # Nginx container
│   └── default.conf        # Reverse proxy configuration
└── mynotes/                # React frontend source (don't touch)
```

---

## ⚙️ Prerequisites

Make sure the following are installed on your server:

- [Docker](https://docs.docker.com/get-docker/)
- [Docker Compose](https://docs.docker.com/compose/install/)
- [Node.js & npm](https://nodejs.org/) — only needed to build the React frontend once
- AWS EC2 instance — **t3.small or higher recommended** (t3.micro struggles with 3 containers)

---

## 🚀 Deployment Steps

### Step 1 — Clone the repository
```bash
git clone https://github.com/aman-mishra-05/django-notes-app-production-deployment.git
cd django-notes-app-production-deployment
```

### Step 2 — Create your `.env` file
```bash
nano .env
```

Add the following variables:
```env
SECRET_KEY=your_django_secret_key
DB_NAME=notesdb
DB_USER=root
DB_PASSWORD=your_mysql_password
DB_HOST=mysql
DB_PORT=3306
MYSQL_ROOT_PASSWORD=your_mysql_password
MYSQL_DATABASE=notesdb
```

### Step 3 — Build the React frontend
This is a one-time step. The build output gets copied into the Docker image automatically.
```bash
cd mynotes
npm install
npm run build
cd ..
```

### Step 4 — Add swap memory
Required if you're on a t3.micro (1GB RAM). Prevents the instance from freezing.
```bash
sudo fallocate -l 1G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

### Step 5 — Build and start all containers
```bash
docker compose up --build -d
```

### Step 6 — Verify everything is running
```bash
docker ps
```

All 3 containers should show status **Up** and **Healthy**:

| Container | Status |
|-----------|--------|
| `nginx_cont` | Up |
| `django_cont` | Healthy |
| `mysql` | Healthy |

### Step 7 — Access the app
Open your browser and go to:
```
http://<your-ec2-public-ip>
```

---

## 🔧 Useful Commands

| Task | Command |
|------|---------|
| Start containers | `docker compose up -d` |
| Stop containers | `docker compose down` |
| Rebuild containers | `docker compose up --build -d` |
| View all container logs | `docker compose logs` |
| View specific container logs | `docker logs <container_name>` |
| Free up Docker disk space | `docker system prune -af` |
| Check disk usage | `df -h` |
| Check memory usage | `free -h` |

---

## 🌐 Nginx Configuration

Nginx acts as a reverse proxy, forwarding all traffic from port **80** to Django on port **8000**.

```nginx
upstream django {
    server django:8000;
}

server {
    listen 80;
    server_name localhost;

    location / {
        proxy_pass http://django;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

---

## ⚠️ Important Notes

- Never commit your `.env` file — it contains sensitive credentials
- The `mynotes/build` and `mynotes/node_modules` folders are excluded from git
- MySQL data is persisted using a Docker volume (`mysql-data`) — data survives container restarts
- All containers have `restart: always` — they auto-start after EC2 reboots
- Containers start in order: MySQL → Django → Nginx using `healthcheck` + `depends_on`

---

## 🔒 AWS Security Group — Required Inbound Rules

| Port | Protocol | Source | Purpose |
|------|----------|--------|---------|
| 22 | TCP | Your IP | SSH access |
| 80 | TCP | 0.0.0.0/0 | HTTP (Nginx) |
| 8000 | TCP | 0.0.0.0/0 | Django (optional) |

---

> Used for DevOps learning and production deployment practice
