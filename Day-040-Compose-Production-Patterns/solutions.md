# Day-040: Compose Production Patterns - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Multi-file Compose

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    image: nginx
  api:
    image: node:18-alpine
```

**docker-compose.prod.yml:**
```yaml
services:
  web:
    ports:
      - "80:80"
    restart: always
  api:
    environment:
      - NODE_ENV=production
    restart: always
```

**Usage:**
```bash
$ docker compose -f docker-compose.yml -f docker-compose.prod.yml up
```

---

## ✅ BÀI TẬP 2: Production Setup

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    restart: always
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 3s
      retries: 3

  api:
    build: ./api
    environment:
      - NODE_ENV=${NODE_ENV}
      - DB_HOST=${DB_HOST}
    restart: always
    depends_on:
      - db
    healthcheck:
      test: ["CMD", "node", "healthcheck.js"]
      interval: 30s
      timeout: 3s
      retries: 3

  db:
    image: postgres:14-alpine
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - db-data:/var/lib/postgresql/data
    restart: always
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  db-data:
```

**Justification:**
- Health checks: Monitor service health
- Restart policies: Automatic recovery
- Environment variables: No hardcoded values
- Volumes: Data persistence

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

