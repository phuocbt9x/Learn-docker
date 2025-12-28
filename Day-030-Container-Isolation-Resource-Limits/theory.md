# Day-030: Container Isolation & Resource Limits

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được container isolation
- Biết cách set resource limits
- Hiểu được CPU và memory limits
- Biết cách monitor resources
- Áp dụng được trong production

---

## 🔒 PHẦN 1: CONTAINER ISOLATION

### 1.1. Isolation là gì?

**Container Isolation** là **tách biệt containers** với nhau và với host.

**Isolation levels:**
- **Process isolation**: Processes isolated
- **Network isolation**: Networks isolated
- **Filesystem isolation**: Filesystems isolated
- **Resource isolation**: Resources isolated

### 1.2. Network Isolation

**Custom networks:**
```bash
$ docker network create app-network
$ docker run --network app-network app
```

**Benefits:**
- **Isolated communication**: Containers isolated
- **Security**: Better security
- **Control**: Better control

---

## 📊 PHẦN 2: RESOURCE LIMITS

### 2.1. CPU Limits

**Set CPU limit:**
```bash
$ docker run --cpus="1.5" app
# Limit to 1.5 CPUs
```

**CPU shares:**
```bash
$ docker run --cpu-shares=512 app
# Relative CPU priority
```

### 2.2. Memory Limits

**Set memory limit:**
```bash
$ docker run -m 512m app
# Limit to 512MB
```

**Memory reservation:**
```bash
$ docker run --memory-reservation=256m app
# Reserve 256MB
```

### 2.3. Monitor Resources

**Container stats:**
```bash
$ docker stats
# Real-time resource usage
```

**Inspect:**
```bash
$ docker inspect <container> | grep -i memory
```

---

## 🏭 PRODUCTION STORY: Resource Exhaustion

### Context

**Công ty:** E-commerce, 800 employees
**Issue:** Container consume all resources
**Root cause:** No resource limits

### Fix

**Solution: Resource limits**
```bash
$ docker run -m 1g --cpus="2" app
# Set memory and CPU limits
```

**Results:**
- No resource exhaustion
- Better stability
- Predictable performance

---

## 🎓 TÓM TẮT

**Isolation:**
- Process, network, filesystem isolation
- Custom networks
- Security benefits

**Resource limits:**
- CPU limits
- Memory limits
- Monitor resources

---

## 🚀 BƯỚC TIẾP THEO

**Phase 6 hoàn thành!** Bạn đã nắm vững Security & Hardening.

**Phase tiếp theo (Phase 7)** sẽ đi sâu vào:
- Production Operations
- Monitoring và logging

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-031: Container-Health-Checks](../Day-031-Container-Health-Checks/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
