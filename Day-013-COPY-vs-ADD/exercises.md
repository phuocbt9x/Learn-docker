# Day-013: COPY vs ADD - Exercises

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi hoàn thành các bài tập này, bạn sẽ:
- Viết được Dockerfile với COPY/ADD đúng cách
- Hiểu được khi nào dùng COPY, khi nào dùng ADD
- Tối ưu được layer caching với COPY
- Phân tích được trade-offs giữa COPY và ADD
- Debug được các vấn đề liên quan đến COPY/ADD

---

## 📝 BÀI TẬP 1: COPY Basics

### Yêu cầu

Tạo Dockerfile cho Python application với:
- Base image: `python:3.9-slim`
- Copy `requirements.txt` vào `/app/requirements.txt`
- Copy `app.py` vào `/app/app.py`
- Copy `config/` directory vào `/app/config/`
- Install dependencies và run application

### Câu hỏi

1. **Tại sao copy `requirements.txt` trước `app.py`?**
2. **Làm thế nào để optimize layer caching?**
3. **Test**: Build image và verify files được copy đúng

### Deliverables

- Dockerfile
- Commands để build và verify
- Giải thích layer caching optimization

---

## 📝 BÀI TẬP 2: .dockerignore

### Yêu cầu

Tạo `.dockerignore` file để exclude:
- `node_modules/`
- `.git/`
- `*.log`
- `.env`
- `__pycache__/`
- `*.pyc`

### Câu hỏi

1. **Tại sao cần .dockerignore?**
2. **Impact của .dockerignore lên build time?**
3. **Test**: Build image với và không có .dockerignore, so sánh build time

### Deliverables

- .dockerignore file
- Comparison (build time với/không có .dockerignore)
- Analysis

---

## 📝 BÀI TẬP 3: COPY vs ADD Comparison

### Yêu cầu

Tạo 2 Dockerfiles cho cùng một application:

**Dockerfile 1: COPY**
```dockerfile
FROM python:3.9-slim
COPY requirements.txt /app/
RUN pip install -r /app/requirements.txt
COPY app.py /app/app.py
```

**Dockerfile 2: ADD**
```dockerfile
FROM python:3.9-slim
ADD requirements.txt /app/
RUN pip install -r /app/requirements.txt
ADD app.py /app/app.py
```

### Câu hỏi

1. **So sánh behavior:**
   - Có khác biệt gì không?
   - Khi nào nên dùng COPY? Khi nào nên dùng ADD?

2. **Test**: Build cả 2 images và so sánh:
   - Build time
   - Image size
   - Behavior

3. **Recommendation**: Nên dùng cái nào? Tại sao?

### Deliverables

- 2 Dockerfiles
- Comparison table
- Recommendation với justification

---

## 📝 BÀI TẬP 4: ADD với URL

### Yêu cầu

**Scenario:** Cần download file từ URL

**Option 1: ADD với URL**
```dockerfile
ADD https://example.com/file.txt /app/file.txt
```

**Option 2: RUN với curl**
```dockerfile
RUN curl -o /app/file.txt https://example.com/file.txt
```

**Option 3: Multi-stage build**
```dockerfile
FROM alpine:latest AS downloader
RUN apk add --no-cache curl
RUN curl -o /app/file.txt https://example.com/file.txt

FROM python:3.9-slim
COPY --from=downloader /app/file.txt /app/file.txt
```

### Câu hỏi

1. **So sánh 3 options:**
   - Cache behavior?
   - Build time?
   - Image size?
   - Error handling?

2. **Test**: Build cả 3 options và so sánh

3. **Recommendation**: Option nào tốt nhất? Tại sao?

### Deliverables

- 3 Dockerfiles
- Comparison table
- Test results
- Recommendation

---

## 📝 BÀI TẬP 5: ADD với Auto-extraction

### Yêu cầu

**Scenario:** Cần extract archive `app.tar.gz`

**Option 1: ADD auto-extraction**
```dockerfile
ADD app.tar.gz /app/
```

**Option 2: Explicit extraction**
```dockerfile
COPY app.tar.gz /tmp/
RUN tar -xzf /tmp/app.tar.gz -C /app/ && rm /tmp/app.tar.gz
```

**Option 3: Pre-extract**
```dockerfile
COPY app/ /app/
```

### Câu hỏi

1. **So sánh 3 options:**
   - Predictability?
   - Control?
   - Build time?
   - Image size?

2. **Test**: Tạo archive và test cả 3 options

3. **Recommendation**: Option nào tốt nhất? Tại sao?

### Deliverables

- 3 Dockerfiles
- Test archive
- Comparison table
- Recommendation

---

## 📝 BÀI TẬP 6: Layer Caching Optimization

### Yêu cầu

Tối ưu Dockerfile để maximize layer caching:

**Dockerfile ban đầu:**
```dockerfile
FROM python:3.9-slim
COPY . /app/
RUN pip install -r /app/requirements.txt
CMD ["python", "/app/app.py"]
```

