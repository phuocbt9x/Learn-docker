# Day-044: Multi-arch Builds

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được multi-arch builds là gì
- Biết cách build cho multiple architectures
- Hiểu được buildx
- Biết cách push multi-arch images
- Áp dụng trong production

---

## 🏗️ PHẦN 1: MULTI-ARCH BUILDS

### 1.1. Multi-arch là gì?

**Multi-arch builds** là **build images cho multiple architectures** (x86_64, ARM64, etc.).

**Purpose:**
- **Cross-platform**: Support multiple platforms
- **ARM support**: Support ARM-based systems
- **Cloud compatibility**: Compatible với different cloud providers

### 1.2. Buildx

**Docker Buildx** là **extended build capabilities** cho Docker.

**Features:**
- **Multi-arch**: Build cho multiple architectures
- **Advanced caching**: Advanced caching options
- **BuildKit**: Uses BuildKit

**Setup:**
```bash
$ docker buildx create --use
```

### 1.3. Build Multi-arch

**Build:**
```bash
$ docker buildx build --platform linux/amd64,linux/arm64 -t my-app .
```

**Build và push:**
```bash
$ docker buildx build --platform linux/amd64,linux/arm64 \
  -t my-app:latest --push .
```

---

## 🏭 PRODUCTION STORY: ARM Support

### Context

**Công ty:** Cloud provider, 1000 employees
**Issue:** Images không chạy trên ARM
**Solution:** Multi-arch builds

### Fix

**Solution: Multi-arch builds**
```yaml
- name: Build
  uses: docker/build-push-action@v2
  with:
    platforms: linux/amd64,linux/arm64
    push: true
```

**Results:**
- Support multiple architectures
- Cloud compatibility
- Broader deployment options

---

## 🎓 TÓM TẮT

**Multi-arch builds:**
- Build cho multiple architectures
- Use buildx
- Push multi-arch images

**Use cases:**
- Cross-platform support
- ARM-based systems
- Cloud compatibility

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-045)** sẽ đi sâu vào:
- CI/CD Best Practices
- Production patterns

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.

