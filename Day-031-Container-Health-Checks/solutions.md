# Day-031: Container Health Checks - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Basic Health Check

**Dockerfile:**
```dockerfile
FROM node:18-alpine
COPY . .
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
CMD ["node", "app.js"]
```

**Test:**
```bash
$ docker build -t my-app .
$ docker run -d --name app my-app
$ docker ps
# STATUS: (healthy)
```

---

## ✅ BÀI TẬP 2: Health Check Options

**Fast detection:**
```dockerfile
HEALTHCHECK --interval=10s --timeout=2s --retries=1 \
  CMD curl -f http://localhost:3000/health || exit 1
```

**Stable detection:**
```dockerfile
HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
```

**Startup period:**
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1
```

**Recommendations:**
- Fast: Development, testing
- Stable: Production
- Startup period: Applications cần startup time

---

## ✅ BÀI TẬP 3: Health Check Types

**HTTP:**
```dockerfile
HEALTHCHECK CMD curl -f http://localhost:3000/health || exit 1
```

**TCP:**
```dockerfile
HEALTHCHECK CMD nc -z localhost 3306 || exit 1
```

**Custom script:**
```dockerfile
HEALTHCHECK CMD /app/healthcheck.sh || exit 1
```

**Use cases:**
- HTTP: Web applications
- TCP: Databases, services
- Custom: Complex checks

---

## ✅ BÀI TẬP 4: Debug Health Check

**Problem:** Health endpoint không có

**Fix:**
```dockerfile
# Add health endpoint trong application
# hoặc
HEALTHCHECK CMD curl -f http://localhost:3000/ || exit 1
```

**Results:**
- Health check works
- Container healthy

---

## ✅ BÀI TẬP 5: Production Health Check

**Dockerfile:**
```dockerfile
FROM node:18-alpine
COPY . .
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"
CMD ["node", "app.js"]
```

**Justification:**
- Interval 30s: Balance detection và load
- Timeout 5s: Tolerate network hiccups
- Start period 10s: Cho application startup
- Retries 3: Avoid false positives

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

