# Day-027: Non-root User & Capabilities - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Create Non-root User

**Dockerfile:**
```dockerfile
FROM alpine:latest
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser
USER appuser
WORKDIR /app
COPY . .
CMD ["app"]
```

**Verification:**
```bash
$ docker run --rm my-app whoami
# Output: appuser
```

---

## ✅ BÀI TẬP 2: Fix Permissions

**Dockerfile:**
```dockerfile
FROM alpine:latest
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser
USER appuser
WORKDIR /app
COPY --chown=appuser:appuser . .
CMD ["app"]
```

**Results:**
- Files owned by appuser
- Correct permissions
- Access works

---

## ✅ BÀI TẬP 3: Drop Capabilities

**Commands:**
```bash
$ docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx
```

**Analysis:**
- Drop all capabilities
- Add only needed
- Reduce attack surface

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

