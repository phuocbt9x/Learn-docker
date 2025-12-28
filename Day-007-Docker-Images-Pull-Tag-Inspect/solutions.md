# Day-007: Giải Pháp & Giải Thích

## 🎯 LƯU Ý QUAN TRỌNG

Các giải pháp dưới đây là commands và explanations thực tế. Quan trọng là bạn:

- **Thực hành** các commands trên terminal
- **Hiểu** tại sao mỗi command hoạt động
- **Experiment** với các options khác nhau
- **Apply** best practices trong production

---

## 📝 BÀI TẬP 1: PULL IMAGES

### 1.1. Pull Images từ Docker Hub

**Commands:**
```bash
$ docker pull nginx:latest
$ docker pull nginx:1.21
$ docker pull nginx:alpine
```

**So sánh sizes:**
```bash
$ docker images nginx
REPOSITORY   TAG       IMAGE ID       SIZE
nginx        latest    133MB
nginx        1.21      133MB
nginx        alpine    23MB
```

**Giải thích:**
- `nginx:latest` và `nginx:1.21`: ~133MB (dùng Debian base)
- `nginx:alpine`: ~23MB (dùng Alpine Linux base - nhỏ hơn nhiều)
- Alpine variant nhỏ hơn vì base image nhỏ hơn

### 1.2. Pull từ User/Organization

**Commands:**
```bash
$ docker pull ubuntu:20.04
$ docker pull python:3.9-slim
```

**Image name format:**
```
[registry/][username/]image-name[:tag]
```

**Giải thích:**
- `ubuntu:20.04` → `docker.io/library/ubuntu:20.04` (official image)
- `python:3.9-slim` → `docker.io/library/python:3.9-slim` (official image)
- `myuser/myapp:v1.0` → `docker.io/myuser/myapp:v1.0` (user image)

### 1.3. Pull từ Private Registry

**Command:**
```bash
$ docker pull registry.example.com/my-app:v1.0
```

**Authentication:**
```bash
# Login trước
$ docker login registry.example.com
Username: myuser
Password: ****

# Sau đó pull
$ docker pull registry.example.com/my-app:v1.0
```

**Lưu ý:**
- Private registry thường cần authentication
- Dùng `docker login` để authenticate
- Credentials được lưu trong `~/.docker/config.json`

### 1.4. Pull Process

**Khi pull image, Docker làm gì:**

1. **Check local**: Image đã có local chưa?
   ```bash
   # Nếu có → skip pull
   # Nếu không → tiếp tục
   ```

2. **Connect registry**: Connect đến registry (Docker Hub hoặc private)

3. **Download manifest**: Download image manifest (metadata)

4. **Download layers**: Download các layers chưa có local
   ```bash
   # Layers đã có → skip
   # Layers chưa có → download
   ```

5. **Verify**: Verify image integrity (checksums)

6. **Store**: Lưu image local

**Tại sao một số layers "Already exists"?**

**Layer sharing:**
- Layers được share giữa images
- Nếu layer đã có (từ image khác) → skip download
- **Efficient**: Tiết kiệm bandwidth, storage

**Ví dụ:**
```bash
$ docker pull nginx:latest
# Download layers

$ docker pull nginx:1.21
# Một số layers "Already exists" (shared với nginx:latest)
```

---

## 📝 BÀI TẬP 2: IMAGE TAGS

### 2.1. Tag Naming

**Giải thích tags:**

- `nginx:latest`: Latest version (default tag)
- `nginx:1.21`: Version 1.21
- `nginx:1.21-alpine`: Version 1.21, Alpine variant
- `my-app:v1.0.0`: Version 1.0.0 (semantic versioning)
- `my-app:production`: Production environment tag

**Tag nào nên dùng trong production?**

**✅ Nên dùng:**
- `my-app:v1.0.0` (specific version)
- `my-app:1.0.0` (semantic versioning)
- `my-app:production` (environment tag, nhưng phải pin version)

**❌ Không nên dùng:**
- `my-app:latest` (mutable, không reliable)

**Lý do:**
- Specific versions: Immutable, reliable
- Latest tag: Mutable, có thể thay đổi

### 2.2. Latest Tag

**Latest tag là gì?**

