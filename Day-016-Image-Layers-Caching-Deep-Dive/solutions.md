# Day-016: Layer Caching - Build Performance - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Basic Layer Ordering

**Optimized Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
CMD ["node", "index.js"]
```

**Changes:**
- Copy package.json trước source code
- npm install được cache khi source code đổi

**Results:**
- Cache hits: 0% → 80%
- Build time: 5 phút → 1 phút

---

## ✅ BÀI TẬP 2: Dependencies Before Source

**Optimized Dockerfile:**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

**Results:**
- Cache hits: 0% → 85%
- Build time: 8 phút → 1.5 phút

---

## ✅ BÀI TẬP 3: Combine RUN Commands

**Optimized Dockerfile:**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update && \
    apt-get install -y curl wget && \
    apt-get clean
```

**Results:**
- Layers: 4 → 1
- Build time: Similar (nhưng ít layers hơn)

---

## ✅ BÀI TẬP 4: Cache Analysis

**Test results:**
- Build 1: 5 phút (all layers)
- Build 2 (source change): 30s (cache npm install)
- Build 3 (package.json change): 5 phút (rebuild từ npm install)

**Analysis:**
- npm install được cache khi source code đổi
- npm install rebuild khi package.json đổi

---

## ✅ BÀI TẬP 5: Cache Debugging

**Problem:** collectstatic generate files → cache miss

**Fixed Dockerfile:**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
# Don't run collectstatic in build
CMD ["gunicorn", "app.wsgi:application"]
```

**Fix:** Remove collectstatic from build, run at runtime

---

## ✅ BÀI TẬP 6: Practical Optimization

**Optimized Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install
COPY . .
RUN npm run build
CMD ["node", "dist/index.js"]
```

**Results:**
- Cache hits: 20% → 85%
- Build time: 20 phút → 4 phút

---

## ✅ BÀI TẬP 7: Multi-stage Cache

**Optimized Dockerfile:**
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package.json .
RUN npm install --production
CMD ["node", "dist/index.js"]
```

**Cache optimization:**
- Builder stage: Cache npm install
- Production stage: Cache production install

---

## ✅ BÀI TẬP 8: Production Analysis

**Optimized Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install
COPY src/ src/
COPY config/ config/
RUN npm run build
CMD ["node", "dist/index.js"]
```

**Results:**
- Build time: 20 phút → 4 phút
- Cache hits: 20% → 85%

---

## ✅ BÀI TẬP 9: Conditional Caching

**Dockerfile:**
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
ARG NODE_ENV=production
RUN if [ "$NODE_ENV" = "production" ]; then \
      npm run build:prod; \
    else \
      npm run build:dev; \
    fi

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package.json .
RUN npm install --production
CMD ["node", "dist/index.js"]
```

**Cache behavior:**
- npm install cached cho cả dev và prod
- Build step conditional nhưng cache vẫn tốt

---

## ✅ BÀI TẬP 10: Cache Monitoring

**Script:**
```bash
#!/bin/bash
echo "Building image..."
START=$(date +%s)
docker build -t my-app . 2>&1 | tee build.log
END=$(date +%s)
BUILD_TIME=$((END - START))

CACHE_HITS=$(grep -c "Using cache" build.log)
TOTAL_STEPS=$(grep -c "Step" build.log)
CACHE_PERCENT=$((CACHE_HITS * 100 / TOTAL_STEPS))

echo "Build time: ${BUILD_TIME}s"
echo "Cache hits: ${CACHE_HITS}/${TOTAL_STEPS} (${CACHE_PERCENT}%)"
```

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

