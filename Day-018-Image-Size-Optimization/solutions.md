# Day-018: Image Size Optimization - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Base Image Comparison

**Results:**
- ubuntu:20.04: 200MB
- debian:bullseye-slim: 80MB
- alpine:latest: 5MB

**Recommendation:** Alpine cho minimal size

---

## ✅ BÀI TẬP 2: Cleanup Techniques

**Optimized:**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update && \
    apt-get install -y curl wget && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
COPY . .
```

**Results:**
- Before: 250MB
- After: 220MB (12% reduction)

---

## ✅ BÀI TẬP 3: Multi-stage Optimization

**Optimized:**
```dockerfile
FROM node:18-alpine AS builder
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

FROM alpine:latest
RUN apk add --no-cache nodejs
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/index.js"]
```

**Results:**
- Before: 400MB
- After: 50MB (87% reduction)

---

## ✅ BÀI TẬP 4: Production Optimization

**Techniques applied:**
1. Multi-stage builds
2. Alpine base
3. Remove build tools
4. Cleanup caches
5. Production dependencies only

**Results:**
- Before: 1GB
- After: 250MB (75% reduction)

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