**`latest`** là **default tag** khi không specify tag:
```bash
$ docker pull nginx
# Tương đương: docker pull nginx:latest
```

**Tại sao không nên dùng latest trong production?**

**Vấn đề:**
- **Mutable**: Latest tag có thể thay đổi
- **Unpredictable**: Không biết version nào sẽ được deploy
- **Risk**: Có thể deploy version chưa test

**Ví dụ:**
```bash
# Day 1
$ docker pull my-app:latest
# → Version 1.0.0

# Day 2 (new build)
$ docker pull my-app:latest
# → Version 1.1.0 (different version!)
```

**Khi nào có thể dùng latest?**

**✅ Có thể dùng:**
- Development environment
- Testing
- Local development

**❌ Không nên dùng:**
- Production
- Staging (nên pin version)

### 2.3. Tag Conventions

**Semantic Versioning:**
- Format: `vMAJOR.MINOR.PATCH`
- Example: `v1.0.0`, `v1.1.0`, `v2.0.0`
- **Use case**: Version tracking

**Environment Tags:**
- `dev`: Development
- `staging`: Staging
- `production`: Production
- **Use case**: Environment-specific deployments
- **Lưu ý**: Vẫn nên pin version (e.g., `my-app:v1.0.0-production`)

**Date Tags:**
- `2023-01-15`: Date-based
- `20230115`: Date compact
- **Use case**: Date-based releases

### 2.4. Tag Management

**List tags của image:**

**Không có command trực tiếp:**
- Phải check trên registry website
- Hoặc dùng registry API

**Check local tags:**
```bash
$ docker images nginx
REPOSITORY   TAG       IMAGE ID
nginx        latest    abc123...
nginx        1.21      def456...
nginx        alpine    ghi789...
```

**Một image có thể có nhiều tags không?**

**✅ CÓ:**
```bash
$ docker tag nginx:latest my-nginx:v1.0
$ docker tag nginx:latest my-nginx:latest
$ docker tag nginx:latest my-nginx:production

$ docker images nginx
REPOSITORY   TAG          IMAGE ID
nginx        latest       abc123...
my-nginx     v1.0         abc123...  # ← Same image ID
my-nginx     latest       abc123...  # ← Same image ID
my-nginx     production   abc123...  # ← Same image ID
```

**Tag có tốn storage không?**

**❌ KHÔNG:**
- Tag chỉ là **pointer** đến image
- Không tốn storage
- Một image có thể có nhiều tags

---

## 📝 BÀI TẬP 3: TAG IMAGES

### 3.1. Tag Local Images

**Commands:**
```bash
# Tag nginx:latest thành my-nginx:v1.0
$ docker tag nginx:latest my-nginx:v1.0

# Tag với registry
$ docker tag nginx:latest registry.example.com/my-nginx:v1.0

# Verify
$ docker images | grep nginx
REPOSITORY                    TAG       IMAGE ID
nginx                         latest    abc123...
my-nginx                      v1.0      abc123...  # ← Same ID
registry.example.com/my-nginx v1.0      abc123...  # ← Same ID
```

### 3.2. Tag After Build

**Commands:**
```bash
# Build image
$ docker build -t my-app:latest .

# Tag với version
$ docker tag my-app:latest my-app:v1.0.0

# Tag với environment
$ docker tag my-app:latest my-app:production

# Tag với registry
$ docker tag my-app:latest registry.example.com/my-app:v1.0.0

# Verify
$ docker images my-app
REPOSITORY                    TAG          IMAGE ID
my-app                        latest       abc123...
my-app                        v1.0.0       abc123...
my-app                        production   abc123...
registry.example.com/my-app   v1.0.0       abc123...
```

### 3.3. Multiple Tags

**Một image có thể có nhiều tags:**

**✅ CÓ:**
```bash
$ docker tag nginx:latest my-nginx:v1.0
$ docker tag nginx:latest my-nginx:latest
$ docker tag nginx:latest my-nginx:production

$ docker images | grep nginx
REPOSITORY   TAG          IMAGE ID
nginx        latest       abc123...
my-nginx     v1.0         abc123...  # ← All same ID
my-nginx     latest       abc123...
my-nginx     production   abc123...
```

**Remove một tag:**

```bash
$ docker rmi my-nginx:v1.0
# Chỉ remove tag, không remove image
# Image vẫn còn (có tags khác)
```

