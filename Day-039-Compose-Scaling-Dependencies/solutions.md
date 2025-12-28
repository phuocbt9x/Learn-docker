# Day-039: Compose Scaling & Dependencies - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Scale Services

**Commands:**
```bash
$ docker compose up --scale web=3
```

**Limitations:**
- Cannot scale services với published ports
- Single host only
- Use orchestrator cho production

---

## ✅ BÀI TẬP 2: Advanced Dependencies

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  api:
    image: node:18-alpine
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
      interval: 10s
      timeout: 5s
      retries: 5
```

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

