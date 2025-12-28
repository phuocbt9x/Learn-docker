# Day-038: Compose Environment Variables

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được environment variables trong Compose
- Biết cách define environment variables
- Hiểu được .env file
- Biết cách override environment variables
- Áp dụng trong production

---

## 🔧 PHẦN 1: ENVIRONMENT VARIABLES

### 1.1. Define trong Compose File

**Inline:**
```yaml
services:
  api:
    image: node:18-alpine
    environment:
      - NODE_ENV=production
      - DB_HOST=db
```

**Environment file:**
```yaml
services:
  api:
    image: node:18-alpine
    env_file:
      - .env
      - .env.production
```

### 1.2. .env File

**.env file:**
```
NODE_ENV=production
DB_HOST=db
DB_PORT=5432
```

**Use trong Compose:**
```yaml
services:
  api:
    image: node:18-alpine
    environment:
      - NODE_ENV=${NODE_ENV}
      - DB_HOST=${DB_HOST}
```

### 1.3. Override

**Override với command:**
```bash
$ NODE_ENV=development docker compose up
```

**Override với .env file:**
```bash
$ docker compose --env-file .env.dev up
```

---

## 🏭 PRODUCTION STORY: Environment Management

### Context

**Công ty:** SaaS, 600 employees
**Issue:** Environment variables hardcoded
**Solution:** .env files và environment management

### Fix

**Solution: .env files**
```yaml
# docker-compose.yml
services:
  api:
    environment:
      - NODE_ENV=${NODE_ENV}
      - DB_HOST=${DB_HOST}
```

**.env:**
```
NODE_ENV=production
DB_HOST=db
```

**Results:**
- No hardcoded values
- Easy configuration
- Environment-specific configs

---

## 🎓 TÓM TẮT

**Environment variables:**
- Define trong Compose file
- Use .env file
- Override khi cần

**Best practices:**
- Use .env files
- No hardcoded values
- Environment-specific configs

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-039)** sẽ đi sâu vào:
- Compose Scaling & Dependencies
- Production patterns

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-039: Compose-Scaling-Dependencies](../Day-039-Compose-Scaling-Dependencies/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
