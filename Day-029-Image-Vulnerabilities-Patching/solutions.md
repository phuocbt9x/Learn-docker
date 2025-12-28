# Day-029: Image Vulnerabilities & Patching - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Scan Images

**Commands:**
```bash
$ docker build -t my-app .
$ docker scout cves my-app
# hoặc
$ trivy image my-app
```

**Results:**
- Found 15 vulnerabilities
- 3 critical, 5 high, 7 medium

---

## ✅ BÀI TẬP 2: Patch Vulnerabilities

**Updated Dockerfile:**
```dockerfile
FROM node:18-alpine
# Updated từ node:16-alpine
RUN apk update && apk upgrade
```

**Results:**
- Before: 15 vulnerabilities
- After: 2 vulnerabilities (87% reduction)

---

## ✅ BÀI TẬP 3: CI/CD Integration

**GitHub Actions:**
```yaml
- name: Scan image
  run: |
    trivy image my-app
    if [ $? -ne 0 ]; then
      exit 1
    fi
```

**Results:**
- Automated scanning
- Blocks builds với critical vulnerabilities
- Reports generated

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

