# Day-015: Multi-stage Builds - Tối ưu Image Size

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được multi-stage builds là gì và tại sao cần
- Biết cách viết multi-stage Dockerfile
- Hiểu được cách tối ưu image size với multi-stage
- Biết được các patterns phổ biến cho multi-stage builds
- Viết được production-ready multi-stage Dockerfile

---

## 🏗️ PHẦN 1: MULTI-STAGE BUILDS LÀ GÌ?

### 1.1. Vấn đề với Single-stage Build

**Single-stage Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build
CMD ["node", "dist/index.js"]
```

**Vấn đề:**
- **Large image**: Image chứa build tools, source code, dependencies
- **Security**: Build tools không cần trong production
- **Size**: Image size lớn không cần thiết

**Image contents:**
- Node.js runtime ✅ (cần)
- npm, build tools ❌ (không cần)
- Source code ❌ (không cần)
- node_modules (dev) ❌ (không cần)
- Build artifacts ✅ (cần)

### 1.2. Multi-stage Builds là gì?

**Multi-stage builds** cho phép dùng **multiple FROM statements** trong một Dockerfile, mỗi stage có thể dùng base image khác nhau và copy artifacts từ stage trước.

**Syntax:**
```dockerfile
FROM image AS stage-name
# Build stage

FROM another-image
# Final stage
COPY --from=stage-name /path /path
```

**Ví dụ:**
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
COPY --from=builder /app/node_modules ./node_modules
COPY package.json .
CMD ["node", "dist/index.js"]
```

**Kết quả:**
- **Smaller image**: Chỉ có runtime và artifacts
- **No build tools**: Không có build tools
- **Security**: Ít attack surface hơn

### 1.3. Tại sao Multi-stage Builds tồn tại?

**Vấn đề:**
- Single-stage builds tạo image lớn
- Build tools không cần trong production
- Security concerns với build tools

**Multi-stage builds giải quyết:**
- **Smaller images**: Chỉ copy artifacts cần thiết
- **Security**: Loại bỏ build tools
- **Separation**: Tách biệt build và runtime

---

## 🔨 PHẦN 2: MULTI-STAGE BUILD SYNTAX

### 2.1. Basic Syntax

**Multiple stages:**
```dockerfile
FROM image1 AS stage1
# Stage 1 instructions

FROM image2 AS stage2
# Stage 2 instructions

FROM image3
# Final stage (no name)
```

**Naming stages:**
```dockerfile
FROM node:18-alpine AS builder
FROM python:3.9-slim AS tester
FROM nginx:alpine
```

**Copy from stage:**
```dockerfile
COPY --from=stage-name /source /dest
COPY --from=0 /source /dest  # Use index
```

### 2.2. Stage Naming

**Named stages:**
```dockerfile
FROM node:18-alpine AS builder
FROM node:18-alpine AS tester
FROM node:18-alpine
```

**Benefits:**
- **Clear intent**: Rõ ràng mục đích của stage
- **Easy reference**: Dễ reference khi copy
- **Documentation**: Self-documenting

**Best practice:**
- **Name all stages**: Đặt tên cho tất cả stages
- **Descriptive names**: Tên mô tả rõ mục đích

### 2.3. Copy from Stages

**Copy from named stage:**
```dockerfile
COPY --from=builder /app/dist ./dist
```

**Copy from index:**
```dockerfile
COPY --from=0 /app/dist ./dist
```

**Copy from external image:**
```dockerfile
COPY --from=nginx:alpine /etc/nginx/nginx.conf /etc/nginx/
```

**Best practice:**
- **Use named stages**: Dùng tên thay vì index
- **Clear source**: Rõ ràng source path

---

## 📦 PHẦN 3: COMMON PATTERNS

### 3.1. Build + Runtime Pattern

**Pattern:** Build artifacts trong stage 1, copy vào runtime stage 2

