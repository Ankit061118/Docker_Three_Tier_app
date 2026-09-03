# Docker Setup Guide

This document provides comprehensive instructions for building and running the Student-Teacher Portal application using Docker.

## Overview

The application consists of three main Docker services:
- **MySQL Database** - Data persistence
- **Backend** - Node.js API server
- **Frontend** - React web application with Nginx

## Prerequisites

- Docker installed and running
- Docker Compose (optional, for easier orchestration)
- Basic understanding of Docker commands

## Architecture Setup

### 1. Create Docker Network

Create a network to enable communication between containers:

```bash
docker network create three-tier-network
```

### 2. Create Docker Volume

Create a volume for persistent database storage:

```bash
docker volume create mysql-volume
```

## Building and Running Services

### Database (MySQL)

#### Dockerfile Location
`database/Dockerfile`

#### Build MySQL Image

```bash
docker build -t mysql-image database/
```

#### Run MySQL Container

```bash
docker run --name mysql-container -d \
  -p 3306:3306 \
  --network three-tier-network \
  -v mysql-data:/var/lib/mysql \
  mysql-image
```

**Container Details:**
- Container Name: `mysql-container`
- Port: `3306`
- Network: `three-tier-network`
- Volume: `mysql-data:/var/lib/mysql`
- Credentials: 
  - Root Password: `mysql123`
  - Database: `school`

---

### Backend (Node.js API)

#### Dockerfile Location
`backend/Dockerfile`

#### Build Backend Image

```bash
docker build -t backend backend/
```

#### Run Backend Container

```bash
docker run --name backend-container -d \
  -p 3500:3500 \
  --network three-tier-network \
  backend
```

**Container Details:**
- Container Name: `backend-container`
- Port: `3500`
- Network: `three-tier-network`
- Runtime: Node.js v20-alpine

---

### Frontend (React Application)

#### Dockerfile Location
`frontend/Dockerfile`

#### Build Frontend Image

```bash
docker build -t frontend \
  --build-arg REACT_APP_API_BASE_URL=http://localhost:3500 \
  frontend/
```

**Build Arguments:**
- `REACT_APP_API_BASE_URL`: Backend API endpoint (default: `http://localhost:3500`)

#### Run Frontend Container

```bash
docker run -d \
  --name frontend-container \
  --network three-tier-network \
  -p 80:3000 \
  frontend
```

**Container Details:**
- Container Name: `frontend-container`
- Port: `80` (external) → `3000` (internal)
- Network: `three-tier-network`
- Server: Nginx serving React build

---

## Complete Startup Sequence

Follow these steps to bring up the entire application:

```bash
# Step 1: Create network and volume
docker network create three-tier-network
docker volume create mysql-volume

# Step 2: Build and run MySQL
docker build -t mysql-image database/
docker run --name mysql-container -d -p 3306:3306 --network three-tier-network -v mysql-data:/var/lib/mysql mysql-image

# Step 3: Build and run Backend
docker build -t backend backend/
docker run --name backend-container -d -p 3500:3500 --network three-tier-network backend

# Step 4: Build and run Frontend
docker build -t frontend --build-arg REACT_APP_API_BASE_URL=http://localhost:3500 frontend/
docker run -d --name frontend-container --network three-tier-network -p 80:3000 frontend
```

## Accessing the Application

Once all containers are running:

- **Frontend**: http://localhost
- **Backend API**: http://localhost:3500
- **Database**: localhost:3306

## Common Docker Commands

### View Running Containers
```bash
docker ps
```

### View Container Logs
```bash
docker logs <container-name>
```

Example:
```bash
docker logs mysql-container
docker logs backend-container
docker logs frontend-container
```

### Inspect Network
```bash
docker network inspect three-tier-network
```

### Stop Containers
```bash
docker stop mysql-container backend-container frontend-container
```

### Remove Containers
```bash
docker rm mysql-container backend-container frontend-container
```

### Remove Images
```bash
docker rmi mysql-image backend frontend
```

### Clean Up Everything
```bash
docker network rm three-tier-network
docker volume rm mysql-volume
```

## Environment Variables

### MySQL
- `MYSQL_ROOT_PASSWORD`: `mysql123`
- `MYSQL_DATABASE`: `school`

### Frontend
- `REACT_APP_API_BASE_URL`: Backend API endpoint (set during build)

## Port Mappings

| Service | Internal Port | External Port | Purpose |
|---------|---|---|---|
| MySQL | 3306 | 3306 | Database |
| Backend | 3500 | 3500 | API Server |
| Frontend | 3000 | 80 | Web Application |

## Troubleshooting

### Port Already in Use
If a port is already in use, you can map to a different external port:
```bash
docker run -p <new-port>:<container-port> ...
```

### Container Won't Start
Check logs:
```bash
docker logs <container-name>
```

### Network Issues
Verify network connectivity:
```bash
docker network inspect three-tier-network
```

### Volume Issues
List all volumes:
```bash
docker volume ls
```

## Notes

- The application follows a three-tier architecture pattern
- All services communicate through the `three-tier-network` Docker network
- Database data persists using the `mysql-volume` Docker volume
- Each service can be independently built and deployed
