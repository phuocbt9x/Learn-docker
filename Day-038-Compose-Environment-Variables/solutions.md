# Day-038: Compose Environment Variables - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Environment Variables

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  api:
    image: node:18-alpine
    environment:
      - NODE_ENV=production
      - DB_HOST=db
```

---

## ✅ BÀI TẬP 2: .env File

**.env:**
```
NODE_ENV=production
DB_HOST=db
DB_PORT=5432
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  api:
    image: node:18-alpine
    environment:
      - NODE_ENV=${NODE_ENV}
      - DB_HOST=${DB_HOST}
      - DB_PORT=${DB_PORT}
```

---

## ✅ BÀI TẬP 3: Environment Override

**Command line:**
```bash
$ NODE_ENV=development docker compose up
```

**Different .env:**
```bash
$ docker compose --env-file .env.dev up
```

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

