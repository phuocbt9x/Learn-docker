# Day-029: Image Vulnerabilities & Patching

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được image vulnerabilities
- Biết cách scan images
- Biết cách patch vulnerabilities
- Hiểu được vulnerability management
- Áp dụng được trong production

---

## 🔍 PHẦN 1: IMAGE VULNERABILITIES

### 1.1. Vulnerabilities là gì?

**Vulnerabilities** là **security flaws** trong software có thể bị exploit.

**Sources:**
- **Base images**: Vulnerabilities trong base images
- **Packages**: Vulnerabilities trong packages
- **Dependencies**: Vulnerabilities trong dependencies

### 1.2. Scan Images

**Docker Scout:**
```bash
$ docker scout cves my-app
```

**Trivy:**
```bash
$ trivy image my-app
```

**Snyk:**
```bash
$ snyk test --docker my-app
```

### 1.3. Patch Vulnerabilities

**Update base image:**
```dockerfile
FROM node:18-alpine
# Update từ node:16-alpine
```

**Update packages:**
```dockerfile
RUN apt-get update && apt-get upgrade -y
```

---

## 🏭 PRODUCTION STORY: Vulnerability Management

### Context

**Công ty:** Fintech, 500 employees
**Issue:** 50+ vulnerabilities trong images
**Solution:** Automated scanning và patching

### Implementation

**CI/CD integration:**
```yaml
- name: Scan image
  run: trivy image my-app
  # Block build nếu có critical vulnerabilities
```

**Results:**
- Automated scanning
- Vulnerabilities fixed
- Compliance achieved

---

## 🎓 TÓM TẮT

**Vulnerability management:**
- Regular scanning
- Automated patching
- CI/CD integration
- Update regularly

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-030)** sẽ đi sâu vào:
- Container Isolation & Resource Limits
- Resource management

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.