**Example: Node.js**
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
COPY --from=builder /app/package.json .
RUN npm install --production
CMD ["node", "dist/index.js"]
```

**Benefits:**
- **Smaller image**: Chỉ có runtime và artifacts
- **No build tools**: Không có npm, build tools
- **Production-ready**: Optimized cho production

### 3.2. Compiler Pattern

**Pattern:** Compile code trong stage 1, copy binary vào minimal stage 2

**Example: Go**
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

**Benefits:**
- **Tiny image**: Alpine base (~5MB)
- **No Go compiler**: Không có Go compiler
- **Static binary**: Static binary, không cần dependencies

### 3.3. Multi-service Pattern

**Pattern:** Build multiple services, combine vào final stage

**Example:**
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

### 3.4. Test + Build Pattern

**Pattern:** Test trong stage 1, build trong stage 2, runtime trong stage 3

**Example:**
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

## 🎯 PHẦN 4: OPTIMIZATION TECHNIQUES

### 4.1. Minimize Final Stage

**Use minimal base image:**
```dockerfile
# ❌ Bad: Large base
FROM ubuntu:20.04

# ✅ Good: Minimal base
FROM alpine:latest
```

**Copy only needed files:**
```dockerfile
# ❌ Bad: Copy everything
COPY --from=builder /app .

# ✅ Good: Copy only artifacts
COPY --from=builder /app/dist ./dist
```

### 4.2. Layer Optimization

**Combine COPY commands:**
```dockerfile
# ❌ Bad: Multiple layers
COPY --from=builder /app/file1 .
COPY --from=builder /app/file2 .

# ✅ Good: Single layer
COPY --from=builder /app/file1 /app/file2 .
```

**Use .dockerignore:**
```dockerfile
# .dockerignore
node_modules/
.git/
*.log
```

### 4.3. Dependency Optimization

**Production dependencies only:**
```dockerfile
# Stage 1: Install all
RUN npm install

# Stage 2: Install production only
RUN npm install --production
```

**Separate dependencies:**
```dockerfile
# Copy package.json first
COPY package.json .
RUN npm install --production
# Then copy code
COPY . .
```

---

## 🏭 PRODUCTION STORY #1: Image Size Reduction

### Context

**Công ty:** Fintech, 400 employees
**Hệ thống:** Node.js microservices với Docker
**Traffic:** 10M requests/day
**Team:** 25 backend engineers

### Problem

**Tháng 5/2023:**
- **Image size quá lớn**: 800MB per image
- **Slow deployments**: Pull images mất 5-10 phút
- **Storage costs**: High storage costs
- **Root cause**: Single-stage builds với build tools

**Timeline:**
- **9:00 AM**: Deploy new version
- **9:05 AM**: Pull image (800MB) → 8 phút
- **9:13 AM**: Deploy complete
- **9:15 AM**: Team investigate image size

**Impact:**
- **Deployment time**: 8 phút per deployment
- **Storage costs**: $500/month cho image storage
- **Network bandwidth**: High bandwidth usage

### Investigation

**Root cause:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install  # ← Install all dependencies (dev + prod)
COPY . .
RUN npm run build
CMD ["node", "dist/index.js"]
```

**Image contents:**
- Node.js runtime: 150MB
- npm, build tools: 50MB
- node_modules (dev): 400MB ❌
- Source code: 100MB ❌
- Build artifacts: 100MB ✅
- **Total: 800MB**

**Analysis:**
- **Build tools không cần**: npm, build tools không cần trong production
- **Dev dependencies không cần**: Dev dependencies không cần
- **Source code không cần**: Chỉ cần build artifacts

### Fix

