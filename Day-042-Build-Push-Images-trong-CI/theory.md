# Day-042: Build & Push Images trong CI

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Biết cách build images trong CI
- Biết cách push images to registry
- Hiểu được authentication với registries
- Biết cách tag images
- Áp dụng trong production

---

## 🔨 PHẦN 1: BUILD IMAGES TRONG CI

### 1.1. Basic Build

**GitHub Actions:**
```yaml
- name: Build
  run: docker build -t my-app .
```

**GitLab CI:**
```yaml
build:
  script:
    - docker build -t my-app .
```

### 1.2. Build với Tags

**Tag images:**
```yaml
- name: Build
  run: |
    docker build -t my-app:latest .
    docker build -t my-app:${{ github.sha }} .
```

### 1.3. Build Options

**Build với options:**
```yaml
- name: Build
  run: docker build --target production -t my-app .
```

---

## 📤 PHẦN 2: PUSH IMAGES

### 2.1. Authentication

**Docker Hub:**
```yaml
- name: Login
  uses: docker/login-action@v2
  with:
    username: ${{ secrets.DOCKER_USERNAME }}
    password: ${{ secrets.DOCKER_PASSWORD }}
```

**Private Registry:**
```yaml
- name: Login
  uses: docker/login-action@v2
  with:
    registry: registry.example.com
    username: ${{ secrets.REGISTRY_USERNAME }}
    password: ${{ secrets.REGISTRY_PASSWORD }}
```

### 2.2. Push Images

**Push:**
```yaml
- name: Push
  run: |
    docker push my-app:latest
    docker push my-app:${{ github.sha }}
```

### 2.3. Conditional Push

**Push only on main:**
```yaml
- name: Push
  if: github.ref == 'refs/heads/main'
  run: docker push my-app:latest
```

---

## 🏭 PRODUCTION STORY: Image Push Failures

### Context

**Công ty:** Fintech, 500 employees
**Issue:** Images không push được
**Root cause:** Authentication issues

### Fix

**Solution: Proper authentication**
```yaml
- name: Login
  uses: docker/login-action@v2
  with:
    username: ${{ secrets.DOCKER_USERNAME }}
    password: ${{ secrets.DOCKER_PASSWORD }}
```

**Results:**
- Authentication works
- Images push successfully
- No failures

---

## 🎓 TÓM TẮT

**Build images:**
- Basic build
- Tag images
- Build options

**Push images:**
- Authentication
- Push to registry
- Conditional push

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-043)** sẽ đi sâu vào:
- Docker Layer Caching trong CI
- Cache strategies

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-043: Docker-Layer-Caching-trong-CI](../Day-043-Docker-Layer-Caching-trong-CI/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
