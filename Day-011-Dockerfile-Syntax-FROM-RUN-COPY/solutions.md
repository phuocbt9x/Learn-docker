# Day-011: Giải Pháp & Giải Thích

## 🎯 LƯU Ý QUAN TRỌNG

Các giải pháp dưới đây là Dockerfiles và explanations thực tế. Quan trọng là bạn:

- **Thực hành** viết và build Dockerfiles
- **Hiểu** layer caching mechanism
- **Apply** best practices
- **Measure** build time và image size

---

## 📝 BÀI TẬP 1: DOCKERFILE BASICS

### 1.1. Viết Dockerfile Cơ Bản

**Dockerfile:**
```dockerfile
FROM python:3.9

# Set working directory
WORKDIR /app

# Copy requirements first (for better caching)
COPY requirements.txt /app/

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY app.py /app/

# Run application
CMD ["python", "app.py"]
```

**Giải thích:**
- `FROM python:3.9`: Base image
- `WORKDIR /app`: Set working directory
- `COPY requirements.txt`: Copy dependencies trước (cache)
- `RUN pip install`: Install dependencies
- `COPY app.py`: Copy code sau
- `CMD`: Run application

### 1.2. Build và Test

**Build:**
```bash
$ docker build -t my-app:latest .
```

**Run:**
```bash
$ docker run -d -p 5000:5000 my-app:latest
```

**Verify:**
```bash
$ curl http://localhost:5000
# Hoặc check logs
$ docker logs <container-id>
```

### 1.3. Layers và Caching

**Layers:**
1. Layer 1: `FROM python:3.9` (base image)
2. Layer 2: `WORKDIR /app` (metadata)
3. Layer 3: `COPY requirements.txt` (file)
4. Layer 4: `RUN pip install` (dependencies)
5. Layer 5: `COPY app.py` (code)
6. Layer 6: `CMD` (metadata)

**Caching:**
- **Build 1**: Tất cả layers được build
- **Build 2** (không thay đổi): Tất cả cached
- **Build 3** (chỉ thay đổi app.py): Layers 1-4 cached, layer 5 rebuild

**Optimize:**
- Copy requirements.txt trước → pip install được cache
- Copy code sau → chỉ rebuild khi code thay đổi

---

## 📝 BÀI TẬP 2: FROM INSTRUCTION

### 2.1. Python Application

**Options:**
- `python:3.9`: ~900MB (full)
- `python:3.9-slim`: ~120MB (slim)
- `python:3.9-alpine`: ~45MB (alpine)

**Recommendation: `python:3.9-slim`**

**Lý do:**
- ✅ Đủ packages cho most use cases
- ✅ Nhỏ hơn full (87% nhỏ hơn)
- ✅ Dễ hơn alpine (không cần compile nhiều packages)
- ✅ Good balance giữa size và compatibility

**So sánh:**
- **Full**: Quá lớn, không cần
- **Slim**: Perfect cho most cases
- **Alpine**: Nhỏ nhất nhưng có thể có compatibility issues

### 2.2. Node.js Application

**Options:**
- `node:16`: ~900MB (full)
- `node:16-slim`: ~200MB (slim)
- `node:16-alpine`: ~170MB (alpine)

**Recommendation: `node:16-slim` hoặc `node:16-alpine`**

**Lý do:**
- **Slim**: Nếu cần build tools
- **Alpine**: Nếu không cần build tools, muốn nhỏ nhất

### 2.3. Nginx Static Site

**Options:**
- `nginx:latest`: ~133MB
- `nginx:alpine`: ~23MB
- `nginx:1.21`: ~133MB

**Recommendation: `nginx:alpine`**

**Lý do:**
- ✅ Nhỏ nhất (83% nhỏ hơn)
- ✅ Đủ cho static sites
- ✅ Security tốt
- ✅ Pin version: `nginx:1.21-alpine`

### 2.4. Best Practices

**Không dùng latest:**
- **Mutable**: Có thể thay đổi
- **Unpredictable**: Không biết version
- **Production risk**: Có thể deploy version chưa test

**Alpine vs Slim:**
- **Alpine**: Nhỏ nhất, nhưng có thể có compatibility issues
- **Slim**: Balance tốt, recommended cho most cases
- **Full**: Chỉ khi cần nhiều packages

**Trade-offs:**
- **Alpine**: Size nhỏ, nhưng có thể cần compile packages
- **Slim**: Size trung bình, compatibility tốt
- **Full**: Size lớn, nhưng có đủ mọi thứ

---

## 📝 BÀI TẬP 3: RUN INSTRUCTION

### 3.1. Optimize Dockerfile

