# Day-026: Container Security Fundamentals - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Security Analysis

**Security Issues:**
1. Large base image (ubuntu:20.04)
2. Unnecessary packages (vim, git)
3. Running as root
4. No image scanning

**Refactored Dockerfile:**
```dockerfile
FROM alpine:latest
RUN apk add --no-cache python3
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser
USER appuser
COPY . .
CMD ["app"]
```

**Improvements:**
- Minimal base image
- Only necessary packages
- Non-root user
- Reduced attack surface

---

## ✅ BÀI TẬP 2: Minimal Base Image

**Refactored:**
```dockerfile
FROM python:3.9-alpine
# hoặc
FROM alpine:latest
RUN apk add --no-cache python3
```

**Comparison:**
- ubuntu:20.04: 200MB, many packages
- alpine:latest: 5MB, minimal packages
- **Security**: Alpine has smaller attack surface

---

## ✅ BÀI TẬP 3: Security Scanning

**Scan:**
```bash
$ docker build -t my-app .
$ docker scout cves my-app
# hoặc
$ trivy image my-app
```

**Fix:**
- Update base image
- Remove vulnerable packages
- Apply security patches

---

## ✅ BÀI TẬP 4: Production Security

**Secure Dockerfile:**
```dockerfile
FROM alpine:latest
RUN apk add --no-cache python3 && \
    addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser
USER appuser
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "app.py"]
```

**Security measures:**
- Minimal base image
- Non-root user
- No unnecessary packages
- Production dependencies only

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

