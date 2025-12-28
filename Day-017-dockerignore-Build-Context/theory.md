# Day-017: .dockerignore & Build Context

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được build context là gì và cách hoạt động
- Biết cách sử dụng .dockerignore để optimize build
- Hiểu được impact của build context size lên build performance
- Biết được best practices cho .dockerignore
- Tối ưu được build context size

---

## 📦 PHẦN 1: BUILD CONTEXT

### 1.1. Build Context là gì?

**Build context** là **thư mục và files** được gửi đến Docker daemon khi build image.

**Command:**
```bash
$ docker build -t my-app .
#                                    ↑
#                              Build context (current directory)
```

**What is sent:**
- **All files** trong build context directory
- **Subdirectories** và files trong đó
- **Everything** except files trong .dockerignore

**Important:**
- **Sent to daemon**: Build context được gửi đến Docker daemon
- **Size matters**: Context size ảnh hưởng build time
- **One-way**: Chỉ gửi từ client → daemon (không ngược lại)

### 1.2. Build Context Size Impact

**Large build context:**
- **Slow transfer**: Mất thời gian gửi files
- **Memory usage**: Tốn memory trên daemon
- **Build time**: Build chậm hơn

**Small build context:**
- **Fast transfer**: Gửi files nhanh
- **Less memory**: Ít memory hơn
- **Faster builds**: Build nhanh hơn

**Example:**
```bash
# Large context (500MB)
$ docker build -t my-app .
# Sending build context: 500MB
# Time: 2 phút

# Small context (50MB với .dockerignore)
$ docker build -t my-app .
# Sending build context: 50MB
# Time: 10 giây
```

### 1.3. Build Context Best Practices

**1. Minimize context size:**
- Use .dockerignore
- Exclude unnecessary files
- Only include needed files

**2. Use specific paths:**
```bash
# ❌ Bad: Large context
$ docker build -t my-app .

# ✅ Good: Specific path
$ docker build -t my-app ./app
```

**3. Organize project:**
- Separate build context
- Keep Dockerfile close to needed files

---

## 🚫 PHẦN 2: .dockerignore

### 2.1. .dockerignore là gì?

**.dockerignore** là file giống `.gitignore`, dùng để **exclude files/folders** khỏi build context.

**Location:**
- **Same directory** với Dockerfile
- **Root** của build context

**Syntax:**
```
pattern
!exception
**/pattern
```

**Ví dụ:**
```
node_modules/
.git/
*.log
.env
```

### 2.2. .dockerignore Patterns

**Basic patterns:**
```
node_modules/        # Exclude directory
*.log                # Exclude all .log files
.git/                # Exclude .git directory
.env                 # Exclude .env file
```

**Advanced patterns:**
```
**/node_modules/     # Exclude node_modules ở mọi level
**/*.log             # Exclude .log files ở mọi level
!important.log       # Exception: include important.log
src/**/*.tmp         # Exclude .tmp files trong src/
```

**Important:**
- **Similar to .gitignore**: Syntax giống .gitignore
- **Pattern matching**: Support wildcards
- **Order matters**: Patterns được check theo thứ tự

### 2.3. Common .dockerignore Patterns

**Node.js:**
```
node_modules/
npm-debug.log
.npm
.env
.git/
.DS_Store
```

**Python:**
```
__pycache__/
*.pyc
*.pyo
*.pyd
.Python
venv/
.env
.git/
```

**General:**
```
.git/
.gitignore
*.log
.env
.DS_Store
.vscode/
.idea/
```

### 2.4. .dockerignore Best Practices

**1. Exclude dependencies:**
```
node_modules/
vendor/
```

**2. Exclude version control:**
```
.git/
.gitignore
.svn/
```

**3. Exclude IDE files:**
```
.vscode/
.idea/
*.swp
```

**4. Exclude build artifacts:**
```
dist/
build/
*.o
```

**5. Exclude sensitive files:**
```
.env
*.key
*.pem
secrets/
```

---

## 🏭 PRODUCTION STORY #1: Large Build Context

### Context

**Công ty:** E-commerce, 600 employees
**Hệ thống:** Node.js applications với Docker
**Traffic:** 15M requests/day
**Team:** 30 backend engineers

### Problem

**Tháng 10/2023:**
- **Build times quá chậm**: 10-15 phút per build
- **Root cause**: Build context quá lớn (800MB)
- **Impact**: CI/CD pipeline chậm

**Timeline:**
- **10:00 AM**: Developer push code
- **10:01 AM**: CI/CD start build
- **10:12 AM**: Build complete (11 phút)
- **10:15 AM**: Team investigate

