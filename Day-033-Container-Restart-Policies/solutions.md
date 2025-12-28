# Day-033: Container Restart Policies - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Restart Policies

**Test:**
```bash
# no (default)
$ docker run -d --restart=no app
# Không restart

# always
$ docker run -d --restart=always app
# Luôn restart

# on-failure
$ docker run -d --restart=on-failure app
# Restart khi fail

# unless-stopped
$ docker run -d --restart=unless-stopped app
# Restart trừ khi stopped
```

**Recommendations:**
- always: Production
- on-failure: Development
- unless-stopped: Production với manual control

---

## ✅ BÀI TẬP 2: Production Policy

**Choice:** always hoặc unless-stopped

**Justification:**
- High availability requirement
- Automatic recovery
- unless-stopped cho manual control

**Implementation:**
```bash
$ docker run -d --restart=always app
```

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

