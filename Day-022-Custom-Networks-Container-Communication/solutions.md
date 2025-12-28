# Day-022: Custom Networks & Container Communication - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Create Custom Network

**Command:**
```bash
$ docker network create \
  --subnet=10.0.0.0/24 \
  --gateway=10.0.0.1 \
  app-network
```

**Inspection:**
```bash
$ docker network inspect app-network
# Verify subnet and gateway
```

---

## ✅ BÀI TẬP 2: Container Communication

**Commands:**
```bash
$ docker network create app-network

$ docker run -d --network app-network --name web nginx
$ docker run -d --network app-network --name api alpine

# Test DNS
$ docker exec api ping web
# ✅ DNS resolution works
```

---

## ✅ BÀI TẬP 3: Multi-container Setup

**Commands:**
```bash
$ docker network create app-network

$ docker run -d --network app-network --name web nginx
$ docker run -d --network app-network --name api api-service
$ docker run -d --network app-network --name db postgres

# Test communication
$ docker exec web ping api
$ docker exec api ping db
# ✅ All containers communicate
```

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

