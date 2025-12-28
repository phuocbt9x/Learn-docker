# Day-010: Docker Hub & Registry Basics

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được Docker Hub là gì và cách sử dụng
- Biết cách push/pull images từ Docker Hub
- Hiểu được Docker Registry và cách hoạt động
- Biết cách setup và sử dụng private registry
- Hiểu được image tags và versioning trong registry
- Biết cách authenticate với registries

---

## 📖 PHẦN 1: DOCKER HUB LÀ GÌ?

### 1.1. Docker Hub là gì?

**Docker Hub** là **public registry** lớn nhất cho Docker images:
- **Public images**: Hàng triệu images public
- **Official images**: Images được maintain bởi Docker và vendors
- **Private repositories**: Lưu trữ private images
- **Automated builds**: Tự động build từ GitHub/Bitbucket

**URL:** https://hub.docker.com

**Đặc điểm:**
- **Free tier**: 1 private repository, unlimited public
- **Paid tiers**: Unlimited private repositories
- **Rate limits**: Anonymous (100 pulls/6h), Authenticated (200 pulls/6h)
- **CDN**: Global CDN cho fast downloads

### 1.2. Tại sao Docker Hub tồn tại?

**Vấn đề:**
- Developers cần share images
- Không có central place để store images
- Khó tìm images

**Docker Hub giải quyết:**
- **Central repository**: Một nơi để store và share images
- **Discovery**: Dễ tìm images
- **Versioning**: Track versions với tags
- **Collaboration**: Share images với team

### 1.3. Khi nào dùng Docker Hub?

**Use cases:**

1. **Public images**: Share images với community
2. **Development**: Quick access to images
3. **CI/CD**: Pull images trong CI/CD pipelines
4. **Learning**: Explore existing images

**Khi không nên dùng:**
- **Private/proprietary code**: Cần private registry
- **High security requirements**: Cần self-hosted registry
- **High volume**: Rate limits có thể là vấn đề

---

## 🔐 PHẦN 2: AUTHENTICATION

### 2.1. Docker Login

**Login to Docker Hub:**
```bash
$ docker login
Username: myusername
Password: ****
Login Succeeded
```

**Login với username:**
```bash
$ docker login -u myusername
Password: ****
```

**Login với token:**
```bash
$ echo "mytoken" | docker login -u myusername --password-stdin
```

### 2.2. Docker Logout

**Logout:**
```bash
$ docker logout
```

**Logout từ specific registry:**
```bash
$ docker logout registry.example.com
```

### 2.3. Credentials Storage

**Credentials được lưu ở đâu?**

**Linux:**
- `~/.docker/config.json`

**macOS:**
- Keychain (default)
- Hoặc `~/.docker/config.json`

**Windows:**
- Credential Manager
- Hoặc `%USERPROFILE%\.docker\config.json`

**Security:**
- Credentials được encrypt
- **Không commit** config.json vào Git
- Use tokens thay vì passwords khi có thể

---

## 📤 PHẦN 3: PUSH IMAGES

### 3.1. Tag Image cho Registry

**Tag image:**
```bash
$ docker tag my-app:latest myusername/my-app:latest
```

**Tag với registry:**
```bash
$ docker tag my-app:latest docker.io/myusername/my-app:latest
```

**Tag format:**
```
[registry/][username/]image-name[:tag]
```

### 3.2. Push Image

**Push to Docker Hub:**
```bash
$ docker push myusername/my-app:latest
```

**Push process:**
```
The push refers to repository [docker.io/myusername/my-app]
abc123...: Pushing [==================================================>] 10MB/10MB
def456...: Pushing [==================================================>] 5MB/5MB
latest: digest: sha256:... size: 2
```

**Giải thích:**
1. **Check authentication**: Verify credentials
2. **Check image exists**: Image phải được tag đúng
3. **Push layers**: Push các layers chưa có trên registry
4. **Push manifest**: Push image manifest
5. **Complete**: Image available trên registry

### 3.3. Push Multiple Tags

**Push nhiều tags:**
```bash
$ docker tag my-app:latest myusername/my-app:v1.0.0
$ docker tag my-app:latest myusername/my-app:latest
$ docker push myusername/my-app:v1.0.0
$ docker push myusername/my-app:latest
```

**Hoặc push tất cả tags:**
```bash
$ docker push myusername/my-app --all-tags
```

### 3.4. Push to Private Registry

**Push to private registry:**
```bash
# Login trước
$ docker login registry.example.com

# Tag với registry
$ docker tag my-app:latest registry.example.com/my-app:latest

# Push
$ docker push registry.example.com/my-app:latest
```

---

## 📥 PHẦN 4: PULL IMAGES

### 4.1. Pull từ Docker Hub

