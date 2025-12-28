# Day-019: BuildKit & Advanced Build Features

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được BuildKit là gì và benefits
- Biết cách enable BuildKit
- Hiểu được advanced build features
- Biết được build secrets và cache mounts
- Sử dụng được BuildKit trong production

---

## 🔧 PHẦN 1: BUILDKIT

### 1.1. BuildKit là gì?

**BuildKit** là **next-generation build engine** cho Docker, cung cấp:
- **Faster builds**: Build nhanh hơn
- **Better caching**: Cache tốt hơn
- **Parallel builds**: Build parallel
- **Advanced features**: Secrets, cache mounts, etc.

### 1.2. Enable BuildKit

**Environment variable:**
```bash
export DOCKER_BUILDKIT=1
docker build -t my-app .
```

**Dockerfile syntax:**
```dockerfile
# syntax=docker/dockerfile:1.4
FROM node:18-alpine
```

### 1.3. Build Secrets

**Syntax:**
```dockerfile
# syntax=docker/dockerfile:1.4
RUN --mount=type=secret,id=mysecret \
    cat /run/secrets/mysecret
```

**Build:**
```bash
docker build --secret id=mysecret,src=./secret.txt -t my-app .
```

### 1.4. Cache Mounts

**Syntax:**
```dockerfile
# syntax=docker/dockerfile:1.4
RUN --mount=type=cache,target=/root/.npm \
    npm install
```

**Benefits:**
- **Persistent cache**: Cache persist giữa builds
- **Faster installs**: npm install nhanh hơn

---

## 🏭 PRODUCTION STORY: BuildKit Performance

### Context

**Công ty:** E-commerce, 600 employees
**Build time:** 15 phút
**Requirement:** < 5 phút

### Fix

**Enable BuildKit:**
```bash
export DOCKER_BUILDKIT=1
```

**Results:**
- Build time: 15 phút → 4 phút (73% faster)

---

## 🎓 TÓM TẮT

**BuildKit benefits:**
- Faster builds
- Better caching
- Advanced features
- Production-ready

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-020)** sẽ đi sâu vào:
- Image Security Scanning
- Vulnerability management

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.

