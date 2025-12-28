# Day-030: Container Isolation & Resource Limits - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Resource Limits

**Commands:**
```bash
$ docker run -d -m 512m --cpus="1" --name app nginx

# Monitor
$ docker stats app
```

**Results:**
- Memory limit: 512MB
- CPU limit: 1 CPU
- Limits enforced

---

## ✅ BÀI TẬP 2: Network Isolation

**Commands:**
```bash
$ docker network create app1-network
$ docker network create app2-network

$ docker run -d --network app1-network --name app1 nginx
$ docker run -d --network app2-network --name app2 nginx

# Test isolation
$ docker exec app1 ping app2
# ❌ Cannot reach (isolated)
```

---

## ✅ BÀI TẬP 3: Production Configuration

**Configuration:**
```bash
$ docker run -d \
  -m 1g \
  --cpus="2" \
  --network app-network \
  --name app \
  nginx
```

**Justification:**
- Memory limit: Prevent exhaustion
- CPU limit: Predictable performance
- Network isolation: Security

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

