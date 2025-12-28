# Day-022: Custom Networks & Container Communication

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được cách tạo và quản lý custom networks
- Biết cách containers communicate với nhau
- Hiểu được DNS resolution trong Docker networks
- Biết cách configure network settings
- Debug được container communication issues

---

## 🌐 PHẦN 1: CUSTOM NETWORKS

### 1.1. Tạo Custom Network

**Create bridge network:**
```bash
$ docker network create my-network
```

**Create với options:**
```bash
$ docker network create \
  --driver bridge \
  --subnet=172.20.0.0/16 \
  --gateway=172.20.0.1 \
  my-network
```

### 1.2. Network Configuration

**Subnet và gateway:**
```bash
$ docker network create \
  --subnet=10.0.0.0/24 \
  --gateway=10.0.0.1 \
  my-network
```

**IP ranges:**
```bash
$ docker network create \
  --ip-range=10.0.0.0/28 \
  my-network
```

---

## 💬 PHẦN 2: CONTAINER COMMUNICATION

### 2.1. DNS Resolution

**Automatic DNS:**
```bash
$ docker network create my-network
$ docker run -d --network my-network --name web nginx
$ docker run -d --network my-network --name app alpine

# app có thể ping web bằng tên
$ docker exec app ping web
```

**Benefits:**
- **Automatic DNS**: Containers resolve tên tự động
- **Service discovery**: Dễ dàng discover services
- **No IP needed**: Không cần biết IP addresses

### 2.2. Container Linking (Legacy)

**Legacy linking:**
```bash
$ docker run -d --name web nginx
$ docker run -d --link web:webapp alpine
# Legacy, không recommended
```

**Modern approach:**
- **Use custom networks**: Thay vì linking
- **DNS resolution**: Automatic DNS trong networks

---

## 🏭 PRODUCTION STORY: Container Communication

### Context

**Công ty:** SaaS, 600 employees
**Hệ thống:** Microservices architecture
**Issue:** Containers không communicate được

### Fix

**Solution: Custom networks với DNS**
```bash
$ docker network create app-network
$ docker run -d --network app-network --name api api-service
$ docker run -d --network app-network --name web web-service
# Containers communicate via DNS
```

**Results:**
- Containers communicate successfully
- DNS resolution works
- Service discovery improved

---

## 🎓 TÓM TẮT

**Custom networks:**
- Better isolation
- DNS resolution
- Custom configuration

**Container communication:**
- DNS resolution
- Service discovery
- Network isolation

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-023)** sẽ đi sâu vào:
- Docker Volumes - Named vs Anonymous
- Data persistence

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-023: Docker-Volumes-Named-vs-Anonymous](../Day-023-Docker-Volumes-Named-vs-Anonymous/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
