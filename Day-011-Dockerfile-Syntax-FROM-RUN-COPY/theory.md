# Day-011: Dockerfile Syntax - FROM, RUN, COPY

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được Dockerfile là gì và cách hoạt động
- Thành thạo instruction `FROM` - chọn base image đúng
- Thành thạo instruction `RUN` - execute commands
- Thành thạo instruction `COPY` - copy files vào image
- Hiểu được layer caching và cách optimize
- Viết được Dockerfile cơ bản production-ready

---

## 📖 PHẦN 1: DOCKERFILE LÀ GÌ?

### 1.1. Dockerfile là gì?

**Dockerfile** là một **text file** chứa instructions để build Docker image.

**Ví dụ đơn giản:**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y nginx
COPY index.html /var/www/html/
CMD ["nginx", "-g", "daemon off;"]
```

**Đặc điểm:**
- **Text file**: Plain text, không binary
- **Instructions**: Mỗi dòng là một instruction
- **Case-sensitive**: Instructions phải viết hoa
- **Order matters**: Thứ tự instructions quan trọng

### 1.2. Tại sao Dockerfile tồn tại?

**Vấn đề:**
- Manual image creation phức tạp
- Không reproducible
- Không version control được

**Dockerfile giải quyết:**
- **Automation**: Tự động build image
- **Reproducible**: Cùng Dockerfile → cùng image
- **Version control**: Track changes trong Git
- **Documentation**: Dockerfile là documentation

### 1.3. Dockerfile Format

**Basic format:**
```dockerfile
# Comment
INSTRUCTION arguments
```

**Rules:**
- **Instructions**: Viết hoa (FROM, RUN, COPY, etc.)
- **Arguments**: Tùy instruction
- **Comments**: Bắt đầu với `#`
- **Line continuation**: Dùng `\` để continue line

**Ví dụ:**
```dockerfile
# This is a comment
FROM ubuntu:20.04

# Install packages
RUN apt-get update && \
    apt-get install -y nginx

# Copy files
COPY index.html /var/www/html/
```

### 1.4. Build Process

**Khi build image:**
```bash
$ docker build -t my-app:latest .
```

**Process:**
1. **Read Dockerfile**: Đọc Dockerfile từ build context
2. **Parse instructions**: Parse từng instruction
3. **Execute instructions**: Execute từng instruction
4. **Create layers**: Mỗi instruction tạo một layer
5. **Final image**: Combine tất cả layers thành image

---

## 🏗️ PHẦN 2: FROM INSTRUCTION

### 2.1. FROM là gì?

**FROM** là instruction **đầu tiên** trong Dockerfile, chỉ định **base image**.

**Syntax:**
```dockerfile
FROM <image>[:<tag>]
# Hoặc
FROM <image>[@<digest>]
```

**Ví dụ:**
```dockerfile
FROM ubuntu:20.04
FROM python:3.9-slim
FROM nginx:1.21-alpine
FROM my-image@sha256:abc123...
```

**Đặc điểm:**
- **Bắt buộc**: Phải có trong mọi Dockerfile
- **Đầu tiên**: Phải là instruction đầu tiên (trừ ARG)
- **Base**: Tất cả instructions sau dựa trên base image này

### 2.2. Tại sao FROM quan trọng?

**FROM quyết định:**
- **OS**: Operating system (Ubuntu, Alpine, etc.)
- **Runtime**: Python, Node.js, Java, etc.
- **Size**: Base image size ảnh hưởng final image size
- **Security**: Base image có vulnerabilities → final image có vulnerabilities

**Ví dụ:**
```dockerfile
# Large base image
FROM ubuntu:20.04
# Final image: ~200MB

