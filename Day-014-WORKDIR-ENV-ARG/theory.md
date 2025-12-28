# Day-014: WORKDIR, ENV, ARG - Environment Management

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được WORKDIR và cách sử dụng
- Hiểu được ENV (runtime environment variables)
- Hiểu được ARG (build-time variables)
- Biết được sự khác biệt giữa ENV và ARG
- Biết khi nào dùng ENV, khi nào dùng ARG
- Viết được Dockerfile với WORKDIR, ENV, ARG đúng cách

---

## 📁 PHẦN 1: WORKDIR INSTRUCTION

### 1.1. WORKDIR là gì?

**WORKDIR** là instruction để **set working directory** cho các instructions tiếp theo.

**Syntax:**
```dockerfile
WORKDIR /path/to/directory
```

**Ví dụ:**
```dockerfile
WORKDIR /app
RUN pwd
# Output: /app
```

**Đặc điểm:**
- **Set working directory**: Set directory cho các instructions sau
- **Create if not exists**: Tự động tạo directory nếu chưa có
- **Affects subsequent instructions**: Ảnh hưởng đến tất cả instructions sau
- **Relative paths**: Có thể dùng relative paths (relative to previous WORKDIR)

### 1.2. WORKDIR Behavior

**Absolute path:**
```dockerfile
WORKDIR /app
COPY app.py .
# Copy app.py vào /app/app.py
```

**Relative path:**
```dockerfile
WORKDIR /app
WORKDIR src
# Current directory: /app/src
```

**Multiple WORKDIR:**
```dockerfile
WORKDIR /app
WORKDIR src
WORKDIR api
# Current directory: /app/src/api
```

**Important:**
- **Persists**: WORKDIR persist qua các layers
- **Affects COPY, RUN, CMD, ENTRYPOINT**: Tất cả relative paths dựa trên WORKDIR
- **Create directories**: Tự động tạo nếu chưa có

### 1.3. WORKDIR vs RUN cd

**WORKDIR (Recommended):**
```dockerfile
WORKDIR /app
RUN npm install
COPY app.js .
```

**RUN cd (Not recommended):**
```dockerfile
RUN mkdir -p /app && cd /app
RUN npm install
COPY app.js /app/
```

**So sánh:**
- **WORKDIR**: Persist qua layers, clearer
- **RUN cd**: Không persist, mỗi RUN là new shell

**Best practice:**
- **Luôn dùng WORKDIR**: Thay vì RUN cd
- **Set early**: Set WORKDIR sớm trong Dockerfile
- **Use absolute paths**: Dùng absolute paths cho clarity

### 1.4. WORKDIR Use Cases

**Use cases cho WORKDIR:**
- **Set application directory**: Set directory cho application
- **Simplify paths**: Simplify COPY, RUN paths
- **Standard practice**: Best practice trong Dockerfiles

