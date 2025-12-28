# Day-043: Docker Layer Caching trong CI - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Basic Caching

**GitHub Actions:**
```yaml
- name: Build
  uses: docker/build-push-action@v2
  with:
    cache-from: type=registry,ref=my-registry/my-app:buildcache
    cache-to: type=registry,ref=my-registry/my-app:buildcache
```

**Results:**
- Build time: 20 phút → 5 phút
- Cache hits: 85%

---

## ✅ BÀI TẬP 2: Cache Strategy

**Choice:** Registry cache

**Implementation:**
```yaml
- name: Build
  uses: docker/build-push-action@v2
  with:
    cache-from: type=registry,ref=my-registry/my-app:buildcache
    cache-to: type=registry,ref=my-registry/my-app:buildcache,mode=max
```

**Justification:**
- Persistent across runs
- Fast cache retrieval
- Production-ready

---

## ✅ BÀI TẬP 3: Cache Optimization

**Optimized Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build
```

**Optimization:**
- Copy package.json trước
- npm install cached
- Source code copy sau

**Results:**
- Cache hits: 60% → 90%
- Build time: Further reduced

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

