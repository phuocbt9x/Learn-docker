# Day-010: Giải Pháp & Giải Thích

## 🎯 LƯU Ý QUAN TRỌNG

Các giải pháp dưới đây là commands và explanations thực tế. Quan trọng là bạn:

- **Thực hành** các commands trên terminal
- **Hiểu** registry mechanism
- **Apply** best practices trong production
- **Secure** credentials

---

## 📝 BÀI TẬP 1: DOCKER HUB

### 1.1. Docker Hub Account

**Tạo account:**
- Truy cập: https://hub.docker.com/
- Sign up với email
- Verify email

**Login:**
```bash
$ docker login
Username: myusername
Password: ****
Login Succeeded
```

**Verify:**
```bash
# Check credentials
$ cat ~/.docker/config.json
{
  "auths": {
    "https://index.docker.io/v1/": {
      "auth": "base64encodedcredentials"
    }
  }
}
```

### 1.2. Explore Docker Hub

**Popular images:**
- nginx (official)
- redis (official)
- postgres (official)
- node (official)
- python (official)

**Official vs User images:**
- **Official**: Maintained bởi Docker/vendors, verified
- **User**: Maintained bởi community, không verified

**Check tags:**
- Trên Docker Hub website: Browse tags
- Hoặc dùng API: `curl https://hub.docker.com/v2/repositories/library/nginx/tags/`

### 1.3. Rate Limits

**Docker Hub rate limits:**
- **Anonymous**: 100 pulls/6 hours per IP
- **Authenticated**: 200 pulls/6 hours per account
- **Paid**: Unlimited

**Check usage:**
- Docker Hub website: Account settings
- Hoặc check error messages

**Increase limits:**
- **Authenticate**: Login để tăng từ 100 → 200
- **Paid plan**: Upgrade để unlimited

### 1.4. Public vs Private

**Public repositories:**
- ✅ Free, unlimited
- ✅ Shareable với community
- ❌ Visible to everyone
- **Use case**: Open source, learning

**Private repositories:**
- ✅ Private, secure
- ❌ Free tier: 1 private repo
- ❌ Paid: Unlimited
- **Use case**: Proprietary code, production

**Free tier limitations:**
- 1 private repository
- Unlimited public repositories
- Rate limits apply

---

## 📝 BÀI TẬP 2: AUTHENTICATION

### 2.1. Docker Login

**Login:**
```bash
$ docker login
Username: myusername
Password: ****
Login Succeeded
```

**Verify:**
```bash
# Check credentials
$ cat ~/.docker/config.json
```

**Logout:**
```bash
$ docker logout
```

### 2.2. Login với Token

**Tạo token:**
- Docker Hub: Account Settings → Security → New Access Token

**Login với token:**
```bash
$ echo "mytoken" | docker login -u myusername --password-stdin
```

**Security benefits:**
- ✅ Tokens có thể revoke
- ✅ Không expose password
- ✅ Better cho CI/CD
- ✅ Fine-grained permissions

### 2.3. Multiple Registries

**Login multiple registries:**
```bash
# Docker Hub
$ docker login

# Private registry
$ docker login registry.example.com
```

**Check credentials:**
```bash
$ cat ~/.docker/config.json
{
  "auths": {
    "https://index.docker.io/v1/": {...},
    "registry.example.com": {...}
  }
}
```

**Logout từ specific:**
```bash
$ docker logout registry.example.com
```

### 2.4. Credentials Security

**Storage location:**
- Linux: `~/.docker/config.json`
- macOS: Keychain (default) hoặc `~/.docker/config.json`
- Windows: Credential Manager

**Secure credentials:**
- ✅ Use tokens thay vì passwords
- ✅ Rotate tokens regularly
- ✅ Don't commit config.json vào Git
- ✅ Use `.dockerignore` hoặc `.gitignore`

**Best practices:**
- Add `~/.docker/config.json` vào `.gitignore`
- Use environment variables cho CI/CD
- Use secrets management tools

---

## 📝 BÀI TẬP 3: PUSH IMAGES

### 3.1. Tag Image

**Tag cho Docker Hub:**
```bash
$ docker tag my-app:latest myusername/my-app:latest
```

**Tag format:**
```
[registry/][username/]image-name[:tag]
```

**Examples:**
- `myusername/my-app:latest` → `docker.io/myusername/my-app:latest`
- `registry.example.com/my-app:latest` → Private registry

**Verify:**
```bash
$ docker images | grep my-app
REPOSITORY              TAG       IMAGE ID
my-app                  latest    abc123...
myusername/my-app       latest    abc123...  # Same ID
```

