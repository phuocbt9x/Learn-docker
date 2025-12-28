# Day-005: Giải Pháp & Giải Thích

## 🎯 LƯU Ý QUAN TRỌNG

Các giải pháp dưới đây giải thích Image vs Container, Layers, và Copy-on-Write ở mức sâu. Quan trọng là bạn hiểu được:

- **Tại sao** Image và Container khác nhau
- **Cách** Layers hoạt động và được cache
- **Cách** Copy-on-Write tiết kiệm storage
- **Khi nào** cần optimize và làm thế nào

---

## 📝 BÀI TẬP 1: HIỂU IMAGE VS CONTAINER

### 1.1. Sự Khác Biệt

**Image là gì?**

**Image** là một **read-only template** dùng để tạo containers:
- Chứa application code, dependencies, runtime
- **Immutable**: Không thể modify sau khi tạo
- **Layered**: Được tạo từ nhiều layers
- **Persistent**: Tồn tại cho đến khi delete

**Container là gì?**

**Container** là một **running instance** của image:
- Chạy application từ image
- **Mutable**: Có thể modify (trong container layer)
- **Ephemeral**: Data mất khi delete (trừ volumes)
- **Stateful**: Có state (processes, data)

**Mối quan hệ:**

```
Image (template)
    ↓ docker run
Container (instance)
├── Image layers (read-only, shared)
└── Container layer (read-write, unique)
```

**Khi nào dùng Image?**

- **Build application**: Tạo image từ Dockerfile
- **Share với team**: Push image lên registry
- **Version control**: Tags để track versions
- **CI/CD**: Build image một lần, dùng nhiều lần

**Khi nào dùng Container?**

- **Run application**: Chạy application từ image
- **Development**: Test, debug
- **Production**: Deploy applications
- **Scaling**: Tạo nhiều containers từ một image

### 1.2. Diagram

```
Image: my-app:1.0
├── Layer 1: ubuntu:20.04 (200MB) ← Base
├── Layer 2: install python (50MB)
└── Layer 3: copy app (10MB)
Total: 260MB

Container 1 (running)
├── Image layers (shared) ← 260MB (shared với Container 2)
│   ├── Layer 1: ubuntu:20.04
│   ├── Layer 2: install python
│   └── Layer 3: copy app
└── Container 1 layer (unique) ← 5MB (chỉ Container 1)
    ├── /app/logs/app.log
    └── /app/data/temp.json

Container 2 (running)
├── Image layers (shared) ← 260MB (shared với Container 1)
│   ├── Layer 1: ubuntu:20.04
│   ├── Layer 2: install python
│   └── Layer 3: copy app
└── Container 2 layer (unique) ← 3MB (chỉ Container 2)
    └── /app/logs/app.log

Total Storage:
- Image: 260MB
- Container 1: 5MB (unique)
- Container 2: 3MB (unique)
- Total: 268MB (không phải 260MB × 2 = 520MB!)
```

**Key Points:**
- **Image layers**: Shared giữa tất cả containers
- **Container layers**: Unique cho mỗi container
- **Storage efficient**: Chỉ lưu image layers 1 lần

### 1.3. Lifecycle

**Image Lifecycle:**

```
Build → Tag → Push → Pull → Run
  ↓
Persistent (tồn tại cho đến khi delete)
```

**Chi tiết:**
1. **Build**: `docker build -t my-app:1.0 .`
2. **Tag**: `docker tag my-app:1.0 registry/my-app:1.0`
3. **Push**: `docker push registry/my-app:1.0`
4. **Pull**: `docker pull registry/my-app:1.0`
5. **Run**: `docker run my-app:1.0` (tạo container)

**Container Lifecycle:**

```
Create → Start → Running → Stop → Delete
  ↓
Ephemeral (mất khi delete, trừ volumes)
```

**Chi tiết:**
1. **Create**: `docker create my-app:1.0`
2. **Start**: `docker start <container-id>`
3. **Running**: Container đang chạy
4. **Stop**: `docker stop <container-id>`
5. **Delete**: `docker rm <container-id>`

**Khi container delete:**

- **Container layer**: **Bị xóa** (mất data, logs, changes)
- **Image layers**: **Vẫn còn** (shared với containers khác)
- **Image**: **Không bị ảnh hưởng** (vẫn tồn tại)