# Small base image
FROM alpine:3.15
# Final image: ~5MB
```

### 2.3. Chọn Base Image

**Tiêu chí chọn base image:**

1. **Size**: Càng nhỏ càng tốt
   - `alpine`: ~5MB (nhỏ nhất)
   - `slim`: ~50-100MB
   - `full`: ~200-500MB

2. **Security**: 
   - Official images (maintained bởi vendors)
   - Regular updates
   - Vulnerability scanning

3. **Compatibility**:
   - Application requirements
   - Dependencies
   - Performance needs

4. **Maintenance**:
   - Actively maintained
   - Regular updates
   - Good documentation

**Recommendations:**
- **Alpine**: Khi có thể (nhỏ, secure)
- **Slim**: Khi Alpine không đủ
- **Official**: Luôn dùng official images
- **Pin version**: Không dùng `latest` trong production

### 2.4. FROM Best Practices

**Best practices:**

1. **Pin version:**
   ```dockerfile
   # Good
   FROM ubuntu:20.04
   
   # Bad
   FROM ubuntu:latest
   ```

2. **Use official images:**
   ```dockerfile
   # Good
   FROM python:3.9-slim
   
   # Bad
   FROM someuser/python:3.9
   ```

3. **Use slim/alpine variants:**
   ```dockerfile
   # Good
   FROM python:3.9-slim
   FROM nginx:alpine
   
   # Bad
   FROM python:3.9
   FROM nginx:latest
   ```

4. **Multi-stage builds** (sẽ học Day-015):
   ```dockerfile
   FROM node:16 as builder
   # Build stage
   
   FROM node:16-slim
   # Runtime stage
   ```

---

## ⚙️ PHẦN 3: RUN INSTRUCTION

### 3.1. RUN là gì?

**RUN** execute commands trong container **khi build**.

**Syntax:**
```dockerfile
RUN <command>
# Hoặc
RUN ["executable", "param1", "param2"]
```

**Ví dụ:**
```dockerfile
# Shell form
RUN apt-get update
RUN apt-get install -y nginx

# Exec form
RUN ["apt-get", "update"]
RUN ["apt-get", "install", "-y", "nginx"]
```

**Đặc điểm:**
- **Execute during build**: Chạy khi build, không phải khi run
- **Creates layer**: Mỗi RUN tạo một layer
- **Changes persist**: Changes được lưu trong image

### 3.2. Shell Form vs Exec Form

**Shell form:**
```dockerfile
RUN apt-get update && apt-get install -y nginx
```

**Đặc điểm:**
- Chạy trong shell (`/bin/sh -c`)
- Có thể dùng shell features (variables, pipes, etc.)
- **Không có signal handling** (SIGTERM không work)

**Exec form:**
```dockerfile
RUN ["apt-get", "update"]
RUN ["apt-get", "install", "-y", "nginx"]
```

**Đặc điểm:**
- Chạy trực tiếp executable
- **Không có shell** → không dùng shell features
- **Có signal handling** (SIGTERM work)

**Khi nào dùng gì?**
- **Shell form**: Khi cần shell features (pipes, variables, etc.)
- **Exec form**: Khi cần signal handling (ít dùng với RUN)

### 3.3. Combine RUN Commands

**Bad: Nhiều RUN commands**
```dockerfile
RUN apt-get update
RUN apt-get install -y package1
RUN apt-get install -y package2
RUN apt-get clean
```

**Vấn đề:**
- **Nhiều layers**: 4 layers thay vì 1
- **Larger image**: Mỗi layer có metadata overhead
- **Slower builds**: Nhiều layers hơn

**Good: Combine RUN commands**
```dockerfile
RUN apt-get update && \
    apt-get install -y package1 package2 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

**Lợi ích:**
- **1 layer**: Chỉ 1 layer thay vì 4
- **Smaller image**: Ít metadata overhead
- **Faster builds**: Ít layers hơn

### 3.4. RUN Best Practices

**Best practices:**

1. **Combine commands:**
   ```dockerfile
   RUN apt-get update && \
       apt-get install -y package1 package2 && \
       apt-get clean
   ```

2. **Remove cache:**
   ```dockerfile
   RUN apt-get update && \
       apt-get install -y package && \
       apt-get clean && \
       rm -rf /var/lib/apt/lists/*
   ```

3. **Order matters:**
   ```dockerfile
   # Install dependencies trước
   RUN apt-get update && apt-get install -y dependencies
   
   # Copy code sau
   COPY . /app
   ```

4. **Use --no-install-recommends:**
   ```dockerfile
   RUN apt-get update && \
       apt-get install -y --no-install-recommends package && \
       apt-get clean
   ```

---

## 📋 PHẦN 4: COPY INSTRUCTION

### 4.1. COPY là gì?

**COPY** copy files từ **build context** vào image.

**Syntax:**
```dockerfile
COPY <src> <dest>
# Hoặc
COPY ["<src>", "<dest>"]
```

**Ví dụ:**
```dockerfile
COPY index.html /var/www/html/
COPY app.py /app/
COPY config/ /app/config/
COPY . /app/
```

**Đặc điểm:**
- **Build context**: Chỉ copy từ build context
- **Creates layer**: Mỗi COPY tạo một layer
- **File permissions**: Preserve file permissions