**Remove image (khi chỉ còn 1 tag):**
```bash
$ docker rmi my-nginx:latest
# Remove tag và image (nếu là tag cuối cùng)
```

### 3.4. Retag Images

**Retag process:**
```bash
# Retag nginx:1.21 thành nginx:stable
$ docker tag nginx:1.21 nginx:stable

# Retag my-app:v1.0 thành my-app:latest
$ docker tag my-app:v1.0 my-app:latest
```

**Giải thích:**
- Retag = tạo tag mới point đến cùng image
- Tag cũ vẫn còn (nếu không remove)
- Không tốn storage (tag chỉ là pointer)

---

## 📝 BÀI TẬP 4: INSPECT IMAGES

### 4.1. Basic Inspect

**Command:**
```bash
$ docker inspect nginx:latest
```

**Các fields quan trọng:**

- **Id**: Image ID (SHA256 hash)
- **RepoTags**: Tags của image
- **RepoDigests**: Digests của image
- **Created**: Thời gian tạo
- **Size**: Image size
- **Architecture**: Architecture (amd64, arm64, etc.)
- **Os**: Operating system (linux, windows)
- **Config**: Image configuration (Env, Cmd, ExposedPorts, etc.)

**Image ID vs Digest:**

- **Image ID**: Local identifier (có thể thay đổi)
- **Digest**: Content-based identifier (immutable, SHA256)
- **Digest reliable hơn** cho production (không thay đổi)

### 4.2. Inspect Specific Fields

**Commands:**
```bash
# Image ID
$ docker inspect --format='{{.Id}}' nginx:latest

# Created date
$ docker inspect --format='{{.Created}}' nginx:latest

# Size
$ docker inspect --format='{{.Size}}' nginx:latest

# Architecture
$ docker inspect --format='{{.Architecture}}' nginx:latest

# OS
$ docker inspect --format='{{.Os}}' nginx:latest

# Environment variables
$ docker inspect --format='{{.Config.Env}}' nginx:latest

# Exposed ports
$ docker inspect --format='{{.Config.ExposedPorts}}' nginx:latest

# CMD
$ docker inspect --format='{{.Config.Cmd}}' nginx:latest

# Entrypoint
$ docker inspect --format='{{.Config.Entrypoint}}' nginx:latest
```

### 4.3. Image History

**Command:**
```bash
$ docker history nginx:latest
```

**Output:**
```
IMAGE          CREATED        CREATED BY                                      SIZE
abc123...      2 weeks ago    /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B
def456...      2 weeks ago    /bin/sh -c #(nop)  EXPOSE 80                    0B
ghi789...      2 weeks ago    /bin/sh -c apt-get update && apt-get install…   50MB
jkl012...      2 weeks ago    /bin/sh -c #(nop)  ADD file:... in /             10MB
mno345...      2 weeks ago    /bin/sh -c #(nop)  MAINTAINER NGINX Docker M…   0B
pqr678...      2 weeks ago    /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
```

**Giải thích:**
- **IMAGE**: Layer ID
- **CREATED**: Thời gian tạo layer
- **CREATED BY**: Command tạo layer
- **SIZE**: Layer size

**Layers nào lớn nhất?**

- Layers install packages thường lớn nhất
- Base image layers cũng lớn
- Metadata layers (CMD, EXPOSE) = 0B

**Xem command tạo layer:**
```bash
$ docker history nginx:latest --no-trunc
# Hiển thị full command
```

### 4.4. Compare Images

**Inspect và so sánh:**
```bash
$ docker inspect nginx:latest
$ docker inspect nginx:alpine
```

**So sánh sizes:**
```bash
$ docker images nginx
REPOSITORY   TAG       SIZE
nginx        latest    133MB
nginx        alpine    23MB
```

**So sánh layers:**
```bash
$ docker history nginx:latest
$ docker history nginx:alpine
```

**Tại sao alpine nhỏ hơn?**

- **Base image**: Alpine Linux (~5MB) vs Debian (~70MB)
- **Packages**: Alpine dùng musl libc (nhỏ hơn glibc)
- **Optimization**: Alpine được optimize cho size

---

## 📝 BÀI TẬP 5: IMAGE MANAGEMENT

