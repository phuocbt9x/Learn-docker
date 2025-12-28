# Day-007: Docker Images - Pull, Tag, Inspect

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được Docker Image là gì và cách quản lý
- Biết cách pull images từ Docker Hub và các registries khác
- Hiểu được image tags và versioning
- Biết cách tag và retag images
- Biết cách inspect images để xem thông tin chi tiết
- Hiểu được image layers và cách chúng được quản lý

---

## 📖 PHẦN 1: DOCKER IMAGE LÀ GÌ?

### 1.1. Image là gì?

**Docker Image** là một **read-only template** dùng để tạo containers. Image chứa:

- **Application code**
- **Dependencies** (libraries, frameworks)
- **Runtime environment** (Python, Node.js, etc.)
- **System libraries**
- **Configuration files**
- **Metadata** (environment variables, exposed ports, etc.)

**Đặc điểm:**
- **Read-only**: Không thể modify sau khi tạo
- **Layered**: Được tạo từ nhiều layers
- **Immutable**: Một khi tạo, không thay đổi
- **Reusable**: Có thể dùng để tạo nhiều containers

### 1.2. Image Format

**Image name format:**
```
[registry/][username/]image-name[:tag]
```

**Ví dụ:**
- `nginx` → `docker.io/library/nginx:latest`
- `nginx:1.21` → `docker.io/library/nginx:1.21`
- `myuser/myapp:v1.0` → `docker.io/myuser/myapp:v1.0`
- `registry.example.com/myapp:latest` → Private registry

**Components:**
- **Registry**: docker.io (Docker Hub), hoặc private registry
- **Username**: Organization/user name (optional)
- **Image name**: Tên image
- **Tag**: Version (default: latest)

### 1.3. Image Layers

**Image được tạo từ layers:**
```
Image: nginx:latest
├── Layer 1: Base image (debian:bullseye-slim)
├── Layer 2: Install nginx
├── Layer 3: Copy config files
└── Layer 4: Set CMD
```

**Lợi ích:**
- **Sharing**: Layers được share giữa images
- **Caching**: Layers được cache khi build
- **Efficiency**: Chỉ pull/push layers thay đổi

### 1.4. Image Storage

**Images được lưu ở đâu?**

**Local storage:**
- `/var/lib/docker/image/` (Linux)
- `~/Library/Containers/com.docker.docker/Data/` (macOS)
- `C:\ProgramData\docker\` (Windows)

**Registry storage:**
- Docker Hub (public)
- Private registries (Harbor, GitLab Registry, AWS ECR, etc.)

---

## 📥 PHẦN 2: PULL IMAGES

### 2.1. Pull từ Docker Hub

**Basic pull:**
```bash
$ docker pull nginx
```

**Giải thích:**
- Pull image `nginx` với tag `latest` (default)
- Tương đương: `docker pull nginx:latest`
- Pull từ Docker Hub (docker.io)

**Pull với tag cụ thể:**
```bash
$ docker pull nginx:1.21
$ docker pull nginx:1.21-alpine
$ docker pull nginx:alpine
```

**Pull từ user/organization:**
```bash
$ docker pull myuser/myapp:v1.0
```

### 2.2. Pull từ Private Registry

**Pull từ private registry:**
```bash
$ docker pull registry.example.com/myapp:latest
```

**Với authentication:**
```bash
$ docker login registry.example.com
Username: myuser
Password: ****
$ docker pull registry.example.com/myapp:latest
```

### 2.3. Pull Options

**Pull và không hiển thị output:**
```bash
$ docker pull -q nginx
```

**Pull tất cả tags:**
```bash
# Không có option trực tiếp
# Phải pull từng tag một
$ docker pull nginx:latest
$ docker pull nginx:1.21
$ docker pull nginx:alpine
```

**Pull và verify:**
```bash
$ docker pull nginx
$ docker images nginx
# Verify image đã được pull
```

### 2.4. Pull Process

**Khi pull image, Docker làm gì?**

1. **Check local**: Image đã có local chưa?
2. **Connect registry**: Connect đến registry
3. **Download manifest**: Download image manifest
4. **Download layers**: Download các layers chưa có
5. **Verify**: Verify image integrity
6. **Store**: Lưu image local

**Ví dụ:**
```bash
$ docker pull nginx:1.21
1.21: Pulling from library/nginx
a2abf6c4d29d: Already exists      # Layer đã có
a9edb18cadd1: Already exists      # Layer đã có
589b7251471a: Pull complete       # Pull layer mới
186b1aaa4aa6: Pull complete       # Pull layer mới
b4df32aa5a72: Pull complete       # Pull layer mới
a0bcbecc962e: Pull complete       # Pull layer mới
Digest: sha256:...
Status: Downloaded newer image for nginx:1.21
```

---

## 🏷️ PHẦN 3: IMAGE TAGS

### 3.1. Tags là gì?

**Tag** là một **label** để identify version của image.

**Ví dụ:**
- `nginx:latest` → Latest version
- `nginx:1.21` → Version 1.21
- `nginx:1.21-alpine` → Version 1.21, Alpine variant
- `myapp:v1.0.0` → Version 1.0.0
- `myapp:production` → Production tag

**Đặc điểm:**
- **Mutable**: Tag có thể point đến image khác
- **Multiple tags**: Một image có thể có nhiều tags
- **Default**: `latest` là default tag

### 3.2. Tag Naming Conventions

**Semantic Versioning:**
- `v1.0.0` → Major.Minor.Patch
- `v1.0` → Major.Minor
- `1.0.0` → Không có 'v' prefix

**Environment tags:**
- `latest` → Latest version
- `stable` → Stable version
- `dev` → Development version
- `production` → Production version

**Variant tags:**
- `alpine` → Alpine Linux variant
- `slim` → Slim variant
- `full` → Full variant

**Date tags:**
- `2023-01-15` → Date-based
- `20230115` → Date compact

### 3.3. Latest Tag

**Latest tag là gì?**

**`latest`** là **default tag** khi không specify tag.

**Ví dụ:**
```bash
$ docker pull nginx
# Tương đương: docker pull nginx:latest
```

**Lưu ý:**
- **Latest không phải newest**: Latest tag có thể không phải version mới nhất
- **Mutable**: Latest tag có thể thay đổi
- **Production**: Không nên dùng `latest` trong production

**Best practices:**
- **Development**: Có thể dùng `latest`
- **Production**: Dùng specific version tags
- **CI/CD**: Pin specific versions

### 3.4. Tag Management

**List tags của image:**
```bash
# Không có command trực tiếp
# Phải check trên registry website
# Hoặc dùng API
```

**Check local tags:**
```bash
$ docker images nginx
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
nginx        latest    abc123...      2 weeks ago    133MB
nginx        1.21      def456...      2 weeks ago    133MB
nginx        alpine    ghi789...      2 weeks ago    23MB
```

---

## 🏷️ PHẦN 4: TAG IMAGES

### 4.1. Tag Local Images

**Tag image:**
```bash
$ docker tag source-image:tag target-image:tag
```

**Ví dụ:**
```bash
# Tag nginx:latest thành my-nginx:v1.0
$ docker tag nginx:latest my-nginx:v1.0