**Solution: Multi-stage build**
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
RUN npm install --production  # ← Only production dependencies
CMD ["node", "dist/index.js"]
```

**Kết quả:**
- **Image size**: 800MB → 200MB (75% reduction)
- **Deployment time**: 8 phút → 2 phút
- **Storage costs**: $500/month → $125/month

### Result

**Trước:**
- Single-stage build
- **Image size**: 800MB
- **Deployment time**: 8 phút

**Sau:**
- Multi-stage build
- **Image size**: 200MB
- **Deployment time**: 2 phút

### Lesson Learned

1. **Always use multi-stage**: Cho applications cần build
2. **Minimize final stage**: Chỉ copy artifacts cần thiết
3. **Production dependencies only**: Chỉ install production dependencies
4. **Monitor image size**: Track image size để optimize

---

## 🏭 PRODUCTION STORY #2: Security Improvement

### Context

**Công ty:** Healthcare, 600 employees
**Hệ thống:** Python applications với Docker
**Traffic:** 15M requests/day
**Team:** 30 backend engineers

### Problem

**Tháng 8/2023:**
- **Security scan**: Phát hiện vulnerabilities trong build tools
- **Root cause**: Build tools (compilers, dev dependencies) trong production image
- **Impact**: High risk vulnerabilities

**Timeline:**
- **10:00 AM**: Security scan
- **10:15 AM**: 50+ high severity vulnerabilities found
- **10:30 AM**: Team investigate
- **10:45 AM**: Root cause: Build tools trong image

**Impact:**
- **Security risk**: High risk vulnerabilities
- **Compliance**: Không đạt compliance requirements
- **Remediation**: Cần fix ngay

### Investigation

**Root cause:**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
RUN apt-get update && apt-get install -y \
    gcc g++ make  # ← Build tools
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
```

**Vulnerabilities:**
- **gcc, g++**: Compilers với known vulnerabilities
- **make**: Build tool với vulnerabilities
- **Dev dependencies**: Python dev packages với vulnerabilities

**Analysis:**
- **Build tools không cần**: Không cần trong production
- **Attack surface**: Build tools tăng attack surface
- **Compliance**: Không đạt security compliance

### Fix

**Solution: Multi-stage build**
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

**Kết quả:**
- **Vulnerabilities**: 50+ → 5 (90% reduction)
- **Attack surface**: Giảm đáng kể
- **Compliance**: Đạt compliance requirements

### Result

**Trước:**
- Single-stage với build tools
- **Vulnerabilities**: 50+ high severity
- **Compliance**: ❌ Không đạt

**Sau:**
- Multi-stage, no build tools
- **Vulnerabilities**: 5 low severity
- **Compliance**: ✅ Đạt

### Lesson Learned

1. **Remove build tools**: Loại bỏ build tools khỏi production
2. **Multi-stage for security**: Multi-stage giúp improve security
3. **Regular scans**: Scan images regularly
4. **Minimize attack surface**: Giảm attack surface

---

## 🎓 TÓM TẮT

### Multi-stage Builds

**Chức năng:**
- Multiple stages trong một Dockerfile
- Copy artifacts giữa stages
- Tối ưu image size và security

**Benefits:**
- **Smaller images**: Chỉ có runtime và artifacts
- **Security**: Loại bỏ build tools
- **Separation**: Tách biệt build và runtime

### Common Patterns

**Build + Runtime:**
- Build artifacts trong stage 1
- Copy vào minimal runtime stage 2

**Compiler Pattern:**
- Compile trong stage 1
- Copy binary vào minimal stage 2

**Test + Build:**
- Test trong stage 1
- Build trong stage 2
- Runtime trong stage 3

### Best Practices

**1. Minimize final stage:**
- Use minimal base image
- Copy only needed files

**2. Name stages:**
- Descriptive names
- Clear intent

**3. Optimize layers:**
- Combine COPY commands
- Use .dockerignore

**4. Production dependencies:**
- Only install production dependencies
- Separate dev and prod

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ bạn đã:
- ✅ Hiểu multi-stage builds
- ✅ Biết các patterns phổ biến
- ✅ Tối ưu image size

**Phase 3 hoàn thành!** Bạn đã nắm vững Dockerfile fundamentals.

**Phase tiếp theo (Phase 4)** sẽ đi sâu vào:
- Image Optimization
- Advanced Dockerfile techniques
- Production patterns

---

## 📚 TÀI LIỆU THAM KHẢO

- Multi-stage builds: https://docs.docker.com/build/building/multi-stage/

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-016: Image-Layers-Caching-Deep-Dive](../Day-016-Image-Layers-Caching-Deep-Dive/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
