# Day-051: Docker BuildKit Advanced Features

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu sâu về BuildKit advanced features
- Biết cách sử dụng build secrets
- Hiểu được cache mounts
- Biết cách sử dụng SSH mounts
- Áp dụng advanced features trong production

---

## 🔧 PHẦN 1: BUILDKIT ADVANCED FEATURES

### 1.1. Build Secrets

**Build secrets** cho phép **pass secrets** vào build process mà **không lưu trong image**.

**Syntax:**
```dockerfile
# syntax=docker/dockerfile:1.4
RUN --mount=type=secret,id=mysecret \
    cat /run/secrets/mysecret
```

**Build:**
```bash
$ docker build --secret id=mysecret,src=./secret.txt -t my-app .
```

**Benefits:**
- **Security**: Secrets không có trong image
- **Flexibility**: Pass secrets at build time
- **CI/CD**: Useful trong CI/CD

### 1.2. Cache Mounts

**Cache mounts** cho phép **persistent cache** giữa builds.

**Syntax:**
```dockerfile
# syntax=docker/dockerfile:1.4
RUN --mount=type=cache,target=/root/.npm \
    npm install
```

**Benefits:**
- **Persistent cache**: Cache persist giữa builds
- **Faster builds**: npm install nhanh hơn
- **CI/CD**: Very useful trong CI/CD

### 1.3. SSH Mounts

**SSH mounts** cho phép **access private repositories** trong build.

**Syntax:**
```dockerfile
# syntax=docker/dockerfile:1.4
RUN --mount=type=ssh \
    git clone git@github.com:user/repo.git
```

**Build:**
```bash
$ docker build --ssh default -t my-app .
```

---

## 🏭 PRODUCTION STORY: Build Secrets

### Context

**Công ty:** Fintech, 500 employees
**Issue:** Secrets trong images
**Solution:** Build secrets

### Fix

**Solution: Build secrets**
```dockerfile
# syntax=docker/dockerfile:1.4
RUN --mount=type=secret,id=apikey \
    echo "API key configured"
```

**Results:**
- No secrets in image
- Secure builds
- CI/CD compatible

---

## 🎓 TÓM TẮT

**BuildKit features:**
- Build secrets: Secure secret handling
- Cache mounts: Persistent cache
- SSH mounts: Private repo access

**Benefits:**
- Security
- Performance
- Flexibility

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-052)** sẽ đi sâu vào:
- Image Manifest & Multi-arch
- Image formats

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-052: Image-Manifest-Multi-arch](../Day-052-Image-Manifest-Multi-arch/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
