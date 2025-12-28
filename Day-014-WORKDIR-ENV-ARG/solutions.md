# Day-014: WORKDIR, ENV, ARG - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

Tài liệu này cung cấp giải pháp chi tiết cho tất cả các bài tập.

---

## ✅ BÀI TẬP 1: WORKDIR Basics

**Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
CMD ["node", "index.js"]
```

**Test:**
```bash
$ docker build -t my-app .
$ docker run --rm my-app pwd
# Output: /app
```

**Giải thích:**
- WORKDIR set current directory
- COPY và RUN dùng relative paths
- Persist qua layers

---

## ✅ BÀI TẬP 2: ENV Basics

**Dockerfile:**
```dockerfile
FROM python:3.9-slim
ENV NODE_ENV=production
ENV PORT=3000
COPY app.py .
CMD ["python", "app.py"]
```

**Test:**
```bash
$ docker build -t my-app .
$ docker run --rm my-app env | grep -E "NODE_ENV|PORT"
# Output: NODE_ENV=production, PORT=3000

# Override
$ docker run --rm -e PORT=8080 my-app env | grep PORT
# Output: PORT=8080
```

---

## ✅ BÀI TẬP 3: ARG Basics

**Dockerfile:**
```dockerfile
FROM alpine:latest
ARG VERSION=latest
RUN echo "Building version: $VERSION"
```

**Build:**
```bash
$ docker build --build-arg VERSION=1.0.0 -t my-app .
# Output: Building version: 1.0.0

# Verify ARG not in runtime
$ docker run --rm my-app env | grep VERSION
# Output: (empty - ARG not available)
```

---

## ✅ BÀI TẬP 4: ENV vs ARG

**Comparison:**
- ENV: Runtime, in image, override với `-e`
- ARG: Build-time, not in image, pass với `--build-arg`

**Recommendation:**
- ENV cho runtime config
- ARG cho build-time config

---

## ✅ BÀI TẬP 5: ENV từ ARG

**Dockerfile:**
```dockerfile
FROM alpine:latest
ARG VERSION=latest
ENV APP_VERSION=$VERSION
RUN echo "Building version: $VERSION"
```

**Test:**
```bash
$ docker build --build-arg VERSION=1.0.0 -t my-app .
$ docker run --rm my-app env | grep APP_VERSION
# Output: APP_VERSION=1.0.0
```

---

## ✅ BÀI TẬP 6: Practical Dockerfile

**Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
ARG VERSION=latest
ENV NODE_ENV=production
ENV PORT=3000
ENV APP_VERSION=$VERSION
COPY package.json .
RUN npm install
COPY . .
CMD ["node", "index.js"]
```

---

## ✅ BÀI TẬP 7: Troubleshooting

**Fixed Dockerfile:**
```dockerfile
FROM node:18-alpine
ARG DB_HOST=localhost
ENV DB_HOST=$DB_HOST
COPY app.js .
CMD ["node", "app.js"]
```

**Giải thích:**
- ARG không available khi runtime
- Set ENV từ ARG để runtime access

---

## ✅ BÀI TẬP 8: Production Analysis

**Refactored Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
ARG VERSION=latest
ENV NODE_ENV=production
ENV PORT=3000
ENV APP_VERSION=$VERSION
COPY package.json .
RUN npm install
COPY . .
CMD ["node", "index.js"]
```

**Changes:**
- Add WORKDIR
- Add ENV defaults
- Add ARG for version
- Set ENV from ARG

---

## ✅ BÀI TẬP 9: Build Metadata

**Dockerfile:**
```dockerfile
FROM node:18-alpine
ARG VERSION
ARG BUILD_DATE
ARG GIT_COMMIT
ENV APP_VERSION=$VERSION
ENV BUILD_DATE=$BUILD_DATE
ENV GIT_COMMIT=$GIT_COMMIT
WORKDIR /app
COPY . .
CMD ["node", "index.js"]
```

**Build script:**
```bash
#!/bin/bash
docker build \
  --build-arg VERSION=$(git describe --tags) \
  --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
  --build-arg GIT_COMMIT=$(git rev-parse HEAD) \
  -t my-app .
```

---

## ✅ BÀI TẬP 10: Conditional Build

**Dockerfile:**
```dockerfile
FROM node:18-alpine
ARG NODE_ENV=production
ENV NODE_ENV=$NODE_ENV
WORKDIR /app
COPY package.json .
RUN if [ "$NODE_ENV" = "production" ]; then \
      npm install --production; \
    else \
      npm install; \
    fi
COPY . .
CMD ["node", "index.js"]
```

**Build:**
```bash
# Production
$ docker build --build-arg NODE_ENV=production -t my-app:prod .

# Development
$ docker build --build-arg NODE_ENV=development -t my-app:dev .
```

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