**Impact:**
- **Build time**: 10-15 phút
- **CI/CD costs**: High costs
- **Developer productivity**: Lost time

### Investigation

**Root cause:**
```bash
$ du -sh .
# 800MB

$ docker build -t my-app .
# Sending build context: 800MB
# Time: 8 phút chỉ để send context
```

**Vấn đề:**
- **node_modules/**: 600MB (không cần)
- **.git/**: 100MB (không cần)
- **dist/**: 50MB (không cần)
- **Other files**: 50MB

**No .dockerignore:**
- Tất cả files được gửi
- Build context quá lớn

### Fix

**Solution: Create .dockerignore**
```
node_modules/
.git/
dist/
*.log
.env
.DS_Store
```

**Kết quả:**
```bash
$ du -sh .
# 800MB (total)

$ docker build -t my-app .
# Sending build context: 50MB
# Time: 30 giây để send context
```

**Build time:**
- **Before**: 11 phút
- **After**: 3 phút (73% reduction)

### Result

**Trước:**
- No .dockerignore
- **Build context**: 800MB
- **Build time**: 11 phút

**Sau:**
- With .dockerignore
- **Build context**: 50MB
- **Build time**: 3 phút

### Lesson Learned

1. **Always use .dockerignore**: Best practice
2. **Exclude dependencies**: node_modules, vendor, etc.
3. **Exclude version control**: .git, .svn, etc.
4. **Monitor context size**: Track context size

---

## 🏭 PRODUCTION STORY #2: Sensitive Files in Image

### Context

**Công ty:** Fintech, 400 employees
**Hệ thống:** Python applications với Docker
**Traffic:** 8M requests/day
**Team:** 20 backend engineers

### Problem

**Tháng 12/2023:**
- **Security incident**: Sensitive files trong image
- **Root cause**: .env file được copy vào image
- **Impact**: Security breach

**Timeline:**
- **2:00 PM**: Security scan
- **2:15 PM**: Phát hiện .env trong image
- **2:30 PM**: Team investigate
- **3:00 PM**: Fix và rotate secrets

**Impact:**
- **Security breach**: Sensitive data exposed
- **Compliance**: Không đạt compliance
- **Reputation**: Damage to reputation

### Investigation

**Root cause:**
```dockerfile
FROM python:3.9-slim
COPY . .
# ← Copy tất cả, including .env
```

**No .dockerignore:**
- .env file được copy vào image
- Sensitive data (API keys, passwords) trong image
- Image pushed to registry → exposed

**Test:**
```bash
$ docker run my-app cat .env
# Output: API_KEY=secret123
# → Sensitive data exposed
```

### Fix

**Solution 1: .dockerignore**
```
.env
*.key
*.pem
secrets/
```

**Solution 2: Don't copy .env**
```dockerfile
FROM python:3.9-slim
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY app.py .
# Don't copy .env
```

**Kết quả:**
- .env không có trong image
- Sensitive data không exposed
- Security improved

### Result

**Trước:**
- No .dockerignore
- **.env in image**: ✅ (security risk)
- **Sensitive data exposed**: ✅

**Sau:**
- With .dockerignore
- **.env in image**: ❌
- **Sensitive data exposed**: ❌

### Lesson Learned

1. **Always exclude sensitive files**: .env, keys, secrets
2. **Use .dockerignore**: Best practice
3. **Security scans**: Regular security scans
4. **Don't copy .env**: Never copy .env into image

---

## 🎓 TÓM TẮT

### Build Context

**Definition:**
- Files và directories gửi đến Docker daemon
- Size ảnh hưởng build performance

**Best practices:**
- Minimize context size
- Use specific paths
- Organize project

### .dockerignore

**Purpose:**
- Exclude files khỏi build context
- Reduce context size
- Improve security

**Best practices:**
- Exclude dependencies
- Exclude version control
- Exclude sensitive files
- Exclude build artifacts

### Common Patterns

**Node.js:**
- node_modules/, .npm, .env

**Python:**
- __pycache__/, venv/, .env

**General:**
- .git/, *.log, .DS_Store

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ bạn đã:
- ✅ Hiểu build context
- ✅ Biết cách dùng .dockerignore
- ✅ Tối ưu build context size

**Day tiếp theo (Day-018)** sẽ đi sâu vào:
- Image Size Optimization Strategies
- Techniques để giảm image size
- Best practices

---

## 📚 TÀI LIỆU THAM KHẢO

- .dockerignore: https://docs.docker.com/engine/reference/builder/#dockerignore-file
- Build context: https://docs.docker.com/build/guide/context/

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-018: Image-Size-Optimization](../Day-018-Image-Size-Optimization/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
