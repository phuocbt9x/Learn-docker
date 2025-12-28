# Day-021: Docker Networking - Bridge, Host, None

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được các network drivers trong Docker
- Biết được sự khác biệt giữa Bridge, Host, None networks
- Hiểu được khi nào dùng network driver nào
- Biết cách tạo và quản lý networks
- Debug được network issues

---

## 🌐 PHẦN 1: DOCKER NETWORKING OVERVIEW

### 1.1. Docker Networking là gì?

**Docker Networking** là cơ chế Docker **kết nối containers** với nhau và với external world.

**Key concepts:**
- **Network drivers**: Các loại network (bridge, host, none, etc.)
- **Network isolation**: Containers có thể isolated hoặc connected
- **Port mapping**: Map container ports ra host ports
- **DNS resolution**: Containers có thể resolve tên của nhau

### 1.2. Default Network

**Khi tạo container không specify network:**
```bash
$ docker run nginx
# Container được connect vào default bridge network
```

**Default bridge network:**
- **Name**: `bridge`
- **Driver**: `bridge`
- **Isolation**: Containers isolated by default
- **DNS**: Không có DNS resolution giữa containers

### 1.3. List Networks

**View all networks:**
```bash
$ docker network ls
```

**Inspect network:**
```bash
$ docker network inspect bridge
```

---

## 🌉 PHẦN 2: BRIDGE NETWORK

### 2.1. Bridge Network là gì?

**Bridge network** là **default network driver** trong Docker, tạo một **internal network** trên host.

**Characteristics:**
- **Isolated**: Containers isolated từ host network
- **Port mapping**: Cần port mapping để access từ host
- **Internal communication**: Containers có thể communicate với nhau
- **Default**: Default network cho containers

### 2.2. Bridge Network Behavior

**Create container:**
```bash
$ docker run -d --name web nginx
# Container connect vào default bridge network
```

**Port mapping:**
```bash
$ docker run -d -p 8080:80 --name web nginx
# Map container port 80 → host port 8080
```

**Internal communication:**
```bash
# Container 1
$ docker run -d --name app1 nginx

# Container 2
$ docker run -d --name app2 nginx

# app1 và app2 có thể communicate qua IP addresses
```

### 2.3. Custom Bridge Network

**Create custom bridge:**
```bash
$ docker network create my-network
```

**Connect container:**
```bash
$ docker run -d --network my-network --name web nginx
```

**Benefits:**
- **DNS resolution**: Containers có thể resolve tên
- **Better isolation**: Isolated từ default bridge
- **Custom configuration**: Có thể config subnet, gateway, etc.

---

## 🏠 PHẦN 3: HOST NETWORK

### 3.1. Host Network là gì?

**Host network** là network driver **sử dụng host network stack** trực tiếp.

**Characteristics:**
- **No isolation**: Container sử dụng host network
- **No port mapping**: Không cần port mapping
- **Direct access**: Container ports accessible trực tiếp
- **Performance**: Better performance (no NAT)

### 3.2. Host Network Behavior

**Use host network:**
```bash
$ docker run -d --network host nginx
# Container sử dụng host network
# Port 80 accessible trực tiếp trên host
```

**Important:**
- **Linux only**: Chỉ work trên Linux
- **Port conflicts**: Có thể conflict với host ports
- **Security**: Less isolation

### 3.3. Host Network Use Cases

**Use cases:**
- **High performance**: Cần performance cao
- **Direct access**: Cần direct access từ host
- **Linux only**: Chỉ dùng trên Linux

---

## 🚫 PHẦN 4: NONE NETWORK

### 4.1. None Network là gì?

**None network** là network driver **không có network connectivity**.

**Characteristics:**
- **No network**: Container không có network
- **Complete isolation**: Hoàn toàn isolated
- **Use cases**: Special cases (testing, security)

### 4.2. None Network Behavior

**Use none network:**
```bash
$ docker run -d --network none alpine
# Container không có network
```

**Use cases:**
- **Security testing**: Test applications không cần network
- **Isolated workloads**: Workloads cần complete isolation
- **Special scenarios**: Special use cases

---

## 🔄 PHẦN 5: NETWORK COMPARISON

### 5.1. So Sánh

| Tiêu chí | Bridge | Host | None |
|----------|--------|------|------|
| **Isolation** | ✅ Isolated | ❌ No isolation | ✅ Complete isolation |
| **Port mapping** | ✅ Required | ❌ Not needed | N/A |
| **DNS resolution** | ✅ (custom) | ❌ | N/A |
| **Performance** | ⚠️ NAT overhead | ✅ Best | N/A |
| **Use case** | Most cases | High performance | Special cases |

### 5.2. Khi Nào Dùng Gì?

**Bridge (Recommended):**
- **Most cases**: 90% use cases
- **Isolation**: Cần isolation
- **Port mapping**: Cần port mapping