### 5.1. List Images

**List all images:**
```bash
$ docker images
```

**List với filter:**
```bash
# Filter by name
$ docker images nginx

# Filter by tag
$ docker images nginx:latest

# Filter dangling images
$ docker images -f "dangling=true"

# Filter before date
$ docker images -f "before=nginx:latest"
```

**List với format:**
```bash
$ docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

### 5.2. Remove Images

**Remove image:**
```bash
$ docker rmi nginx:latest
# Hoặc
$ docker image rm nginx:latest
```

**Remove by ID:**
```bash
$ docker rmi abc123def456
```

**Force remove:**
```bash
$ docker rmi -f nginx:latest
# Remove even if used by containers
```

**Remove multiple:**
```bash
$ docker rmi nginx:latest nginx:1.21
```

**Remove all unused images:**
```bash
$ docker image prune
# Hoặc
$ docker image prune -a  # Remove all unused images
```

### 5.3. Image Size

**Check image size:**
```bash
$ docker images
REPOSITORY   TAG       SIZE
nginx        latest    133MB
ubuntu       20.04     72MB
```

**Detailed size:**
```bash
$ docker system df
```

**Size vs VirtualSize:**

- **Size**: Compressed size (trong registry)
- **VirtualSize**: Uncompressed size (local)
- **Shared size**: Size của shared layers

**Giảm image sizes:**

1. **Use slim base images**: alpine, slim variants
2. **Multi-stage builds**: Remove build artifacts
3. **Optimize layers**: Combine commands
4. **Remove unnecessary packages**: Chỉ install cần thiết

### 5.4. Image Search

**Search trên Docker Hub:**
```bash
$ docker search nginx
```

**Output:**
```
NAME                      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
nginx                     Official build of Nginx.                       18000     [OK]
jwilder/nginx-proxy      Automated nginx reverse proxy                  2000
...
```

**Search options:**
```bash
# Limit results
$ docker search nginx --limit 5

# Filter official
$ docker search nginx --filter "is-official=true"

# Filter automated
$ docker search nginx --filter "is-automated=true"
```

---

## 📝 BÀI TẬP 6: PRACTICAL SCENARIOS

### Scenario 1: Development Workflow

**6.1. Setup development images:**
```bash
$ docker pull python:3.9-slim
$ docker pull node:16-alpine

# Tag với tên dễ nhớ
$ docker tag python:3.9-slim dev-python:3.9
$ docker tag node:16-alpine dev-node:16

# Verify
$ docker images | grep -E "python|node"
```

**6.2. Image inspection:**
```bash
# Inspect python:3.9-slim
$ docker inspect python:3.9-slim

# Environment variables
$ docker inspect --format='{{.Config.Env}}' python:3.9-slim

# Exposed ports
$ docker inspect --format='{{.Config.ExposedPorts}}' python:3.9-slim

# CMD/ENTRYPOINT
$ docker inspect --format='{{.Config.Cmd}}' python:3.9-slim
$ docker inspect --format='{{.Config.Entrypoint}}' python:3.9-slim
```

**6.3. Cleanup:**
```bash
# Remove unused images
$ docker image prune

# Check disk usage
$ docker system df

# Best practices:
# - Regular cleanup
# - Remove old/unused images
# - Monitor disk usage
```

### Scenario 2: Production Deployment

**6.4. Tag strategy:**

**Semantic versioning:**
```bash
my-app:v1.0.0
my-app:v1.1.0
my-app:v2.0.0
```

**Environment tags (với version):**
```bash
my-app:v1.0.0-dev
my-app:v1.0.0-staging
my-app:v1.0.0-production
```

**Best practices:**
- Pin specific versions trong production
- Không dùng `latest`
- Use semantic versioning
- Tag với environment (nhưng vẫn pin version)

**6.5. Version pinning:**

**Tại sao cần pin versions?**
- Predictable deployments
- Reproducible builds
- Rollback dễ dàng

**Làm thế nào pin versions?**
```yaml
# docker-compose.yml
services:
  app:
    image: my-app:v1.0.0  # ← Pin version
```

```bash
# Kubernetes
apiVersion: v1
kind: Pod
spec:
  containers:
  - image: my-app:v1.0.0  # ← Pin version
