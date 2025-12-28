# Day-045: CI/CD Best Practices - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Security Best Practices

**CI config:**
```yaml
- name: Build
  uses: docker/build-push-action@v2
  with:
    tags: my-app:latest

- name: Scan
  uses: aquasecurity/trivy-action@master
  with:
    image-ref: my-app:latest
    exit-code: '1'
```

**Security measures:**
- Secrets management
- Image scanning
- Security checks

---

## ✅ BÀI TẬP 2: Pipeline Optimization

**Optimizations:**
- Layer caching
- Parallel builds
- Build only changed

**Results:**
- Build time: 30 phút → 8 phút
- Cache hits: 90%

---

## ✅ BÀI TẬP 3: Production Pipeline

**Complete pipeline:**
```yaml
name: CI/CD
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build
        uses: docker/build-push-action@v2
        with:
          cache-from: type=registry,ref=my-registry/my-app:buildcache
          cache-to: type=registry,ref=my-registry/my-app:buildcache
          push: true
      - name: Scan
        uses: aquasecurity/trivy-action@master
      - name: Test
        run: docker run my-app npm test
```

**Features:**
- Caching
- Security scanning
- Testing
- Production-ready

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

