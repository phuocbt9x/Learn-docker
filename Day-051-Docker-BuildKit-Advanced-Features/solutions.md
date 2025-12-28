# Day-051: Docker BuildKit Advanced Features - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Build Secrets

**Dockerfile:**
```dockerfile
# syntax=docker/dockerfile:1.4
FROM alpine:latest
RUN --mount=type=secret,id=apikey \
    cat /run/secrets/apikey
```

**Build:**
```bash
$ docker build --secret id=apikey,src=./api.key -t my-app .
```

**Verification:**
```bash
$ docker run --rm my-app cat /run/secrets/apikey
# No output → secret not in image ✅
```

---

## ✅ BÀI TẬP 2: Cache Mounts

**Dockerfile:**
```dockerfile
# syntax=docker/dockerfile:1.4
FROM node:18-alpine
RUN --mount=type=cache,target=/root/.npm \
    npm install
```

**Results:**
- First build: 5 phút
- Second build: 1 phút (cache hit)

---

## ✅ BÀI TẬP 3: SSH Mounts

**Dockerfile:**
```dockerfile
# syntax=docker/dockerfile:1.4
FROM alpine:latest
RUN apk add --no-cache git openssh-client
RUN --mount=type=ssh \
    git clone git@github.com:user/repo.git
```

**Build:**
```bash
$ docker build --ssh default -t my-app .
```

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

