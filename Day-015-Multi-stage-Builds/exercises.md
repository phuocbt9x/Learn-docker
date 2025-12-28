# Day-015: Multi-stage Builds - Exercises

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi hoàn thành các bài tập này, bạn sẽ:
- Viết được multi-stage Dockerfile
- Tối ưu được image size
- Áp dụng được các patterns phổ biến
- Phân tích được trade-offs
- Debug được các vấn đề liên quan đến multi-stage builds

---

## 📝 BÀI TẬP 1: Basic Multi-stage

### Yêu cầu

Convert single-stage Dockerfile thành multi-stage:

**Single-stage:**
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

1. **Tạo multi-stage Dockerfile:**
   - Stage 1: Build
   - Stage 2: Production

2. **So sánh image size:**
   - Single-stage vs multi-stage
   - Giải thích sự khác biệt

3. **Test**: Build và verify

### Deliverables

- Multi-stage Dockerfile
- Image size comparison
- Explanation

---

## 📝 BÀI TẬP 2: Go Application

### Yêu cầu

Tạo multi-stage Dockerfile cho Go application:

**Requirements:**
- Stage 1: Build với golang image
- Stage 2: Runtime với alpine
- Compile Go binary
- Copy binary vào minimal image

### Câu hỏi

1. **Viết multi-stage Dockerfile**
2. **Test**: Build và verify image size
3. **Compare**: So sánh với single-stage

### Deliverables

- Multi-stage Dockerfile
- Test results
- Comparison

---

## 📝 BÀI TẬP 3: Python Application

### Yêu cầu

Tạo multi-stage Dockerfile cho Python application cần build tools:

**Requirements:**
- Stage 1: Build với build tools (gcc, g++)
- Stage 2: Runtime không có build tools
- Install Python dependencies

### Câu hỏi

1. **Viết multi-stage Dockerfile**
2. **Test**: Verify build tools không có trong final image
3. **Security**: Phân tích security improvements

### Deliverables

- Multi-stage Dockerfile
- Test results
- Security analysis

---

## 📝 BÀI TẬP 4: Test + Build Pattern

### Yêu cầu

Tạo multi-stage Dockerfile với test stage:

**Requirements:**
- Stage 1: Test
- Stage 2: Build
- Stage 3: Production
- Run tests trước khi build

### Câu hỏi

1. **Viết multi-stage Dockerfile**
2. **Test**: Verify tests chạy
3. **Failure handling**: Test với failing tests

### Deliverables

- Multi-stage Dockerfile
- Test results
- Failure handling explanation

---

## 📝 BÀI TẬP 5: Multi-service Pattern

### Yêu cầu

Tạo multi-stage Dockerfile cho multiple services:

**Requirements:**
- Stage 1: Build API
- Stage 2: Build Frontend
- Stage 3: Combine vào nginx

### Câu hỏi

1. **Viết multi-stage Dockerfile**
2. **Test**: Build và verify
3. **Optimization**: Tối ưu nếu có thể

### Deliverables

- Multi-stage Dockerfile
- Test results
- Optimization notes

---

## 📝 BÀI TẬP 6: Image Size Optimization

### Yêu cầu

Tối ưu multi-stage Dockerfile để minimize image size:

**Current:**
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app .
CMD ["node", "dist/index.js"]
```

### Câu hỏi

1. **Phân tích current Dockerfile:**
   - Vấn đề gì?
   - Có thể optimize gì?

2. **Optimize Dockerfile:**
   - Minimize final stage
   - Copy only needed files
   - Production dependencies only

3. **Test**: Compare image size

### Deliverables

- Optimized Dockerfile
- Analysis
- Image size comparison

---

## 📝 BÀI TẬP 7: Troubleshooting

### Scenario

**Dockerfile:**
```dockerfile
FROM node:18-alpine AS builder
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build

FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist .
CMD ["node", "index.js"]
```

**Problem:**
- Container không start
- Error: Cannot find module

### Câu hỏi

1. **Root cause là gì?**
2. **Fix Dockerfile**
3. **Test**: Verify fix

### Deliverables

- Fixed Dockerfile
- Explanation
- Test results

---

## 📝 BÀI TẬP 8: Production Analysis

### Scenario

**Company:** SaaS platform
**Application:** Node.js microservice
**Current Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json .
RUN npm install
COPY . .
RUN npm run build
CMD ["node", "dist/index.js"]
```

**Requirements:**
- Image size < 200MB
- No build tools in production
- Production dependencies only

### Câu hỏi

1. **Phân tích current Dockerfile:**
   - Ưu điểm?
   - Nhược điểm?
   - Image size?

2. **Refactor thành multi-stage:**
   - Optimize image size
   - Remove build tools
   - Production dependencies only

3. **Justification**: Giải thích changes

### Deliverables

- Analysis
- Refactored Dockerfile
- Justification

---

## 📝 BÀI TẬP 9: Advanced - Conditional Stages

### Yêu cầu

Tạo multi-stage Dockerfile với conditional logic:

**Requirements:**
- Build stage với different targets
- Conditional copy based on target
- Support dev và prod builds

### Câu hỏi

1. **Viết multi-stage Dockerfile**
2. **Test**: Build với different targets
3. **Compare**: So sánh image sizes

### Deliverables

- Multi-stage Dockerfile
- Build commands
- Comparison results

---

## 📝 BÀI TẬP 10: Advanced - Build Cache Optimization

### Yêu cầu

Tối ưu multi-stage Dockerfile để maximize build cache:

**Requirements:**
- Optimize layer caching
- Minimize rebuilds
- Fast builds

### Câu hỏi

1. **Phân tích current Dockerfile:**
   - Cache behavior?
   - Rebuild triggers?

2. **Optimize Dockerfile:**
   - Optimize COPY order
   - Minimize layer changes

3. **Test**: Verify cache behavior

### Deliverables

- Optimized Dockerfile
- Cache analysis
- Test results

---

## 🎯 CHECKLIST

Trước khi submit, đảm bảo:

- [ ] Đã viết multi-stage Dockerfile
- [ ] Đã tối ưu image size
- [ ] Đã test build
- [ ] Đã so sánh với single-stage
- [ ] Đã giải thích decisions

---

## 📚 HINTS

1. **Multi-stage syntax:**
   - `FROM image AS stage-name`
   - `COPY --from=stage-name /source /dest`

2. **Optimization:**
   - Minimize final stage
   - Copy only artifacts
   - Production dependencies only

3. **Patterns:**
   - Build + Runtime
   - Compiler pattern
   - Test + Build

---

**Lưu ý:** Tất cả số liệu trong exercises là illustrative/approximate cho mục đích giáo dục.

