# Day-017: .dockerignore & Build Context - Exercises

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi hoàn thành các bài tập này, bạn sẽ:
- Tạo được .dockerignore file đúng cách
- Tối ưu được build context size
- Phân tích được impact của build context
- Debug được các vấn đề liên quan đến build context

---

## 📝 BÀI TẬP 1: Basic .dockerignore

### Yêu cầu

Tạo .dockerignore cho Node.js project:

**Files structure:**
```
.
├── Dockerfile
├── package.json
├── node_modules/ (600MB)
├── .git/ (100MB)
├── dist/
├── *.log
└── .env
```

### Câu hỏi

1. **Tạo .dockerignore file**
2. **Test**: Measure build context size với và không có .dockerignore
3. **Compare**: So sánh build time

### Deliverables

- .dockerignore file
- Comparison results
- Analysis

---

## 📝 BÀI TẬP 2: Python .dockerignore

### Yêu cầu

Tạo .dockerignore cho Python project:

**Files structure:**
```
.
├── Dockerfile
├── requirements.txt
├── __pycache__/
├── venv/
├── .env
├── *.pyc
└── .git/
```

### Câu hỏi

1. **Tạo .dockerignore file**
2. **Test**: Verify files được exclude
3. **Security**: Verify .env không có trong image

### Deliverables

- .dockerignore file
- Test results
- Security verification

---

## 📝 BÀI TẬP 3: Build Context Analysis

### Yêu cầu

Analyze build context của project:

**Task:**
1. Measure build context size
2. Identify large files/directories
3. Create .dockerignore để optimize

### Câu hỏi

1. **Analyze**: Identify files cần exclude
2. **Create .dockerignore**
3. **Measure**: Compare before/after

### Deliverables

- Analysis report
- .dockerignore file
- Comparison results

---

## 📝 BÀI TẬP 4: Advanced Patterns

### Yêu cầu

Tạo .dockerignore với advanced patterns:

**Requirements:**
- Exclude node_modules ở mọi level
- Exclude .log files nhưng keep important.log
- Exclude .tmp files trong src/

### Câu hỏi

1. **Tạo .dockerignore với advanced patterns**
2. **Test**: Verify patterns work
3. **Document**: Document patterns

### Deliverables

- .dockerignore file
- Test results
- Documentation

---

## 📝 BÀI TẬP 5: Security Analysis

### Yêu cầu

Analyze security risks trong build context:

**Task:**
1. Identify sensitive files
2. Verify .dockerignore excludes them
3. Test image không có sensitive files

### Câu hỏi

1. **Identify**: List sensitive files
2. **Verify**: Check .dockerignore
3. **Test**: Verify files không có trong image

### Deliverables

- Security analysis
- .dockerignore file
- Test results

---

## 📝 BÀI TẬP 6: Multi-language .dockerignore

### Yêu cầu

Tạo .dockerignore cho multi-language project:

**Files:**
- Node.js: node_modules/, .npm
- Python: venv/, __pycache__/
- Go: vendor/, *.o
- General: .git/, *.log

### Câu hỏi

1. **Tạo .dockerignore**
2. **Test**: Verify all patterns work
3. **Organize**: Organize patterns by language

### Deliverables

- .dockerignore file
- Test results
- Organization notes

---

## 📝 BÀI TẬP 7: Build Context Optimization

### Yêu cầu

Optimize build context cho large project:

**Current:**
- Build context: 1GB
- Build time: 20 phút

**Requirement:**
- Build context: < 100MB
- Build time: < 5 phút

### Câu hỏi

1. **Analyze**: Identify optimization opportunities
2. **Optimize**: Create .dockerignore
3. **Measure**: Verify improvements

### Deliverables

- Analysis
- Optimized .dockerignore
- Results

---

## 📝 BÀI TẬP 8: Troubleshooting

### Scenario

**Problem:** Build context vẫn lớn sau khi có .dockerignore

**Current .dockerignore:**
```
node_modules/
.git/
```

**Build context:** 800MB (expected 50MB)

### Câu hỏi

1. **Root cause**: Tại sao vẫn lớn?
2. **Fix**: Update .dockerignore
3. **Test**: Verify fix

### Deliverables

- Analysis
- Fixed .dockerignore
- Test results

---

## 📝 BÀI TẬP 9: Production Analysis

### Scenario

**Company:** SaaS platform
**Build context:** 2GB
**Build time:** 30 phút
**Requirement:** < 200MB, < 5 phút

### Câu hỏi

1. **Analyze current state**
2. **Create optimization plan**
3. **Implement và measure**

### Deliverables

- Analysis
- Optimization plan
- Results

---

## 📝 BÀI TẬP 10: Best Practices

### Yêu cầu

Create comprehensive .dockerignore với best practices:

**Requirements:**
- Cover all common cases
- Well organized
- Documented

### Câu hỏi

1. **Create .dockerignore**
2. **Document**: Document patterns
3. **Test**: Verify all patterns

### Deliverables

- .dockerignore file
- Documentation
- Test results

---

## 🎯 CHECKLIST

- [ ] Đã tạo .dockerignore
- [ ] Đã test build context size
- [ ] Đã verify security
- [ ] Đã document patterns

---

**Lưu ý:** Tất cả số liệu trong exercises là illustrative/approximate cho mục đích giáo dục.