**Before:**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get install -y python3-pip
RUN apt-get install -y curl
RUN apt-get install -y wget
RUN apt-get clean
```

**After:**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    python3 \
    python3-pip \
    curl \
    wget && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

**Optimizations:**
1. **Combine RUN**: 6 layers → 1 layer
2. **--no-install-recommends**: Chỉ install packages cần
3. **apt-get clean**: Remove package cache
4. **rm -rf /var/lib/apt/lists/***: Remove package lists

**Better: Use slim base**
```dockerfile
FROM python:3.9-slim
# Không cần install python3, python3-pip
# Chỉ install packages cần thiết
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    curl \
    wget && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### 3.2. Shell Form vs Exec Form

**Shell form:**
```dockerfile
RUN apt-get update && apt-get install -y nginx
```

**Đặc điểm:**
- Chạy trong shell (`/bin/sh -c`)
- Có thể dùng shell features (&&, ||, pipes, etc.)
- **Không có signal handling**

**Exec form:**
```dockerfile
RUN ["apt-get", "update"]
RUN ["apt-get", "install", "-y", "nginx"]
```

**Đặc điểm:**
- Chạy trực tiếp executable
- **Không có shell** → không dùng shell features
- **Có signal handling**

**Khi nào dùng:**
- **Shell form**: Khi cần shell features (most cases với RUN)
- **Exec form**: Khi cần signal handling (ít dùng với RUN, thường dùng với CMD/ENTRYPOINT)

### 3.3. Combine Commands

**Tại sao cần combine:**
- **Giảm số layers**: Ít layers = nhỏ hơn, nhanh hơn
- **Better caching**: Ít layers = dễ cache hơn
- **Atomic operations**: Commands liên quan nên cùng layer

**Cách combine đúng:**
```dockerfile
# Good: Combine related commands
RUN apt-get update && \
    apt-get install -y package1 package2 && \
    apt-get clean

# Bad: Separate unrelated commands
RUN apt-get update
RUN npm install
RUN pip install
```

**Best practices:**
- Combine related commands
- Use `&&` để chain commands
- Use `\` để continue line
- Remove cache trong cùng RUN

### 3.4. Remove Cache

**Tại sao cần remove:**
- **Reduce image size**: Cache chiếm nhiều space
- **Security**: Không cần cache trong final image

**Cách remove:**
```dockerfile
# apt-get
RUN apt-get update && \
    apt-get install -y package && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# pip
RUN pip install --no-cache-dir -r requirements.txt

# npm
RUN npm ci --only=production && \
    npm cache clean --force
```

---

## 📝 BÀI TẬP 4: COPY INSTRUCTION

### 4.1. Copy Strategy

**Order quan trọng:**
```dockerfile
FROM python:3.9-slim

# Step 1: Copy dependencies (ít thay đổi)
COPY requirements.txt /app/
RUN pip install -r /app/requirements.txt

