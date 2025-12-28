# Day-024: Bind Mounts vs Volumes - Production Use Cases

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được sự khác biệt giữa bind mounts và volumes
- Biết khi nào dùng bind mount, khi nào dùng volume
- Hiểu được production use cases
- Biết cách debug storage issues

---

## 📁 PHẦN 1: BIND MOUNTS

### 1.1. Bind Mount là gì?

**Bind Mount** là **mount host directory** trực tiếp vào container.

**Syntax:**
```bash
$ docker run -v /host/path:/container/path nginx
```

**Characteristics:**
- **Direct mapping**: Map trực tiếp host directory
- **Host dependency**: Phụ thuộc vào host filesystem
- **Performance**: Good performance
- **Portability**: Less portable

### 1.2. Bind Mount Use Cases

**Use cases:**
- **Development**: Development với live code reload
- **Configuration**: Mount config files
- **Logs**: Mount log directories
- **Host access**: Cần access host filesystem

---

## 💾 PHẦN 2: VOLUMES

### 2.1. Volume là gì?

**Volume** là **Docker-managed storage** độc lập với host filesystem.

**Syntax:**
```bash
$ docker run -v my-volume:/container/path nginx
```

**Characteristics:**
- **Managed**: Docker quản lý
- **Portable**: Portable across hosts
- **Isolated**: Isolated từ host
- **Backup**: Dễ backup

### 2.2. Volume Use Cases

**Use cases:**
- **Production**: Production data storage
- **Database**: Database data
- **Backup**: Backup và restore
- **Multi-container**: Share data giữa containers

---

## 🔄 PHẦN 3: BIND MOUNTS VS VOLUMES

### 3.1. So Sánh

| Tiêu chí | Bind Mount | Volume |
|----------|------------|--------|
| **Location** | Host path | Docker-managed |
| **Portability** | ❌ Host-dependent | ✅ Portable |
| **Performance** | ✅ Direct | ⚠️ Slight overhead |
| **Backup** | ⚠️ Manual | ✅ Easy |
| **Use case** | Development | Production |

### 3.2. Khi Nào Dùng Gì?

**Bind Mount:**
- **Development**: Development với live reload
- **Config files**: Mount config files
- **Host access**: Cần access host

**Volume:**
- **Production**: Production data
- **Database**: Database storage
- **Backup**: Cần backup

---

## 🏭 PRODUCTION STORY: Storage Choice

### Context

**Công ty:** SaaS, 600 employees
**Issue:** Data portability issues
**Root cause:** Bind mounts trong production

### Fix

**Solution: Volumes cho production**
```bash
$ docker volume create app-data
$ docker run -d -v app-data:/data app
```

**Results:**
- Data portable
- Easy backup
- Production-ready

---

## 🎓 TÓM TẮT

**Bind Mounts:**
- Host directory mapping
- Development use cases
- Less portable

**Volumes:**
- Docker-managed storage
- Production use cases
- Portable và manageable

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-025)** sẽ đi sâu vào:
- Volume Backup & Restore Strategies
- Data management

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-025: Volume-Backup-Restore](../Day-025-Volume-Backup-Restore/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
