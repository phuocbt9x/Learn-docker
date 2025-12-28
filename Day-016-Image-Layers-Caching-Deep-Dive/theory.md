# Day-016: Layer Caching - Build Performance

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu sâu về Docker layer caching mechanism
- Biết cách optimize layer caching để tăng build performance
- Hiểu được cache invalidation và cách tránh
- Biết được các techniques để maximize cache hits
- Viết được Dockerfile tối ưu cho caching

---

## 🧱 PHẦN 1: LAYER CACHING MECHANISM

### 1.1. Layer Caching là gì?

**Layer caching** là cơ chế Docker **cache các layers** đã build trước đó để **tái sử dụng** khi build lại, giúp **tăng tốc build time**.

**How it works:**
1. **Build lần đầu**: Tất cả layers được build và cache
2. **Build lần sau**: Docker check từng layer
3. **Cache hit**: Nếu layer không đổi → dùng cache
4. **Cache miss**: Nếu layer đổi → rebuild từ đó

**Ví dụ:**
```dockerfile
FROM node:18-alpine
COPY package.json .
RUN npm install
COPY . .
```

**Build lần 1:**
- Layer 1: FROM → Build
- Layer 2: COPY package.json → Build
- Layer 3: RUN npm install → Build (5 phút)
- Layer 4: COPY . → Build

**Build lần 2 (chỉ đổi source code):**
- Layer 1: FROM → Cache hit ✅
- Layer 2: COPY package.json → Cache hit ✅
- Layer 3: RUN npm install → Cache hit ✅
- Layer 4: COPY . → Cache miss → Rebuild

**Kết quả:** Build time giảm từ 5 phút → 30 giây

### 1.2. Cache Key

**Cache key** là cách Docker **identify** một layer có thể cache được hay không.

**Cache key được tính từ:**
- **Instruction**: Loại instruction (FROM, COPY, RUN, etc.)
- **Content**: Nội dung của instruction
- **Previous layers**: Các layers trước đó

**Ví dụ:**
```dockerfile
COPY package.json .
```

**Cache key:**
- Instruction: COPY
- Source: package.json (content hash)
- Dest: . (relative to WORKDIR)
- Previous layers: Hash của tất cả layers trước

**Important:**
- **Content hash**: Nếu file thay đổi → cache miss
- **Order matters**: Thứ tự layers ảnh hưởng cache key
- **Previous layers**: Thay đổi layer trước → invalidate layers sau

### 1.3. Cache Invalidation

**Cache invalidation** xảy ra khi một layer **không thể dùng cache** và phải rebuild.

**Causes:**
1. **Content change**: File/content thay đổi
2. **Instruction change**: Instruction thay đổi
3. **Previous layer change**: Layer trước thay đổi
4. **Force rebuild**: `docker build --no-cache`

**Ví dụ:**
```dockerfile
FROM node:18-alpine
COPY package.json .
RUN npm install
COPY app.js .
```

**Scenario 1: Thay đổi app.js**
- Layer 1-3: Cache hit ✅
- Layer 4: Cache miss → Rebuild

**Scenario 2: Thay đổi package.json**
- Layer 1: Cache hit ✅
- Layer 2: Cache miss → Rebuild
- Layer 3: Cache miss (do layer 2 đổi) → Rebuild
- Layer 4: Cache miss (do layer 3 đổi) → Rebuild

**Result:** Thay đổi package.json → rebuild từ layer 2

### 1.4. Cache Layers

**Cache layers** là các layers được **lưu trong Docker cache** để tái sử dụng.

**Cache storage:**
- **Local cache**: Lưu trên local machine
- **BuildKit cache**: Advanced caching với BuildKit
- **Registry cache**: Cache từ registry (nếu push)

**View cache:**
```bash
$ docker system df
# Show cache usage

$ docker builder prune
# Clean cache
```

**Cache size:**
- **Can grow large**: Cache có thể lớn
- **Manage cache**: Cần manage cache size
- **Clean old cache**: Clean cache cũ để giải phóng space

---

## ⚡ PHẦN 2: OPTIMIZATION TECHNIQUES

### 2.1. Order Instructions by Change Frequency

**Principle:** Đặt instructions **ít thay đổi trước**, **thường xuyên thay đổi sau**.

**Bad order:**
```dockerfile
FROM node:18-alpine
COPY . .              # ← Thay đổi thường xuyên
RUN npm install        # ← Phải rebuild mỗi lần
```

**Good order:**
```dockerfile
FROM node:18-alpine
COPY package.json .    # ← Ít thay đổi
RUN npm install        # ← Cache tốt
COPY . .               # ← Thay đổi thường xuyên
```

**Benefits:**
- **More cache hits**: Nhiều layers được cache
- **Faster rebuilds**: Rebuild nhanh hơn
- **Less work**: Ít work hơn khi rebuild

### 2.2. Copy Dependencies Before Source Code

**Principle:** Copy **dependencies files** (package.json, requirements.txt) trước **source code**.

