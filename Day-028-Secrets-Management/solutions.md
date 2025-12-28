# Day-028: Secrets Management - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Docker Secrets

**Commands:**
```bash
# Initialize Swarm
$ docker swarm init

# Create secret
$ echo "mysecret" | docker secret create my_secret -

# Use secret
$ docker service create \
  --secret my_secret \
  --name my-service \
  nginx
```

**Verification:**
```bash
$ docker exec <container> cat /run/secrets/my_secret
# Output: mysecret
```

---

## ✅ BÀI TẬP 2: Runtime Injection

**Commands:**
```bash
# Inject at runtime
$ docker run -e API_KEY=$API_KEY app

# Verify not in image
$ docker inspect <image> | grep API_KEY
# No output → secret not in image ✅
```

---

## ✅ BÀI TẬP 3: Secrets Audit

**Analysis:**
- Found: ENV API_KEY=secret123
- Issue: Secret hardcoded

**Refactored:**
```dockerfile
# Remove ENV API_KEY
# Inject at runtime
```

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

