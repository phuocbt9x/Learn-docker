# Day-024: Bind Mounts vs Volumes - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Bind Mount

**Commands:**
```bash
# Create container với bind mount
$ docker run -d -v /host/data:/container/data nginx

# Test
$ echo "test" > /host/data/file.txt
$ docker exec <container> cat /container/data/file.txt
# ✅ File accessible
```

**Results:**
- Bind mount works
- Host-container sync
- Direct access

---

## ✅ BÀI TẬP 2: Volume

**Commands:**
```bash
# Create volume
$ docker volume create app-data

# Use volume
$ docker run -d -v app-data:/data nginx

# Test
$ docker exec <container> echo "test" > /data/file.txt
# ✅ Data persist
```

**Results:**
- Volume works
- Data persist
- Portable

---

## ✅ BÀI TẬP 3: Production Scenario

**Choice:** Volume

**Justification:**
- Production data
- Need backup
- Portable

**Implementation:**
```bash
$ docker volume create db-data
$ docker run -d -v db-data:/var/lib/postgresql/data postgres
```

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