### 4.2. Build Context

**Build context là gì?**

**Build context** là directory được specify khi build:
```bash
$ docker build -t my-app .
# '.' là build context
```

**Rules:**
- **Only files in context**: Chỉ copy files trong build context
- **Outside context**: Không thể copy files ngoài context
- **.dockerignore**: Exclude files khỏi context

**Ví dụ:**
```
project/
├── Dockerfile
├── app.py
├── requirements.txt
└── data/
    └── data.txt

# Build context: project/
# Có thể copy: app.py, requirements.txt, data/
# Không thể copy: files ngoài project/
```

### 4.3. COPY Patterns

**Copy single file:**
```dockerfile
COPY app.py /app/app.py
```

**Copy directory:**
```dockerfile
COPY config/ /app/config/
```

**Copy với wildcard:**
```dockerfile
COPY *.txt /app/
```

**Copy multiple files:**
```dockerfile
COPY app.py requirements.txt /app/
```

**Copy all files:**
```dockerfile
COPY . /app/
```

### 4.4. COPY vs ADD

**COPY:**
- ✅ Simple: Chỉ copy files
- ✅ Recommended: Best practice
- ✅ Predictable: Behavior rõ ràng

**ADD:**
- ⚠️ Complex: Có thể download, extract
- ⚠️ Less predictable: Behavior phức tạp hơn
- ⚠️ Security risk: Có thể download từ URLs

**Recommendation:**
- **Luôn dùng COPY** trừ khi cần features của ADD
- **ADD chỉ khi cần**: Download từ URL, extract tar files

**Ví dụ khi cần ADD:**
```dockerfile
# Download và extract
ADD https://example.com/file.tar.gz /tmp/
RUN tar -xzf /tmp/file.tar.gz -C /app/
```

**Nhưng tốt hơn:**
```dockerfile
# Download trong RUN
RUN curl -o /tmp/file.tar.gz https://example.com/file.tar.gz && \
    tar -xzf /tmp/file.tar.gz -C /app/ && \
    rm /tmp/file.tar.gz
```

### 4.5. COPY Best Practices

**Best practices:**

1. **Copy dependencies trước code:**
   ```dockerfile
   # Dependencies (ít thay đổi)
   COPY requirements.txt /app/
   RUN pip install -r /app/requirements.txt
   
   # Code (thay đổi thường xuyên)
   COPY . /app/
   ```

2. **Use .dockerignore:**
   ```dockerfile
   # .dockerignore
   node_modules
   .git
   *.log
   .env
   ```

3. **Be specific:**
   ```dockerfile
   # Good: Specific
   COPY requirements.txt /app/
   
   # Bad: Copy all
   COPY . /app/
   ```

4. **Copy only needed files:**
   ```dockerfile
   # Good: Only needed
   COPY app.py config.json /app/
   
   # Bad: Copy everything
   COPY . /app/
   ```

---

## 🔄 PHẦN 5: LAYER CACHING

### 5.1. Layer Caching là gì?

**Layer caching** là mechanism Docker cache layers để **build nhanh hơn**.

**Cách hoạt động:**
1. **Build lần đầu**: Tất cả layers được build
2. **Build lần 2**: Docker check từng layer
3. **Nếu không đổi**: Dùng cached layer
4. **Nếu đổi**: Rebuild từ layer đó

**Ví dụ:**
```dockerfile
FROM ubuntu:20.04          # Layer 1: Cached
RUN apt-get update         # Layer 2: Cached
RUN apt-get install nginx  # Layer 3: Cached
COPY app.py /app/          # Layer 4: Rebuild (code changed)
```

### 5.2. Cache Invalidation

**Khi nào cache bị invalidate?**

1. **Instruction thay đổi:**
   ```dockerfile
   # Build 1
   RUN apt-get install -y nginx
   
   # Build 2 (changed)
   RUN apt-get install -y nginx curl
   # → Cache invalidate từ layer này
   ```

2. **Files thay đổi (COPY):**
   ```dockerfile
   # Build 1
   COPY app.py /app/
   
   # Build 2 (app.py changed)
   COPY app.py /app/
   # → Cache invalidate từ layer này
   ```

3. **Base image thay đổi:**
   ```dockerfile
   # Build 1
   FROM ubuntu:20.04
   
   # Build 2 (base changed)
   FROM ubuntu:22.04
   # → Cache invalidate từ đầu
   ```