**Pull public image:**
```bash
$ docker pull nginx
# Tương đương: docker pull docker.io/library/nginx:latest
```

**Pull với tag:**
```bash
$ docker pull nginx:1.21
$ docker pull nginx:alpine
```

**Pull từ user:**
```bash
$ docker pull myusername/my-app:latest
```

### 4.2. Pull từ Private Registry

**Pull từ private registry:**
```bash
# Login trước
$ docker login registry.example.com

# Pull
$ docker pull registry.example.com/my-app:latest
```

### 4.3. Pull Process

**Khi pull, Docker làm gì?**

1. **Check local**: Image đã có local chưa?
2. **Connect registry**: Connect đến registry
3. **Download manifest**: Download image manifest
4. **Download layers**: Download các layers chưa có
5. **Verify**: Verify image integrity
6. **Store**: Lưu image local

**Layer sharing:**
- Layers đã có local → skip download
- **Efficient**: Tiết kiệm bandwidth

---

## 🏢 PHẦN 5: PRIVATE REGISTRY

### 5.1. Private Registry là gì?

**Private Registry** là registry riêng của organization:
- **Self-hosted**: Chạy trên infrastructure riêng
- **Private**: Chỉ accessible bởi authorized users
- **Control**: Full control over images
- **Security**: Better security cho proprietary code

**Use cases:**
- Proprietary applications
- High security requirements
- Compliance requirements
- High volume (no rate limits)

### 5.2. Registry Options

**Self-hosted:**
- **Docker Registry**: Official, simple
- **Harbor**: Enterprise-grade, features
- **GitLab Registry**: Integrated với GitLab
- **Nexus**: Artifact repository

**Cloud-managed:**
- **AWS ECR**: Amazon Elastic Container Registry
- **GCP GCR**: Google Container Registry
- **Azure ACR**: Azure Container Registry
- **GitHub Container Registry**: Integrated với GitHub

### 5.3. Docker Registry (Self-hosted)

**Run Docker Registry:**
```bash
$ docker run -d -p 5000:5000 --name registry registry:2
```

**Push to local registry:**
```bash
# Tag
$ docker tag my-app:latest localhost:5000/my-app:latest

# Push
$ docker push localhost:5000/my-app:latest
```

**Pull from local registry:**
```bash
$ docker pull localhost:5000/my-app:latest
```

**Lưu ý:**
- `localhost:5000` chỉ cho local development
- Production cần proper domain và HTTPS

### 5.4. Harbor (Enterprise Registry)

**Harbor features:**
- **RBAC**: Role-based access control
- **Vulnerability scanning**: Scan images cho vulnerabilities
- **Replication**: Replicate images giữa registries
- **Web UI**: User-friendly interface
- **Helm charts**: Store Helm charts

**Setup:**
- Phức tạp hơn Docker Registry
- Cần database, Redis
- Recommended cho production

---

## 🏷️ PHẦN 6: IMAGE TAGS & VERSIONING

### 6.1. Tags trong Registry

**Tags trong registry:**
- **Mutable**: Tags có thể thay đổi
- **Multiple tags**: Một image có thể có nhiều tags
- **Versioning**: Dùng tags để track versions

**Ví dụ:**
```
my-app:latest      → Points to v1.2.0
my-app:v1.2.0      → Points to specific version
my-app:v1.2        → Points to v1.2.0
my-app:v1          → Points to v1.2.0
```

### 6.2. Semantic Versioning

**Format:**
```
vMAJOR.MINOR.PATCH
```

**Examples:**
- `v1.0.0`: Initial release
- `v1.0.1`: Patch (bug fixes)
- `v1.1.0`: Minor (new features, backward compatible)
- `v2.0.0`: Major (breaking changes)

**Best practices:**
- Use semantic versioning
- Tag releases
- Don't reuse tags

### 6.3. Image Digests

**Digest là gì?**

**Digest** là **immutable identifier** của image:
- **SHA256 hash**: Content-based
- **Immutable**: Không thay đổi khi content không đổi
- **Reliable**: Better cho production (không bị tag confusion)

**Pull by digest:**
```bash
$ docker pull nginx@sha256:abc123def456...
```

**Use case:**
- **Production**: Pin specific digest
- **Security**: Ensure image không bị tamper
- **Reproducibility**: Guarantee cùng image

### 6.4. Tag Best Practices

**Best practices:**

1. **Semantic versioning**: `v1.0.0`, `v1.1.0`, etc.
2. **Don't reuse tags**: Mỗi tag point đến một version
3. **Latest tag**: Chỉ cho development, không dùng production
4. **Pin digests**: Production nên pin digests
5. **Tag strategy**: Document tag strategy

---

## 🏭 PRODUCTION STORY #1: Docker Hub Rate Limiting

### Context

