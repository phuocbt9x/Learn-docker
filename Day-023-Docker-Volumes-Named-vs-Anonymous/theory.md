# Day-023: Docker Volumes - Named vs Anonymous

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được Docker volumes là gì
- Biết được sự khác biệt giữa named và anonymous volumes
- Hiểu được khi nào dùng volume nào
- Biết cách tạo và quản lý volumes
- Debug được volume issues

---

## 💾 PHẦN 1: DOCKER VOLUMES

### 1.1. Volume là gì?

**Docker Volume** là **persistent storage** cho containers, data **persist** sau khi container bị xóa.

**Characteristics:**
- **Persistent**: Data persist sau khi container xóa
- **Managed**: Docker quản lý volumes
- **Portable**: Volumes có thể share giữa containers
- **Backup**: Dễ dàng backup và restore

### 1.2. Named Volumes

**Create named volume:**
```bash
$ docker volume create my-volume
```

**Use named volume:**
```bash
$ docker run -d -v my-volume:/data nginx
```

**Benefits:**
- **Named**: Có tên, dễ quản lý
- **Reusable**: Có thể reuse
- **Manageable**: Dễ manage và backup

### 1.3. Anonymous Volumes

**Create anonymous volume:**
```bash
$ docker run -d -v /data nginx
# Anonymous volume, tên tự động generate
```

**Characteristics:**
- **No name**: Không có tên
- **Auto-generated**: Tên tự động generate
- **Hard to manage**: Khó quản lý

---

## 🔄 PHẦN 2: NAMED VS ANONYMOUS

### 2.1. So Sánh

| Tiêu chí | Named | Anonymous |
|----------|-------|-----------|
| **Name** | ✅ Có tên | ❌ Không có tên |
| **Manageable** | ✅ Dễ quản lý | ❌ Khó quản lý |
| **Reusable** | ✅ Có thể reuse | ⚠️ Khó reuse |
| **Backup** | ✅ Dễ backup | ❌ Khó backup |

### 2.2. Khi Nào Dùng Gì?

**Named Volumes (Recommended):**
- **Production**: Production use cases
- **Data persistence**: Cần persist data
- **Backup**: Cần backup data
- **Multi-container**: Share data giữa containers

**Anonymous Volumes:**
- **Testing**: Testing scenarios
- **Temporary**: Temporary data
- **Quick setup**: Quick setup

---

## 🏭 PRODUCTION STORY: Data Loss

### Context

**Công ty:** E-commerce, 800 employees
**Issue:** Data loss khi container bị xóa
**Root cause:** Anonymous volumes

### Fix

**Solution: Named volumes**
```bash
$ docker volume create db-data
$ docker run -d -v db-data:/var/lib/postgresql/data postgres
```

**Results:**
- Data persist
- Easy backup
- No data loss

---

## 🎓 TÓM TẮT

**Volumes:**
- Persistent storage
- Data persist sau container deletion
- Managed by Docker

**Named vs Anonymous:**
- Named: Recommended cho production
- Anonymous: Testing, temporary

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-024)** sẽ đi sâu vào:
- Bind Mounts vs Volumes
- Production use cases

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-024: Bind-Mounts-vs-Volumes](../Day-024-Bind-Mounts-vs-Volumes/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
