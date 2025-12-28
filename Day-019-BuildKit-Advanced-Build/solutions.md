# Day-019: BuildKit & Advanced Build - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Enable BuildKit

**Enable:**
```bash
export DOCKER_BUILDKIT=1
docker build -t my-app .
```

**Results:**
- Without BuildKit: 15 phút
- With BuildKit: 4 phút (73% faster)

---

## ✅ BÀI TẬP 2: Build Secrets

**Dockerfile:**
```dockerfile
# syntax=docker/dockerfile:1.4
FROM node:18-alpine
RUN --mount=type=secret,id=apikey \
    echo "API key configured"
```

**Build:**
```bash
docker build --secret id=apikey,src=./api.key -t my-app .
```

**Verification:**
```bash
docker run --rm my-app cat /run/secrets/apikey
# No output → secret not in image ✅
```

---

## ✅ BÀI TẬP 3: Cache Mounts

**Dockerfile:**
```dockerfile
# syntax=docker/dockerfile:1.4
FROM node:18-alpine
RUN --mount=type=cache,target=/root/.npm \
    npm install
```

**Results:**
- Without cache mount: 5 phút
- With cache mount: 1 phút (80% faster)

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

