# Day-016: Layer Caching - Build Performance - Exercises

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi hoàn thành các bài tập này, bạn sẽ:
- Tối ưu được Dockerfile cho layer caching
- Phân tích được cache behavior
- Debug được cache issues
- Measure được build performance improvements

---

## 📝 BÀI TẬP 1: Basic Layer Ordering

### Yêu cầu

Tối ưu Dockerfile để maximize cache hits:

**Current:**
```dockerfile
FROM node:18-alpine
COPY . .
RUN npm install
CMD ["node", "index.js"]
```

### Câu hỏi

1. **Phân tích vấn đề:**
   - Tại sao cache không tốt?
   - Layers nào rebuild mỗi lần?

2. **Optimize Dockerfile:**
   - Reorder instructions
   - Maximize cache hits

3. **Test**: Build và measure cache hits

### Deliverables

- Optimized Dockerfile
- Analysis
- Test results

---

## 📝 BÀI TẬP 2: Dependencies Before Source

### Yêu cầu

Tối ưu Dockerfile cho Python application:

**Current:**
```dockerfile
FROM python:3.9-slim
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "app.py"]
```

### Câu hỏi

1. **Optimize Dockerfile**
2. **Test**: Build với và không có cache, so sánh
3. **Measure**: Measure build time improvements

### Deliverables

- Optimized Dockerfile
- Comparison results
- Analysis

---

## 📝 BÀI TẬP 3: Combine RUN Commands

### Yêu cầu

Tối ưu Dockerfile bằng cách combine RUN commands:

**Current:**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y wget
RUN apt-get clean
```

### Câu hỏi

1. **Combine RUN commands**
2. **Test**: Compare layer count và build time
3. **Trade-offs**: Phân tích trade-offs

### Deliverables

- Optimized Dockerfile
- Comparison
- Trade-off analysis

---

## 📝 BÀI TẬP 4: Cache Analysis

### Yêu cầu

Analyze cache behavior của Dockerfile:

**Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build
CMD ["node", "dist/index.js"]
```

### Câu hỏi

1. **Build lần 1**: Measure build time
2. **Thay đổi source code**: Rebuild và check cache
3. **Thay đổi package.json**: Rebuild và check cache
4. **Analysis**: Phân tích cache hits/misses

### Deliverables

- Test results
- Cache analysis
- Recommendations

---

## 📝 BÀI TẬP 5: Cache Debugging

### Scenario

**Problem:** Build time tăng đột ngột, cache không work

**Dockerfile:**
```dockerfile
FROM python:3.9-slim
COPY requirements.txt .
RUN pip install -r requirements.txt
COPY . .
RUN python manage.py collectstatic
CMD ["gunicorn", "app.wsgi:application"]
```

### Câu hỏi

1. **Root cause**: Tại sao cache không work?
2. **Fix**: Optimize Dockerfile
3. **Test**: Verify fix

### Deliverables

- Analysis
- Fixed Dockerfile
- Test results

---

## 📝 BÀI TẬP 6: Practical Optimization

### Yêu cầu

Tối ưu Dockerfile cho Node.js application với:

**Requirements:**
- Maximize cache hits
- Fast rebuilds
- Production-ready

**Current Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build
CMD ["node", "dist/index.js"]
```

### Câu hỏi

1. **Optimize Dockerfile**
2. **Test**: Measure improvements
3. **Document**: Document optimizations

### Deliverables

- Optimized Dockerfile
- Test results
- Documentation

---

## 📝 BÀI TẬP 7: Multi-stage Cache

### Yêu cầu

Tối ưu multi-stage Dockerfile cho caching:

**Current:**
```dockerfile
FROM node:18-alpine AS builder
COPY . .
RUN npm install
RUN npm run build

FROM node:18-alpine
COPY --from=builder /app/dist ./dist
CMD ["node", "dist/index.js"]
```

### Câu hỏi

1. **Optimize cho caching**
2. **Test**: Verify cache behavior
3. **Compare**: Single-stage vs multi-stage cache

### Deliverables

- Optimized Dockerfile
- Test results
- Comparison

---

## 📝 BÀI TẬP 8: Production Analysis

### Scenario

**Company:** SaaS platform
**Build time:** 20 phút
**Cache hits:** 20%
**Requirement:** Build time < 5 phút, cache hits > 80%

### Câu hỏi

1. **Analyze current Dockerfile**
2. **Optimize**: Apply all optimization techniques
3. **Measure**: Verify improvements

### Deliverables

- Analysis
- Optimized Dockerfile
- Results

---

## 📝 BÀI TẬP 9: Advanced - Conditional Caching

### Yêu cầu

Tối ưu Dockerfile với conditional logic mà vẫn maximize cache:

**Requirements:**
- Support dev và prod builds
- Maximize cache cho cả 2 cases

### Câu hỏi

1. **Design Dockerfile**
2. **Test**: Build với different targets
3. **Analysis**: Cache behavior

### Deliverables

- Dockerfile
- Test results
- Analysis

---

## 📝 BÀI TẬP 10: Cache Monitoring

### Yêu cầu

Create script để monitor cache behavior:

**Requirements:**
- Measure build time
- Count cache hits/misses
- Generate report

### Câu hỏi

1. **Create monitoring script**
2. **Test**: Run với different scenarios
3. **Report**: Generate cache report

### Deliverables

- Monitoring script
- Test results
- Report

---

## 🎯 CHECKLIST

- [ ] Đã optimize Dockerfile cho caching
- [ ] Đã test cache behavior
- [ ] Đã measure improvements
- [ ] Đã document optimizations

---

**Lưu ý:** Tất cả số liệu trong exercises là illustrative/approximate cho mục đích giáo dục.

