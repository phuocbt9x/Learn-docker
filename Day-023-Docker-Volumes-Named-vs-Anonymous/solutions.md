# Day-023: Docker Volumes - Named vs Anonymous - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Named Volumes

**Commands:**
```bash
# Create volume
$ docker volume create my-data

# Use volume
$ docker run -d -v my-data:/data nginx

# Verify
$ docker exec <container> ls /data
```

**Results:**
- Named volume created
- Data persist
- Easy to manage

---

## ✅ BÀI TẬP 2: Anonymous Volumes

**Commands:**
```bash
# Create container với anonymous volume
$ docker run -d -v /data nginx

# Identify volume
$ docker inspect <container> | grep Mounts
# Volume name auto-generated
```

**Comparison:**
- Named: Easy to manage
- Anonymous: Hard to manage

---

## ✅ BÀI TẬP 3: Volume Management

**Commands:**
```bash
# List volumes
$ docker volume ls

# Inspect
$ docker volume inspect my-data

# Remove
$ docker volume rm my-data
```

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

