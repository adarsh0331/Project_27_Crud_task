# MEAN Stack Application - Complete Deployment Guide

## 📋 Table of Contents

1. [Overview](#overview)
2. [Repository Setup](#repository-setup)
3. [Docker Configuration](#docker-configuration)
4. [Docker Hub Setup](#docker-hub-setup)
5. [Ubuntu VM Setup](#ubuntu-vm-setup)
6. [CI/CD Pipeline](#cicd-pipeline)
7. [Nginx Configuration](#nginx-configuration)
8. [Deployment Steps](#deployment-steps)
9. [Verification](#verification)
10. [Troubleshooting](#troubleshooting)

## 🎯 Overview

This is a complete MEAN stack (MongoDB, Express, Angular, Node.js) CRUD application with:
- **Backend**: Node.js/Express REST API
- **Frontend**: Angular 15 application
- **Database**: MongoDB
- **Reverse Proxy**: Nginx
- **Containerization**: Docker & Docker Compose
- **CI/CD**: GitHub Actions
- **Deployment**: Ubuntu VM on cloud platform

## 📦 Repository Setup

### Step 1: Create GitHub Repository

1. Go to GitHub and create a new repository
2. Name it: `mean-crud-app` (or your preferred name)
3. Initialize with README (optional)

### Step 2: Push Code to GitHub

```bash
# Navigate to project directory
cd crud-dd-task-mean-app

# Initialize git (if not already done)
git init

# Add all files
git add .

# Commit
git commit -m "Initial commit: MEAN stack CRUD application with Docker and CI/CD"

# Add remote repository
git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git

# Push to GitHub
git branch -M main
git push -u origin main
```

## 🐳 Docker Configuration

### Dockerfiles Created

1. **Backend Dockerfile** (`backend/Dockerfile`)
   - Uses Node.js 18 Alpine
   - Installs dependencies
   - Exposes port 8080

2. **Frontend Dockerfile** (`frontend/Dockerfile`)
   - Multi-stage build
   - Builds Angular app
   - Serves with Nginx
   - Exposes port 80

3. **Docker Compose** (`docker-compose.yml`)
   - MongoDB service
   - Backend service
   - Frontend service
   - Nginx reverse proxy

### Build Docker Images Locally (Optional)

```bash
# Build backend
cd backend
docker build -t yourusername/mean-backend:latest .
docker push yourusername/mean-backend:latest

# Build frontend
cd ../frontend
docker build -t yourusername/mean-frontend:latest .
docker push yourusername/mean-frontend:latest
```

## 🏷️ Docker Hub Setup

### Step 1: Create Docker Hub Account

1. Go to https://hub.docker.com
2. Sign up for a free account
3. Note your username

### Step 2: Create Access Token

1. Go to Account Settings → Security
2. Click "New Access Token"
3. Name it: `github-actions`
4. Copy the token (you'll need it for GitHub secrets)

### Step 3: Login to Docker Hub

```bash
docker login -u yourusername
# Enter your password or access token
```

## 🖥️ Ubuntu VM Setup

### Step 1: Launch Ubuntu VM

**AWS EC2:**
1. Launch EC2 instance
2. Choose Ubuntu 22.04 LTS
3. Select instance type (t2.micro is fine for testing)
4. Configure security group:
   - SSH (22) from your IP
   - HTTP (80) from anywhere (0.0.0.0/0)
5. Launch and download key pair

### Step 2: Install Docker on VM

SSH into your VM:

```bash
ssh -i your-key.pem ubuntu@your-vm-ip
```

Install Docker:

```bash
# Update system
sudo apt-get update

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add user to docker group
sudo usermod -aG docker $USER

# Install Docker Compose
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

# Verify installations
docker --version
docker-compose --version

# Logout and login again for group changes
exit
# SSH back in
```

### Step 4: Clone Repository on VM

```bash
cd ~
git clone https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git mean-app
cd mean-app
```

## 🔄 CI/CD Pipeline

### GitHub Actions Workflow

The workflow file (`.github/workflows/ci-cd.yml`) is already created. It:
1. Builds Docker images on code push
2. Pushes images to Docker Hub
3. Deploys to VM automatically

### Configure GitHub Secrets

1. Go to your GitHub repository
2. Navigate to: **Settings** → **Secrets and variables** → **Actions**
3. Click **New repository secret** and add:

   | Secret Name | Value | Description |
   |------------|-------|-------------|
   | `DOCKER_HUB_USERNAME` | your-dockerhub-username | Your Docker Hub username |
   | `DOCKER_HUB_TOKEN` | your-dockerhub-token | Docker Hub access token |
   | `VM_HOST` | your-vm-ip-or-domain | VM public IP or domain |
   | `VM_USER` | ubuntu | SSH username |
   | `VM_SSH_KEY` | content-of-private-key | Private SSH key content |

**To get SSH key content:**
```bash
Copy the private key
# Copy the entire output including -----BEGIN and -----END lines
```

### Test CI/CD Pipeline

1. Make a small change to the code
2. Commit and push:
   ```bash
   git add .
   git commit -m "Test CI/CD pipeline"
   git push origin main
   ```
3. Go to GitHub → Actions tab
4. Watch the workflow run
5. Check deployment on VM

## 🌐 Nginx Configuration

Nginx is configured as a reverse proxy:
- **Port 80**: Main entry point
- **Frontend**: Served from `/`
- **Backend API**: Proxied from `/api/`

Configuration file: `nginx/nginx.conf`

## 🚀 Deployment Steps

### Option 1: Manual Deployment

```bash
# On your VM
cd ~/mean-app

# Update docker-compose.yml with your Docker Hub username
# Or use build context (default)

# Start services
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f
```

### Option 2: Automated via CI/CD

Just push to main/master branch - GitHub Actions handles everything!

## ✅ Verification

### 1. Check Container Status

```bash
docker-compose ps
```

Expected output:
```
NAME              STATUS          PORTS
mean-backend      Up              0.0.0.0:8080->8080/tcp
mean-frontend     Up              0.0.0.0:8081->80/tcp
mean-mongodb      Up              0.0.0.0:27017->27017/tcp
mean-nginx        Up              0.0.0.0:80->80/tcp
```

### 2. Check Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f backend
docker-compose logs -f frontend
docker-compose logs -f mongodb
docker-compose logs -f nginx
```

### 3. Test Application

1. Open browser: `http://your-vm-ip`
2. You should see the Angular application
3. Test CRUD operations:
   - Add a tutorial
   - View tutorials
   - Search tutorials
   - Edit tutorial
   - Delete tutorial

### 4. Test API Directly

```bash
# Test backend health
curl http://your-vm-ip/api/tutorials

# Should return: [] (empty array) or list of tutorials
```

## 🔧 Troubleshooting

### Issue: Containers not starting

```bash
# Check logs
docker-compose logs

# Check Docker daemon
sudo systemctl status docker

# Restart Docker
sudo systemctl restart docker
```

### Issue: MongoDB connection error

```bash
# Check MongoDB logs
docker-compose logs mongodb

# Verify MongoDB is running
docker-compose ps mongodb

# Restart MongoDB
docker-compose restart mongodb
```

### Issue: Frontend not loading

```bash
# Check frontend build
docker-compose logs frontend

# Rebuild frontend
cd frontend
docker-compose build frontend
docker-compose up -d frontend
```

### Issue: CORS errors

- Verify CORS is enabled in `backend/server.js`
- Check Nginx configuration includes CORS headers
- Restart backend and nginx:
  ```bash
  docker-compose restart backend nginx
  ```

### Issue: Port 80 already in use

```bash
# Find what's using port 80
sudo netstat -tulpn | grep :80

# Stop conflicting service or change Nginx port in docker-compose.yml
```

## 📊 Architecture Diagram

```
┌─────────────────────────────────────────┐
│         Internet (Port 80)              │
└─────────────────┬───────────────────────┘
                  │
                  ▼
        ┌─────────────────┐
        │   Nginx Proxy   │
        │   (Port 80)     │
        └────────┬────────┘
                 │
        ┌────────┴────────┐
        │                 │
        ▼                 ▼
┌──────────────┐  ┌──────────────┐
│   Frontend   │  │   Backend    │
│  (Angular)   │  │  (Express)   │
│  Port 8081   │  │  Port 8080   │
└──────────────┘  └──────┬───────┘
                         │
                         ▼
                  ┌──────────────┐
                  │   MongoDB    │
                  │  Port 27017  │
                  └──────────────┘
```

## 🔐 Security Considerations

1. **Firewall**: Only open necessary ports (80, 22)
2. **SSH Keys**: Use SSH keys instead of passwords
3. **Docker Hub**: Use access tokens, not passwords
4. **Secrets**: Never commit secrets to repository
5. **HTTPS**: Consider adding SSL/TLS in production
6. **MongoDB**: Consider adding authentication in production

## 📝 Environment Variables

### Backend
- `PORT`: Server port (default: 8080)
- `MONGODB_URL`: MongoDB connection string

### Frontend
- `API_URL`: Backend API URL (set in environment files)

## 🔄 Updating Application

### Manual Update
```bash
cd ~/mean-app
git pull
docker-compose down
docker-compose build
docker-compose up -d
```

### Automatic Update
- Push changes to main/master branch
- CI/CD pipeline handles the rest

## 📞 Support

For issues:
1. Check logs: `docker-compose logs`
2. Check GitHub Actions: Repository → Actions
3. Verify Docker Hub: Images pushed correctly
4. Test connectivity: `curl http://your-vm-ip`

## ✅ Checklist

- [ ] GitHub repository created and code pushed
- [ ] Docker Hub account created
- [ ] Docker images built and pushed
- [ ] Ubuntu VM launched
- [ ] Docker installed on VM
- [ ] Docker Compose installed on VM
- [ ] SSH key configured for GitHub Actions
- [ ] GitHub secrets configured
- [ ] Repository cloned on VM
- [ ] Application deployed
- [ ] Application accessible on port 80
- [ ] CI/CD pipeline tested
- [ ] All CRUD operations working

---
<img width="1917" height="822" alt="image" src="https://github.com/user-attachments/assets/8812fa5d-ea1d-4506-b2b0-f771bc0375ad" />

<img width="1916" height="831" alt="image" src="https://github.com/user-attachments/assets/bba23efb-c51c-446c-9605-0daa0ab4bdef" />

<img width="1914" height="875" alt="image" src="https://github.com/user-attachments/assets/5f4c8788-fd7b-43ac-830f-29cc92712687" />

<img width="1919" height="838" alt="image" src="https://github.com/user-attachments/assets/705844f7-851b-4c65-ac58-b5b14082fe44" />

<img width="1919" height="801" alt="image" src="https://github.com/user-attachments/assets/bb96f378-daf5-4002-a069-fd9f4d2757ba" />

## Browser → Frontend → API call → Backend → MongoDB → Save data → Return response → UI updates