# Step 2: Copy code (thay đổi thường xuyên)
COPY . /app/
```

**Tại sao order quan trọng:**
- **Dependencies ít thay đổi**: requirements.txt thay đổi ít
- **Code thay đổi thường xuyên**: app.py thay đổi nhiều
- **Cache behavior**: Dependencies được cache, chỉ rebuild code layer

### 4.2. Optimize COPY

**Bad:**
```dockerfile
FROM python:3.9-slim
COPY . /app/
RUN pip install -r requirements.txt
```

**Vấn đề:**
- Copy tất cả trước → invalidate cache
- Mỗi lần code thay đổi → rebuild pip install

**Good:**
```dockerfile
FROM python:3.9-slim
COPY requirements.txt /app/
RUN pip install -r /app/requirements.txt
COPY . /app/
```

**Lợi ích:**
- Copy requirements.txt trước → pip install được cache
- Copy code sau → chỉ rebuild code layer khi code thay đổi
- **80-90% faster builds** khi chỉ code thay đổi

### 4.3. .dockerignore

**.dockerignore file:**
```
node_modules
.git
*.log
.env
.DS_Store
dist
build
*.pyc
__pycache__
.vscode
.idea
```

**Tại sao quan trọng:**
- **Exclude files không cần**: Giảm build context size
- **Prevent cache invalidation**: Files không cần không trigger rebuild
- **Security**: Không copy sensitive files (.env, credentials)

**Best practices:**
- Exclude dependencies (node_modules, venv)
- Exclude version control (.git)
- Exclude build artifacts (dist, build)
- Exclude IDE files (.vscode, .idea)
- Exclude logs (*.log)

### 4.4. COPY vs ADD

**COPY:**
```dockerfile
COPY app.py /app/
COPY config/ /app/config/
```

**Đặc điểm:**
- ✅ Simple: Chỉ copy files
- ✅ Recommended: Best practice
- ✅ Predictable: Behavior rõ ràng

**ADD:**
```dockerfile
ADD https://example.com/file.tar.gz /tmp/
ADD file.tar.gz /tmp/
```

**Đặc điểm:**
- ⚠️ Complex: Có thể download, extract
- ⚠️ Less predictable: Behavior phức tạp
- ⚠️ Security risk: Download từ URLs

**Recommendation:**
- **Luôn dùng COPY** trừ khi cần ADD features
- **ADD chỉ khi**: Download từ URL, extract tar files

**Khi cần ADD:**
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

---

## 📝 BÀI TẬP 5: LAYER CACHING

### 5.1. Analyze Dockerfile

**Dockerfile:**
```dockerfile
FROM ubuntu:20.04
COPY . /app/
RUN apt-get update
RUN apt-get install -y python3 python3-pip
RUN pip3 install -r /app/requirements.txt
CMD ["python3", "/app/app.py"]
```

**Vấn đề:**
1. **COPY đặt đầu**: Mỗi lần code thay đổi → invalidate tất cả layers
2. **Nhiều RUN**: 3 RUN commands → 3 layers
3. **Không remove cache**: apt cache không được remove
4. **Base image**: ubuntu:20.04 lớn, nên dùng python:3.9-slim

**Cache behavior:**
- **Build 1**: Tất cả layers build
- **Build 2** (code thay đổi): Tất cả layers rebuild (không cache được)
- **Build 3** (code thay đổi): Tất cả layers rebuild lại

### 5.2. Optimize Order

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
1. **Slim base**: python:3.9-slim thay vì ubuntu
2. **Combine RUN**: 3 RUN → 1 RUN
3. **Order**: Dependencies trước code
4. **Remove cache**: apt-get clean, --no-cache-dir

### 5.3. Test Caching

**Before optimization:**
- **Build 1**: 15 phút (tất cả layers)
- **Build 2** (không thay đổi): 15 phút (không cache được do COPY đầu)
- **Build 3** (code thay đổi): 15 phút (rebuild toàn bộ)

**After optimization:**
- **Build 1**: 5 phút (tất cả layers)
- **Build 2** (không thay đổi): 30 giây (cached)
- **Build 3** (code thay đổi): 2 phút (chỉ rebuild COPY . /app/)

**Improvement:**
- **90% faster** khi code thay đổi
- **95% faster** khi không thay đổi

### 5.4. Cache Invalidation

**Khi nào cache bị invalidate:**

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

**Prevent unnecessary invalidation:**
- **Order instructions**: Ít thay đổi trước
- **Use .dockerignore**: Exclude files không cần
- **Be specific**: Copy specific files
- **Pin versions**: Pin base image version

---

## 📝 BÀI TẬP 6: PRACTICAL DOCKERFILE

### 6.1. Python Web Application

**Dockerfile:**
```dockerfile
FROM python:3.9-slim

# Install system dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    gcc && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /app

# Copy requirements first (for caching)
COPY requirements.txt /app/
RUN pip install --no-cache-dir -r requirements.txt

# Copy application files
COPY app.py /app/
COPY config/ /app/config/
COPY static/ /app/static/
COPY templates/ /app/templates/

# Expose port
EXPOSE 5000

# Run application
CMD ["python", "app.py"]
```

**Optimize:**
- ✅ Slim base image
- ✅ Dependencies trước code
- ✅ Combine RUN commands
- ✅ Remove cache
- ✅ Specific COPY (không copy all)

### 6.2. Optimize

**.dockerignore:**
```
__pycache__
*.pyc
.env
.git
.vscode
.idea
*.log
```

**Further optimization:**
```dockerfile
FROM python:3.9-slim

WORKDIR /app

# System dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Python dependencies
COPY requirements.txt /app/
RUN pip install --no-cache-dir -r requirements.txt

# Application code
COPY app.py config/ static/ templates/ /app/

EXPOSE 5000
CMD ["python", "app.py"]
```

### 6.3. Node.js Application

**Dockerfile:**
```dockerfile
FROM node:16-slim

WORKDIR /app

# Copy package files
COPY package*.json /app/

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY src/ /app/src/
COPY public/ /app/public/

# Build (if needed)
RUN npm run build

# Expose port
EXPOSE 3000

