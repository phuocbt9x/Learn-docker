# Day-005: Image vs Container - Layers & Filesystem

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được sự khác biệt giữa Image và Container
- Hiểu được Image Layers và cách chúng hoạt động
- Hiểu được Container Filesystem (Copy-on-Write)
- Biết cách tối ưu image size bằng cách optimize layers
- Có thể debug image/container filesystem issues

---

## 📖 PHẦN 1: IMAGE LÀ GÌ?

### 1.1. Nó là gì?

**Image** là một **read-only template** dùng để tạo containers. Image chứa:

- **Application code**
- **Dependencies** (libraries, frameworks)
- **Runtime environment** (Python, Node.js, etc.)
- **System libraries**
- **Configuration files**
- **Metadata** (environment variables, exposed ports, etc.)

**Ví dụ:**
```
nginx:latest image
├── Base layer: Ubuntu 20.04
├── Layer 2: Install nginx
├── Layer 3: Copy config files
└── Layer 4: Set CMD ["nginx"]
```

**Đặc điểm:**
- **Read-only**: Không thể modify image
- **Immutable**: Một khi tạo, không thay đổi
- **Layered**: Image được tạo từ nhiều layers
- **Reusable**: Có thể dùng để tạo nhiều containers

### 1.2. Tại sao Image tồn tại?

**Vấn đề trước khi có Image:**

1. **Manual setup**: Phải setup environment mỗi lần
2. **Inconsistency**: Mỗi lần setup có thể khác nhau
3. **Time-consuming**: Mất nhiều thời gian
4. **No versioning**: Không thể track versions

**Image giải quyết:**
- **Automation**: Tự động setup environment
- **Consistency**: Cùng image = cùng environment
- **Speed**: Pull image nhanh hơn setup thủ công
- **Versioning**: Tags để track versions

### 1.3. Khi nào dùng Image?

**Use cases:**

1. **Create containers**: Image là template để tạo containers
2. **Share applications**: Push image lên registry, share với team
3. **Version control**: Tags để track versions
4. **CI/CD**: Build image một lần, dùng nhiều lần
5. **Reproducibility**: Cùng image = cùng behavior

### 1.4. Hậu quả nếu không hiểu Image?

**Hậu quả:**

1. **Large images**: Image quá lớn → pull chậm, tốn storage
2. **Inefficient builds**: Build không tối ưu → chậm
3. **Security risks**: Image có vulnerabilities → security issues
4. **Debug issues**: Không hiểu layers → khó debug

---

## 🐳 PHẦN 2: CONTAINER LÀ GÌ?

### 2.1. Nó là gì?

**Container** là một **running instance** của image. Container có:

- **Read-write layer**: Layer có thể write (Copy-on-Write)
- **Running process**: Application đang chạy
- **State**: Data, logs, changes
- **Network**: IP address, ports
- **Resources**: CPU, memory limits

**Ví dụ:**
```
nginx:latest image (read-only)
    ↓ docker run
Container (running instance)
├── Image layers (read-only)
└── Container layer (read-write)
    ├── Logs
    ├── Data
    └── Changes
```

**Đặc điểm:**
- **Writable**: Có thể modify (trong container layer)
- **Ephemeral**: Data mất khi container delete (trừ volumes)
- **Isolated**: Mỗi container có filesystem riêng
- **Stateful**: Có state (processes, data)

### 2.2. Tại sao Container tồn tại?

**Vấn đề:**

1. **Image là read-only**: Không thể chạy application trực tiếp
2. **Cần writable layer**: Application cần write logs, data
3. **Cần isolation**: Mỗi instance cần filesystem riêng

**Container giải quyết:**
- **Writable layer**: Copy-on-Write cho phép write
- **Isolation**: Mỗi container có filesystem riêng
- **State management**: Quản lý state của application

### 2.3. Khi nào dùng Container?

**Use cases:**

1. **Run applications**: Chạy applications từ images
2. **Development**: Test, debug applications
3. **Production**: Deploy applications
4. **Scaling**: Tạo nhiều containers từ một image

