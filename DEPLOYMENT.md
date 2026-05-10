# RAG System — Detailed Deployment Guide

## Prerequisites

Ensure the following are installed on your server:
- **Docker** (≥ 24.x)
- **Docker Compose** (≥ 2.x)
- **Git**
- **16 GB RAM** recommended for optimal performance.

## Deployment Procedure

### 1. Clone Repositories

Create a project directory and clone all parts:
```bash
mkdir project && cd project

git clone https://github.com/Deatrix09/ass-lover-infra
git clone https://github.com/Deatrix09/ass-lover-frontend
git clone https://github.com/Deatrix09/ass-lover-backend
```

### 2. Configure Environment

The system relies on environment variables. We use a central `.env` file in the infra directory.

```bash
cd ass-lover-infra
cp .env.prod.example .env
# Edit .env with your actual values (IP, API Keys, Passwords)
nano .env
```

### 3. Launch the Stack

```bash
# Initial build may take 10-20 minutes
docker compose -f docker-compose.prod.yaml up --build -d

# Monitor logs
docker compose -f docker-compose.prod.yaml logs -f backend
```

## Keycloak (User Management)

Authentication is handled by **Keycloak**. On first launch, the `rag` realm is automatically imported with default users:

| User | Password | Role |
|---|---|---|
| `admin` | `admin` | `admin` |
| `user` | `user` | `user` |

> **Security Warning:** Change default passwords immediately after deployment!

### Admin Console Access
`https://<your-server-ip>/auth/admin/`
Default credentials: `kcadmin` / `kcadmin`

## Verification

After starting:
- **Frontend**: `https://<your-server-ip>`
- **Backend API**: `https://<your-server-ip>/api/docs`
- **Keycloak**: `https://<your-server-ip>/auth`

## Updating the System

When code is updated in the repositories:
```bash
cd ass-lover-infra
docker compose -f docker-compose.prod.yaml pull
docker compose -f docker-compose.prod.yaml up -d --build
```

## Stopping the Stack

```bash
docker compose -f docker-compose.prod.yaml down

# To also delete persistent volumes:
docker compose -f docker-compose.prod.yaml down -v
```
