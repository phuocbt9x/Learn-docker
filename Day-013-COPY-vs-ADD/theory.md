# Day-013: COPY vs ADD - Trade-offs & Best Practices

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được sự khác biệt giữa COPY và ADD
- Biết khi nào dùng COPY, khi nào dùng ADD
- Hiểu được các tính năng đặc biệt của ADD (URL, tar extraction)
- Biết được trade-offs và best practices
- Viết được Dockerfile với COPY/ADD đúng cách

---

## 📋 PHẦN 1: COPY INSTRUCTION

### 1.1. COPY là gì?

**COPY** là instruction để **copy files/folders** từ build context vào image.

**Syntax:**
```dockerfile
COPY <src>... <dest>
COPY ["<src>",..., "<dest>"]
```

**Ví dụ:**
```dockerfile
COPY app.py /app/app.py
COPY config/ /app/config/
COPY *.txt /app/
```

**Đặc điểm:**
- **Simple**: Chỉ copy files từ build context
- **No URL**: Không hỗ trợ URL
- **No extraction**: Không tự động extract archives
- **Recommended**: Recommended cho hầu hết use cases

### 1.2. COPY Behavior

**Copy single file:**
```dockerfile
COPY app.py /app/app.py
```

**Copy directory:**
```dockerfile
COPY src/ /app/src/
```

**Copy multiple files:**
```dockerfile
COPY file1.txt file2.txt /app/
```

**Copy với wildcards:**
```dockerfile
COPY *.txt /app/
COPY config/*.json /app/config/
```

**Important:**
- **Build context**: Chỉ copy từ build context (thư mục chứa Dockerfile)
- **Absolute paths**: Dest path phải là absolute hoặc relative to WORKDIR
- **Permissions**: Giữ nguyên permissions của source files

### 1.3. COPY với .dockerignore

**.dockerignore** giống `.gitignore`, loại trừ files khỏi build context:

**.dockerignore:**
```
node_modules/
.git/
*.log
.env
```

**Kết quả:**
- `node_modules/` không được copy vào image
- `.git/` không được copy vào image
- `*.log` không được copy vào image

**Best practice:**
- **Luôn dùng .dockerignore**: Giảm build context size
- **Faster builds**: Ít files → faster COPY
- **Smaller images**: Không copy unnecessary files

### 1.4. COPY Use Cases

**Use cases cho COPY:**
- **Application code**: Copy source code
- **Configuration files**: Copy config files
- **Static assets**: Copy static files
- **Dependencies**: Copy dependencies (nếu không dùng package manager)

**Ví dụ:**
```dockerfile
FROM python:3.9-slim
COPY requirements.txt /app/
RUN pip install -r /app/requirements.txt
COPY app.py /app/app.py
COPY config/ /app/config/
```

---

## 📦 PHẦN 2: ADD INSTRUCTION

### 2.1. ADD là gì?

**ADD** là instruction để **copy files/folders** từ build context hoặc **URL** vào image, và có thể **auto-extract** archives.

**Syntax:**
```dockerfile
ADD <src>... <dest>
ADD ["<src>",..., "<dest>"]
```

**Ví dụ:**
```dockerfile
ADD app.py /app/app.py
ADD https://example.com/file.tar.gz /app/
ADD archive.tar.gz /app/
```

**Đặc điểm:**
- **URL support**: Có thể download từ URL
- **Auto-extract**: Tự động extract archives (tar, gzip, etc.)
- **More features**: Nhiều tính năng hơn COPY
- **Less predictable**: Khó predict behavior hơn COPY

### 2.2. ADD với URL

**ADD có thể download từ URL:**

```dockerfile
ADD https://example.com/file.txt /app/file.txt
ADD https://github.com/user/repo/archive/main.tar.gz /app/
```

**Behavior:**
- **Download**: Download file từ URL
- **Permissions**: File được tạo với permissions 600
- **Cache**: Không cache URL downloads (luôn download mới)

**Use cases:**
- **Download dependencies**: Download files từ external sources
- **Download releases**: Download application releases
- **Download configs**: Download configuration files

**Important:**
- **No authentication**: Không hỗ trợ authentication
- **No cache**: Không cache → slower builds
- **Not recommended**: Không recommended cho production

### 2.3. ADD với Auto-extraction

**ADD tự động extract archives:**

```dockerfile
ADD archive.tar.gz /app/
ADD file.zip /app/
ADD data.tgz /app/
```

**Supported formats:**
- **tar**: `.tar`
- **gzip**: `.tar.gz`, `.tgz`
- **bzip2**: `.tar.bz2`, `.tbz2`
- **xz**: `.tar.xz`
- **zip**: `.zip`