### 2.4. Hậu quả nếu không hiểu Container?

**Hậu quả:**

1. **Data loss**: Không hiểu ephemeral → mất data
2. **Performance issues**: Không hiểu Copy-on-Write → chậm
3. **Storage issues**: Không hiểu layers → tốn storage
4. **Debug issues**: Không hiểu filesystem → khó debug

---

## 📚 PHẦN 3: IMAGE LAYERS

### 3.1. Layers là gì?

**Layer** là một **read-only filesystem** chứa changes so với layer trước.

**Ví dụ:**
```dockerfile
FROM ubuntu:20.04          # Layer 1: Base image
RUN apt-get update         # Layer 2: Update packages
RUN apt-get install nginx  # Layer 3: Install nginx
COPY nginx.conf /etc/      # Layer 4: Copy config
CMD ["nginx"]              # Layer 5: Metadata
```

**Mỗi instruction tạo một layer:**
- `FROM`: Base layer
- `RUN`: Tạo layer với changes
- `COPY`: Tạo layer với files
- `CMD`: Metadata (không tạo layer)

**Image structure:**
```
Image: my-app:1.0
├── Layer 1 (ubuntu:20.04)
├── Layer 2 (apt-get update)
├── Layer 3 (apt-get install nginx)
├── Layer 4 (COPY nginx.conf)
└── Metadata (CMD, ENV, etc.)
```

### 3.2. Tại sao Layers tồn tại?

**Vấn đề:**

1. **Large images**: Image lớn → pull/push chậm
2. **Inefficient storage**: Duplicate data giữa images
3. **Slow builds**: Build lại toàn bộ mỗi lần

**Layers giải quyết:**
- **Deduplication**: Share layers giữa images
- **Caching**: Cache layers, build nhanh hơn
- **Efficiency**: Chỉ pull/push layers thay đổi

### 3.3. Layer Caching

**Cách hoạt động:**

1. **Build lần đầu:**
   ```dockerfile
   FROM ubuntu:20.04          # Pull base image
   RUN apt-get update         # Build layer 2
   RUN apt-get install nginx  # Build layer 3
   COPY nginx.conf /etc/      # Build layer 4
   ```

2. **Build lần 2 (chỉ thay đổi COPY):**
   ```dockerfile
   FROM ubuntu:20.04          # ✅ Cached
   RUN apt-get update         # ✅ Cached
   RUN apt-get install nginx  # ✅ Cached
   COPY nginx.conf /etc/      # ❌ Rebuild (changed)
   ```

**Kết quả:**
- Layers 1-3: **Cached** → không rebuild
- Layer 4: **Rebuild** → chỉ rebuild layer này
- **Time saved**: 70-80% build time

### 3.4. Layer Sharing

**Cách hoạt động:**

**Image A:**
```
Layer 1: ubuntu:20.04
Layer 2: apt-get update
Layer 3: install python
```

**Image B:**
```
Layer 1: ubuntu:20.04      # ← Share với Image A
Layer 2: apt-get update     # ← Share với Image A
Layer 3: install nodejs    # ← Khác Image A
```

**Storage:**
- Layer 1, 2: **Shared** → chỉ lưu 1 lần
- Layer 3: **Different** → lưu riêng
- **Storage saved**: 50-70%

### 3.5. Hậu quả nếu không hiểu Layers?

**Hậu quả:**

1. **Large images**: Nhiều layers không cần thiết → image lớn
2. **Slow builds**: Không tận dụng cache → build chậm
3. **Storage waste**: Duplicate layers → tốn storage
4. **Security risks**: Layers có vulnerabilities → security issues

---

## 💾 PHẦN 4: CONTAINER FILESYSTEM (COPY-ON-WRITE)

### 4.1. Copy-on-Write (CoW) là gì?

**Copy-on-Write** là một mechanism cho phép **share** read-only data và **copy** khi cần write.

**Ví dụ:**

**Image layers (read-only):**
```
Layer 1: /bin/bash
Layer 2: /usr/bin/python
Layer 3: /app/app.py
```

