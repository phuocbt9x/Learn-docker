# Day-049: Permission & Filesystem Issues - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Permission Debugging

**Commands:**
```bash
$ docker exec <container> ls -la /path
# Check permissions

$ docker exec <container> whoami
# Check user
```

**Fix:**
```dockerfile
COPY --chown=user:group file /path
```

---

## ✅ BÀI TẬP 2: Volume Permissions

**Fix:**
```yaml
services:
  app:
    user: "1000:1000"
    volumes:
      - data:/app/data
```

**Results:**
- Permissions correct
- Application works

---

## ✅ BÀI TẬP 3: Production Fix

**Dockerfile:**
```dockerfile
FROM alpine:latest
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser
USER appuser
COPY --chown=appuser:appuser . /app
```

**Results:**
- Non-root user
- Correct permissions
- Production-ready

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