**Ví dụ:**
```bash
# Tạo container
$ docker run -d my-app:1.0
container-id-1

# Container tạo file
$ docker exec container-id-1 touch /app/data.txt

# Delete container
$ docker rm -f container-id-1

# Image vẫn còn
$ docker images my-app:1.0
REPOSITORY   TAG   IMAGE ID
my-app       1.0   abc123...

# Data mất (không có volumes)
# File /app/data.txt không còn
```

### 1.4. 10 Containers từ Cùng Image

**Storage Usage:**

**Image:**
- Image size: 500MB (5 layers)
- **Storage: 500MB** (chỉ lưu 1 lần, shared)

**Containers:**
- **Nếu không modify gì**: Mỗi container ~1-5MB (metadata)
- **Total containers**: 10 × 2MB = 20MB

**Total Storage:**
- Image: 500MB
- Containers: 20MB
- **Total: 520MB** (không phải 500MB × 10 = 5GB!)

**Nếu một container modify file trong image:**

**Scenario:**
- Container 1 modify `/app/config.json` (file trong image layer 3)

**Điều gì xảy ra:**

1. **Copy-on-Write:**
   - File được copy từ image layer → container layer
   - Modify file trong container layer
   - Image layer vẫn read-only (unchanged)

2. **Storage:**
   - Container 1 layer: +10MB (file được copy)
   - Image layers: Không thay đổi
   - Containers khác: Không thay đổi

3. **Isolation:**
   - Container 1 thấy file đã modify
   - Containers 2-10 thấy file gốc (từ image)
   - **Hoàn toàn isolated**

**Containers khác có bị ảnh hưởng không?**

**❌ KHÔNG:**
- Container 1 modify file → chỉ affect Container 1
- Containers 2-10 vẫn thấy file gốc từ image
- **Copy-on-Write đảm bảo isolation**

---

## 📝 BÀI TẬP 2: IMAGE LAYERS

### 2.1. Phân Tích Layers

**Dockerfile:**
```dockerfile
FROM ubuntu:20.04                    # Layer 1
RUN apt-get update                   # Layer 2
RUN apt-get install -y python3 python3-pip  # Layer 3
COPY requirements.txt /app/          # Layer 4
RUN pip3 install -r /app/requirements.txt  # Layer 5
COPY . /app/                         # Layer 6
CMD ["python3", "/app/app.py"]       # Metadata (không tạo layer)
```

**Layers:**

1. **Layer 1 (FROM ubuntu:20.04)**: ~200MB
   - Base Ubuntu image
   - System libraries, binaries

2. **Layer 2 (RUN apt-get update)**: ~10MB
   - Updated package lists
   - Metadata

3. **Layer 3 (RUN apt-get install)**: ~100MB
   - Python 3, pip
   - Dependencies

4. **Layer 4 (COPY requirements.txt)**: ~1MB
   - requirements.txt file

5. **Layer 5 (RUN pip3 install)**: ~200MB
   - Python packages từ requirements.txt

6. **Layer 6 (COPY . /app/)**: ~10MB
   - Application code

**Total: ~521MB**

### 2.2. Layer Caching

**Build lần đầu:**
- **Tất cả layers** được build (không có cache)
- Time: ~10 phút

**Build lần 2 (không thay đổi gì):**
- **Tất cả layers** được cache
- Time: ~10 giây (chỉ verify)

**Build lần 3 (chỉ thay đổi app.py):**

**Cache analysis:**
- Layer 1: ✅ Cached (FROM không đổi)
- Layer 2: ✅ Cached (RUN apt-get update không đổi)
- Layer 3: ✅ Cached (RUN apt-get install không đổi)
- Layer 4: ✅ Cached (requirements.txt không đổi)
- Layer 5: ✅ Cached (pip install không đổi)
- Layer 6: ❌ **Rebuild** (COPY . /app/ thay đổi vì app.py thay đổi)

**Time:**
- Layers 1-5: Cached (~10 giây)
- Layer 6: Rebuild (~30 giây)
- **Total: ~40 giây** (thay vì 10 phút)

### 2.3. Optimize Dockerfile

**Vấn đề hiện tại:**
- COPY . /app/ đặt trước RUN pip install
- Mỗi lần code thay đổi → rebuild layer 5 (pip install)
- **Không tận dụng cache**

