# Day-018: Image Size Optimization Strategies

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được các techniques để giảm image size
- Biết cách chọn base image phù hợp
- Hiểu được cách cleanup trong Dockerfile
- Biết được các tools để analyze image size
- Viết được Dockerfile tối ưu cho size

---

## 📦 PHẦN 1: BASE IMAGE SELECTION

### 1.1. Base Image Size Comparison

**Common base images:**
- **ubuntu:20.04**: ~200MB
- **debian:bullseye**: ~120MB
- **alpine:latest**: ~5MB
- **distroless**: ~20MB

**Recommendation:**
- **Alpine**: Smallest, good for most cases
- **Distroless**: Minimal, security-focused
- **Slim variants**: python:3.9-slim, node:18-alpine

### 1.2. Multi-stage với Minimal Base

**Pattern:**
```dockerfile
FROM node:18-alpine AS builder
# Build stage

FROM alpine:latest
# Minimal runtime
```

**Benefits:**
- **Smaller final image**: Chỉ có runtime
- **Security**: Ít attack surface
- **Fast pulls**: Pull nhanh hơn

---

## 🧹 PHẦN 2: CLEANUP TECHNIQUES

### 2.1. Remove Package Cache

**apt-get:**
```dockerfile
RUN apt-get update && \
    apt-get install -y package && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

**apk (Alpine):**
```dockerfile
RUN apk add --no-cache package
```

**pip:**
```dockerfile
RUN pip install --no-cache-dir -r requirements.txt
```

### 2.2. Remove Build Tools

**Multi-stage:**
```dockerfile
FROM node:18-alpine AS builder
RUN npm install

FROM node:18-alpine
# No build tools in final image
```

### 2.3. Remove Temporary Files

**Cleanup in same RUN:**
```dockerfile
RUN build-command && \
    rm -rf /tmp/* && \
    rm -rf /var/tmp/*
```

---

## 🏭 PRODUCTION STORY: Image Size Reduction

### Context

**Công ty:** SaaS, 500 employees
**Image size:** 1.5GB
**Pull time:** 10 phút
**Requirement:** < 300MB, < 2 phút

### Fix

**Optimizations:**
1. Multi-stage builds
2. Alpine base
3. Remove build tools
4. Cleanup caches

**Results:**
- Image size: 1.5GB → 250MB
- Pull time: 10 phút → 1.5 phút

---

## 🎓 TÓM TẮT

**Techniques:**
- Use minimal base images
- Multi-stage builds
- Remove build tools
- Cleanup caches
- Production dependencies only

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-019)** sẽ đi sâu vào:
- BuildKit & Advanced Build Features
- Build performance improvements

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.