# Run application
CMD ["node", "src/index.js"]
```

**Optimize với multi-stage (sẽ học Day-015):**
```dockerfile
# Build stage
FROM node:16-slim as builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Runtime stage
FROM node:16-slim
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/index.js"]
```

### 6.4. Optimize

**Best practices applied:**
- ✅ Slim base image
- ✅ Dependencies trước code
- ✅ Combine RUN commands
- ✅ Remove cache
- ✅ Multi-stage build (remove build dependencies)

---

## 📝 BÀI TẬP 7: TROUBLESHOOTING

### 7.1. Build Fails

**Error:**
```
E: Unable to locate package package
```

**Root cause:**
- Package không tồn tại
- Typo trong package name
- Repository chưa update

**Fix:**
```dockerfile
# Update trước khi install
RUN apt-get update && \
    apt-get install -y package

# Hoặc check package name
RUN apt-get update && \
    apt-get install -y python3  # Not python
```

**Best practices:**
- Always `apt-get update` trước install
- Verify package names
- Use `apt-cache search` để tìm packages

### 7.2. Build Chậm

**Root cause:**
- Không tận dụng cache
- Order instructions sai
- Nhiều layers không cần thiết

**Fix:**
- Optimize order (dependencies trước code)
- Combine RUN commands
- Use .dockerignore
- Measure và optimize

### 7.3. Image Quá Lớn

**Root cause:**
- Base image lớn
- Nhiều layers
- Không remove cache
- Copy files không cần

**Fix:**
- Use slim/alpine base images
- Combine RUN commands
- Remove cache (apt-get clean, --no-cache-dir)
- Use .dockerignore
- Multi-stage builds

### 7.4. Cache Không Work

**Root cause:**
- Order instructions sai
- Files thay đổi trigger rebuild
- Base image thay đổi

**Fix:**
- Reorder instructions
- Use .dockerignore
- Pin base image version
- Copy dependencies trước code

---

## 📝 BÀI TẬP 8: REFACTOR DOCKERFILE

### 8.1. Refactor

**Before:**
```dockerfile
FROM ubuntu:latest
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get install -y python3-pip
RUN apt-get install -y git
RUN apt-get install -y curl
RUN apt-get install -y wget
RUN apt-get install -y vim
RUN apt-get install -y htop
COPY . /app/
RUN pip3 install -r requirements.txt
CMD python3 app.py
```

**Issues:**
1. ❌ `ubuntu:latest` → nên pin version
2. ❌ Nhiều RUN commands → nên combine
3. ❌ Không remove cache
4. ❌ COPY trước dependencies → sai order
5. ❌ Install packages không cần (vim, htop)
6. ❌ CMD shell form → nên exec form

**After:**
```dockerfile
FROM python:3.9-slim

# Install only needed packages
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
    git \
    curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app

# Copy requirements first (for caching)
COPY requirements.txt /app/
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . /app/

CMD ["python", "app.py"]
```

**Optimizations:**
1. ✅ Slim base image (python:3.9-slim)
2. ✅ Combine RUN commands
3. ✅ Remove cache
4. ✅ Order: Dependencies trước code
5. ✅ Chỉ install packages cần
6. ✅ CMD exec form

### 8.2. So Sánh

**Before:**
- **Size**: ~800MB
- **Layers**: 10 layers
- **Build time**: 10 phút
- **Cache hit rate**: 0%

**After:**
- **Size**: ~200MB (75% giảm)
- **Layers**: 5 layers (50% giảm)
- **Build time**: 2 phút (80% nhanh hơn)
- **Cache hit rate**: 80-90%

### 8.3. Best Practices Applied

**Best practices:**
1. ✅ **Pin version**: python:3.9-slim (không dùng latest)
2. ✅ **Slim base**: Giảm image size
3. ✅ **Combine RUN**: Giảm số layers
4. ✅ **Remove cache**: Giảm image size
5. ✅ **Order**: Dependencies trước code
6. ✅ **Specific packages**: Chỉ install cần thiết
7. ✅ **CMD exec form**: Better signal handling
8. ✅ **.dockerignore**: Exclude files không cần

**Trade-offs:**
- **Slim base**: Có thể thiếu packages → cần install thêm
- **Combine RUN**: Khó debug hơn (nhưng worth it)
- **Order**: Phải plan trước (nhưng better caching)

---

## ✅ TỔNG KẾT

Các bài tập này giúp bạn:

1. **Write Dockerfiles**: FROM, RUN, COPY
2. **Choose base images**: Slim, alpine, official
3. **Optimize layers**: Combine commands, order đúng
4. **Cache optimization**: Maximize cache hits
5. **Best practices**: Production-ready Dockerfiles

**Key takeaways:**
- **FROM**: Pin version, use slim/alpine, official images
- **RUN**: Combine commands, remove cache
- **COPY**: Dependencies trước code, use .dockerignore
- **Caching**: Order quan trọng, measure và optimize

---

**Chúc bạn học tốt! Tiếp tục với Day-012 để học CMD vs ENTRYPOINT.**