# Tag với registry
$ docker tag nginx:latest registry.example.com/my-nginx:v1.0
```

**Multiple tags:**
```bash
# Một image có thể có nhiều tags
$ docker tag nginx:latest my-nginx:v1.0
$ docker tag nginx:latest my-nginx:latest
$ docker tag nginx:latest my-nginx:production
```

### 4.2. Tag After Build

**Tag khi build:**
```bash
$ docker build -t my-app:v1.0 .
$ docker build -t my-app:latest .
$ docker build -t registry.example.com/my-app:v1.0 .
```

**Tag sau khi build:**
```bash
$ docker build -t my-app:latest .
$ docker tag my-app:latest my-app:v1.0
$ docker tag my-app:latest registry.example.com/my-app:v1.0
```

### 4.3. Retag Images

**Retag (thay đổi tag):**
```bash
# Tag mới
$ docker tag nginx:1.21 nginx:stable

# Remove tag cũ (nếu cần)
$ docker rmi nginx:1.21
# Chỉ remove tag, không remove image nếu có tags khác
```

**Lưu ý:**
- Tag chỉ là pointer → không tốn storage
- Một image có thể có nhiều tags
- Remove tag không remove image (nếu có tags khác)

### 4.4. Tag Best Practices

**Best practices:**

1. **Use semantic versioning:**
   ```bash
   my-app:v1.0.0
   my-app:v1.1.0
   my-app:v2.0.0
   ```

2. **Tag important versions:**
   ```bash
   my-app:latest
   my-app:stable
   my-app:production
   ```

3. **Tag builds:**
   ```bash
   my-app:build-123
   my-app:commit-abc123
   ```

4. **Avoid latest in production:**
   - Dùng specific versions
   - Pin versions trong deployment

---

## 🔍 PHẦN 5: INSPECT IMAGES

### 5.1. Inspect Image

**Basic inspect:**
```bash
$ docker inspect nginx:latest
```

**Output (JSON):**
```json
[
  {
    "Id": "sha256:abc123...",
    "RepoTags": ["nginx:latest"],
    "RepoDigests": ["nginx@sha256:def456..."],
    "Parent": "",
    "Comment": "",
    "Created": "2023-01-15T10:00:00Z",
    "Container": "",
    "ContainerConfig": {
      "Hostname": "",
      "User": "",
      "Env": ["PATH=/usr/local/sbin:..."],
      "Cmd": ["nginx", "-g", "daemon off;"],
      ...
    },
    "DockerVersion": "24.0.0",
    "Author": "",
    "Config": {
      "Image": "sha256:...",
      "Env": ["PATH=..."],
      "Cmd": ["nginx", "-g", "daemon off;"],
      "ExposedPorts": {"80/tcp": {}},
      "Volumes": null,
      "WorkingDir": "",
      "Entrypoint": null,
      ...
    },
    "Architecture": "amd64",
    "Os": "linux",
    "Size": 133000000,
    "VirtualSize": 133000000,
    ...
  }
]
```

### 5.2. Inspect Specific Fields

**Inspect với format:**
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

### 5.3. Inspect Image History

**Image history:**
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
- **CREATED**: Thời gian tạo
- **CREATED BY**: Command tạo layer
- **SIZE**: Layer size

### 5.4. Inspect Image Layers

**List image layers:**
```bash
# Không có command trực tiếp
# Dùng docker history
$ docker history nginx:latest --no-trunc
```

**Inspect layer details:**
```bash
# Inspect từng layer
$ docker inspect <layer-id>
```

**Check layer sharing:**
```bash
# Images share layers
$ docker images
# Xem VirtualSize vs Size
```

---

## 📊 PHẦN 6: IMAGE MANAGEMENT

### 6.1. List Images

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

### 6.2. Remove Images

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

### 6.3. Image Size

**Check image size:**
```bash
$ docker images
REPOSITORY   TAG       IMAGE ID       SIZE
nginx        latest    abc123...      133MB
ubuntu       20.04     def456...      72MB
```

**Detailed size:**
```bash
$ docker system df
```

**Size breakdown:**
- **Size**: Compressed size (trong registry)
- **VirtualSize**: Uncompressed size (local)
- **Shared size**: Size của shared layers

### 6.4. Image Search

**Search trên Docker Hub:**
```bash
$ docker search nginx
```

**Output:**
```
NAME                      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
nginx                     Official build of Nginx.                       18000     [OK]
jwilder/nginx-proxy       Automated nginx reverse proxy                  2000
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

