# Day-027: Non-root User & Capabilities

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được tại sao cần non-root user
- Biết cách tạo và sử dụng non-root user
- Hiểu được Linux capabilities
- Biết cách drop capabilities
- Áp dụng được trong production

---

## 👤 PHẦN 1: NON-ROOT USER

### 1.1. Tại sao cần Non-root User?

**Risks của root user:**
- **Full privileges**: Root có full system privileges
- **Security risk**: Nếu container bị compromise, attacker có full access
- **Compliance**: Nhiều compliance requirements yêu cầu non-root

**Benefits của non-root:**
- **Reduced risk**: Giảm risk nếu container bị compromise
- **Principle of least privilege**: Chỉ có privileges cần thiết
- **Compliance**: Đạt compliance requirements

### 1.2. Tạo Non-root User

**Alpine:**
```dockerfile
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser
USER appuser
```

**Debian/Ubuntu:**
```dockerfile
RUN groupadd -r appuser && \
    useradd -r -g appuser appuser
USER appuser
```

### 1.3. Fix Permissions

**Fix file permissions:**
```dockerfile
COPY --chown=appuser:appuser . /app
# Set ownership khi copy
```

---

## 🔐 PHẦN 2: LINUX CAPABILITIES

### 2.1. Capabilities là gì?

**Linux Capabilities** là **fine-grained permissions** thay vì all-or-nothing root access.

**Common capabilities:**
- **NET_BIND_SERVICE**: Bind to ports < 1024
- **CHOWN**: Change file ownership
- **DAC_OVERRIDE**: Bypass file permissions

### 2.2. Drop Capabilities

**Drop all capabilities:**
```dockerfile
FROM alpine:latest
USER appuser
# Drop all capabilities by default
```

**Drop specific capabilities:**
```bash
$ docker run --cap-drop=ALL --cap-add=NET_BIND_SERVICE nginx
```

---

## 🏭 PRODUCTION STORY: Root Access Exploit

### Context

**Công ty:** E-commerce, 700 employees
**Incident:** Container bị exploit
**Root cause:** Root access

### Fix

**Solution: Non-root user**
```dockerfile
FROM alpine:latest
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser
USER appuser
```

**Results:**
- Reduced risk
- Compliance achieved
- No exploits

---

## 🎓 TÓM TẮT

**Non-root user:**
- Reduce security risk
- Compliance requirement
- Best practice

**Capabilities:**
- Fine-grained permissions
- Drop unnecessary capabilities
- Principle of least privilege

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-028)** sẽ đi sâu vào:
- Secrets Management trong Docker
- Docker Secrets
- Best practices

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.