**Host:**
- **High performance**: Cần performance cao
- **Linux only**: Chỉ trên Linux
- **Direct access**: Cần direct access

**None:**
- **Special cases**: Testing, security
- **Complete isolation**: Cần complete isolation

---

## 🏭 PRODUCTION STORY #1: Network Performance Issues

### Context

**Công ty:** E-commerce, 700 employees
**Hệ thống:** High-traffic microservices
**Traffic:** 20M requests/day
**Team:** 35 backend engineers

### Problem

**Tháng 1/2024:**
- **Network latency cao**: 50ms per request
- **Root cause**: Bridge network với NAT overhead
- **Impact**: Slow response times

**Timeline:**
- **10:00 AM**: Performance degradation reported
- **10:15 AM**: Team investigate
- **10:30 AM**: Root cause: Bridge network overhead
- **11:00 AM**: Fix implemented

**Impact:**
- **Latency**: 50ms → 5ms (90% reduction)
- **Throughput**: Tăng 2x

### Investigation

**Root cause:**
```bash
$ docker run -d -p 8080:80 nginx
# Bridge network với NAT
# Overhead: ~45ms per request
```

**Vấn đề:**
- **NAT overhead**: Bridge network có NAT overhead
- **Performance**: Ảnh hưởng performance

### Fix

**Solution: Host network (Linux)**
```bash
$ docker run -d --network host nginx
# Host network, no NAT
# Latency: ~5ms per request
```

**Kết quả:**
- **Latency**: 50ms → 5ms
- **Throughput**: Tăng 2x
- **Performance**: Improved significantly

### Result

**Trước:**
- Bridge network
- **Latency**: 50ms
- **Throughput**: 10K req/s

**Sau:**
- Host network
- **Latency**: 5ms
- **Throughput**: 20K req/s

### Lesson Learned

1. **Choose network driver carefully**: Ảnh hưởng performance
2. **Host network cho performance**: Dùng khi cần performance
3. **Measure**: Measure performance để optimize
4. **Linux only**: Host network chỉ work trên Linux

---

## 🏭 PRODUCTION STORY #2: Network Isolation Issues

### Context

**Công ty:** SaaS platform, 500 employees
**Hệ thống:** Multi-tenant applications
**Traffic:** 15M requests/day
**Team:** 25 backend engineers

### Problem

**Tháng 3/2024:**
- **Security incident**: Containers access được containers khác
- **Root cause**: Default bridge network không có isolation
- **Impact**: Security breach

**Timeline:**
- **2:00 PM**: Security scan
- **2:15 PM**: Phát hiện containers communicate với nhau
- **2:30 PM**: Team investigate
- **3:00 PM**: Fix implemented

**Impact:**
- **Security risk**: High risk
- **Compliance**: Không đạt compliance

### Investigation

**Root cause:**
```bash
$ docker run -d --name app1 nginx
$ docker run -d --name app2 nginx
# Cả 2 containers trong default bridge
# Có thể communicate với nhau qua IP
```

**Vấn đề:**
- **No isolation**: Default bridge không có isolation
- **Security risk**: Containers có thể access nhau

### Fix

**Solution: Custom networks**
```bash
$ docker network create app1-network
$ docker network create app2-network

$ docker run -d --network app1-network --name app1 nginx
$ docker run -d --network app2-network --name app2 nginx
# Containers isolated
```

**Kết quả:**
- **Isolation**: Containers isolated
- **Security**: Improved
- **Compliance**: Đạt compliance

### Result

**Trước:**
- Default bridge
- **Isolation**: ❌
- **Security**: ⚠️ Risk

**Sau:**
- Custom networks
- **Isolation**: ✅
- **Security**: ✅

### Lesson Learned

1. **Use custom networks**: Better isolation
2. **Default bridge risks**: Default bridge có security risks
3. **Network isolation**: Critical cho multi-tenant
4. **Security best practices**: Always isolate networks

---

## 🎓 TÓM TẮT

### Network Drivers

**Bridge:**
- Default network driver
- Isolated containers
- Port mapping required
- Most common use case

**Host:**
- Uses host network
- No isolation
- Best performance
- Linux only

**None:**
- No network
- Complete isolation
- Special use cases

### Best Practices

**1. Use custom bridge networks:**
- Better isolation
- DNS resolution
- Custom configuration

**2. Avoid default bridge:**
- Security risks
- No DNS resolution
- Limited configuration

**3. Choose based on requirements:**
- Bridge: Most cases
- Host: High performance (Linux)
- None: Special cases

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ bạn đã:
- ✅ Hiểu network drivers
- ✅ Biết khi nào dùng gì
- ✅ Debug network issues

**Day tiếp theo (Day-022)** sẽ đi sâu vào:
- Custom Networks & Container Communication
- DNS resolution
- Network configuration

---

## 📚 TÀI LIỆU THAM KHẢO

- Docker Networking: https://docs.docker.com/network/
- Network drivers: https://docs.docker.com/network/drivers/

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.