**Bad:**
```dockerfile
FROM node:18-alpine
COPY . .               # ← Copy tất cả (source code thay đổi thường xuyên)
RUN npm install        # ← Phải rebuild mỗi lần
```

**Good:**
```dockerfile
FROM node:18-alpine
COPY package.json .    # ← Dependencies ít thay đổi
RUN npm install        # ← Cache tốt
COPY . .               # ← Source code thay đổi thường xuyên
```

**Benefits:**
- **Dependencies cached**: npm install được cache
- **Source code changes**: Chỉ rebuild COPY . khi code đổi
- **Faster builds**: Build nhanh hơn

### 2.3. Combine RUN Commands

**Principle:** **Combine multiple RUN commands** thành một để giảm số layers.

**Bad:**
```dockerfile
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y wget
RUN apt-get clean
```

**Good:**
```dockerfile
RUN apt-get update && \
    apt-get install -y curl wget && \
    apt-get clean
```

**Benefits:**
- **Fewer layers**: Ít layers hơn
- **Better caching**: Cache tốt hơn
- **Atomic operations**: Operations atomic

**Trade-off:**
- **Less granular cache**: Cache ít granular hơn
- **Rebuild all**: Phải rebuild tất cả nếu một phần đổi

### 2.4. Use .dockerignore

**Principle:** Dùng **.dockerignore** để exclude files không cần thiết khỏi build context.

**.dockerignore:**
```
node_modules/
.git/
*.log
.env
```

**Benefits:**
- **Smaller build context**: Build context nhỏ hơn
- **Faster COPY**: COPY nhanh hơn
- **Better cache**: Cache tốt hơn (ít files thay đổi)

### 2.5. Minimize Layer Changes

**Principle:** **Minimize changes** trong các layers để maximize cache hits.

**Techniques:**
- **Copy specific files**: Copy specific files thay vì COPY .
- **Group changes**: Group related changes
- **Avoid unnecessary changes**: Tránh changes không cần thiết

**Example:**
```dockerfile
# ❌ Bad: Copy all
COPY . .

# ✅ Good: Copy specific
COPY src/ src/
COPY config/ config/
```

---

## 🔍 PHẦN 3: CACHE DEBUGGING

### 3.1. Check Cache Usage

**View cache during build:**
```bash
$ docker build -t my-app .
# Output shows: Using cache or Building
```

**Verbose output:**
```bash
$ docker build --progress=plain -t my-app .
# Show detailed cache information
```

**Check cache layers:**
```bash
$ docker history my-app
# Show layers and cache status
```

### 3.2. Force Rebuild

**Force rebuild all:**
```bash
$ docker build --no-cache -t my-app .
# Rebuild tất cả, không dùng cache
```

**Force rebuild from specific stage:**
```bash
$ docker build --no-cache-from=builder -t my-app .
# Rebuild từ stage builder
```

### 3.3. Cache Analysis

**Analyze cache behavior:**
```bash
# Build và measure time
$ time docker build -t my-app .

# Check which layers used cache
$ docker history my-app
```

**Identify cache misses:**
- **Check build output**: Look for "Building" vs "Using cache"
- **Analyze changes**: Identify what changed
- **Optimize**: Optimize based on analysis

---

## 🏭 PRODUCTION STORY #1: Slow Build Times

### Context

**Công ty:** E-commerce, 800 employees
**Hệ thống:** Node.js microservices với Docker
**Traffic:** 20M requests/day
**Team:** 40 backend engineers

### Problem

**Tháng 9/2023:**
- **Build times quá chậm**: 15-20 phút per build
- **CI/CD bottleneck**: CI/CD pipeline bị bottleneck
- **Developer productivity**: Developers phải chờ build
- **Root cause**: Poor layer caching

**Timeline:**
- **10:00 AM**: Developer push code
- **10:01 AM**: CI/CD start build
- **10:16 AM**: Build complete (15 phút)
- **10:20 AM**: Team investigate build times

**Impact:**
- **Build time**: 15-20 phút per build
- **CI/CD costs**: High costs do build time lâu
- **Developer productivity**: 30% time lost waiting

### Investigation

**Root cause:**
```dockerfile
FROM node:18-alpine
COPY . .                    # ← Copy tất cả trước
RUN npm install             # ← Phải rebuild mỗi lần
RUN npm run build
CMD ["node", "dist/index.js"]
```

**Vấn đề:**
- **Copy order**: Copy tất cả trước → npm install phải rebuild mỗi lần
- **No cache**: npm install không được cache
- **Slow builds**: Build chậm do phải install dependencies mỗi lần

**Test:**
```bash
$ time docker build -t my-app .
# Real: 15m30s
# → npm install mất 12 phút mỗi lần
```

### Fix

**Solution: Optimize layer order**
```dockerfile
FROM node:18-alpine
WORKDIR /app

# Copy dependencies first
COPY package.json package-lock.json ./
RUN npm install             # ← Cache tốt

# Copy source code after
COPY . .
RUN npm run build
CMD ["node", "dist/index.js"]
```