**Behavior:**
- **Auto-extract**: Tự động extract nếu là archive
- **No extract**: Nếu không phải archive, copy như bình thường
- **Dest path**: Extract vào dest path

**Ví dụ:**
```dockerfile
ADD app.tar.gz /app/
# Extract app.tar.gz vào /app/
# → /app/app/ (nếu archive có folder app/)
```

**Important:**
- **Unpredictable**: Khó predict structure sau khi extract
- **Not recommended**: Không recommended cho production

### 2.4. ADD Use Cases

**Use cases cho ADD:**
- **Download từ URL**: Khi cần download files
- **Extract archives**: Khi cần extract archives
- **Legacy code**: Khi maintain legacy Dockerfiles

**Ví dụ:**
```dockerfile
# Download và extract
ADD https://example.com/app.tar.gz /app/
# Download app.tar.gz và extract vào /app/
```

---

## 🔄 PHẦN 3: COPY vs ADD

### 3.1. So Sánh

| Tiêu chí | COPY | ADD |
|----------|------|-----|
| **URL support** | ❌ Không | ✅ Có |
| **Auto-extract** | ❌ Không | ✅ Có |
| **Predictability** | ✅ Predictable | ⚠️ Less predictable |
| **Cache** | ✅ Cache tốt | ⚠️ URL không cache |
| **Recommendation** | ✅ Recommended | ⚠️ Use sparingly |
| **Use cases** | Most cases | Special cases |

### 3.2. Khi Nào Dùng COPY?

**Dùng COPY khi:**
- **Copy files từ build context**: 99% use cases
- **Predictable behavior**: Muốn behavior predictable
- **Better caching**: Muốn better caching
- **Production**: Production Dockerfiles

**Ví dụ:**
```dockerfile
FROM python:3.9-slim
COPY requirements.txt /app/
RUN pip install -r /app/requirements.txt
COPY app.py /app/app.py
COPY config/ /app/config/
```

**Lý do:**
- **Simple**: Đơn giản, dễ hiểu
- **Predictable**: Behavior rõ ràng
- **Cache**: Cache tốt
- **Recommended**: Best practice

### 3.3. Khi Nào Dùng ADD?

**Dùng ADD khi:**
- **Download từ URL**: Cần download files
- **Extract archives**: Cần extract archives
- **Legacy code**: Maintain legacy Dockerfiles

**Ví dụ:**
```dockerfile
# Download từ URL
ADD https://example.com/file.txt /app/file.txt

# Extract archive
ADD app.tar.gz /app/
```

**Lưu ý:**
- **Not recommended**: Không recommended cho production
- **Use sparingly**: Chỉ dùng khi thực sự cần
- **Consider alternatives**: Nên consider alternatives

### 3.4. Alternatives cho ADD

**Thay vì ADD với URL:**

**Option 1: Download trong RUN**
```dockerfile
RUN curl -o /app/file.txt https://example.com/file.txt
```

**Pros:**
- **Better control**: Control tốt hơn
- **Error handling**: Có thể handle errors
- **Cache**: Cache tốt hơn

**Option 2: Multi-stage build**
```dockerfile
FROM alpine:latest AS downloader
RUN apk add --no-cache curl
RUN curl -o /app/file.txt https://example.com/file.txt

FROM python:3.9-slim
COPY --from=downloader /app/file.txt /app/file.txt
```

**Pros:**
- **Separation**: Tách biệt download và runtime
- **Smaller image**: Runtime image nhỏ hơn

**Thay vì ADD với extraction:**

**Option 1: Extract trong RUN**
```dockerfile
COPY app.tar.gz /tmp/
RUN tar -xzf /tmp/app.tar.gz -C /app/ && rm /tmp/app.tar.gz
```

**Pros:**
- **Explicit**: Rõ ràng, dễ hiểu
- **Control**: Control tốt hơn
- **Cleanup**: Có thể cleanup archive sau khi extract

**Option 2: Pre-extract**
```dockerfile
# Extract trước khi build
COPY app/ /app/
```

**Pros:**
- **Simple**: Đơn giản nhất
- **No extraction**: Không cần extraction

---

## 🎯 PHẦN 4: BEST PRACTICES

### 4.1. General Best Practices

**1. Luôn dùng COPY trừ khi cần ADD:**
```dockerfile
# ✅ Good
COPY app.py /app/app.py

# ⚠️ Only if needed
ADD app.py /app/app.py
```

**2. Dùng .dockerignore:**
```dockerfile
# .dockerignore
node_modules/
.git/
*.log
```

**3. Copy dependencies trước source code:**
```dockerfile
# ✅ Good: Dependencies trước
COPY requirements.txt /app/
RUN pip install -r /app/requirements.txt
COPY app.py /app/app.py

# ❌ Bad: Source code trước
COPY app.py /app/app.py
COPY requirements.txt /app/
RUN pip install -r /app/requirements.txt
```