```

**Track versions:**
- Document versions trong deployment configs
- Use version control (Git)
- Tag releases
- Maintain changelog

---

## 📝 BÀI TẬP 7: IMAGE LAYERS

### 7.1. Layer Structure

**Inspect và history:**
```bash
$ docker inspect nginx:latest
$ docker history nginx:latest
```

**Layers:**
- **Base image layers**: Debian/Ubuntu base
- **Package installation layers**: Install nginx, dependencies
- **Configuration layers**: Copy config files
- **Metadata layers**: CMD, EXPOSE (0B)

### 7.2. Layer Sharing

**Pull và so sánh:**
```bash
$ docker pull nginx:latest
$ docker pull nginx:alpine

$ docker images nginx
REPOSITORY   TAG       SIZE
nginx        latest    133MB
nginx        alpine    23MB
```

**Layers được share:**
- Nếu có layers giống nhau → share
- Nginx latest và alpine có base khác → không share base layers
- Nhưng có thể share một số application layers

**Storage saved:**
- Depends on shared layers
- Typically 20-50% nếu share base

### 7.3. Layer Optimization

**Giảm số layers:**
```dockerfile
# Bad: Nhiều layers
RUN apt-get update
RUN apt-get install -y package1
RUN apt-get install -y package2

# Good: Combine
RUN apt-get update && \
    apt-get install -y package1 package2 && \
    apt-get clean
```

**Giảm layer size:**
- Remove unnecessary files
- Clean package cache
- Use .dockerignore

### 7.4. Layer Inspection

**Inspect từng layer:**
```bash
$ docker history nginx:latest --no-trunc
# Xem full commands

# Identify optimization:
# - Combine RUN commands
# - Remove cache
# - Remove unnecessary files
```

---

## 📝 BÀI TẬP 8: TROUBLESHOOTING

### 8.1. Pull Failures

**Root cause:**
- Network issues
- Firewall blocking
- DNS issues
- Registry unavailable

**Debug:**
```bash
# Check internet
$ ping 8.8.8.8

# Check DNS
$ nslookup registry-1.docker.io

# Check firewall
$ sudo ufw status

# Check Docker logs
$ sudo journalctl -u docker -n 50
```

**Fix:**
- Fix network connection
- Configure DNS
- Allow Docker ports trong firewall
- Check registry status

### 8.2. Tag Not Found

**Root cause:**
- Tag không tồn tại
- Typo trong tag name
- Registry không có tag đó

**Check available tags:**
- Check trên Docker Hub website
- Hoặc dùng registry API

**Fix:**
- Verify tag name
- Check available tags
- Use correct tag

### 8.3. Image Too Large

**Giảm image size:**

1. **Use slim base images:**
   ```dockerfile
   FROM python:3.9-slim  # Thay vì python:3.9
   ```

2. **Multi-stage builds:**
   ```dockerfile
   FROM node:16 as builder
   # Build
   
   FROM node:16-slim
   COPY --from=builder /app/dist ./dist
   ```

3. **Remove unnecessary files:**
   ```dockerfile
   RUN apt-get update && \
       apt-get install -y package && \
       apt-get clean && \
       rm -rf /var/lib/apt/lists/*
   ```

### 8.4. Tag Confusion

**Prevent:**

1. **Pin versions:**
   ```yaml
   image: my-app:v1.0.0  # ← Pin version
   ```

2. **Tag strategy:**
   - Latest chỉ cho dev
   - Production dùng specific versions

3. **CI/CD:**
   - Automate version tagging
   - Pin versions trong deployment

---

## ✅ TỔNG KẾT

Các bài tập này giúp bạn:

1. **Pull images**: Từ Docker Hub và private registries
2. **Understand tags**: Latest, versions, conventions
3. **Tag images**: Tag, retag, multiple tags
4. **Inspect images**: Thông tin chi tiết, layers, history
5. **Manage images**: List, remove, cleanup, search

**Key takeaways:**
- **Tags**: Latest mutable, pin versions trong production
- **Layers**: Share giữa images, optimize để giảm size
- **Inspection**: Hiểu image structure, config
- **Management**: Regular cleanup, monitor disk usage

---

**Chúc bạn học tốt! Tiếp tục với Day-008 để học Container Lifecycle.**