**Optimized Dockerfile:**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    python3 python3-pip && \
    rm -rf /var/lib/apt/lists/*

# Copy requirements trước (ít thay đổi)
COPY requirements.txt /app/
RUN pip3 install --no-cache-dir -r /app/requirements.txt

# Copy code sau (thay đổi thường xuyên)
COPY . /app/

CMD ["python3", "/app/app.py"]
```

**Optimizations:**
1. **Combine RUN commands**: Giảm số layers
2. **COPY requirements trước code**: Cache pip install layer
3. **Remove apt cache**: Giảm image size
4. **--no-install-recommends**: Chỉ install packages cần

**Cache behavior:**
- Thay đổi code → chỉ rebuild layer COPY . /app/
- Thay đổi requirements.txt → rebuild từ layer COPY requirements.txt
- **Maximize cache hits**

### 2.4. Layer Sharing

**Image A:**
```
Layer 1: ubuntu:20.04 (200MB)
Layer 2: apt-get update (10MB)
Layer 3: install python3 (100MB)
```

**Image B:**
```
Layer 1: ubuntu:20.04 (200MB)      # ← Share với Image A
Layer 2: apt-get update (10MB)     # ← Share với Image A
Layer 3: install nodejs (150MB)   # ← Khác Image A
```

**Storage:**

**Without sharing:**
- Image A: 310MB
- Image B: 360MB
- **Total: 670MB**

**With sharing:**
- Layer 1: 200MB (shared)
- Layer 2: 10MB (shared)
- Layer 3 (python): 100MB (Image A only)
- Layer 3 (nodejs): 150MB (Image B only)
- **Total: 460MB**

**Storage saved: 210MB (31%)**

---

## 📝 BÀI TẬP 3: COPY-ON-WRITE

### 3.1. Giải Thích Copy-on-Write

**Copy-on-Write (CoW) là gì?**

**CoW** là một mechanism:
- **Share** read-only data (image layers)
- **Copy** khi cần write
- **Efficient**: Tiết kiệm storage, memory

**Cách hoạt động:**

1. **Initial state:**
   ```
   Image layers (read-only)
   └── /app/config.json
   
   Container 1, 2, 3 (read image layers)
   └── All see /app/config.json from image
   ```

2. **Container 1 modify file:**
   ```
   Container 1 modify /app/config.json
       ↓
   Copy file từ image layer → container 1 layer
       ↓
   Modify file trong container 1 layer
       ↓
   Image layer vẫn read-only (unchanged)
   ```

3. **Result:**
   ```
   Image layer: /app/config.json (original, read-only)
   Container 1 layer: /app/config.json (modified, read-write)
   Container 2, 3: Still see original from image
   ```

**Tại sao CoW efficient?**

- **Storage**: Chỉ copy khi cần (không copy toàn bộ image)
- **Memory**: Share memory pages (nếu không modify)
- **Speed**: Fast startup (không cần copy files)

### 3.2. Container 1 Modify File

**Scenario:**
- File `/app/config.json` trong image layer 3 (read-only)
- Container 1 modify file này

**Điều gì xảy ra:**

1. **Copy file:**
   - File được copy từ image layer 3 → container 1 layer
   - **Storage**: +10MB (file size)

2. **Modify file:**
   - Modify file trong container 1 layer
   - Image layer 3 vẫn read-only (unchanged)

3. **Result:**
   ```
   Image layer 3: /app/config.json (original, unchanged)
   Container 1 layer: /app/config.json (modified)
   ```

**Containers 2, 3 có bị ảnh hưởng không?**

**❌ KHÔNG:**
- Containers 2, 3 vẫn thấy file gốc từ image layer 3
- **Isolation hoàn toàn**
- Container 1's changes không affect containers khác

### 3.3. Container 2 Tạo File Mới

**Scenario:**
- Container 2 tạo file `/app/data.txt`

**File được lưu ở đâu?**

**Container 2 layer:**
- File được lưu trong **container 2 layer** (read-write)
- **Không** trong image layers (read-only)

**Containers 1, 3 có thấy file không?**

**❌ KHÔNG:**
- File chỉ có trong container 2 layer
- Containers 1, 3 có filesystem riêng
- **Isolation hoàn toàn**

**Ví dụ:**
```bash
# Container 2
$ docker exec container-2 touch /app/data.txt

# Container 1
$ docker exec container-1 ls /app/data.txt
# ❌ File not found (không thấy)

# Container 3
$ docker exec container-3 ls /app/data.txt
# ❌ File not found (không thấy)
```

### 3.4. Storage Analysis

**Given:**
- Image: 500MB (3 layers)
- Container 1: Modify 10 files (mỗi file 1MB)
- Container 2: Tạo 5 files mới (mỗi file 2MB)
- Container 3: Không modify gì

**Calculation:**

**Image:**
- 500MB (shared giữa tất cả containers)

**Container 1:**
- Modify 10 files × 1MB = 10MB (copied to container layer)

**Container 2:**
- Tạo 5 files × 2MB = 10MB (in container layer)

**Container 3:**
- 0MB (không modify gì, chỉ metadata ~1MB)

**Total Storage:**
- Image: 500MB
- Container 1: 10MB
- Container 2: 10MB
- Container 3: 1MB
- **Total: 521MB**

**So sánh với nếu không có CoW:**
- Mỗi container copy toàn bộ image: 500MB × 3 = 1500MB
- **CoW tiết kiệm: 979MB (65%)**

---

## 📝 BÀI TẬP 4: TỐI ƯU IMAGE SIZE

### 4.1. Phân Tích Image Size

**Dockerfile:**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update
RUN apt-get install -y python3 python3-pip nodejs npm build-essential git vim curl wget htop net-tools
COPY . /app
RUN pip3 install -r requirements.txt
RUN npm install
RUN npm run build
CMD ["python3", "app.py"]
```

**Phân tích:**

1. **Base image (ubuntu:20.04)**: ~200MB
2. **Packages**: ~800MB
   - python3, python3-pip: ~50MB
   - nodejs, npm: ~100MB
   - build-essential: ~200MB
   - git, vim, curl, wget, htop, net-tools: ~450MB
3. **Dependencies (requirements.txt)**: ~500MB
4. **Dependencies (npm)**: ~300MB
5. **Build artifacts (npm run build)**: ~200MB
6. **Application code**: ~100MB

**Total: ~2.5GB**

### 4.2. Tối Ưu Dockerfile

**Optimized Dockerfile (Multi-stage):**
```dockerfile
# Stage 1: Build
FROM node:16-slim as builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Stage 2: Python dependencies
FROM python:3.9-slim as python-deps
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --user -r requirements.txt

# Stage 3: Runtime
FROM python:3.9-slim
WORKDIR /app

# Copy Python dependencies
COPY --from=python-deps /root/.local /root/.local

# Copy built assets from builder (không copy source, node_modules)
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/public ./public

# Copy Python app
COPY . .

# Chỉ install packages cần thiết cho runtime
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    curl && \
    rm -rf /var/lib/apt/lists/*

ENV PATH=/root/.local/bin:$PATH
CMD ["python3", "app.py"]
```

**Optimizations:**

1. **Multi-stage build**: Tách build và runtime
2. **Slim base images**: python:3.9-slim, node:16-slim
3. **Chỉ install packages cần**: curl thay vì 10+ packages
4. **Remove build artifacts**: Không copy node_modules, source code
5. **--no-install-recommends**: Chỉ install packages cần
6. **Remove apt cache**: `rm -rf /var/lib/apt/lists/*`
7. **npm ci**: Faster, more reliable
8. **--no-cache-dir**: Không cache pip packages

### 4.3. Estimate Image Size

**After optimization:**

1. **Base image (python:3.9-slim)**: ~120MB (thay vì ubuntu:20.04 200MB)
2. **Packages (chỉ curl)**: ~5MB (thay vì 800MB)
3. **Python dependencies**: ~300MB (thay vì 500MB, remove cache)
4. **Built assets (dist, public)**: ~50MB (thay vì 200MB build artifacts)
5. **Application code**: ~100MB

**Total: ~575MB**

**Giảm: 2.5GB → 575MB = 77% giảm**

### 4.4. Trade-offs

**Lợi ích của image nhỏ:**

1. **Faster pull/push**: 77% nhanh hơn
2. **Less bandwidth**: Tiết kiệm bandwidth
3. **Less storage**: Tiết kiệm storage
4. **Faster deployments**: Deploy nhanh hơn

**Rủi ro của image quá nhỏ:**

1. **Missing packages**: Thiếu packages cần thiết
2. **Debugging**: Khó debug (thiếu tools)
3. **Compatibility**: Có thể có compatibility issues

**Best Practices:**

1. **Start với slim image**: Dùng slim base images
2. **Add packages khi cần**: Chỉ install packages thực sự cần
3. **Test kỹ**: Test để đảm bảo không thiếu gì
4. **Document**: Document packages cần thiết
5. **Review regularly**: Review image size thường xuyên

---

## 📝 BÀI TẬP 5: CONTAINER DATA LOSS

### 5.1. Phân Tích Vấn Đề

**Tại sao data mất khi container restart?**

**Root cause:**
- Data được lưu trong **container layer** (ephemeral)
- Container layer **bị xóa** khi container delete
- **Không persistent** qua container lifecycle

**Data được lưu ở đâu?**

**Container layer:**
```
Container
├── Image layers (read-only)
└── Container layer (read-write, ephemeral)
    └── /app/data/results.json  # ← Lưu ở đây
```

**Khi container restart:**
- Container layer **bị xóa**
- Data **mất**

**Tại sao data không persistent?**

- Container filesystem là **ephemeral** by design
- Data chỉ persist nếu dùng **volumes** hoặc **bind mounts**

### 5.2. Giải Pháp

**Solution 1: Volumes**

**docker-compose.yml:**
```yaml
services:
  app:
    image: my-app:1.0
    volumes:
      - app-data:/app/data  # Named volume

volumes:
  app-data:  # Docker quản lý
```

**docker run:**
```bash
$ docker run -v app-data:/app/data my-app:1.0
```

**Solution 2: Bind Mounts**

**docker-compose.yml:**
```yaml
services:
  app:
    image: my-app:1.0
    volumes:
      - ./data:/app/data  # Bind mount (host path)
```

**docker run:**
```bash
$ docker run -v $(pwd)/data:/app/data my-app:1.0
```

**So sánh Volumes vs Bind Mounts:**

| Feature | Volumes | Bind Mounts |
|---------|---------|-------------|
| **Managed by** | Docker | Host filesystem |
| **Location** | Docker directory | Host path |
| **Portability** | ✅ Portable | ❌ Path-dependent |
| **Performance** | ✅ Better (Linux) | ⚠️ Depends |
| **Backup** | ✅ Easy | ⚠️ Manual |

**Recommendation:**
- **Volumes**: Production, portable
- **Bind Mounts**: Development, specific paths

### 5.3. Implementation

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  app:
    image: my-app:1.0
    volumes:
      - app-data:/app/data
    restart: unless-stopped

volumes:
  app-data:
    driver: local
```

**Verify:**
```bash
# Start container
$ docker-compose up -d

# Create data
$ docker-compose exec app touch /app/data/test.txt

# Restart container
$ docker-compose restart app

# Verify data still exists
$ docker-compose exec app ls /app/data/test.txt
# ✅ File exists!
```

### 5.4. Best Practices

**Khi nào dùng volumes:**
- Production data
- Database data
- Application logs (nếu cần persistent)
- Shared data giữa containers

**Khi nào dùng bind mounts:**
- Development (mount source code)
- Configuration files
- Specific host paths

**Backup Strategy:**
```bash
# Backup volume
$ docker run --rm -v app-data:/data -v $(pwd):/backup \
  ubuntu tar czf /backup/app-data-backup.tar.gz /data

# Restore volume
$ docker run --rm -v app-data:/data -v $(pwd):/backup \
  ubuntu tar xzf /backup/app-data-backup.tar.gz -C /
```

**Data Migration:**
- Export data từ container
- Import vào volume
- Verify data integrity

---

## 📝 BÀI TẬP 6: LAYER CACHING OPTIMIZATION

### 6.1. Phân Tích Vấn Đề

**Dockerfile:**
```dockerfile
FROM ubuntu:20.04
COPY . /app              # ← Vấn đề: Copy trước
RUN apt-get update
RUN apt-get install -y python3 python3-pip
RUN pip3 install -r /app/requirements.txt
RUN python3 /app/setup.py
CMD ["python3", "/app/app.py"]
```

**Vấn đề:**

1. **COPY . /app đặt đầu**: Mỗi lần code thay đổi → invalidate tất cả layers sau
2. **Không cache được**: Layers sau COPY không được cache
3. **Rebuild toàn bộ**: Mỗi lần code thay đổi → rebuild từ layer 2

**Tại sao không cache được?**

- Docker cache dựa trên **layer content**
- Nếu layer trước thay đổi → layers sau **không cache được**
- COPY . /app thay đổi → tất cả layers sau invalidate

### 6.2. Optimize Layer Order

**Optimized Dockerfile:**
```dockerfile
FROM ubuntu:20.04

# Install system packages (ít thay đổi)
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    python3 python3-pip && \
    rm -rf /var/lib/apt/lists/*

# Copy requirements trước (ít thay đổi)
COPY requirements.txt /app/
RUN pip3 install --no-cache-dir -r /app/requirements.txt

# Copy code sau (thay đổi thường xuyên)
COPY . /app/
RUN python3 /app/setup.py

CMD ["python3", "/app/app.py"]
```

**Optimizations:**

1. **Combine RUN commands**: Giảm số layers
2. **COPY requirements trước code**: Cache pip install
3. **Remove apt cache**: Giảm image size
4. **--no-install-recommends**: Chỉ install packages cần

**Cache behavior:**

- **Thay đổi code**: Chỉ rebuild từ COPY . /app/
- **Thay đổi requirements.txt**: Rebuild từ COPY requirements.txt
- **Không thay đổi gì**: Tất cả cached

### 6.3. Estimate Build Time

**Before optimization:**

- Build lần đầu: 10 phút
- Build lần 2 (không thay đổi): 10 phút (không cache được)
- Build lần 3 (chỉ thay đổi code): 10 phút (rebuild toàn bộ)

**After optimization:**

- Build lần đầu: 10 phút
- Build lần 2 (không thay đổi): **30 giây** (cached)
- Build lần 3 (chỉ thay đổi code): **2 phút** (chỉ rebuild từ COPY . /app/)

**Time saved: 80-95%**

### 6.4. Advanced Optimization

**Multi-stage build:**
```dockerfile
# Stage 1: Build
FROM python:3.9-slim as builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --user -r requirements.txt

# Stage 2: Runtime
FROM python:3.9-slim
WORKDIR /app
COPY --from=builder /root/.local /root/.local
COPY . .
ENV PATH=/root/.local/bin:$PATH
CMD ["python3", "app.py"]
```

**.dockerignore:**
```
node_modules
.git
*.log
.env
dist
build
```

**Layer combination:**
```dockerfile
# Bad: Nhiều layers
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get clean

# Good: Combine
RUN apt-get update && \
    apt-get install -y python3 && \
    apt-get clean
```

**Best Practices:**

1. **Order matters**: Copy dependencies trước code
2. **Combine commands**: Giảm số layers
3. **Use .dockerignore**: Exclude files không cần
4. **Multi-stage builds**: Remove build artifacts
5. **Regular review**: Review và optimize thường xuyên

---

## 📝 BÀI TẬP 7: DEBUG FILESYSTEM ISSUES

### 7.1. Phân Tích Vấn Đề

**Vấn đề:**
- Container không thấy file `/app/config.json`
- Nhưng file có trong image

**Root cause analysis:**

1. **File có trong image không?**
   ```bash
   $ docker run --rm my-app:1.0 ls /app/config.json
   /app/config.json  # ✅ Có trong image
   ```

2. **Container layer có override không?**
   ```bash
   $ docker diff container-id
   D /app/config.json  # ← File bị delete trong container layer!
   ```

3. **Volume mount có override không?**
   ```bash
   $ docker inspect container-id | grep Mounts
   # Check volumes
   ```

**Kết luận:**
- File bị **delete** trong container layer
- Hoặc bị **override** bởi volume mount

### 7.2. Debug Steps

**Step 1: Check image layers**
```bash
$ docker history my-app:1.0
# Xem layers, tìm layer có config.json
```

**Step 2: Check container layer**
```bash
$ docker diff container-id
# Xem changes trong container layer
```

**Step 3: Check file permissions**
```bash
$ docker exec container-id ls -la /app/config.json
# Check permissions
```

**Step 4: Check mount points**
```bash
$ docker inspect container-id | grep -A 10 Mounts
# Check volumes, bind mounts
```

### 7.3. Common Issues

**Issue 1: File bị delete trong container layer**
- **Cause**: Application hoặc script delete file
- **Fix**: Restart container (recreate container layer)

**Issue 2: File bị override bởi volume**
- **Cause**: Volume mount override file
- **Fix**: Check volume mount, adjust path

**Issue 3: Permission issues**
- **Cause**: File permissions không đúng
- **Fix**: Fix permissions trong Dockerfile

**Issue 4: Namespace issues**
- **Cause**: Mount namespace không đúng
- **Fix**: Check namespace configuration

### 7.4. Solutions

**Fix file bị delete:**
```bash
# Restart container (recreate container layer)
$ docker restart container-id

# Hoặc recreate container
$ docker rm container-id
$ docker run my-app:1.0
```

**Fix volume override:**
```yaml
# docker-compose.yml
volumes:
  - ./config:/app/config  # ← Override /app/config.json
  # Fix: Mount to different path
  - ./config:/app/config-external
```

**Prevent trong tương lai:**
- **Read-only root filesystem**: Prevent modifications
- **Document**: Document file locations
- **Test**: Test file access
- **Monitoring**: Monitor file changes

---

## 📝 BÀI TẬP 8: IMAGE VS CONTAINER STORAGE

### 8.1. Tính Toán Storage

**Given:**
- 1 Image: `my-app:1.0` (500MB, 5 layers)
- 10 Containers chạy từ image đó

**Calculation:**

**Image storage:**
- Image: 500MB (5 layers)
- **Storage: 500MB** (chỉ lưu 1 lần, shared)

**Container storage (nếu không modify gì):**
- Mỗi container: ~2MB (metadata, container layer empty)
- 10 containers: 10 × 2MB = 20MB

**Total storage:**
- Image: 500MB
- Containers: 20MB
- **Total: 520MB**

**So sánh với nếu mỗi container copy toàn bộ image:**
- 10 containers × 500MB = 5000MB
- **CoW tiết kiệm: 4480MB (90%)**

### 8.2. Scenario: 5 Containers Modify Files

**Given:**
- 5 containers modify 1 file mỗi container (10MB mỗi file)

**Calculation:**

**Image:**
- 500MB (unchanged, shared)

**Containers:**
- 5 containers modify files: 5 × 10MB = 50MB
- 5 containers không modify: 5 × 2MB = 10MB

**Total storage:**
- Image: 500MB
- Containers: 60MB
- **Total: 560MB**

**So sánh với nếu mỗi container copy toàn bộ image:**
- 10 containers × 500MB = 5000MB
- **CoW tiết kiệm: 4440MB (89%)**

### 8.3. Scenario: Container Delete

**Given:**
- Delete 1 container (đã modify 1 file 10MB)

**Storage freed:**
- Container layer: 10MB (file modified) + 2MB (metadata) = 12MB
- **Freed: 12MB**

**Image storage:**
- **Không thay đổi**: 500MB (vẫn còn, shared với containers khác)

**Containers còn lại:**
- 9 containers vẫn chạy
- Image vẫn shared với 9 containers

### 8.4. Storage Optimization

**Image optimization:**
- Multi-stage builds
- Slim base images
- Remove unnecessary layers
- Optimize layer order

**Container cleanup:**
```bash
# Remove stopped containers
$ docker container prune

# Remove unused images
$ docker image prune

# Remove unused volumes
$ docker volume prune

# Remove everything unused
$ docker system prune -a
```

**Best practices:**
- **Regular cleanup**: Clean unused containers, images
- **Monitor storage**: Track storage usage
- **Optimize images**: Giảm image size
- **Use volumes**: Persistent data ngoài container layer

---

## ✅ TỔNG KẾT

Các bài tập này giúp bạn:

1. **Hiểu Image vs Container**: Template vs instance
2. **Hiểu Layers**: Caching, sharing, optimization
3. **Hiểu Copy-on-Write**: Efficient storage
4. **Optimize images**: Giảm size, tăng cache hits
5. **Debug filesystem**: Troubleshoot issues

**Key takeaways:**
- **Image**: Read-only template, layered, persistent
- **Container**: Running instance, writable layer, ephemeral
- **Layers**: Cache, share, optimize order
- **Copy-on-Write**: Efficient storage, isolation
- **Volumes**: Persistent data

---

**Chúc bạn học tốt! Tiếp tục với Day-006 để học Docker Installation & First Container.**