### 3.2. Push Image

**Push:**
```bash
$ docker push myusername/my-app:latest
```

**Output:**
```
The push refers to repository [docker.io/myusername/my-app]
abc123...: Pushing [==================================================>] 10MB/10MB
def456...: Pushing [==================================================>] 5MB/5MB
latest: digest: sha256:... size: 2
```

**Push process:**
1. Check authentication
2. Check image exists
3. Push layers (chưa có trên registry)
4. Push manifest
5. Complete

**Verify trên Docker Hub:**
- Truy cập: https://hub.docker.com/r/myusername/my-app
- Image phải visible

### 3.3. Push Multiple Tags

**Tag nhiều tags:**
```bash
$ docker tag my-app:latest myusername/my-app:v1.0.0
$ docker tag my-app:latest myusername/my-app:v1.0
$ docker tag my-app:latest myusername/my-app:latest
```

**Push từng tag:**
```bash
$ docker push myusername/my-app:v1.0.0
$ docker push myusername/my-app:v1.0
$ docker push myusername/my-app:latest
```

**Push all tags:**
```bash
$ docker push myusername/my-app --all-tags
```

### 3.4. Push to Private Registry

**Setup local registry:**
```bash
$ docker run -d -p 5000:5000 --name registry registry:2
```

**Tag và push:**
```bash
# Tag
$ docker tag my-app:latest localhost:5000/my-app:latest

# Push
$ docker push localhost:5000/my-app:latest
```

**Verify:**
```bash
# List repositories
$ curl http://localhost:5000/v2/_catalog
{"repositories":["my-app"]}

# List tags
$ curl http://localhost:5000/v2/my-app/tags/list
{"name":"my-app","tags":["latest"]}
```

---

## 📝 BÀI TẬP 4: PULL IMAGES

### 4.1. Pull từ Docker Hub

**Commands:**
```bash
# Public image
$ docker pull nginx
# Tương đương: docker pull docker.io/library/nginx:latest

# Specific tag
$ docker pull nginx:1.21
$ docker pull nginx:alpine

# User repository
$ docker pull myusername/my-app:latest
```

**So sánh pull times:**
- Depends on:
  - Image size
  - Network speed
  - Layers đã có local
  - CDN location

### 4.2. Pull từ Private Registry

**Commands:**
```bash
# Login
$ docker login registry.example.com

# Pull
$ docker pull registry.example.com/my-app:latest
```

**So sánh với Docker Hub:**
- **Private registry**: Faster (local network)
- **Docker Hub**: Slower (internet, CDN)
- **Both**: Layer sharing efficient

### 4.3. Pull by Digest

**Tìm digest:**
```bash
# Inspect image
$ docker inspect nginx:latest --format='{{.RepoDigests}}'
[nginx@sha256:abc123def456...]

# Hoặc từ registry
$ curl https://hub.docker.com/v2/repositories/library/nginx/tags/latest
```

**Pull by digest:**
```bash
$ docker pull nginx@sha256:abc123def456...
```

**So sánh với tag:**
- **Tag**: Mutable, có thể thay đổi
- **Digest**: Immutable, không thay đổi
- **Production**: Nên dùng digest

### 4.4. Pull Process

**Khi pull, Docker làm gì:**

1. **Check local**: Image đã có local chưa?
   ```bash
   # Nếu có → skip
   # Nếu không → tiếp tục
   ```

2. **Connect registry**: Connect đến registry

3. **Download manifest**: Download image manifest

4. **Download layers**: Download các layers chưa có
   ```bash
   # Layers đã có → "Already exists"
   # Layers chưa có → Download
   ```

5. **Verify**: Verify image integrity

6. **Store**: Lưu image local

**Optimize pull time:**
- Pre-pull images
- Use local registry
- Cache layers
- Use CDN

---

## 📝 BÀI TẬP 5: PRIVATE REGISTRY

### 5.1. Docker Registry

**Run registry:**
```bash
$ docker run -d -p 5000:5000 --name registry registry:2
```

**Push image:**
```bash
# Tag
$ docker tag my-app:latest localhost:5000/my-app:latest

# Push
$ docker push localhost:5000/my-app:latest
```

**Pull image:**
```bash
$ docker pull localhost:5000/my-app:latest
```

**Verify:**
```bash
# List repositories
$ curl http://localhost:5000/v2/_catalog

# List tags
$ curl http://localhost:5000/v2/my-app/tags/list
```

### 5.2. Registry với Authentication