**Container 1:**
```
Image layers (read-only) ← Share
Container layer (read-write)
├── /app/data.txt (new file)
└── /app/logs/ (new directory)
```

**Container 2:**
```
Image layers (read-only) ← Share (same as Container 1)
Container layer (read-write)
├── /app/data.txt (different file)
└── /app/logs/ (different directory)
```

**Khi Container 1 modify file trong image:**
```
Container 1 modify /app/app.py
    ↓
Copy file từ image layer → container layer
    ↓
Modify file trong container layer
    ↓
Image layer vẫn read-only (unchanged)
```

### 4.2. Tại sao Copy-on-Write tồn tại?

**Vấn đề:**

1. **Storage waste**: Copy toàn bộ image cho mỗi container → tốn storage
2. **Slow startup**: Copy files mất thời gian
3. **Memory waste**: Duplicate data trong memory

**Copy-on-Write giải quyết:**
- **Share layers**: Containers share image layers
- **Copy only when needed**: Chỉ copy khi modify
- **Efficient**: Tiết kiệm storage, memory

### 4.3. Container Layer

**Container layer là gì?**

**Container layer** là một **thin read-write layer** trên cùng image layers.

**Chức năng:**
- **Store changes**: Lưu tất cả changes so với image
- **Isolation**: Mỗi container có layer riêng
- **Ephemeral**: Mất khi container delete

**Ví dụ:**
```
Container running
├── Image layers (read-only, shared)
│   ├── /bin/bash
│   ├── /usr/bin/python
│   └── /app/app.py
└── Container layer (read-write, unique)
    ├── /app/data.txt (new)
    ├── /app/logs/app.log (new)
    └── /app/app.py (modified - copied from image)
```

**Khi container delete:**
- Container layer **bị xóa**
- Image layers **vẫn còn** (shared với containers khác)

### 4.4. Storage Drivers

**Storage driver** quản lý cách Docker lưu image layers và container layers.

**Các storage drivers:**

1. **overlay2** (recommended):
   - Modern, efficient
   - Support nhiều layers
   - Good performance

2. **aufs** (legacy):
   - Old, deprecated
   - Limited layers
   - Slower

3. **devicemapper**:
   - Block-level
   - Good cho production
   - Complex setup

4. **btrfs, zfs**:
   - Advanced features
   - Complex setup
   - Limited use cases

**Recommendation: overlay2**

### 4.5. Hậu quả nếu không hiểu Copy-on-Write?

**Hậu quả:**

1. **Storage issues**: Không hiểu → nghĩ mỗi container copy toàn bộ image
2. **Performance issues**: Modify nhiều files → nhiều copies → chậm
3. **Data loss**: Không hiểu ephemeral → mất data khi container delete
4. **Debug issues**: Không hiểu filesystem → khó debug

---

## 🔄 PHẦN 5: IMAGE VS CONTAINER

### 5.1. So Sánh

| Tiêu chí | Image | Container |
|----------|-------|-----------|
| **Type** | Read-only template | Running instance |
| **State** | Immutable | Mutable (container layer) |
| **Storage** | Layers (shared) | Container layer (unique) |
| **Lifecycle** | Persistent | Ephemeral (trừ volumes) |
| **Usage** | Create containers | Run applications |
| **Modification** | Không thể | Có thể (container layer) |
| **Sharing** | Share layers | Isolated |

### 5.2. Relationship

**Image → Container:**

```
Image (read-only)
    ↓ docker run
Container (running)
├── Image layers (read-only, shared)
└── Container layer (read-write, unique)
```

**Multiple Containers từ một Image:**

```
Image: nginx:latest
    ↓
Container 1 (running)
├── Image layers (shared)
└── Container 1 layer (unique)

Container 2 (running)
├── Image layers (shared) ← Same as Container 1
└── Container 2 layer (unique) ← Different from Container 1
```

**Storage:**
- Image layers: **Shared** (chỉ lưu 1 lần)
- Container layers: **Unique** (mỗi container có layer riêng)

