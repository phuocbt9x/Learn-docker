# Day-048: Network Connectivity Problems - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Connectivity Testing

**Commands:**
```bash
$ docker exec container1 ping container2
# Test connectivity

$ docker exec container1 nc -zv container2 80
# Test port connectivity
```

**Fix:**
- Ensure containers trong cùng network
- Check network configuration

---

## ✅ BÀI TẬP 2: DNS Resolution

**Commands:**
```bash
$ docker exec container nslookup hostname
# Test DNS
```

**Fix:**
- Use custom networks
- Check DNS configuration

---

## ✅ BÀI TẬP 3: Network Isolation

**Fix:**
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
- Network isolation maintained

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