**Lý do:**
- **Layer caching**: Dependencies ít thay đổi → cache tốt
- **Faster rebuilds**: Chỉ rebuild khi source code thay đổi

**4. Use absolute paths:**
```dockerfile
# ✅ Good
COPY app.py /app/app.py

# ⚠️ Works but less clear
WORKDIR /app
COPY app.py .
```

### 4.2. COPY Best Practices

**1. Copy specific files:**
```dockerfile
# ✅ Good: Specific
COPY requirements.txt /app/

# ⚠️ Less specific
COPY . /app/
```

**2. Copy directories với trailing slash:**
```dockerfile
# ✅ Good: Trailing slash
COPY src/ /app/src/

# ⚠️ Less clear
COPY src /app/src
```

**3. Use wildcards carefully:**
```dockerfile
# ✅ Good: Specific pattern
COPY *.txt /app/

# ⚠️ Too broad
COPY * /app/
```

### 4.3. ADD Best Practices

**1. Avoid ADD với URL:**
```dockerfile
# ❌ Not recommended
ADD https://example.com/file.txt /app/file.txt

# ✅ Better: Use RUN
RUN curl -o /app/file.txt https://example.com/file.txt
```

**2. Avoid ADD với auto-extraction:**
```dockerfile
# ❌ Not recommended
ADD app.tar.gz /app/

# ✅ Better: Explicit extraction
COPY app.tar.gz /tmp/
RUN tar -xzf /tmp/app.tar.gz -C /app/ && rm /tmp/app.tar.gz
```

**3. Document khi dùng ADD:**
```dockerfile
# Use ADD only for URL download
# Consider using RUN curl instead
ADD https://example.com/file.txt /app/file.txt
```

---

## 🏭 PRODUCTION STORY #1: ADD URL Cache Issues

### Context

**Công ty:** Fintech startup, 150 employees
**Hệ thống:** Python microservices với Docker
**Traffic:** 2M requests/day
**Team:** 15 backend engineers

### Problem

**Tháng 3/2023:**
- **Build times tăng đột ngột**: Từ 5 phút → 20 phút
- **Root cause**: ADD với URL không cache → luôn download mới
- **Impact**: CI/CD pipeline chậm, developers phải chờ lâu

**Timeline:**
- **9:00 AM**: Developer push code
- **9:01 AM**: CI/CD start build
- **9:15 AM**: Build vẫn chạy (thường chỉ 5 phút)
- **9:20 AM**: Build complete (20 phút thay vì 5 phút)
- **9:25 AM**: Team investigate

**Impact:**
- **Build time**: Tăng 4x (5 → 20 phút)
- **Developer productivity**: Giảm do chờ build
- **CI/CD costs**: Tăng do build time lâu hơn

### Investigation

**Root cause:**
```dockerfile
# Dockerfile
ADD https://example.com/dependencies.tar.gz /app/deps/
```

**Vấn đề:**
- **No cache**: ADD với URL không cache
- **Always download**: Mỗi build đều download lại
- **Slow network**: Network chậm → build chậm

**Test:**
```bash
$ time docker build -t my-app .
# Real: 20m15s
# → Download dependencies.tar.gz mất 15 phút
```

### Fix

**Solution 1: Use RUN với curl**
```dockerfile
RUN curl -o /tmp/deps.tar.gz https://example.com/dependencies.tar.gz && \
    tar -xzf /tmp/deps.tar.gz -C /app/deps/ && \
    rm /tmp/deps.tar.gz
```

**Kết quả:**
- **Cache**: RUN có thể cache
- **Faster**: Chỉ download khi dependencies thay đổi
- **Build time**: Giảm từ 20 phút → 5 phút

**Solution 2: Download trong build script**
```dockerfile
# build.sh
curl -o dependencies.tar.gz https://example.com/dependencies.tar.gz

# Dockerfile
COPY dependencies.tar.gz /tmp/
RUN tar -xzf /tmp/dependencies.tar.gz -C /app/deps/ && \
    rm /tmp/dependencies.tar.gz
```

**Kết quả:**
- **Better control**: Control tốt hơn
- **Cache**: COPY cache tốt
- **Build time**: Giảm từ 20 phút → 5 phút

### Result

**Trước:**
- ADD với URL
- **Build time**: 20 phút
- **No cache**: Luôn download mới

**Sau:**
- RUN với curl hoặc COPY
- **Build time**: 5 phút
- **Cache**: Cache tốt, chỉ download khi cần

### Lesson Learned

