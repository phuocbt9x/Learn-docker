# Day-032: Resource Limits (CPU, Memory) - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: CPU Limits

**Commands:**
```bash
$ docker run -d --cpus="1.5" --cpu-shares=512 --name app nginx

# Monitor
$ docker stats app
```

**Results:**
- CPU limit: 1.5 CPUs
- CPU shares: 512
- Limits enforced

---

## ✅ BÀI TẬP 2: Memory Limits

**Commands:**
```bash
$ docker run -d -m 512m --memory-reservation=256m --name app nginx

# Test OOM
$ docker exec app sh -c "dd if=/dev/zero of=/tmp/test bs=1M count=600"
# Container bị OOM kill
```

**Results:**
- Memory limit: 512MB
- Memory reservation: 256MB
- OOM handling works

---

## ✅ BÀI TẬP 3: Resource Monitoring

**Commands:**
```bash
$ docker stats
# Monitor all containers

$ docker stats app
# Monitor specific container
```

**Analysis:**
- CPU usage patterns
- Memory usage trends
- Identify bottlenecks

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

