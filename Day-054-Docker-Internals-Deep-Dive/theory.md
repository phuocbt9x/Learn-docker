# Day-054: Docker Internals - Deep Dive

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu sâu về Docker internals
- Biết được Docker architecture chi tiết
- Hiểu được storage drivers
- Biết được execution drivers
- Hiểu được network drivers
- Debug như senior engineer

---

## 🏗️ PHẦN 1: DOCKER ARCHITECTURE

### 1.1. Docker Components

**Main components:**
- **Docker Client**: CLI interface
- **Docker Daemon**: Background service
- **containerd**: Container runtime
- **runc**: OCI runtime
- **Storage drivers**: Manage storage
- **Network drivers**: Manage networking

### 1.2. Request Flow

**Flow:**
1. **Client**: User runs `docker run`
2. **Daemon**: Daemon receives request
3. **containerd**: containerd manages container
4. **runc**: runc creates container
5. **Container**: Container runs

### 1.3. Storage Drivers

**Storage drivers:**
- **overlay2**: Default, recommended
- **devicemapper**: Legacy
- **aufs**: Legacy
- **btrfs**: Btrfs filesystem

**Choose driver:**
- **overlay2**: Recommended cho most cases
- **Performance**: overlay2 tốt nhất
- **Compatibility**: overlay2 compatible nhất

---

## 🔍 PHẦN 2: DEEP DIVE

### 2.1. Image Storage

**Image storage:**
- **Layers**: Stored as layers
- **Metadata**: Stored separately
- **Cache**: Layer cache

**Location:**
- `/var/lib/docker/`: Default storage location
- **overlay2**: `/var/lib/docker/overlay2/`

### 2.2. Container Storage

**Container storage:**
- **Copy-on-Write**: CoW filesystem
- **Container layer**: Writable layer
- **Image layers**: Read-only layers

### 2.3. Network Storage

**Network storage:**
- **Network configs**: Stored in Docker
- **IPAM**: IP Address Management
- **Bridge networks**: Bridge configs

---

## 🏭 PRODUCTION STORY: Storage Driver Issues

### Context

**Công ty:** E-commerce, 800 employees
**Issue:** Slow container operations
**Root cause:** Storage driver

### Fix

**Solution: overlay2**
- Switch to overlay2
- Better performance
- Production-ready

**Results:**
- Performance improved
- Operations faster
- Stability improved

---

## 🎓 TÓM TẮT

**Docker internals:**
- Architecture: Client, Daemon, containerd, runc
- Storage: overlay2 recommended
- Network: Bridge, custom networks

**Deep dive:**
- Image storage
- Container storage
- Network storage

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-055)** sẽ đi sâu vào:
- Production Architecture Patterns
- Senior-level patterns

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-055: Production-Architecture-Patterns](../Day-055-Production-Architecture-Patterns/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
