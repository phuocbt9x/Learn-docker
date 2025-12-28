# Day-026: Container Security Fundamentals

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được container security fundamentals
- Biết được các security risks trong containers
- Hiểu được defense-in-depth approach
- Biết được security best practices
- Áp dụng được security measures

---

## 🔒 PHẦN 1: CONTAINER SECURITY OVERVIEW

### 1.1. Container Security là gì?

**Container Security** là **bảo vệ containers** khỏi các threats và vulnerabilities.

**Key areas:**
- **Image security**: Secure base images
- **Runtime security**: Secure container runtime
- **Network security**: Secure network communication
- **Data security**: Protect sensitive data
- **Access control**: Control access to containers

### 1.2. Security Risks

**Common risks:**
- **Vulnerable images**: Base images với vulnerabilities
- **Root access**: Containers chạy với root privileges
- **Exposed ports**: Expose ports không cần thiết
- **Secrets in images**: Secrets hardcoded trong images
- **Network exposure**: Networks không được isolate

### 1.3. Defense-in-Depth

**Principle:** Multiple layers of security

**Layers:**
1. **Image security**: Secure images
2. **Runtime security**: Secure runtime
3. **Network security**: Secure networks
4. **Access control**: Control access
5. **Monitoring**: Monitor và detect threats

---

## 🛡️ PHẦN 2: SECURITY BEST PRACTICES

### 2.1. Use Minimal Base Images

**Bad:**
```dockerfile
FROM ubuntu:20.04
# Large image với nhiều packages
```

**Good:**
```dockerfile
FROM alpine:latest
# Minimal image, ít attack surface
```

**Benefits:**
- **Smaller attack surface**: Ít packages = ít vulnerabilities
- **Faster scans**: Scan nhanh hơn
- **Less maintenance**: Ít packages cần update

### 2.2. Run as Non-root

**Bad:**
```dockerfile
FROM node:18-alpine
# Chạy với root user
```

**Good:**
```dockerfile
FROM node:18-alpine
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser
USER appuser
# Chạy với non-root user
```

**Benefits:**
- **Reduced risk**: Giảm risk nếu container bị compromise
- **Principle of least privilege**: Chỉ có privileges cần thiết

### 2.3. Remove Unnecessary Packages

**Bad:**
```dockerfile
RUN apt-get install -y curl wget vim git
# Nhiều packages không cần
```

**Good:**
```dockerfile
RUN apt-get install -y --no-install-recommends \
    curl && \
    apt-get clean
# Chỉ install packages cần thiết
```

### 2.4. Scan Images Regularly

**Scan images:**
```bash
$ docker scout cves my-app
# hoặc
$ trivy image my-app
```

**Best practice:**
- **Regular scans**: Scan images regularly
- **CI/CD integration**: Integrate vào CI/CD
- **Fix vulnerabilities**: Fix vulnerabilities ngay

---

## 🏭 PRODUCTION STORY #1: Security Breach

### Context

**Công ty:** Fintech, 500 employees
**Incident:** Container bị compromise
**Root cause:** Root access và vulnerable image

### Problem

**Tháng 2/2024:**
- **Security breach**: Container bị compromise
- **Data exposure**: Sensitive data exposed
- **Root cause**: Root access + vulnerable base image

**Timeline:**
- **10:00 AM**: Security alert
- **10:15 AM**: Container compromised
- **10:30 AM**: Team investigate
- **11:00 AM**: Fix implemented

**Impact:**
- **Data exposure**: Sensitive data exposed
- **Compliance**: Không đạt compliance
- **Reputation**: Damage to reputation

### Investigation

**Root cause:**
```dockerfile
FROM ubuntu:20.04
# Vulnerable base image
RUN apt-get install -y many-packages
# Chạy với root
CMD ["app"]
```

**Vấn đề:**
- **Vulnerable image**: Base image có known vulnerabilities
- **Root access**: Container chạy với root
- **No scanning**: Không scan images

### Fix

**Solution:**
```dockerfile
FROM alpine:latest
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser
USER appuser
# Minimal image, non-root user
```

**Additional measures:**
- **Regular scans**: Scan images trong CI/CD
- **Update images**: Update base images regularly
- **Monitor**: Monitor containers

### Result

**Trước:**
- Vulnerable image
- Root access
- **Security**: ❌ Breached

**Sau:**
- Minimal image
- Non-root user
- **Security**: ✅ Improved

### Lesson Learned

1. **Use minimal images**: Giảm attack surface
2. **Run as non-root**: Giảm risk
3. **Scan regularly**: Detect vulnerabilities
4. **Update regularly**: Fix vulnerabilities

---

## 🏭 PRODUCTION STORY #2: Secrets Exposure

### Context

**Công ty:** SaaS, 600 employees
**Incident:** Secrets exposed trong image
**Root cause:** Secrets hardcoded trong Dockerfile

### Problem

**Tháng 4/2024:**
- **Secrets exposed**: API keys trong image
- **Security scan**: Phát hiện secrets trong image
- **Root cause**: Secrets hardcoded trong Dockerfile

**Impact:**
- **Secrets exposed**: API keys accessible
- **Security risk**: High risk
- **Compliance**: Không đạt compliance

### Investigation

**Root cause:**
```dockerfile
ENV API_KEY=secret123
# Secrets hardcoded trong Dockerfile
```

**Vấn đề:**
- **Secrets in image**: Secrets có trong image
- **Version control**: Secrets trong Git
- **Exposed**: Secrets accessible

### Fix

**Solution: Docker Secrets**
```dockerfile
# Không hardcode secrets
# Use Docker secrets hoặc environment variables
```

**Best practice:**
- **No secrets in images**: Không hardcode secrets
- **Use secrets management**: Dùng secrets management tools
- **Runtime injection**: Inject secrets at runtime

### Result

**Trước:**
- Secrets in image
- **Security**: ❌ Exposed

**Sau:**
- No secrets in image
- **Security**: ✅ Improved

### Lesson Learned

1. **Never hardcode secrets**: Không hardcode secrets
2. **Use secrets management**: Dùng secrets management
3. **Runtime injection**: Inject secrets at runtime
4. **Regular audits**: Audit images for secrets

---

## 🎓 TÓM TẮT

### Security Fundamentals

**Key principles:**
- **Defense-in-depth**: Multiple layers
- **Least privilege**: Minimum privileges
- **Minimal attack surface**: Reduce attack surface
- **Regular updates**: Update regularly

### Best Practices

**1. Use minimal images:**
- Alpine, distroless
- Smaller attack surface

**2. Run as non-root:**
- Create non-root user
- Reduce risk

**3. Remove unnecessary packages:**
- Only install needed packages
- Reduce vulnerabilities

**4. Scan regularly:**
- CI/CD integration
- Fix vulnerabilities

**5. No secrets in images:**
- Use secrets management
- Runtime injection

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ bạn đã:
- ✅ Hiểu container security fundamentals
- ✅ Biết security best practices
- ✅ Áp dụng security measures

**Day tiếp theo (Day-027)** sẽ đi sâu vào:
- Non-root User & Capabilities
- User management
- Capabilities management

---

## 📚 TÀI LIỆU THAM KHẢO

- Container Security: https://docs.docker.com/engine/security/
- Security Best Practices: https://docs.docker.com/develop/security-best-practices/

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.

