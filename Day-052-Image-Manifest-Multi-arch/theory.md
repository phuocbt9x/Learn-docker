# Day-052: Image Manifest & Multi-arch

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được image manifest là gì
- Biết cách inspect manifests
- Hiểu được multi-arch images
- Biết cách build và push multi-arch
- Áp dụng trong production

---

## 📦 PHẦN 1: IMAGE MANIFEST

### 1.1. Manifest là gì?

**Image manifest** là **metadata** mô tả image, bao gồm:
- **Layers**: Image layers
- **Architecture**: Target architecture
- **Config**: Image configuration

### 1.2. Inspect Manifest

**Inspect:**
```bash
$ docker manifest inspect my-app:latest
# Show manifest details
```

**Multi-arch:**
```bash
$ docker buildx imagetools inspect my-app:latest
# Show multi-arch manifest
```

### 1.3. Manifest List

**Manifest list** chứa **multiple manifests** cho different architectures.

**Structure:**
- Manifest list → Multiple manifests
- Each manifest → Specific architecture
- Docker tự động chọn đúng manifest

---

## 🏗️ PHẦN 2: MULTI-ARCH BUILDS

### 2.1. Build Multi-arch

**Build:**
```bash
$ docker buildx build --platform linux/amd64,linux/arm64 -t my-app .
```

**Build và push:**
```bash
$ docker buildx build --platform linux/amd64,linux/arm64 \
  -t my-app:latest --push .
```

### 2.2. Verify Multi-arch

**Inspect:**
```bash
$ docker buildx imagetools inspect my-app:latest
# Shows multiple architectures
```

---

## 🏭 PRODUCTION STORY: Multi-arch Support

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
- Broader deployment

---

## 🎓 TÓM TẮT

**Image manifest:**
- Metadata về image
- Multi-arch support
- Architecture-specific

**Multi-arch:**
- Build cho multiple architectures
- Manifest list
- Automatic selection

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-053)** sẽ đi sâu vào:
- Container Runtime Interface (CRI)
- Runtime integration

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-053: Container-Runtime-Interface-CRI](../Day-053-Container-Runtime-Interface-CRI/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
