# Day-017: .dockerignore & Build Context - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Basic .dockerignore

**.dockerignore:**
```
node_modules/
.git/
dist/
*.log
.env
.DS_Store
```

**Results:**
- Build context: 800MB → 50MB
- Build time: 11 phút → 3 phút

---

## ✅ BÀI TẬP 2: Python .dockerignore

**.dockerignore:**
```
__pycache__/
*.pyc
*.pyo
venv/
.env
.git/
*.log
```

**Security verification:**
```bash
$ docker build -t my-app .
$ docker run --rm my-app ls -la | grep .env
# No output → .env excluded ✅
```

---

## ✅ BÀI TẬP 3: Build Context Analysis

**Analysis:**
- node_modules/: 600MB
- .git/: 100MB
- dist/: 50MB
- Other: 50MB

**.dockerignore:**
```
node_modules/
.git/
dist/
*.log
.env
```

**Results:**
- Build context: 800MB → 50MB (94% reduction)

---

## ✅ BÀI TẬP 4: Advanced Patterns

**.dockerignore:**
```
**/node_modules/
*.log
!important.log
src/**/*.tmp
```

**Test:**
```bash
# Verify node_modules excluded at all levels
# Verify .log excluded except important.log
# Verify .tmp excluded in src/
```

---

## ✅ BÀI TẬP 5: Security Analysis

**Sensitive files:**
- .env
- *.key
- *.pem
- secrets/

**.dockerignore:**
```
.env
*.key
*.pem
secrets/
```

**Verification:**
```bash
$ docker build -t my-app .
$ docker run --rm my-app find / -name ".env" 2>/dev/null
# No output → .env excluded ✅
```

---

## ✅ BÀI TẬP 6: Multi-language .dockerignore

**.dockerignore:**
```
# Node.js
node_modules/
.npm

# Python
__pycache__/
venv/
*.pyc

# Go
vendor/
*.o

# General
.git/
*.log
.env
```

---

## ✅ BÀI TẬP 7: Build Context Optimization

**Analysis:**
- node_modules/: 800MB
- .git/: 200MB
- dist/: 100MB
- Other: 100MB

**Optimized .dockerignore:**
```
node_modules/
.git/
dist/
build/
*.log
.env
.DS_Store
.vscode/
.idea/
```

**Results:**
- Build context: 1GB → 80MB
- Build time: 20 phút → 4 phút

---

## ✅ BÀI TẬP 8: Troubleshooting

**Problem:** Patterns không match correctly

**Fixed .dockerignore:**
```
node_modules/
**/node_modules/
.git/
.gitignore
dist/
build/
*.log
.env
```

**Fix:** Add more specific patterns, check order

---

## ✅ BÀI TẬP 9: Production Analysis

**Optimization plan:**
1. Analyze large directories
2. Create comprehensive .dockerignore
3. Measure improvements

**Results:**
- Build context: 2GB → 150MB
- Build time: 30 phút → 4 phút

---

## ✅ BÀI TẬP 10: Best Practices

**.dockerignore:**
```
# Dependencies
node_modules/
vendor/
__pycache__/

# Version control
.git/
.gitignore
.svn/

# IDE
.vscode/
.idea/
*.swp

# Build artifacts
dist/
build/
*.o

# Logs
*.log

# Environment
.env
.env.*

# OS
.DS_Store
Thumbs.db

# Secrets
*.key
*.pem
secrets/
```

**Documentation:**
- Organized by category
- Comments explain each section
- Covers common cases

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