## 🏭 PRODUCTION STORY #1: Image Tag Confusion tại Production

### Context

**Công ty:** SaaS platform, 300 employees
**Hệ thống:** Production deployment với Docker
**Team:** 20 DevOps engineers
**Issue:** Deploy sai version do tag confusion

### Problem

**Tháng 6/2023:**
- Team deploy `my-app:latest` lên production
- **Unexpected behavior**: Application có bugs
- **Root cause**: `latest` tag đã được update với version mới (chưa test)

**Timeline:**
- **10:00 AM**: Developer push code mới, build image `my-app:latest`
- **10:30 AM**: CI/CD deploy `my-app:latest` lên production
- **11:00 AM**: Production issues reported
- **11:30 AM**: Team investigate
- **12:00 PM**: Root cause: `latest` tag changed

**Impact:**
- **30 minutes downtime**
- **Customer complaints**
- **Revenue loss**

### Investigation

**Root cause:**
```bash
# Before
$ docker images my-app
REPOSITORY   TAG       IMAGE ID
my-app       latest    abc123...  # Version 1.0.0

# After new build
$ docker images my-app
REPOSITORY   TAG       IMAGE ID
my-app       latest    def456...  # Version 1.1.0 (new, untested)
```

**Vấn đề:**
- `latest` tag là **mutable** → có thể thay đổi
- Deploy `latest` → có thể deploy version chưa test
- **Không pin specific version** trong production

### Fix

**Solution 1: Pin Specific Versions**
```yaml
# docker-compose.yml hoặc Kubernetes
services:
  app:
    image: my-app:v1.0.0  # ← Pin specific version
    # Không dùng latest
```

**Solution 2: Use Semantic Versioning**
```bash
# Build với version tags
$ docker build -t my-app:v1.0.0 .
$ docker build -t my-app:v1.1.0 .

# Deploy specific version
$ docker run my-app:v1.0.0
```

**Solution 3: Tag Strategy**
```bash
# Latest chỉ cho development
my-app:latest  # Development only

# Production dùng specific versions
my-app:v1.0.0  # Production
my-app:v1.1.0  # Production
```

### Result

**Trước:**
- Dùng `latest` tag trong production
- **3-4 incidents** mỗi tháng do tag confusion

**Sau:**
- Pin specific versions
- **Zero incidents** trong 6 tháng
- Better deployment control

### Lesson Learned

1. **Không dùng latest trong production**: Latest tag mutable, không reliable
2. **Pin specific versions**: Dùng semantic versioning
3. **Tag strategy**: Latest cho dev, specific versions cho production
4. **CI/CD**: Automate version tagging

