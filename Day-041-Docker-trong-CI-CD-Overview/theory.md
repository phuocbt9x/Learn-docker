# Day-041: Docker trong CI/CD - Overview

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được Docker trong CI/CD là gì
- Biết được benefits của Docker trong CI/CD
- Hiểu được CI/CD pipeline với Docker
- Biết được các CI/CD platforms phổ biến
- Hiểu được best practices
- Áp dụng được trong production

---

## 🔄 PHẦN 1: DOCKER TRONG CI/CD

### 1.1. Docker trong CI/CD là gì?

**Docker trong CI/CD** là **sử dụng Docker** trong Continuous Integration/Continuous Deployment pipelines.

**Purpose:**
- **Consistent environments**: Cùng environment cho build và production
- **Isolated builds**: Builds isolated từ host
- **Reproducible**: Reproducible builds
- **Fast builds**: Fast builds với caching

### 1.2. Tại sao dùng Docker trong CI/CD?

**Benefits:**
- **Consistency**: Cùng environment mọi nơi
- **Isolation**: Builds không ảnh hưởng host
- **Reproducibility**: Cùng code → cùng image
- **Speed**: Fast builds với layer caching
- **Portability**: Build một lần, run mọi nơi

### 1.3. CI/CD Pipeline với Docker

**Typical pipeline:**
1. **Checkout code**: Git checkout
2. **Build image**: Docker build
3. **Test**: Run tests trong container
4. **Push image**: Push to registry
5. **Deploy**: Deploy image to production

**Example:**
```yaml
# GitHub Actions
- name: Build image
  run: docker build -t my-app .

- name: Test
  run: docker run my-app npm test

- name: Push
  run: docker push my-app
```

---

## 🛠️ PHẦN 2: CI/CD PLATFORMS

### 2.1. GitHub Actions

**Features:**
- **Integrated**: Integrated với GitHub
- **Free**: Free cho public repos
- **Docker support**: Native Docker support

**Example:**
```yaml
name: CI
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build
        run: docker build -t my-app .
```

### 2.2. GitLab CI

**Features:**
- **Integrated**: Integrated với GitLab
- **Docker-in-Docker**: Support Docker-in-Docker
- **Runners**: Custom runners

**Example:**
```yaml
build:
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t my-app .
```

### 2.3. Jenkins

**Features:**
- **Flexible**: Rất flexible
- **Plugins**: Nhiều plugins
- **Self-hosted**: Self-hosted option

### 2.4. CircleCI

**Features:**
- **Cloud-based**: Cloud-based
- **Docker support**: Good Docker support
- **Orbs**: Reusable configs

---

## 🏭 PRODUCTION STORY #1: Inconsistent Builds

### Context

**Công ty:** E-commerce, 800 employees
**Hệ thống:** Node.js applications
**Traffic:** 20M requests/day
**Team:** 40 backend engineers

### Problem

**Tháng 5/2024:**
- **"Works on my machine"**: Code work locally nhưng fail trong CI
- **Inconsistent builds**: Builds khác nhau giữa local và CI
- **Time wasted**: Developers waste time debugging
- **Root cause**: Không dùng Docker trong CI

**Timeline:**
- **10:00 AM**: Developer push code
- **10:05 AM**: CI build fail
- **10:10 AM**: Developer investigate
- **10:30 AM**: Found environment difference
- **10:45 AM**: Fix và retry

**Impact:**
- **Build failures**: 30% builds fail do environment
- **Time wasted**: 2-3 giờ per developer per week
- **Frustration**: Developer frustration

### Investigation

**Root cause:**
- **Different environments**: Local và CI có environments khác nhau
- **Node version**: Different Node.js versions
- **Dependencies**: Different dependency versions

**Example:**
- Local: Node.js 18.5.0
- CI: Node.js 18.0.0
- **Issue**: Code work với 18.5.0 nhưng fail với 18.0.0

### Fix

**Solution: Docker trong CI**
```yaml
# .github/workflows/ci.yml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build
        run: docker build -t my-app .
      - name: Test
        run: docker run my-app npm test
```

**Dockerfile:**
```dockerfile
FROM node:18.5.0-alpine
# Pin Node version
```