### Câu hỏi

1. **Phân tích vấn đề:**
   - Tại sao layer caching không tốt?
   - Khi nào cần rebuild?

2. **Optimize Dockerfile:**
   - Copy dependencies trước source code
   - Tối ưu COPY commands

3. **Test**: 
   - Build image
   - Thay đổi source code (không thay đổi requirements.txt)
   - Rebuild và verify chỉ rebuild layers cần thiết

### Deliverables

- Optimized Dockerfile
- Explanation
- Test results

---

## 📝 BÀI TẬP 7: Practical Dockerfile

### Yêu cầu

Tạo Dockerfile cho Node.js application với:

**Requirements:**
- Base image: `node:18-alpine`
- Copy `package.json` và `package-lock.json`
- Install dependencies
- Copy source code
- Expose port 3000
- Run application

**Files structure:**
```
.
├── Dockerfile
├── package.json
├── package-lock.json
├── src/
│   ├── index.js
│   └── routes/
├── public/
│   └── static/
├── node_modules/ (should be excluded)
├── .git/ (should be excluded)
└── .env (should be excluded)
```

### Câu hỏi

1. **Viết Dockerfile hoàn chỉnh:**
   - Dùng COPY hay ADD?
   - Tối ưu layer caching
   - Tạo .dockerignore

2. **Test**: Build và verify application

### Deliverables

- Dockerfile
- .dockerignore
- Commands để test

---

## 📝 BÀI TẬP 8: Troubleshooting

### Scenario

**Dockerfile:**
```dockerfile
FROM python:3.9-slim
ADD https://example.com/deps.tar.gz /app/
RUN pip install -r /app/requirements.txt
COPY app.py /app/app.py
```

**Problem:**
- Build time rất chậm (20+ phút)
- Mỗi build đều download deps.tar.gz lại
- Network timeout errors

### Câu hỏi

1. **Root cause là gì?**
2. **Fix Dockerfile:**
   - Thay ADD với URL bằng alternative
   - Optimize build process

3. **Test**: Verify build time giảm

### Deliverables

- Fixed Dockerfile
- Explanation
- Test results (build time)

---

## 📝 BÀI TẬP 9: Production Analysis

### Scenario

**Company:** SaaS platform
**Application:** Python microservice
**Current Dockerfile:**
```dockerfile
FROM python:3.9-slim
ADD https://github.com/user/repo/archive/main.tar.gz /tmp/
RUN tar -xzf /tmp/main.tar.gz -C /app/ && rm /tmp/main.tar.gz
RUN pip install -r /app/requirements.txt
COPY app.py /app/app.py
```

**Requirements:**
- Build time phải < 5 phút
- Image size phải < 500MB
- Build phải reliable (không fail do network)

### Câu hỏi

1. **Phân tích current Dockerfile:**
   - Ưu điểm?
   - Nhược điểm?
   - Production risks?

2. **Recommendation:**
   - Refactor Dockerfile
   - Optimize build time
   - Improve reliability

3. **Refactor Dockerfile** theo recommendation

### Deliverables

- Analysis document
- Refactored Dockerfile
- Justification

---

## 📝 BÀI TẬP 10: Advanced - Multi-file Copy

### Yêu cầu

**Scenario:** Cần copy nhiều files với patterns khác nhau

**Files structure:**
```
.
├── src/
│   ├── *.js
│   └── *.json
├── config/
│   ├── *.yaml
│   └── *.yml
├── static/
│   └── *.css
└── templates/
    └── *.html
```

### Câu hỏi

1. **Viết Dockerfile:**
   - Copy files với wildcards
   - Tối ưu số lượng COPY commands
   - Maintain layer caching

2. **So sánh approaches:**
   - Option 1: Multiple COPY commands
   - Option 2: Single COPY với wildcards
   - Option 3: Copy directories

3. **Recommendation**: Approach nào tốt nhất?

### Deliverables

- Dockerfile với multiple approaches
- Comparison table
- Recommended approach với justification

---

## 🎯 CHECKLIST

Trước khi submit, đảm bảo:

- [ ] Đã viết Dockerfile với COPY/ADD đúng syntax
- [ ] Đã tạo .dockerignore
- [ ] Đã tối ưu layer caching
- [ ] Đã phân tích trade-offs
- [ ] Đã test build time
- [ ] Đã giải thích decisions

---

## 📚 HINTS

1. **COPY vs ADD:**
   - COPY: Recommended cho hầu hết cases
   - ADD: Chỉ dùng khi cần URL hoặc auto-extraction

2. **Layer caching:**
   - Copy dependencies trước source code
   - Dependencies ít thay đổi → cache tốt

3. **.dockerignore:**
   - Giảm build context size
   - Faster builds
   - Smaller images

4. **ADD alternatives:**
   - URL: RUN với curl
   - Extraction: Explicit extraction trong RUN

---

**Lưu ý:** Tất cả số liệu trong exercises là illustrative/approximate cho mục đích giáo dục.