1. **Avoid ADD với URL**: Không cache → slow builds
2. **Use RUN với curl**: Better control và cache
3. **Consider build scripts**: Download trước khi build
4. **Monitor build times**: Track build times để detect issues

---

## 🏭 PRODUCTION STORY #2: ADD Auto-extraction Issues

### Context

**Công ty:** E-commerce, 500 employees
**Hệ thống:** Java applications với Docker
**Traffic:** 10M requests/day
**Team:** 25 backend engineers

### Problem

**Tháng 7/2023:**
- **Image structure không đúng**: Files extract vào wrong location
- **Root cause**: ADD auto-extraction không predictable
- **Impact**: Application không tìm thấy files → crash

**Timeline:**
- **2:00 PM**: Deploy new version
- **2:05 PM**: Application crash (file not found)
- **2:10 PM**: Team investigate
- **2:15 PM**: Root cause: Files extract vào wrong location
- **2:20 PM**: Fix và redeploy

**Impact:**
- **15 minutes downtime**
- **100K requests failed**
- **Customer complaints**

### Investigation

**Root cause:**
```dockerfile
# Dockerfile
ADD app.tar.gz /app/
```

**Vấn đề:**
- **Unpredictable structure**: Archive có thể có nhiều structure
- **Wrong location**: Files extract vào location không đúng
- **No control**: Không control được extraction process

**Archive structure:**
```
app.tar.gz
  └── app/
      ├── src/
      └── config/
```

**Expected:**
```
/app/
  ├── src/
  └── config/
```

**Actual:**
```
/app/
  └── app/
      ├── src/
      └── config/
```

**Result:**
- Application tìm files ở `/app/src/`
- Files thực tế ở `/app/app/src/`
- → File not found error

### Fix

**Solution: Explicit extraction**
```dockerfile
COPY app.tar.gz /tmp/
RUN tar -xzf /tmp/app.tar.gz -C /tmp/ && \
    mv /tmp/app/* /app/ && \
    rm -rf /tmp/app.tar.gz /tmp/app
```

**Kết quả:**
- **Predictable**: Structure rõ ràng
- **Control**: Control được extraction
- **Correct location**: Files ở đúng location

**Alternative: Pre-extract**
```dockerfile
# Extract trước khi build
COPY app/ /app/
```

**Kết quả:**
- **Simple**: Đơn giản nhất
- **No extraction**: Không cần extraction
- **Predictable**: Structure rõ ràng

### Result

**Trước:**
- ADD auto-extraction
- **Unpredictable structure**: Files ở wrong location
- **Crashes**: Application crash

**Sau:**
- Explicit extraction
- **Predictable structure**: Files ở đúng location
- **No crashes**: Application chạy đúng

### Lesson Learned

1. **Avoid ADD auto-extraction**: Không predictable
2. **Use explicit extraction**: Control tốt hơn
3. **Pre-extract nếu có thể**: Đơn giản nhất
4. **Test image structure**: Verify structure sau khi build

---

## 🎓 TÓM TẮT

### COPY

**Chức năng:**
- Copy files từ build context
- Simple và predictable
- Recommended cho hầu hết use cases

**Best practices:**
- Luôn dùng COPY trừ khi cần ADD
- Dùng .dockerignore
- Copy dependencies trước source code

### ADD

**Chức năng:**
- Copy files từ build context hoặc URL
- Auto-extract archives
- Nhiều tính năng hơn COPY

**Best practices:**
- Chỉ dùng khi thực sự cần
- Tránh ADD với URL (dùng RUN curl)
- Tránh ADD auto-extraction (dùng explicit extraction)

### Decision Matrix

**Dùng COPY khi:**
- Copy files từ build context (99% cases)
- Muốn predictable behavior
- Muốn better caching

**Dùng ADD khi:**
- Download từ URL (nhưng consider alternatives)
- Extract archives (nhưng consider alternatives)
- Legacy code

**Alternatives:**
- URL: RUN với curl
- Extraction: Explicit extraction trong RUN
- Pre-extract: Extract trước khi build

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ bạn đã:
- ✅ Hiểu COPY vs ADD
- ✅ Biết khi nào dùng gì
- ✅ Hiểu trade-offs

**Day tiếp theo (Day-014)** sẽ đi sâu vào:
- WORKDIR, ENV, ARG
- Environment management
- Build-time vs runtime variables

---

## 📚 TÀI LIỆU THAM KHẢO

- COPY: https://docs.docker.com/engine/reference/builder/#copy
- ADD: https://docs.docker.com/engine/reference/builder/#add
- .dockerignore: https://docs.docker.com/engine/reference/builder/#dockerignore-file

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-014: WORKDIR-ENV-ARG](../Day-014-WORKDIR-ENV-ARG/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