### 5.3. Lifecycle

**Image Lifecycle:**

```
Build → Tag → Push → Pull → Run
  ↓
Persistent (tồn tại cho đến khi delete)
```

**Container Lifecycle:**

```
Create → Start → Running → Stop → Delete
  ↓
Ephemeral (mất khi delete, trừ volumes)
```

**Ví dụ:**
```bash
# Image: Persistent
$ docker build -t my-app:1.0 .
$ docker push my-app:1.0
# Image tồn tại cho đến khi delete

# Container: Ephemeral
$ docker run my-app:1.0
$ docker stop <container-id>
$ docker rm <container-id>
# Container layer bị xóa, image vẫn còn
```

### 5.4. Khi Nào Dùng Gì?

**Dùng Image khi:**
- Build application
- Share với team
- Version control
- CI/CD

**Dùng Container khi:**
- Run application
- Development
- Production deployment
- Testing

---

## 🏭 PRODUCTION STORY #1: Image Quá Lớn - Pull Chậm

### Context

**Công ty:** SaaS platform, 400 employees
**Hệ thống:** 100 containers, deploy 10 lần/ngày
**Traffic:** 2M requests/day
**Team:** 25 DevOps engineers

### Problem

**Tháng 3/2023:**
- Image size: **2.5 GB**
- Pull time: **5-10 phút** mỗi lần deploy
- **Deployment time**: 15-20 phút (chủ yếu là pull image)
- **Customer complaints**: Slow deployments

**Root cause:**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update
RUN apt-get install -y \
    python3 \
    python3-pip \
    nodejs \
    npm \
    build-essential \
    git \
    vim \
    curl \
    wget \
    ... (50+ packages)
COPY . /app
RUN pip install -r requirements.txt
```

**Vấn đề:**
- **Nhiều layers không cần thiết**: Install nhiều packages không dùng
- **Không optimize layers**: Mỗi RUN tạo một layer
- **Copy trước khi install**: Copy code trước khi install dependencies → không cache được

### Investigation

**Timeline:**
- **Week 1**: Analyze image size
- **Week 2**: Identify unnecessary layers
- **Week 3**: Optimize Dockerfile
- **Week 4**: Test, deploy

**Findings:**
- **Base image**: 200MB (ubuntu:20.04)
- **Packages**: 1.5GB (nhiều packages không cần)
- **Dependencies**: 500MB (requirements.txt)
- **Application code**: 100MB
- **Build artifacts**: 200MB (không cần trong image)

### Fix

**Optimized Dockerfile:**
```dockerfile
# Multi-stage build
FROM python:3.9-slim as builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