**Ví dụ:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
CMD ["node", "index.js"]
```

---

## 🌍 PHẦN 2: ENV INSTRUCTION

### 2.1. ENV là gì?

**ENV** là instruction để **set environment variables** trong image (runtime).

**Syntax:**
```dockerfile
ENV KEY=value
ENV KEY1=value1 KEY2=value2
ENV KEY="value with spaces"
```

**Ví dụ:**
```dockerfile
ENV NODE_ENV=production
ENV PORT=3000
ENV APP_NAME="My App"
```

**Đặc điểm:**
- **Runtime variables**: Variables available khi container run
- **Persist in image**: Variables được lưu trong image
- **Available to all processes**: Tất cả processes trong container có thể access
- **Overrideable**: Có thể override khi `docker run -e`

### 2.2. ENV Behavior

**Single variable:**
```dockerfile
ENV NODE_ENV=production
```

**Multiple variables:**
```dockerfile
ENV NODE_ENV=production PORT=3000
```

**With spaces:**
```dockerfile
ENV APP_NAME="My Application"
```

**Access in container:**
```bash
$ docker run my-app env
# Output: NODE_ENV=production
```

**Override:**
```bash
$ docker run -e NODE_ENV=development my-app
# Override NODE_ENV
```

### 2.3. ENV Use Cases

**Use cases cho ENV:**
- **Application configuration**: Config cho application
- **Runtime settings**: Settings cần khi container run
- **Default values**: Default values cho application

**Ví dụ:**
```dockerfile
FROM node:18-alpine
ENV NODE_ENV=production
ENV PORT=3000
WORKDIR /app
COPY . .
CMD ["node", "index.js"]
```

### 2.4. ENV Best Practices

**1. Set early:**
```dockerfile
FROM node:18-alpine
ENV NODE_ENV=production
# Set early để có thể dùng trong RUN
```

**2. Use for defaults:**
```dockerfile
ENV PORT=3000
# Default port, có thể override
```

**3. Document:**
```dockerfile
# Application port (override with -e PORT=8080)
ENV PORT=3000
```

---

## 🔧 PHẦN 3: ARG INSTRUCTION

### 3.1. ARG là gì?

**ARG** là instruction để **define build-time variables** (chỉ available khi build).

**Syntax:**
```dockerfile
ARG variable_name[=default_value]
```

**Ví dụ:**
```dockerfile
ARG VERSION=latest
ARG BUILD_DATE
ARG GIT_COMMIT
```

**Đặc điểm:**
- **Build-time only**: Chỉ available khi build
- **Not in image**: Không được lưu trong image
- **Pass with --build-arg**: Pass values khi build
- **Scope**: Chỉ available từ khi define đến end of build

### 3.2. ARG Behavior

**Define ARG:**
```dockerfile
ARG VERSION=latest
```

**Use in Dockerfile:**
```dockerfile
ARG VERSION=latest
RUN echo "Building version: $VERSION"
```

**Pass when build:**
```bash
$ docker build --build-arg VERSION=1.0.0 -t my-app .
```

**Not available at runtime:**
```bash
$ docker run my-app env
# ❌ VERSION không có trong env
```

**Important:**
- **Build-time only**: Chỉ available khi build
- **Not in final image**: Không có trong image
- **Use for build logic**: Dùng cho build-time logic

### 3.3. ARG Use Cases

**Use cases cho ARG:**
- **Build version**: Version của build
- **Build configuration**: Config cho build process
- **Download URLs**: URLs để download dependencies
- **Build flags**: Flags cho build process

**Ví dụ:**
```dockerfile
ARG VERSION=latest
ARG BUILD_DATE
FROM node:18-alpine
WORKDIR /app
RUN echo "Building version: $VERSION"
COPY . .
RUN npm install
```

### 3.4. ARG Best Practices

**1. Set defaults:**
```dockerfile
ARG VERSION=latest
# Default value nếu không pass
```

**2. Document:**
```dockerfile
# Application version (pass with --build-arg VERSION=1.0.0)
ARG VERSION=latest
```

**3. Use for build-time only:**
```dockerfile
ARG NODE_ENV=production
RUN npm install --production
# NODE_ENV chỉ dùng khi build
```

---

## 🔄 PHẦN 4: ENV vs ARG

### 4.1. So Sánh

| Tiêu chí | ENV | ARG |
|----------|-----|-----|
| **Scope** | Runtime (in image) | Build-time only |
| **Available** | Khi container run | Chỉ khi build |
| **Persist** | ✅ Trong image | ❌ Không trong image |
| **Override** | `docker run -e` | `docker build --build-arg` |
| **Use case** | Runtime config | Build config |

### 4.2. Khi Nào Dùng ENV?

**Dùng ENV khi:**
- **Runtime configuration**: Config cần khi container run
- **Application settings**: Settings cho application
- **Default values**: Default values có thể override

**Ví dụ:**
```dockerfile
ENV NODE_ENV=production
ENV PORT=3000
ENV DB_HOST=localhost
```

### 4.3. Khi Nào Dùng ARG?

**Dùng ARG khi:**
- **Build-time configuration**: Config chỉ cần khi build
- **Version information**: Version, build date, etc.
- **Build flags**: Flags cho build process

**Ví dụ:**
```dockerfile
ARG VERSION=latest
ARG BUILD_DATE
ARG GIT_COMMIT
```

### 4.4. ENV từ ARG

**Có thể set ENV từ ARG:**

```dockerfile
ARG VERSION=latest
ENV APP_VERSION=$VERSION
```

**Kết quả:**
- **ARG**: Chỉ available khi build
- **ENV**: Available khi runtime (từ ARG value)

**Use case:**
- **Build version**: Build với version, lưu vào ENV để runtime access

**Ví dụ:**
```dockerfile
ARG VERSION=latest
ENV APP_VERSION=$VERSION
# APP_VERSION available khi runtime
```

---

## 🎯 PHẦN 5: BEST PRACTICES

### 5.1. WORKDIR Best Practices

**1. Set early:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
# Set early
```