### 5.3. Optimize Layer Caching

**Strategy: Order instructions từ ít thay đổi → nhiều thay đổi**

**Bad order:**
```dockerfile
FROM ubuntu:20.04
COPY . /app/              # ← Thay đổi thường xuyên
RUN apt-get update        # ← Ít thay đổi
RUN apt-get install nginx # ← Ít thay đổi
```

**Vấn đề:**
- Code thay đổi → invalidate cache từ COPY
- Phải rebuild apt-get commands mỗi lần

**Good order:**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update        # ← Ít thay đổi (cache được)
RUN apt-get install nginx # ← Ít thay đổi (cache được)
COPY . /app/              # ← Thay đổi thường xuyên (rebuild từ đây)
```

**Lợi ích:**
- Code thay đổi → chỉ rebuild COPY layer
- apt-get commands được cache

### 5.4. Caching Best Practices

**Best practices:**

1. **Order matters:**
   - Dependencies trước code
   - Ít thay đổi trước nhiều thay đổi

2. **Combine RUN commands:**
   - Giảm số layers
   - Better caching

3. **Use .dockerignore:**
   - Exclude files không cần
   - Prevent unnecessary cache invalidation

4. **Be specific:**
   - Copy specific files
   - Không copy all nếu không cần

---

## 🏭 PRODUCTION STORY #1: Dockerfile Build Chậm

### Context

**Công ty:** SaaS platform, 400 employees
**Hệ thống:** Microservices, 50+ services
**CI/CD:** Build images trong CI pipeline
**Team:** 30 backend engineers

### Problem

**Tháng 5/2023:**
- **Build time: 15-20 phút** mỗi service
- **CI pipeline chậm**: Block deployments
- **Root cause**: Dockerfile không optimize, không tận dụng cache

**Timeline:**
- **10:00 AM**: Developer push code
- **10:05 AM**: CI trigger build
- **10:25 AM**: Build complete (20 phút)
- **10:30 AM**: Deploy
- **Total**: 30 phút

**Impact:**
- **Slow deployments**: 30 phút mỗi deployment
- **Developer productivity**: Chờ build
- **CI costs**: Tốn resources

### Investigation

**Root cause:**
```dockerfile
# Bad Dockerfile
FROM ubuntu:20.04
COPY . /app/                    # ← Copy trước
RUN apt-get update              # ← Rebuild mỗi lần
RUN apt-get install -y python3  # ← Rebuild mỗi lần
RUN pip install -r requirements.txt  # ← Rebuild mỗi lần
```

**Vấn đề:**
- COPY đặt đầu → invalidate cache
- Mỗi lần code thay đổi → rebuild toàn bộ
- **Không tận dụng layer cache**

### Fix

**Optimized Dockerfile:**
```dockerfile
FROM python:3.9-slim

# Install system dependencies (ít thay đổi)
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    build-essential && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Copy requirements trước (ít thay đổi)
COPY requirements.txt /app/
RUN pip install --no-cache-dir -r /app/requirements.txt

# Copy code sau (thay đổi thường xuyên)
COPY . /app/
WORKDIR /app

