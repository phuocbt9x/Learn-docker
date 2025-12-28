# Day-018: Image Size Optimization - Exercises

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi hoàn thành các bài tập này, bạn sẽ:
- Tối ưu được image size
- Phân tích được image composition
- Áp dụng được các optimization techniques

---

## 📝 BÀI TẬP 1: Base Image Comparison

### Yêu cầu

So sánh image size với different base images:

**Test:**
1. Build với ubuntu:20.04
2. Build với debian:bullseye-slim
3. Build với alpine:latest

### Câu hỏi

1. **Measure image sizes**
2. **Compare**: So sánh sizes
3. **Recommend**: Base image nào tốt nhất?

### Deliverables

- Comparison table
- Recommendations

---

## 📝 BÀI TẬP 2: Cleanup Techniques

### Yêu cầu

Optimize Dockerfile với cleanup:

**Current:**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update
RUN apt-get install -y curl wget
COPY . .
```

### Câu hỏi

1. **Add cleanup**
2. **Measure**: Compare image size
3. **Document**: Document improvements

### Deliverables

- Optimized Dockerfile
- Size comparison
- Documentation

---

## 📝 BÀI TẬP 3: Multi-stage Optimization

### Yêu cầu

Tối ưu multi-stage Dockerfile:

**Current:**
```dockerfile
FROM node:18-alpine AS builder
COPY . .
RUN npm install
RUN npm run build

FROM node:18-alpine
COPY --from=builder /app .
CMD ["node", "index.js"]
```

### Câu hỏi

1. **Optimize**: Minimize final image
2. **Test**: Measure improvements
3. **Compare**: Before/after

### Deliverables

- Optimized Dockerfile
- Test results
- Comparison

---

## 📝 BÀI TẬP 4: Production Optimization

### Yêu cầu

Optimize Dockerfile từ 1GB → < 300MB:

**Requirements:**
- Use all optimization techniques
- Maintain functionality
- Production-ready

### Câu hỏi

1. **Analyze current Dockerfile**
2. **Apply optimizations**
3. **Measure results**

### Deliverables

- Analysis
- Optimized Dockerfile
- Results

---

## 🎯 CHECKLIST

- [ ] Đã optimize image size
- [ ] Đã measure improvements
- [ ] Đã document techniques

---

**Lưu ý:** Tất cả số liệu trong exercises là illustrative/approximate cho mục đích giáo dục.