**Công ty:** E-commerce, 600 employees
**Hệ thống:** Kubernetes cluster, 100 nodes
**Traffic:** 10M requests/day
**Team:** 35 DevOps engineers

### Problem

**Tháng 8/2023:**
- **Image pull failures** trên 20 nodes
- **Error**: "toomanyrequests: You have reached your pull rate limit"
- **Root cause**: Docker Hub rate limiting

**Timeline:**
- **2:00 AM**: Auto-scaling trigger → 20 nodes mới
- **2:05 AM**: Nodes pull images từ Docker Hub
- **2:10 AM**: **Rate limit exceeded** → pull failures
- **2:15 AM**: Containers không start → service degradation
- **2:30 AM**: Team investigate
- **3:00 AM**: Fix và recover

**Impact:**
- **1 hour service degradation**
- **200K requests failed**
- **Customer complaints**

### Investigation

**Root cause:**
```bash
# Error logs
Error response from daemon: toomanyrequests: You have reached your pull rate limit
```

**Docker Hub rate limits:**
- **Anonymous**: 100 pulls/6 hours per IP
- **Authenticated**: 200 pulls/6 hours per account
- **Paid**: Unlimited

**Vấn đề:**
- 20 nodes × 5 images = 100 pulls
- Nhiều nodes pull cùng lúc → rate limit
- **Không có local registry cache**

### Fix

**Solution 1: Docker Hub Authentication**
```bash
# Authenticate với Docker Hub
$ docker login
# Tăng rate limit từ 100 → 200 pulls/6h
```

**Solution 2: Private Registry**
```bash
# Setup private registry (Harbor)
# Pull từ private registry thay vì Docker Hub
$ docker pull registry.internal.com/my-app:v1.0.0
```

**Solution 3: Image Caching**
```bash
# Pre-pull images trên nodes
# Hoặc dùng image cache trong cluster
```

**Solution 4: Mirror Registry**
```bash
# Setup mirror registry
# Cache images từ Docker Hub
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
3. **Authentication**: Authenticate để tăng limits
4. **Image caching**: Cache images để giảm pulls

---

## 🏭 PRODUCTION STORY #2: Image Tag Confusion

### Context

**Công ty:** SaaS platform, 400 employees
**Hệ thống:** Production deployment với Docker
**Team:** 25 DevOps engineers
**Issue:** Deploy sai version do tag confusion

### Problem

**Tháng 7/2023:**
- Team deploy `my-app:latest` lên production
- **Unexpected behavior**: Application có bugs
- **Root cause**: `latest` tag đã được update với version mới (chưa test)

**Timeline:**
- **10:00 AM**: Developer push code mới, build `my-app:latest`
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

**Solution 2: Use Digests**
```yaml
services:
  app:
    image: my-app@sha256:abc123...  # ← Pin digest
    # Immutable, không thể thay đổi
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
3. **Use digests**: Immutable identifiers
4. **Tag strategy**: Document và enforce tag strategy

---

## 🎓 TÓM TẮT

### Docker Hub

**Là gì:**
- Public registry lớn nhất
- Free tier: 1 private repo
- Rate limits: 100-200 pulls/6h

**Use cases:**
- Public images
- Development
- Learning

### Authentication

**Commands:**
- `docker login`: Login to registry
- `docker logout`: Logout
- Credentials: `~/.docker/config.json`

### Push/Pull

**Push:**
- Tag image với registry format
- `docker push <image>`
- Push layers và manifest

**Pull:**
- `docker pull <image>`
- Download layers chưa có
- Layer sharing efficient

### Private Registry

**Options:**
- Docker Registry: Simple
- Harbor: Enterprise
- Cloud: AWS ECR, GCP GCR, Azure ACR

**Use cases:**
- Proprietary code
- High security
- No rate limits

### Tags & Versioning

**Best practices:**
- Semantic versioning
- Don't reuse tags
- Pin digests trong production
- Latest chỉ cho development

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ bạn đã hoàn thành **Phase 2: Core Docker Usage**!

**Bạn đã học:**
- ✅ Docker Installation & First Container
- ✅ Docker Images (Pull, Tag, Inspect)
- ✅ Container Lifecycle
- ✅ Container Logs & Debugging
- ✅ Docker Hub & Registry Basics

**Phase tiếp theo (Phase 3): Dockerfile Fundamentals**
- Day-011: Dockerfile Syntax - FROM, RUN, COPY
- Day-012: CMD vs ENTRYPOINT
- Day-013: COPY vs ADD
- Day-014: WORKDIR, ENV, ARG
- Day-015: Multi-stage Builds

---

## 📚 TÀI LIỆU THAM KHẢO

- Docker Hub: https://hub.docker.com/
- Docker Registry: https://docs.docker.com/registry/
- Harbor: https://goharbor.io/

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.