**2. Use absolute paths:**
```dockerfile
WORKDIR /app
# Absolute path
```

**3. One WORKDIR:**
```dockerfile
WORKDIR /app
# Set once, dùng cho tất cả
```

### 5.2. ENV Best Practices

**1. Set defaults:**
```dockerfile
ENV PORT=3000
# Default value
```

**2. Document:**
```dockerfile
# Application port (override with -e PORT=8080)
ENV PORT=3000
```

**3. Group related:**
```dockerfile
ENV NODE_ENV=production \
    PORT=3000 \
    DB_HOST=localhost
```

### 5.3. ARG Best Practices

**1. Set defaults:**
```dockerfile
ARG VERSION=latest
# Default value
```

**2. Document:**
```dockerfile
# Build version (pass with --build-arg VERSION=1.0.0)
ARG VERSION=latest
```

**3. Use for build-time only:**
```dockerfile
ARG NODE_ENV=production
RUN npm install --production
# Chỉ dùng khi build
```

### 5.4. ENV từ ARG Pattern

**Pattern: Build version → Runtime version**

```dockerfile
ARG VERSION=latest
ARG BUILD_DATE
ARG GIT_COMMIT

ENV APP_VERSION=$VERSION
ENV BUILD_DATE=$BUILD_DATE
ENV GIT_COMMIT=$GIT_COMMIT
```

**Build:**
```bash
$ docker build \
  --build-arg VERSION=1.0.0 \
  --build-arg BUILD_DATE=$(date -u +'%Y-%m-%dT%H:%M:%SZ') \
  --build-arg GIT_COMMIT=$(git rev-parse HEAD) \
  -t my-app .
```

**Runtime:**
```bash
$ docker run my-app env | grep APP
# APP_VERSION=1.0.0
# BUILD_DATE=2023-01-01T00:00:00Z
# GIT_COMMIT=abc123...
```

---

## 🏭 PRODUCTION STORY #1: ENV vs ARG Confusion

### Context

**Công ty:** SaaS platform, 300 employees
**Hệ thống:** Node.js applications với Docker
**Traffic:** 8M requests/day
**Team:** 20 backend engineers

### Problem

**Tháng 4/2023:**
- **Build fails**: Build fail do ARG không available khi runtime
- **Root cause**: Developer dùng ARG thay vì ENV cho runtime config
- **Impact**: Application không start được

**Timeline:**
- **10:00 AM**: Developer push code
- **10:05 AM**: CI/CD build fail
- **10:10 AM**: Team investigate
- **10:15 AM**: Root cause: ARG không available khi runtime

**Impact:**
- **Build failures**: 5 builds failed
- **Developer productivity**: 30 phút lost
- **Confusion**: Team không hiểu ENV vs ARG

### Investigation

**Root cause:**
```dockerfile
ARG NODE_ENV=production
ENV NODE_ENV=$NODE_ENV
CMD ["node", "index.js"]
```

**Vấn đề:**
- **ARG không persist**: ARG không có trong image
- **ENV từ ARG**: ENV được set từ ARG (OK)
- **Application code**: Application code dùng `process.env.NODE_ENV` (OK)

**Actual issue:**
```dockerfile
ARG DB_HOST=localhost
# ❌ ARG không available khi runtime
# Application code: process.env.DB_HOST → undefined
```

**Test:**
```bash
$ docker build -t my-app .
$ docker run my-app node -e "console.log(process.env.DB_HOST)"
# Output: undefined
```

### Fix

**Solution: Set ENV từ ARG**
```dockerfile
ARG DB_HOST=localhost
ENV DB_HOST=$DB_HOST
# ENV available khi runtime
```

**Kết quả:**
- **ENV persist**: ENV có trong image
- **Runtime access**: Application có thể access
- **Override**: Có thể override với `-e`

### Result