CMD ["python", "app.py"]
```

**Optimizations:**
1. **Order**: Dependencies trước code
2. **Combine RUN**: Giảm số layers
3. **Slim base**: python:3.9-slim thay vì ubuntu
4. **Remove cache**: `--no-cache-dir`, `apt-get clean`

### Result

**Trước:**
- Build time: **15-20 phút**
- Cache hit rate: **0%** (không cache được)

**Sau:**
- Build time: **2-3 phút** (90% nhanh hơn)
- Cache hit rate: **80-90%** (dependencies được cache)
- **First build**: 5 phút (không cache)
- **Subsequent builds**: 2-3 phút (cached)

**Benefits:**
- **Faster deployments**: 30 phút → 10 phút
- **Better developer experience**: Chờ ít hơn
- **Lower CI costs**: Ít resources hơn

### Lesson Learned

1. **Layer order quan trọng**: Ảnh hưởng trực tiếp đến build time
2. **Cache optimization**: Tận dụng cache giảm 80-90% build time
3. **Combine commands**: Giảm số layers
4. **Regular review**: Review Dockerfiles thường xuyên

---

## 🏭 PRODUCTION STORY #2: Image Quá Lớn do Nhiều Layers

### Context

**Công ty:** E-commerce, 600 employees
**Hệ thống:** 100+ containers
**Traffic:** 10M requests/day
**Team:** 35 DevOps engineers

### Problem

**Tháng 7/2023:**
- **Image size: 2.5GB** cho một service đơn giản
- **Pull time: 10-15 phút**
- **Root cause**: Nhiều RUN commands, không optimize

**Timeline:**
- **2:00 AM**: Auto-scaling trigger
- **2:05 AM**: Pull images
- **2:20 AM**: Images pulled (15 phút)
- **2:25 AM**: Containers start
- **Total**: 25 phút để scale

**Impact:**
- **Slow scaling**: 25 phút thay vì 5 phút
- **High bandwidth**: Tốn bandwidth
- **Storage costs**: Tốn storage

### Investigation

**Root cause:**
```dockerfile
# Bad Dockerfile
FROM ubuntu:20.04
RUN apt-get update
RUN apt-get install -y package1
RUN apt-get install -y package2
RUN apt-get install -y package3
RUN apt-get install -y package4
RUN apt-get install -y package5
# ... 20+ RUN commands
COPY . /app/
```

**Vấn đề:**
- **Nhiều layers**: 20+ layers
- **Mỗi layer có overhead**: Metadata, filesystem
- **Không cleanup**: apt cache, build artifacts

### Fix

**Optimized Dockerfile:**
```dockerfile
FROM python:3.9-slim

# Combine tất cả RUN commands
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    package1 package2 package3 package4 package5 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

COPY requirements.txt /app/
RUN pip install --no-cache-dir -r /app/requirements.txt

COPY . /app/
WORKDIR /app

CMD ["python", "app.py"]
```

**Optimizations:**
1. **Combine RUN**: 20+ layers → 3 layers
2. **Remove cache**: `apt-get clean`, `rm -rf /var/lib/apt/lists/*`
3. **Slim base**: python:3.9-slim
4. **--no-install-recommends**: Chỉ install packages cần

### Result

**Trước:**
- Image size: **2.5GB**
- Layers: **25 layers**
- Pull time: **10-15 phút**

**Sau:**
- Image size: **300MB** (88% giảm)
- Layers: **5 layers** (80% giảm)
- Pull time: **1-2 phút** (87% nhanh hơn)

**Benefits:**
- **Faster scaling**: 25 phút → 5 phút
- **Less bandwidth**: 88% ít hơn
- **Lower storage costs**: 88% ít hơn

### Lesson Learned

1. **Combine RUN commands**: Giảm số layers đáng kể
2. **Remove cache**: Cleanup sau install
3. **Slim base images**: Giảm image size
4. **Regular optimization**: Review và optimize thường xuyên

---

## 🎓 TÓM TẮT

### Dockerfile Basics

**Là gì:**
- Text file với instructions
- Build Docker images
- Version controlled

**Format:**
- Instructions viết hoa
- Mỗi instruction = 1 layer
- Order matters

### FROM Instruction

**Chức năng:**
- Chỉ định base image
- Bắt buộc, phải đầu tiên

**Best practices:**
- Pin version (không dùng latest)
- Use official images
- Use slim/alpine variants

### RUN Instruction

**Chức năng:**
- Execute commands khi build
- Tạo layers

**Best practices:**
- Combine commands
- Remove cache
- Order từ ít thay đổi → nhiều thay đổi

### COPY Instruction

**Chức năng:**
- Copy files từ build context
- Tạo layers

**Best practices:**
- Copy dependencies trước code
- Use .dockerignore
- Be specific (không copy all)

### Layer Caching

**Cách hoạt động:**
- Cache layers không đổi
- Invalidate khi đổi

**Optimization:**
- Order instructions đúng
- Combine RUN commands
- Use .dockerignore

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ bạn đã:
- ✅ Hiểu Dockerfile basics
- ✅ Thành thạo FROM, RUN, COPY
- ✅ Hiểu layer caching

**Day tiếp theo (Day-012)** sẽ đi sâu vào:
- CMD vs ENTRYPOINT
- Khi nào dùng CMD? Khi nào dùng ENTRYPOINT?
- Trade-offs và best practices

---

## 📚 TÀI LIỆU THAM KHẢO

- Dockerfile Reference: https://docs.docker.com/engine/reference/builder/
- Best Practices: https://docs.docker.com/develop/develop-images/dockerfile_best-practices/

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-012: CMD-vs-ENTRYPOINT](../Day-012-CMD-vs-ENTRYPOINT/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