**Kết quả:**
- **Consistent builds**: Cùng environment mọi nơi
- **Build failures**: 30% → 5% (83% reduction)
- **Time saved**: 2-3 giờ → 30 phút per week

### Result

**Trước:**
- No Docker trong CI
- **Consistency**: ❌
- **Build failures**: 30%

**Sau:**
- Docker trong CI
- **Consistency**: ✅
- **Build failures**: 5%

### Lesson Learned

1. **Use Docker trong CI**: Đảm bảo consistency
2. **Pin versions**: Pin versions trong Dockerfile
3. **Test trong containers**: Test trong cùng environment
4. **Reproducible**: Reproducible builds

---

## 🏭 PRODUCTION STORY #2: Slow CI Builds

### Context

**Công ty:** SaaS platform, 600 employees
**Hệ thống:** Python applications với Docker
**Traffic:** 15M requests/day
**Team:** 30 backend engineers

### Problem

**Tháng 7/2024:**
- **Slow CI builds**: Builds mất 20-30 phút
- **No caching**: Không có layer caching
- **Developer productivity**: Developers phải chờ builds
- **Root cause**: Không optimize Docker builds trong CI

**Timeline:**
- **10:00 AM**: Developer push code
- **10:01 AM**: CI start build
- **10:25 AM**: Build complete (24 phút)
- **10:30 AM**: Developer frustrated

**Impact:**
- **Build time**: 20-30 phút per build
- **CI costs**: High CI costs
- **Productivity**: Lost productivity

### Investigation

**Root cause:**
```yaml
# CI config
- name: Build
  run: docker build -t my-app .
  # No caching, rebuild everything mỗi lần
```

**Vấn đề:**
- **No cache**: Không có layer caching
- **Rebuild all**: Rebuild tất cả layers mỗi lần
- **Slow**: Build chậm

### Fix

**Solution: Docker layer caching**
```yaml
# CI config
- name: Build
  uses: docker/build-push-action@v2
  with:
    context: .
    push: false
    cache-from: type=registry,ref=my-registry/my-app:buildcache
    cache-to: type=inline
```

**Kết quả:**
- **Build time**: 20-30 phút → 5-8 phút (70% reduction)
- **Cache hits**: 80-90% layers cached
- **CI costs**: Giảm 70%

### Result

**Trước:**
- No caching
- **Build time**: 20-30 phút
- **Cache hits**: 0%

**Sau:**
- Layer caching
- **Build time**: 5-8 phút
- **Cache hits**: 80-90%

### Lesson Learned

1. **Use layer caching**: Critical cho fast builds
2. **Cache strategies**: Implement cache strategies
3. **Monitor cache hits**: Monitor để optimize
4. **Cost savings**: Caching giảm costs đáng kể

---

## 🎓 TÓM TẮT

### Docker trong CI/CD

**Benefits:**
- **Consistency**: Cùng environment
- **Isolation**: Builds isolated
- **Reproducibility**: Reproducible builds
- **Speed**: Fast với caching

### CI/CD Platforms

**Popular platforms:**
- **GitHub Actions**: Integrated với GitHub
- **GitLab CI**: Integrated với GitLab
- **Jenkins**: Flexible, self-hosted
- **CircleCI**: Cloud-based

### Best Practices

**1. Use Docker:**
- Consistent environments
- Reproducible builds

**2. Layer caching:**
- Fast builds
- Cost savings

**3. Pin versions:**
- Reproducibility
- Consistency

**4. Test trong containers:**
- Same environment
- Catch issues early

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ bạn đã:
- ✅ Hiểu Docker trong CI/CD
- ✅ Biết các platforms
- ✅ Hiểu benefits

**Day tiếp theo (Day-042)** sẽ đi sâu vào:
- Build & Push Images trong CI
- Registry integration
- Authentication

---

## 📚 TÀI LIỆU THAM KHẢO

- Docker in CI/CD: https://docs.docker.com/ci-cd/
- GitHub Actions: https://docs.github.com/en/actions
- GitLab CI: https://docs.gitlab.com/ee/ci/

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-042: Build-Push-Images-trong-CI](../Day-042-Build-Push-Images-trong-CI/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