**Trước:**
- ARG cho runtime config
- **Runtime access**: ❌ Không available
- **Application fails**: Application không start

**Sau:**
- ENV từ ARG cho runtime config
- **Runtime access**: ✅ Available
- **Application works**: Application chạy đúng

### Lesson Learned

1. **ENV cho runtime**: Dùng ENV cho runtime config
2. **ARG cho build-time**: Dùng ARG cho build-time only
3. **ENV từ ARG**: Có thể set ENV từ ARG nếu cần
4. **Document**: Document ENV vs ARG usage

---

## 🏭 PRODUCTION STORY #2: WORKDIR Issues

### Context

**Công ty:** E-commerce, 600 employees
**Hệ thống:** Python microservices với Docker
**Traffic:** 12M requests/day
**Team:** 30 backend engineers

### Problem

**Tháng 6/2023:**
- **Files not found**: Application không tìm thấy files
- **Root cause**: WORKDIR không được set, paths không đúng
- **Impact**: Application crash khi start

**Timeline:**
- **2:00 PM**: Deploy new version
- **2:05 PM**: Application crash (file not found)
- **2:10 PM**: Team investigate
- **2:15 PM**: Root cause: WORKDIR không được set

**Impact:**
- **15 minutes downtime**
- **200K requests failed**
- **Customer complaints**

### Investigation

**Root cause:**
```dockerfile
FROM python:3.9-slim
COPY app.py /app/app.py
COPY config/ /app/config/
CMD ["python", "/app/app.py"]
```

**Vấn đề:**
- **No WORKDIR**: Không có WORKDIR
- **Absolute paths**: Dùng absolute paths (OK)
- **Application code**: Application code dùng relative paths

**Application code:**
```python
# app.py
with open('config/app.conf', 'r') as f:
    # ❌ Tìm ở current directory (root /)
    # Không tìm thấy vì file ở /app/config/app.conf
```

**Test:**
```bash
$ docker run my-app
# Error: FileNotFoundError: config/app.conf
```

### Fix

**Solution: Set WORKDIR**
```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY app.py .
COPY config/ config/
CMD ["python", "app.py"]
```

**Kết quả:**
- **WORKDIR set**: Current directory là /app
- **Relative paths work**: Application code dùng relative paths OK
- **Files found**: Application tìm thấy files

### Result

**Trước:**
- No WORKDIR
- **Relative paths**: ❌ Không work
- **Application fails**: Application crash

**Sau:**
- WORKDIR set
- **Relative paths**: ✅ Work
- **Application works**: Application chạy đúng

### Lesson Learned

1. **Always set WORKDIR**: Best practice
2. **Set early**: Set WORKDIR sớm trong Dockerfile
3. **Use absolute paths**: Hoặc dùng absolute paths
4. **Test paths**: Verify paths trong application

---

## 🎓 TÓM TẮT

### WORKDIR

**Chức năng:**
- Set working directory
- Persist qua layers
- Simplify paths

**Best practices:**
- Set early
- Use absolute paths
- One WORKDIR

### ENV

**Chức năng:**
- Runtime environment variables
- Persist in image
- Overrideable với `-e`

**Best practices:**
- Set defaults
- Document
- Group related

### ARG

**Chức năng:**
- Build-time variables
- Not in image
- Pass với `--build-arg`

**Best practices:**
- Set defaults
- Document
- Use for build-time only

### ENV vs ARG

**ENV:**
- Runtime
- In image
- Override với `-e`

**ARG:**
- Build-time
- Not in image
- Pass với `--build-arg`

**Pattern:**
- ARG → ENV: Build version → Runtime version

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ bạn đã:
- ✅ Hiểu WORKDIR, ENV, ARG
- ✅ Biết khi nào dùng gì
- ✅ Hiểu ENV vs ARG

**Day tiếp theo (Day-015)** sẽ đi sâu vào:
- Multi-stage builds
- Tối ưu image size
- Build patterns

---

## 📚 TÀI LIỆU THAM KHẢO

- WORKDIR: https://docs.docker.com/engine/reference/builder/#workdir
- ENV: https://docs.docker.com/engine/reference/builder/#env
- ARG: https://docs.docker.com/engine/reference/builder/#arg

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-015: Multi-stage-Builds](../Day-015-Multi-stage-Builds/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
