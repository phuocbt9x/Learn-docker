# Day-037: Compose Networks & Volumes

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được networks trong Compose
- Hiểu được volumes trong Compose
- Biết cách define custom networks
- Biết cách define volumes
- Hiểu được service dependencies
- Áp dụng trong production

---

## 🌐 PHẦN 1: COMPOSE NETWORKS

### 1.1. Default Network

**Compose tự động tạo network:**
```yaml
services:
  web:
    image: nginx
  api:
    image: node:18-alpine
# Cả 2 services trong cùng network, có thể communicate
```

**Network name:**
- Format: `<project>_default`
- Project name: Directory name hoặc --project-name

### 1.2. Custom Networks

**Define custom network:**
```yaml
services:
  web:
    image: nginx
    networks:
      - frontend

  api:
    image: node:18-alpine
    networks:
      - frontend
      - backend

  db:
    image: postgres
    networks:
      - backend

networks:
  frontend:
  backend:
```

**Benefits:**
- **Isolation**: Isolate services
- **Security**: Better security
- **Organization**: Better organization

---

## 💾 PHẦN 2: COMPOSE VOLUMES

### 2.1. Named Volumes

**Define named volume:**
```yaml
services:
  db:
    image: postgres
    volumes:
      - db-data:/var/lib/postgresql/data

volumes:
  db-data:
```

**Benefits:**
- **Persistent**: Data persist
- **Named**: Easy to manage
- **Reusable**: Có thể reuse

### 2.2. Bind Mounts

**Bind mount:**
```yaml
services:
  web:
    image: nginx
    volumes:
      - ./config:/etc/nginx/conf.d
```

**Use cases:**
- **Development**: Development với live reload
- **Configuration**: Mount config files

---

## 🔗 PHẦN 3: SERVICE DEPENDENCIES

### 3.1. depends_on

**Define dependencies:**
```yaml
services:
  api:
    image: node:18-alpine
    depends_on:
      - db
      - redis

  db:
    image: postgres

  redis:
    image: redis
```

**Behavior:**
- **Start order**: Start db và redis trước api
- **Wait**: Wait cho dependencies start
- **No health check**: Không wait cho healthy (chỉ start)

### 3.2. depends_on với condition

**Health check condition:**
```yaml
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

**Conditions:**
- **service_started**: Wait cho service start
- **service_healthy**: Wait cho service healthy
- **service_completed_successfully**: Wait cho service complete

---

## 🏭 PRODUCTION STORY: Network Isolation

### Context

**Công ty:** Fintech, 500 employees
**Issue:** Services communicate không đúng
**Solution:** Custom networks với isolation

### Fix

**Solution: Custom networks**
```yaml
services:
  web:
    networks:
      - frontend

  api:
    networks:
      - frontend
      - backend

  db:
    networks:
      - backend

networks:
  frontend:
  backend:
```

**Results:**
- Network isolation
- Better security
- Clear architecture

---

## 🎓 TÓM TẮT

**Networks:**
- Default network: Auto-created
- Custom networks: Better isolation
- Service communication: Via network

**Volumes:**
- Named volumes: Persistent data
- Bind mounts: Development, config

**Dependencies:**
- depends_on: Start order
- Health checks: Wait for healthy

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-038)** sẽ đi sâu vào:
- Compose Environment Variables
- Configuration management

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.

