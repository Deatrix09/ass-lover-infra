# A.S.S. Lover — Infrastructure

> Docker Compose, nginx, and deployment configuration for the A.S.S. Lover system.

## Overview

This repository contains all infrastructure configuration required to deploy the A.S.S. Lover system to a production environment.

## Structure

```
rag-infra/
├── docker-compose.prod.yaml   # Production Docker Compose
├── nginx.conf                 # Nginx reverse proxy configuration
├── keycloak/
│   └── realm-export.json      # Keycloak realm configuration (import)
└── ssl/                       # SSL certificates (not in repo)
    ├── server.crt
    └── server.key
```

## Deployment Architecture

```
Internet
    │
    ▼
nginx (80/443)
    ├── /          → frontend:80
    ├── /api/      → backend:8000
    └── /auth/     → keycloak:8080
```

## Requirements

- Docker 24+
- Docker Compose v2
- At least 8 GB RAM recommended

## Deployment Steps

### 1. Clone Repositories

```bash
git clone https://github.com/Deatrix09/ass-lover-backend
git clone https://github.com/Deatrix09/ass-lover-frontend
git clone https://github.com/Deatrix09/ass-lover-infra
```

### 2. Configuration

Copy `.env.prod.example` to `.env` and fill in your actual API keys and server IP.
```bash
cd ass-lover-infra
cp .env.prod.example .env
```

### 3. SSL (Optional)

If you need HTTPS, place your certificates in the `ssl/` directory as `server.crt` and `server.key`.

### 4. Start the System

```bash
docker compose -f docker-compose.prod.yaml up -d --build
```

## License

MIT