---

## 🏭 PRODUCTION STORY #2: Image Pull Failures do Network Issues

### Context

**Công ty:** E-commerce, 500 employees
**Hệ thống:** Kubernetes cluster, 50 nodes
**Traffic:** 5M requests/day
**Team:** 30 DevOps engineers

### Problem

**Tháng 8/2023:**
- **Image pull failures** trên 10 nodes
- **Containers không start được**
- **Root cause**: Network issues, Docker Hub rate limiting

**Timeline:**
- **2:00 AM**: Auto-scaling trigger → tạo 10 nodes mới
- **2:05 AM**: Nodes pull images từ Docker Hub
- **2:10 AM**: **Rate limit exceeded** → pull failures
- **2:15 AM**: Containers không start → service degradation
- **2:30 AM**: Team investigate
- **3:00 AM**: Fix và recover

**Impact:**
- **1 hour service degradation**
- **100K requests failed**
- **Customer complaints**

### Investigation

**Root cause:**
```bash
# Error logs
Error response from daemon: toomanyrequests: You have reached your pull rate limit
```

**Docker Hub rate limits:**
- **Anonymous**: 100 pulls/6 hours
- **Authenticated**: 200 pulls/6 hours
- **Paid**: Unlimited

**Vấn đề:**
- 10 nodes × 5 images = 50 pulls
- Nhiều nodes pull cùng lúc → rate limit
- **Không có local registry cache**

### Fix

**Solution 1: Use Private Registry**
```bash
# Setup private registry (Harbor, GitLab Registry)
# Pull từ private registry thay vì Docker Hub
$ docker pull registry.internal.com/my-app:v1.0.0
```

**Solution 2: Pre-pull Images**
```bash
# Pre-pull images trên nodes trước khi cần
$ docker pull my-app:v1.0.0
$ docker pull nginx:latest
```

**Solution 3: Image Caching**
```bash
# Dùng image cache trong cluster
# Kubernetes: imagePullPolicy: IfNotPresent
```

**Solution 4: Docker Hub Authentication**
```bash
# Authenticate với Docker Hub
$ docker login
# Tăng rate limit từ 100 → 200 pulls/6 hours
```

### Result

**Trước:**
- Pull từ Docker Hub trực tiếp
- **5-10 pull failures** mỗi tháng
- Rate limit issues

**Sau:**
- Private registry với caching
- **Zero pull failures** trong 6 tháng
- Faster pulls (local registry)

### Lesson Learned

1. **Private registry**: Quan trọng cho production
2. **Rate limiting**: Docker Hub có rate limits
3. **Image caching**: Pre-pull hoặc cache images
4. **Monitoring**: Monitor pull failures

---

## 🎓 TÓM TẮT

### Docker Images

**Là gì:**
- Read-only template
- Layered structure
- Immutable

**Format:**
- `[registry/][username/]image-name[:tag]`
- Default registry: docker.io (Docker Hub)
- Default tag: latest

### Pull Images

**Commands:**
- `docker pull <image>`: Pull image
- `docker pull <image>:<tag>`: Pull với tag cụ thể
- `docker pull <registry>/<image>`: Pull từ private registry

**Process:**
- Check local → Connect registry → Download manifest → Download layers → Verify → Store

### Tag Images

**Commands:**
- `docker tag <source> <target>`: Tag image
- `docker build -t <image>:<tag>`: Tag khi build

**Best practices:**
- Semantic versioning
- Không dùng latest trong production
- Tag important versions

### Inspect Images

**Commands:**
- `docker inspect <image>`: Inspect image
- `docker history <image>`: Image history
- `docker images`: List images

**Information:**
- Image ID, tags, size
- Layers, history
- Config, environment, ports

### Image Management

**Commands:**
- `docker images`: List images
- `docker rmi <image>`: Remove image
- `docker image prune`: Cleanup unused images
- `docker search <term>`: Search images

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ bạn đã:
- ✅ Hiểu Docker Images
- ✅ Biết cách pull, tag, inspect images
- ✅ Hiểu image management

**Day tiếp theo (Day-008)** sẽ đi sâu vào:
- Container Lifecycle: Create, Start, Stop, Remove
- Container states
- Container management

---

## 📚 TÀI LIỆU THAM KHẢO

- Docker Images: https://docs.docker.com/engine/reference/commandline/images/
- Docker Pull: https://docs.docker.com/engine/reference/commandline/pull/
- Docker Tag: https://docs.docker.com/engine/reference/commandline/tag/

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-008: Container-Lifecycle-Create-Start-Stop-Remove](../Day-008-Container-Lifecycle-Create-Start-Stop-Remove/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
