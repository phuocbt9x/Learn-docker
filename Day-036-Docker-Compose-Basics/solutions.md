# Day-036: Docker Compose Basics - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Basic Compose File

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
```

**Commands:**
```bash
$ docker compose up -d
$ docker compose ps
```

---

## ✅ BÀI TẬP 2: Multi-service Setup

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"

  api:
    image: node:18-alpine
    command: node app.js

  db:
    image: postgres:14-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
```

**Commands:**
```bash
$ docker compose up -d
$ docker compose ps
```

---

## ✅ BÀI TẬP 3: Compose Commands

**Commands:**
```bash
# Start
$ docker compose up -d

# View logs
$ docker compose logs
$ docker compose logs web

# Stop
$ docker compose stop

# Remove
$ docker compose down
```

---

## ✅ BÀI TẬP 4: Build và Run

**Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
CMD ["node", "app.js"]
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
```

**Commands:**
```bash
$ docker compose up --build
```

---

## ✅ BÀI TẬP 5: Production Scenario

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    image: nginx:alpine
    ports:
      - "80:80"
    environment:
      - NODE_ENV=production

  api:
    build: ./api
    environment:
      - DB_HOST=db
      - NODE_ENV=production

  db:
    image: postgres:14-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
```

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

