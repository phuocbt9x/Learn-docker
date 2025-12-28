# Day-044: Multi-arch Builds - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Setup Buildx

**Commands:**
```bash
$ docker buildx create --name mybuilder --use
$ docker buildx inspect --bootstrap
$ docker buildx ls
```

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
# Shows multiple architectures
```

---

## ✅ BÀI TẬP 3: CI Integration

**GitHub Actions:**
```yaml
- name: Setup Buildx
  uses: docker/setup-buildx-action@v2

- name: Build and Push
  uses: docker/build-push-action@v2
  with:
    platforms: linux/amd64,linux/arm64
    push: true
    tags: my-app:latest
```

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

