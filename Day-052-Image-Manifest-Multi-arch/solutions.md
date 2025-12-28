# Day-052: Image Manifest & Multi-arch - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Inspect Manifest

**Commands:**
```bash
$ docker manifest inspect my-app:latest
# Single-arch manifest

$ docker buildx imagetools inspect my-app:latest
# Multi-arch manifest list
```

**Comparison:**
- Single-arch: One manifest
- Multi-arch: Manifest list với multiple manifests

---

## ✅ BÀI TẬP 2: Multi-arch Build

**Commands:**
```bash
$ docker buildx build --platform linux/amd64,linux/arm64 \
  -t my-app:latest --push .
```

**Verification:**
```bash
$ docker buildx imagetools inspect my-app:latest
# Shows amd64 and arm64
```

---

## ✅ BÀI TẬP 3: Production Multi-arch

**CI/CD:**
```yaml
- name: Build
  uses: docker/build-push-action@v2
  with:
    platforms: linux/amd64,linux/arm64
    push: true
```

**Results:**
- Multi-arch images
- CI/CD integrated
- Production-ready

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