**Setup với authentication:**
```bash
# Create htpasswd file
$ docker run --rm --entrypoint htpasswd httpd:2 \
  -Bbn myuser mypassword > auth/htpasswd

# Run registry với auth
$ docker run -d -p 5000:5000 \
  -v $(pwd)/auth:/auth \
  -e "REGISTRY_AUTH=htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" \
  -e "REGISTRY_AUTH_HTPASSWD_REALM=Registry Realm" \
  --name registry registry:2
```

**Login và push:**
```bash
$ docker login localhost:5000
Username: myuser
Password: ****

$ docker push localhost:5000/my-app:latest
```

### 5.3. Registry Options

**Docker Registry:**
- ✅ Simple, lightweight
- ✅ Easy setup
- ❌ Limited features
- **Use case**: Small teams, simple needs

**Harbor:**
- ✅ Enterprise features (RBAC, scanning, replication)
- ✅ Web UI
- ❌ Complex setup
- **Use case**: Production, enterprise

**So sánh:**

| Feature | Docker Registry | Harbor |
|---------|----------------|--------|
| **Setup** | Simple | Complex |
| **Features** | Basic | Advanced |
| **RBAC** | ❌ | ✅ |
| **Vulnerability Scanning** | ❌ | ✅ |
| **Web UI** | ❌ | ✅ |
| **Use case** | Small scale | Enterprise |

### 5.4. Production Registry

**Architecture:**
```
Load Balancer
    ↓
Registry 1 (Primary)
Registry 2 (Replica)
Registry 3 (Replica)
    ↓
Storage Backend (S3, etc.)
```

**High availability:**
- Multiple registry instances
- Load balancer
- Shared storage backend
- Health checks

**Backup strategy:**
- Regular backups của storage
- Replication giữa registries
- Disaster recovery plan

**Monitoring:**
- Registry health
- Storage usage
- Pull/push metrics
- Error rates

---

## 📝 BÀI TẬP 6: IMAGE TAGS & VERSIONING

### 6.1. Tag Strategy

**Semantic versioning:**
```
vMAJOR.MINOR.PATCH
```

**Examples:**
- `v1.0.0`: Initial release
- `v1.0.1`: Patch (bug fixes)
- `v1.1.0`: Minor (new features)
- `v2.0.0`: Major (breaking changes)

**Environment tags:**
- `v1.0.0-dev`: Development
- `v1.0.0-staging`: Staging
- `v1.0.0-production`: Production

**Date tags:**
- `2023-01-15`: Date-based
- `build-123`: Build number

**Best practices:**
- Use semantic versioning
- Tag releases
- Don't reuse tags
- Document strategy

### 6.2. Tag Management

**Commands:**
```bash
# Tag với version
$ docker tag my-app:latest myusername/my-app:v1.0.0

# Multiple tags
$ docker tag my-app:latest myusername/my-app:v1.0.0
$ docker tag my-app:latest myusername/my-app:v1.0
$ docker tag my-app:latest myusername/my-app:latest

# Update tag
$ docker tag my-app:v1.1.0 myusername/my-app:latest

# Remove tag (local)
$ docker rmi myusername/my-app:old-tag
```

### 6.3. Latest Tag

**Latest tag là gì:**
- Default tag khi không specify
- **Mutable**: Có thể thay đổi
- Points to "latest" version

**Tại sao không nên dùng trong production:**
- **Unpredictable**: Không biết version nào
- **Mutable**: Có thể thay đổi
- **Risk**: Có thể deploy version chưa test

**Khi nào có thể dùng:**
- Development
- Testing
- Learning

**Best practices:**
- Latest chỉ cho development
- Production dùng specific versions
- Document tag strategy

### 6.4. Image Digests

**Digest là gì:**
- **SHA256 hash** của image content
- **Immutable**: Không thay đổi
- **Content-based**: Dựa trên content

**Tại sao quan trọng:**
- **Security**: Ensure image không bị tamper
- **Reproducibility**: Guarantee cùng image
- **Production**: Pin specific version

**Pull by digest:**
```bash
# Tìm digest
$ docker inspect nginx:latest --format='{{.RepoDigests}}'

# Pull by digest
$ docker pull nginx@sha256:abc123def456...
```

**Use trong production:**
```yaml
# docker-compose.yml
services:
  app:
    image: my-app@sha256:abc123...  # ← Pin digest
```

**Best practices:**
- Pin digests trong production
- Track digests trong deployment configs
- Verify digests trước khi deploy

---

