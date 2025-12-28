# Day-021: Docker Networks - Bridge, Host, None - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Bridge Network Basics

**Commands:**
```bash
# Create container
$ docker run -d -p 8080:80 --name web nginx

# Inspect network
$ docker network inspect bridge

# Test
$ curl http://localhost:8080
```

**Results:**
- Container connected to bridge network
- Port mapping: 8080:80
- Accessible from host

---

## ✅ BÀI TẬP 2: Custom Bridge Network

**Commands:**
```bash
# Create network
$ docker network create my-network

# Create containers
$ docker run -d --network my-network --name app1 nginx
$ docker run -d --network my-network --name app2 nginx

# Test DNS
$ docker exec app1 ping app2
# ✅ DNS resolution works
```

**Results:**
- Custom network created
- Containers connected
- DNS resolution works

---

## ✅ BÀI TẬP 3: Host Network

**Commands:**
```bash
# Create container với host network
$ docker run -d --network host nginx

# Test direct access
$ curl http://localhost:80
# ✅ Direct access (no port mapping needed)
```

**Results:**
- Host network works
- Direct access
- No port mapping needed

---

## ✅ BÀI TẬP 4: None Network

**Commands:**
```bash
# Create container với none network
$ docker run -d --network none alpine

# Verify no network
$ docker exec <container> ping 8.8.8.8
# ❌ No network connectivity
```

**Results:**
- No network connectivity
- Complete isolation
- Use cases: Testing, security

---

## ✅ BÀI TẬP 5: Network Comparison

**Comparison Table:**

| Feature | Bridge | Host | None |
|---------|--------|------|------|
| Isolation | ✅ | ❌ | ✅ |
| Port mapping | ✅ | ❌ | N/A |
| DNS | ✅ (custom) | ❌ | N/A |
| Performance | ⚠️ | ✅ | N/A |

**Recommendations:**
- Bridge: Most cases
- Host: High performance (Linux)
- None: Special cases

---

## ✅ BÀI TẬP 6: Network Troubleshooting

**Problem:** No port mapping

**Fix:**
```bash
$ docker run -d -p 8080:80 --name web nginx
# Add port mapping
```

**Results:**
- Port mapping added
- Container accessible

---

## ✅ BÀI TẬP 7: Production Scenario

**Choice:** Host network

**Justification:**
- High performance requirement
- Linux environment
- Direct access needed

**Implementation:**
```bash
$ docker run -d --network host nginx
```

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

