# Day-041: Docker trong CI/CD - Overview - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Basic CI Pipeline

**GitHub Actions:**
```yaml
name: CI
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build
        run: docker build -t my-app .
      - name: Test
        run: docker run my-app npm test
```

---

## ✅ BÀI TẬP 2: CI/CD Platform Comparison

**Comparison:**

| Feature | GitHub Actions | GitLab CI | Jenkins | CircleCI |
|---------|----------------|-----------|---------|----------|
| **Integration** | GitHub | GitLab | Any | Any |
| **Docker** | ✅ | ✅ | ✅ | ✅ |
| **Cost** | Free (public) | Free (self-hosted) | Free | Paid |

**Recommendations:**
- GitHub Actions: GitHub projects
- GitLab CI: GitLab projects
- Jenkins: Self-hosted, flexible
- CircleCI: Cloud-based, enterprise

---

## ✅ BÀI TẬP 3: Docker CI Pipeline

**GitHub Actions:**
```yaml
name: CI
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Build
        run: docker build -t my-app .
      - name: Test
        run: docker run my-app npm test
      - name: Push
        if: github.ref == 'refs/heads/main'
        run: docker push my-app
```

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