## 📝 BÀI TẬP 7: PRACTICAL SCENARIOS

### Scenario 1: Development Workflow

**7.1. Development workflow:**
```bash
# Build
$ docker build -t my-app:latest .

# Tag với version
$ docker tag my-app:latest myusername/my-app:v1.0.0

# Push
$ docker push myusername/my-app:v1.0.0

# Share với team
# Team pull: docker pull myusername/my-app:v1.0.0
```

**7.2. CI/CD integration:**
```bash
#!/bin/bash
# CI/CD script

# Build
docker build -t my-app:$BUILD_NUMBER .

# Tag
docker tag my-app:$BUILD_NUMBER myusername/my-app:$BUILD_NUMBER
docker tag my-app:$BUILD_NUMBER myusername/my-app:latest

# Push
docker push myusername/my-app:$BUILD_NUMBER
docker push myusername/my-app:latest
```

### Scenario 2: Production Deployment

**7.3. Production setup:**
```bash
# 1. Setup private registry (Harbor)
# 2. Configure authentication
# 3. Setup RBAC
# 4. Enable vulnerability scanning
# 5. Setup replication
```

**7.4. Deployment process:**
```bash
# 1. Pull từ private registry
$ docker pull registry.internal.com/my-app:v1.0.0

# 2. Verify image
$ docker inspect registry.internal.com/my-app:v1.0.0

# 3. Deploy
$ docker run -d registry.internal.com/my-app:v1.0.0

# 4. Rollback (nếu cần)
$ docker run -d registry.internal.com/my-app:v0.9.0
```

---

## 📝 BÀI TẬP 8: TROUBLESHOOTING

### 8.1. Push Failures

**Error:**
```bash
denied: requested access to the resource is denied
```

**Root cause:**
- Authentication failed
- No permission
- Wrong repository name

**Fix:**
```bash
# Check login
$ docker login

# Verify repository name
$ docker tag my-app:latest myusername/my-app:latest

# Check permissions
# Docker Hub: Check repository settings
```

### 8.2. Pull Failures - Rate Limiting

**Error:**
```bash
toomanyrequests: You have reached your pull rate limit
```

**Root cause:**
- Docker Hub rate limit exceeded
- Anonymous: 100 pulls/6h
- Authenticated: 200 pulls/6h

**Fix:**
```bash
# Option 1: Authenticate
$ docker login
# Tăng từ 100 → 200 pulls/6h

# Option 2: Use private registry
$ docker pull registry.internal.com/my-app:latest

# Option 3: Wait for rate limit reset
# 6 hours
```

### 8.3. Tag Not Found

**Error:**
```bash
manifest for my-app:v999.999 not found
```

**Root cause:**
- Tag không tồn tại
- Typo trong tag name

**Fix:**
```bash
# Check available tags
# Docker Hub website hoặc API

# Use correct tag
$ docker pull my-app:v1.0.0
```

### 8.4. Registry Connectivity

**Error:**
```bash
dial tcp: lookup registry.example.com
```

**Root cause:**
- DNS issues
- Network connectivity
- Registry không accessible

**Debug:**
```bash
# Check DNS
$ nslookup registry.example.com

# Check connectivity
$ ping registry.example.com
$ curl https://registry.example.com/v2/

# Check firewall
$ sudo ufw status
```

**Fix:**
- Fix DNS
- Fix network
- Configure firewall
- Check registry status

---

## ✅ TỔNG KẾT

Các bài tập này giúp bạn:

1. **Use Docker Hub**: Push/pull images
2. **Authenticate**: Login/logout, manage credentials
3. **Private registry**: Setup và sử dụng
4. **Tags & versioning**: Manage tags, use digests
5. **Best practices**: Security, versioning, production

**Key takeaways:**
- **Docker Hub**: Public registry, rate limits
- **Authentication**: Secure credentials, use tokens
- **Private registry**: Quan trọng cho production
- **Tags**: Semantic versioning, pin digests trong production
- **Best practices**: Security, versioning, monitoring

---

**🎉 Chúc mừng! Bạn đã hoàn thành Phase 2: Core Docker Usage!**

**Phase tiếp theo: Phase 3 - Dockerfile Fundamentals**
- Day-011: Dockerfile Syntax - FROM, RUN, COPY
- Day-012: CMD vs ENTRYPOINT
- Day-013: COPY vs ADD
- Day-014: WORKDIR, ENV, ARG
- Day-015: Multi-stage Builds

---

**Chúc bạn học tốt! Tiếp tục với Phase 3 để học viết Dockerfile production-grade!**

