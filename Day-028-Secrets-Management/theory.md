# Day-028: Secrets Management trong Docker

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được tầm quan trọng của secrets management
- Biết cách sử dụng Docker Secrets
- Hiểu được các alternatives (environment variables, external tools)
- Biết được best practices
- Áp dụng được trong production

---

## 🔐 PHẦN 1: SECRETS MANAGEMENT

### 1.1. Secrets là gì?

**Secrets** là **sensitive data** như passwords, API keys, certificates.

**Characteristics:**
- **Sensitive**: Không được expose
- **Encrypted**: Cần được encrypt
- **Rotated**: Cần rotate regularly
- **Audited**: Cần audit access

### 1.2. Docker Secrets (Swarm)

**Create secret:**
```bash
$ echo "mysecret" | docker secret create my_secret -
```

**Use secret:**
```bash
$ docker service create \
  --secret my_secret \
  nginx
```

**Access trong container:**
```bash
# Secret available tại /run/secrets/my_secret
```

### 1.3. Environment Variables

**Runtime injection:**
```bash
$ docker run -e API_KEY=secret123 app
```

**Best practice:**
- **Runtime only**: Không hardcode trong image
- **External management**: Dùng secrets management tools

---

## 🏭 PRODUCTION STORY: Secrets Exposure

### Context

**Công ty:** SaaS, 600 employees
**Incident:** Secrets exposed trong image
**Root cause:** Secrets hardcoded

### Fix

**Solution: Runtime injection**
```bash
$ docker run -e API_KEY=$API_KEY app
# Secrets từ environment, không trong image
```

**Results:**
- No secrets in image
- Secure management
- Compliance achieved

---

## 🎓 TÓM TẮT

**Secrets management:**
- Never hardcode secrets
- Use Docker Secrets (Swarm)
- Runtime injection
- External tools (Vault, etc.)

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-029)** sẽ đi sâu vào:
- Image Vulnerabilities & Patching
- Vulnerability management

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-029: Image-Vulnerabilities-Patching](../Day-029-Image-Vulnerabilities-Patching/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
