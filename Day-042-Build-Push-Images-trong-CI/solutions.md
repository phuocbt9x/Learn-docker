# Day-042: Build & Push Images trong CI - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Build Image

**GitHub Actions:**
```yaml
- name: Build
  run: docker build -t my-app .
```

---

## ✅ BÀI TẬP 2: Push Image

**GitHub Actions:**
```yaml
- name: Login
  uses: docker/login-action@v2
  with:
    username: ${{ secrets.DOCKER_USERNAME }}
    password: ${{ secrets.DOCKER_PASSWORD }}

- name: Build and Push
  run: |
    docker build -t my-app:${{ github.sha }} .
    docker tag my-app:${{ github.sha }} my-app:latest
    docker push my-app:${{ github.sha }}
    docker push my-app:latest
```

---

## ✅ BÀI TẬP 3: Conditional Push

**GitHub Actions:**
```yaml
- name: Push
  if: github.ref == 'refs/heads/main'
  run: |
    docker push my-app:latest

- name: Push with SHA
  run: |
    docker push my-app:${{ github.sha }}
```

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

