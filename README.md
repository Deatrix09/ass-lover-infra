# A.S.S. Lover — Infrastruktura

> Docker Compose, nginx a deployment konfigurace pro systém A.S.S. Lover

## Přehled

Tento repozitář obsahuje veškerou infrastrukturní konfiguraci pro nasazení systému A.S.S. Lover na produkční server.

## Struktura

```
rag-infra/
├── docker-compose.prod.yaml   # Produkční Docker Compose
├── nginx.conf                 # Nginx reverse proxy konfigurace
├── keycloak/
│   └── realm-export.json      # Keycloak realm konfigurace (import)
└── ssl/                       # SSL certifikáty (není v repozitáři)
    ├── server.crt
    └── server.key
```

## Architektura nasazení

```
Internet
    │
    ▼
nginx (80/443)
    ├── /          → frontend:80
    ├── /api/      → backend:8000
    └── /auth/     → keycloak:8080
         │
         ├── backend (FastAPI)
         │     ├── postgres:5432
         │     ├── qdrant:6333
         │     └── redis:6379
         ├── frontend (React/nginx)
         ├── keycloak:8080
         │     └── postgres:5432
         └── playwright-service:3000
```

## Služby

| Služba | Image | Popis |
|---|---|---|
| `nginx` | nginx:alpine | Reverse proxy, SSL termination |
| `backend` | vlastní build | FastAPI backend |
| `frontend` | vlastní build | React frontend |
| `keycloak` | keycloak:24.0.0 | Autentizace a autorizace |
| `postgres` | postgres:15-alpine | Relační databáze |
| `qdrant` | qdrant/qdrant:latest | Vektorová databáze |
| `redis` | redis:7-alpine | Cache a fronty |
| `playwright-service` | browserless/chrome | Headless prohlížeč pro JS rendering |

## Požadavky

- Docker 24+
- Docker Compose v2
- Server s minimálně 4 GB RAM
- SSL certifikát (self-signed nebo Let's Encrypt)

## Nasazení

### 1. Klonování repozitářů

```bash
mkdir project && cd project
git clone https://github.com/abakan21/rag-backend
git clone https://github.com/abakan21/rag-frontend
git clone https://github.com/abakan21/rag-infra
```

### 2. SSL certifikát

```bash
cd rag-infra
mkdir ssl

# Self-signed (pro vývoj/testování)
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout ssl/server.key \
  -out ssl/server.crt \
  -subj "/CN=váš-server-ip"
```

### 3. Konfigurace

Upravte proměnné prostředí v `docker-compose.prod.yaml`:

```yaml
backend:
  environment:
    OLLAMA_URL: "https://llm.ai.e-infra.cz/v1"
    OLLAMA_API_KEY: "váš-api-klíč"
    LLM_MODEL_NAME: "llama-4-scout-17b-16e-instruct"
    CORS_ORIGINS: "https://váš-server-ip"
    KEYCLOAK_PUBLIC_URL: "https://váš-server-ip/auth"
```

### 4. Spuštění

```bash
cd rag-infra
docker compose -f docker-compose.prod.yaml up -d --build
```

### 5. Ověření

```bash
# Status všech kontejnerů
docker compose -f docker-compose.prod.yaml ps

# Logy backendu
docker logs rag-infra-backend-1 --tail=20

# Test API
curl -k https://váš-server-ip/api/docs
```

## Keycloak konfigurace

Po prvním spuštění se automaticky importuje realm ze souboru `keycloak/realm-export.json`.

### Výchozí nastavení
- Realm: `rag`
- Client ID: `rag-app`
- Admin konzole: `https://váš-server-ip/auth`

### Vytvoření admin uživatele

```bash
docker exec rag-infra-keycloak-1 \
  /opt/keycloak/bin/kcadm.sh create users \
  -r rag \
  -s username=admin \
  -s enabled=true \
  --server http://localhost:8080 \
  --realm master \
  --user admin --password admin
```

## CI/CD Pipeline

Vývojáři pracují lokálně s Dockerem. Workflow:

1. Kód se nahraje do větve v Gitu
2. Pipeline zkontroluje build
3. Po úspěšné kontrole pipeline nasadí na produkční server
4. Runner běží jako Docker instance v produkčním prostředí

## Monitoring

Pro monitoring je připravena integrace s Prometheus + Grafana (viz etapa 10 projektu).

## Volumes

| Volume | Popis |
|---|---|
| `postgres_data` | PostgreSQL data |
| `qdrant_data` | Qdrant vektorová databáze |
| `backend_data` | Extrahované markdown soubory a evidence |

## Hardwarové požadavky

| Komponenta | Minimum | Doporučeno |
|---|---|---|
| CPU | 4 jádra | 8+ jader |
| RAM | 8 GB | 16+ GB |
| Disk | 50 GB SSD | 100+ GB NVMe SSD |
| GPU | není potřeba (LLM v cloudu) | — |

> **Poznámka:** LLM je provozován přes e-infra API (`chat.ai.e-infra.cz`). Embeddingy běží na CPU pomocí modelu `paraphrase-multilingual-MiniLM-L12-v2`.
