# Day-043: Docker Layer Caching trong CI

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được layer caching trong CI
- Biết cách implement cache strategies
- Hiểu được cache types (inline, registry, local)
- Biết cách optimize cache
- Áp dụng trong production

---

## 💾 PHẦN 1: LAYER CACHING

### 1.1. Cache Types

**Inline cache:**
```yaml
- name: Build
  uses: docker/build-push-action@v2
  with:
    cache-from: type=inline
    cache-to: type=inline
```

**Registry cache:**
```yaml
- name: Build
  uses: docker/build-push-action@v2
  with:
    cache-from: type=registry,ref=my-registry/my-app:buildcache
    cache-to: type=registry,ref=my-registry/my-app:buildcache
```

**Local cache:**
```yaml
- name: Build
  uses: docker/build-push-action@v2
  with:
    cache-from: type=local,src=/tmp/.buildx-cache
    cache-to: type=local,dest=/tmp/.buildx-cache
```

### 1.2. Cache Strategies

**Strategy 1: Registry cache**
- **Pros**: Persistent across runs
- **Cons**: Requires registry access

**Strategy 2: Inline cache**
- **Pros**: Simple, no registry needed
- **Cons**: Less efficient

**Strategy 3: Local cache**
- **Pros**: Fast
- **Cons**: Not persistent across runs

---

## 🏭 PRODUCTION STORY: Slow CI Builds

### Context

**Công ty:** SaaS, 600 employees
**Issue:** CI builds mất 20 phút
**Solution:** Layer caching

### Fix

**Solution: Registry cache**
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
- Cost savings: 75%

---

## 🎓 TÓM TẮT

**Cache types:**
- Inline: Simple
- Registry: Persistent
- Local: Fast

**Best practices:**
- Use registry cache cho production
- Monitor cache hits
- Optimize Dockerfile cho caching

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-044)** sẽ đi sâu vào:
- Multi-arch Builds
- ARM, x86_64 builds

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-044: Multi-arch-Builds](../Day-044-Multi-arch-Builds/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