FROM python:3.9-slim
WORKDIR /app
# Chỉ copy dependencies từ builder
COPY --from=builder /root/.local /root/.local
COPY . .
# Chỉ install packages cần thiết
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    curl && \
    rm -rf /var/lib/apt/lists/*
ENV PATH=/root/.local/bin:$PATH
CMD ["python", "app.py"]
```

**Optimizations:**
1. **Multi-stage build**: Tách build và runtime
2. **Slim base image**: python:3.9-slim thay vì ubuntu:20.04
3. **Chỉ install packages cần**: curl thay vì 50+ packages
4. **Layer optimization**: Combine RUN commands
5. **Remove build artifacts**: Không copy vào final image

### Result

**Trước:**
- Image size: **2.5 GB**
- Pull time: **5-10 phút**
- Deployment time: **15-20 phút**

**Sau:**
- Image size: **300 MB** (88% giảm)
- Pull time: **30-60 giây** (90% nhanh hơn)
- Deployment time: **3-5 phút** (75% nhanh hơn)

**Benefits:**
- **Faster deployments**: 75% nhanh hơn
- **Less bandwidth**: 88% ít hơn
- **Better UX**: Customers hài lòng hơn

### Lesson Learned

1. **Image size quan trọng**: Ảnh hưởng trực tiếp đến deployment time
2. **Multi-stage builds**: Giúp giảm image size đáng kể
3. **Optimize layers**: Combine commands, remove unnecessary layers
4. **Regular review**: Review image size thường xuyên

---

## 🏭 PRODUCTION STORY #2: Container Data Loss do không hiểu Ephemeral

### Context

**Công ty:** Analytics platform, 150 employees
**Hệ thống:** Data processing containers
**Traffic:** 100K jobs/day
**Team:** 10 backend engineers

### Problem

**Tháng 7/2023:**
- Containers process data và lưu kết quả vào `/app/results/`
- **Data mất** khi container restart
- **Root cause**: Không hiểu container filesystem là ephemeral

**Timeline:**
- **Day 1**: Container process data → lưu vào `/app/results/`
- **Day 2**: Container restart (update)
- **Day 3**: **Data mất!** → `/app/results/` trống

**Impact:**
- **100+ jobs** mất data
- **Customer complaints**: Data không có
- **Revenue loss**: Phải reprocess → tốn resources

### Investigation

**Root cause:**
```bash
# Container filesystem
/app/results/data.json  # ← Lưu trong container layer
# Khi container restart → container layer bị xóa → data mất
```

**Vấn đề:**
- **Không dùng volumes**: Data lưu trong container layer
- **Không hiểu ephemeral**: Nghĩ data persistent
- **No backup**: Không có backup strategy

### Fix

**Solution 1: Use Volumes**
```yaml
# docker-compose.yml
services:
  processor:
    image: my-app:1.0
    volumes:
      - results-data:/app/results  # ← Persistent volume

volumes:
  results-data:
```

**Solution 2: External Storage**
```dockerfile
# Lưu vào external storage (S3, database)
# Không lưu trong container filesystem
```

**Solution 3: Backup Strategy**
```bash
# Backup data trước khi restart
$ docker cp container:/app/results ./backup/
```

### Result

**Trước:**
- Data lưu trong container layer
- **Mất data** khi container restart
- **100+ jobs** affected

**Sau:**
- Data lưu trong volumes
- **Persistent** qua container restarts
- **Zero data loss** trong 6 tháng

### Lesson Learned

1. **Container filesystem là ephemeral**: Data mất khi container delete
2. **Dùng volumes cho persistent data**: Volumes persist qua container lifecycle
3. **Backup strategy**: Backup data quan trọng
4. **Documentation**: Document data storage strategy

---

## 🎓 TÓM TẮT

### Image

**Là gì:**
- Read-only template
- Layered structure
- Immutable

**Đặc điểm:**
- Persistent
- Shareable
- Versioned

**Tối ưu:**
- Multi-stage builds
- Optimize layers
- Slim base images

### Container

**Là gì:**
- Running instance của image
- Copy-on-Write filesystem
- Ephemeral (trừ volumes)

**Đặc điểm:**
- Writable (container layer)
- Isolated
- Stateful

**Lưu ý:**
- Data mất khi delete (trừ volumes)
- Share image layers
- Unique container layer

### Layers

**Là gì:**
- Read-only filesystem
- Mỗi instruction tạo một layer
- Shareable giữa images

**Lợi ích:**
- Caching
- Deduplication
- Efficiency

### Copy-on-Write

**Là gì:**
- Share read-only layers
- Copy khi cần write
- Efficient storage

**Lợi ích:**
- Tiết kiệm storage
- Fast startup
- Isolation

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ bạn đã hiểu:
- ✅ Image vs Container
- ✅ Layers và cách chúng hoạt động
- ✅ Copy-on-Write filesystem

**Day tiếp theo (Day-006)** sẽ đi sâu vào:
- Docker Installation & First Container
- Docker CLI basics
- Container lifecycle

---

## 📚 TÀI LIỆU THAM KHẢO

- Docker Image Specification: https://docs.docker.com/engine/reference/builder/
- Overlay Filesystem: https://www.kernel.org/doc/Documentation/filesystems/overlayfs.txt
- Copy-on-Write: https://en.wikipedia.org/wiki/Copy-on-write

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-006: Docker-Installation-va-First-Container](../Day-006-Docker-Installation-va-First-Container/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
