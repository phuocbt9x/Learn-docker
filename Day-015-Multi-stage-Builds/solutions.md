# Day-015: Multi-stage Builds - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

Tài liệu này cung cấp giải pháp chi tiết cho tất cả các bài tập.

---

## ✅ BÀI TẬP 1: Basic Multi-stage

**Multi-stage Dockerfile:**
```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package.json .
RUN npm install --production
CMD ["node", "dist/index.js"]
```

**Image size comparison:**
- Single-stage: ~800MB
- Multi-stage: ~200MB
- **Reduction: 75%**

**Giải thích:**
- Multi-stage loại bỏ build tools, dev dependencies, source code
- Chỉ giữ runtime và build artifacts

---

## ✅ BÀI TẬP 2: Go Application

**Multi-stage Dockerfile:**
```dockerfile
# Stage 1: Build
FROM golang:1.20-alpine AS builder
WORKDIR /app
COPY go.mod go.sum .
RUN go mod download
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o app .

# Stage 2: Production
FROM alpine:latest
RUN apk add --no-cache ca-certificates
WORKDIR /app
COPY --from=builder /app/app .
CMD ["./app"]
```

**Image size:**
- Builder stage: ~300MB
- Final image: ~15MB
- **Reduction: 95%**

---

## ✅ BÀI TẬP 3: Python Application

**Multi-stage Dockerfile:**
```dockerfile
# Stage 1: Build
FROM python:3.9-slim AS builder
WORKDIR /app
RUN apt-get update && apt-get install -y gcc g++ make
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Stage 2: Production
FROM python:3.9-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
```

**Security improvements:**
- No build tools in final image
- Reduced attack surface
- Fewer vulnerabilities

---

## ✅ BÀI TẬP 4: Test + Build Pattern

**Multi-stage Dockerfile:**
```dockerfile
# Stage 1: Test
FROM node:18-alpine AS tester
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm test

# Stage 2: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

# Stage 3: Production
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package.json .
RUN npm install --production
CMD ["node", "dist/index.js"]
```

---

## ✅ BÀI TẬP 5: Multi-service Pattern

**Multi-stage Dockerfile:**
```dockerfile
# Stage 1: Build API
FROM node:18-alpine AS api-builder
WORKDIR /app
COPY api/package.json .
RUN npm install
COPY api/ .
RUN npm run build

# Stage 2: Build Frontend
FROM node:18-alpine AS frontend-builder
WORKDIR /app
COPY frontend/package.json .
RUN npm install
COPY frontend/ .
RUN npm run build

# Stage 3: Production
FROM nginx:alpine
COPY --from=api-builder /app/dist /usr/share/nginx/html/api
COPY --from=frontend-builder /app/dist /usr/share/nginx/html
```

---

## ✅ BÀI TẬP 6: Image Size Optimization

**Optimized Dockerfile:**
```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine
WORKDIR /app
# Copy only artifacts
COPY --from=builder /app/dist ./dist
# Copy package.json for production install
COPY package.json .
# Install production dependencies only
RUN npm install --production --ignore-scripts
CMD ["node", "dist/index.js"]
```

**Optimizations:**
- Copy only dist folder
- Production dependencies only
- Ignore scripts for faster install

---

## ✅ BÀI TẬP 7: Troubleshooting

**Problem:** Missing node_modules in final image

**Fixed Dockerfile:**
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package.json .
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

**Fix:** Install production dependencies in final stage

---

## ✅ BÀI TẬP 8: Production Analysis

**Refactored Dockerfile:**
```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package.json .
RUN npm install --production
CMD ["node", "dist/index.js"]
```

**Results:**
- Image size: ~200MB (< 200MB requirement)
- No build tools: ✅
- Production dependencies only: ✅

---

## ✅ BÀI TẬP 9: Conditional Stages

**Multi-stage với ARG:**
```dockerfile
ARG TARGET=production

# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY package.json .
RUN if [ "$TARGET" = "production" ]; then \
      npm install --production; \
    else \
      npm install; \
    fi
CMD ["node", "dist/index.js"]
```

---

## ✅ BÀI TẬP 10: Build Cache Optimization

**Optimized Dockerfile:**
```dockerfile
# Stage 1: Build
FROM node:18-alpine AS builder
WORKDIR /app
# Copy package.json first (changes less frequently)
COPY package.json .
RUN npm install
# Copy source code last (changes frequently)
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine
WORKDIR /app
# Copy artifacts
COPY --from=builder /app/dist ./dist
# Copy package.json for production install
COPY package.json .
RUN npm install --production
CMD ["node", "dist/index.js"]
```

**Cache optimization:**
- package.json copied first → cache npm install layer
- Source code copied last → only rebuild when code changes

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

