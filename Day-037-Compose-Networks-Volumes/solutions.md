# Day-037: Compose Networks & Volumes - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Custom Networks

**docker-compose.yml:**
```yaml
version: '3.8'

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

---

## ✅ BÀI TẬP 2: Volumes

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  db:
    image: postgres
    volumes:
      - db-data:/var/lib/postgresql/data

  web:
    image: nginx
    volumes:
      - ./config:/etc/nginx/conf.d

volumes:
  db-data:
```

---

## ✅ BÀI TẬP 3: Service Dependencies

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  api:
    image: node:18-alpine
    depends_on:
      db:
        condition: service_healthy

  web:
    image: nginx
    depends_on:
      - api

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

