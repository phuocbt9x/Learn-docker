# Day-039: Compose Scaling & Dependencies

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được scaling trong Compose
- Biết cách scale services
- Hiểu được advanced dependencies
- Biết cách manage service lifecycle
- Áp dụng trong production

---

## 📈 PHẦN 1: SCALING SERVICES

### 1.1. Scale Services

**Scale khi start:**
```bash
$ docker compose up --scale web=3
# Scale web service to 3 instances
```

**Scale multiple services:**
```bash
$ docker compose up --scale web=3 --scale api=2
```

### 1.2. Scale Limitations

**Limitations:**
- **Port conflicts**: Cannot scale services với published ports
- **Single host**: Compose scale trên single host
- **Orchestration**: Dùng orchestrator (Kubernetes, Swarm) cho production

### 1.3. Load Balancing

**With load balancer:**
```yaml
services:
  nginx:
    image: nginx
    ports:
      - "80:80"
    depends_on:
      - web

  web:
    image: nginx
    # Multiple instances behind nginx
```

---

## 🔗 PHẦN 2: ADVANCED DEPENDENCIES

### 2.1. Health Check Dependencies

**Wait for healthy:**
```yaml
services:
  api:
    depends_on:
      db:
        condition: service_healthy

  db:
    healthcheck:
      test: ["CMD-SHELL", "pg_isready"]
```

### 2.2. Startup Order

**Control startup:**
```yaml
services:
  api:
    depends_on:
      - db
      - redis
    restart: on-failure
```

---

## 🏭 PRODUCTION STORY: Scaling Issues

### Context

**Công ty:** E-commerce, 800 employees
**Issue:** Cannot scale với Compose
**Solution:** Use orchestrator cho production

### Lesson

**Compose limitations:**
- Single host scaling
- Port conflicts
- Use orchestrator cho production scaling

---

## 🎓 TÓM TẮT

**Scaling:**
- Compose scale: Development, testing
- Production: Use orchestrator
- Load balancing: With load balancer

**Dependencies:**
- depends_on: Basic dependencies
- Health checks: Wait for healthy
- Startup order: Control order

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-040)** sẽ đi sâu vào:
- Compose Production Patterns
- Best practices

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-040: Compose-Production-Patterns](../Day-040-Compose-Production-Patterns/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
