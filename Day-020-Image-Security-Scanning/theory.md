# Day-020: Image Security Scanning

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được tầm quan trọng của security scanning
- Biết cách scan images cho vulnerabilities
- Hiểu được các tools để scan images
- Biết được cách fix vulnerabilities
- Integrate security scanning vào CI/CD

---

## 🔒 PHẦN 1: SECURITY SCANNING

### 1.1. Tại sao cần Security Scanning?

**Risks:**
- **Vulnerabilities**: Known vulnerabilities trong base images
- **Outdated packages**: Packages với security issues
- **Compliance**: Security compliance requirements
- **Reputation**: Security breaches damage reputation

### 1.2. Scanning Tools

**Docker Scout:**
```bash
docker scout cves my-app
```

**Trivy:**
```bash
trivy image my-app
```

**Snyk:**
```bash
snyk test --docker my-app
```

### 1.3. CI/CD Integration

**GitHub Actions:**
```yaml
- name: Scan image
  run: trivy image my-app
```

**Best practices:**
- Scan trong CI/CD
- Block builds với critical vulnerabilities
- Regular scans

---

## 🏭 PRODUCTION STORY: Security Incident

### Context

**Công ty:** Fintech, 400 employees
**Incident:** Critical vulnerability trong production
**Impact:** Security breach

### Fix

**Solution:**
1. Regular security scans
2. Automated scanning trong CI/CD
3. Fix vulnerabilities immediately

**Results:**
- Zero security incidents trong 6 tháng
- Compliance achieved

---

## 🎓 TÓM TẮT

**Security scanning:**
- Critical cho production
- Integrate vào CI/CD
- Regular scans
- Fix immediately

---

## 🚀 BƯỚC TIẾP THEO

**Phase 4 hoàn thành!** Bạn đã nắm vững Image Optimization.

**Phase tiếp theo (Phase 5)** sẽ đi sâu vào:
- Networking & Storage
- Container communication
- Volume management

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-021: Docker-Networks-Bridge-Host-None](../Day-021-Docker-Networks-Bridge-Host-None/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
