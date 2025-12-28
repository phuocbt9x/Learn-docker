# Day-048: Network Connectivity Problems

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được network connectivity issues
- Biết cách debug network problems
- Hiểu được DNS resolution
- Biết cách test connectivity
- Debug network issues
- Áp dụng trong production

---

## 🌐 PHẦN 1: NETWORK DEBUGGING

### 1.1. Connectivity Issues

**Common issues:**
- **Cannot connect**: Containers không thể connect
- **DNS resolution**: DNS không resolve
- **Port issues**: Ports không accessible
- **Network isolation**: Network isolation issues

### 1.2. Debug Tools

**Ping:**
```bash
$ docker exec <container> ping <host>
# Test connectivity
```

**DNS resolution:**
```bash
$ docker exec <container> nslookup <hostname>
# Test DNS
```

**Port check:**
```bash
$ docker exec <container> nc -zv <host> <port>
# Test port connectivity
```

### 1.3. Network Inspection

**Inspect network:**
```bash
$ docker network inspect <network>
# Network details
```

**List networks:**
```bash
$ docker network ls
# List all networks
```

---

## 🏭 PRODUCTION STORY: Network Isolation

### Context

**Công ty:** SaaS, 600 employees
**Issue:** Containers không communicate
**Root cause:** Network isolation

### Fix

**Solution: Custom networks**
```yaml
services:
  api:
    networks:
      - app-network
  db:
    networks:
      - app-network
```

**Results:**
- Containers communicate
- Network isolation
- Security improved

---

## 🎓 TÓM TẮT

**Network debugging:**
- Test connectivity
- Check DNS
- Inspect networks

**Common issues:**
- Network isolation
- DNS resolution
- Port accessibility

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-049)** sẽ đi sâu vào:
- Permission & Filesystem Issues
- File permissions debugging

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-049: Permission-Filesystem-Issues](../Day-049-Permission-Filesystem-Issues/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