**Kết quả:**
- **Build time**: 15 phút → 3 phút (80% reduction)
- **Cache hits**: npm install được cache
- **Faster rebuilds**: Chỉ rebuild khi source code đổi

### Result

**Trước:**
- Poor layer order
- **Build time**: 15-20 phút
- **Cache hits**: 0% (npm install)

**Sau:**
- Optimized layer order
- **Build time**: 3-5 phút
- **Cache hits**: 90% (npm install)

### Lesson Learned

1. **Order matters**: Thứ tự layers ảnh hưởng cache
2. **Dependencies first**: Copy dependencies trước source code
3. **Measure**: Measure build times để optimize
4. **Monitor**: Monitor cache hits trong CI/CD

---

## 🏭 PRODUCTION STORY #2: Cache Invalidation Issues

### Context

**Công ty:** SaaS platform, 500 employees
**Hệ thống:** Python applications với Docker
**Traffic:** 12M requests/day
**Team:** 25 backend engineers

### Problem

**Tháng 11/2023:**
- **Unexpected rebuilds**: Layers rebuild không cần thiết
- **Slow builds**: Build chậm do cache invalidation
- **Root cause**: Unnecessary changes trong layers
- **Impact**: Build time tăng 3x

**Timeline:**
- **2:00 PM**: Developer push code (chỉ đổi 1 file)
- **2:01 PM**: CI/CD start build
- **2:10 PM**: Build complete (9 phút - expected 3 phút)
- **2:15 PM**: Team investigate

**Impact:**
- **Build time**: 3 phút → 9 phút (3x increase)
- **CI/CD costs**: Tăng 3x
- **Developer frustration**: Frustration do slow builds

### Investigation

**Root cause:**
```dockerfile
FROM python:3.9-slim
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
RUN python manage.py collectstatic  # ← Generate files
CMD ["gunicorn", "app.wsgi:application"]
```

**Vấn đề:**
- **collectstatic**: Generate files mỗi lần build
- **Files change**: Generated files thay đổi → cache miss
- **Cascade effect**: Cache miss → rebuild layers sau

**Test:**
```bash
$ docker build -t my-app .
# Step 4: RUN collectstatic → Always rebuild
# → Invalidate layers after
```

### Fix

**Solution 1: Exclude generated files**
```dockerfile
FROM python:3.9-slim
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
# Don't run collectstatic in build
# Run at runtime instead
CMD ["gunicorn", "app.wsgi:application"]
```

**Solution 2: Use .dockerignore**
```dockerfile
# .dockerignore
staticfiles/
*.pyc
__pycache__/
```

**Kết quả:**
- **Build time**: 9 phút → 3 phút
- **Cache hits**: 90% (collectstatic không chạy trong build)
- **Stable cache**: Cache stable hơn

### Result

**Trước:**
- collectstatic trong build
- **Build time**: 9 phút
- **Cache hits**: 30%

**Sau:**
- collectstatic ở runtime
- **Build time**: 3 phút
- **Cache hits**: 90%

### Lesson Learned

1. **Avoid file generation in build**: Tránh generate files trong build
2. **Use .dockerignore**: Exclude generated files
3. **Runtime vs build-time**: Phân biệt runtime và build-time operations
4. **Monitor cache**: Monitor cache behavior

---

## 🎓 TÓM TẮT

### Layer Caching

**Mechanism:**
- Docker cache layers đã build
- Reuse cache khi layer không đổi
- Invalidate cache khi layer đổi

**Benefits:**
- **Faster builds**: Build nhanh hơn
- **Less work**: Ít work hơn
- **Cost savings**: Tiết kiệm costs

### Optimization Techniques

**1. Order by change frequency:**
- Ít thay đổi trước
- Thường xuyên thay đổi sau

**2. Dependencies before source:**
- Copy dependencies trước
- Copy source code sau

**3. Combine RUN commands:**
- Combine multiple RUN
- Giảm số layers

**4. Use .dockerignore:**
- Exclude unnecessary files
- Smaller build context

**5. Minimize changes:**
- Copy specific files
- Avoid unnecessary changes

### Cache Debugging

**Tools:**
- `docker build --progress=plain`
- `docker history`
- `docker system df`

**Analysis:**
- Check cache hits/misses
- Identify bottlenecks
- Optimize based on analysis

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ bạn đã:
- ✅ Hiểu layer caching mechanism
- ✅ Biết cách optimize caching
- ✅ Debug cache issues

**Day tiếp theo (Day-017)** sẽ đi sâu vào:
- .dockerignore & Build Context
- Build context optimization
- Best practices

---

## 📚 TÀI LIỆU THAM KHẢO

- Layer caching: https://docs.docker.com/build/cache/
- Build optimization: https://docs.docker.com/build/guide/optimizing-builds/

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-017: dockerignore-Build-Context](../Day-017-dockerignore-Build-Context/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
