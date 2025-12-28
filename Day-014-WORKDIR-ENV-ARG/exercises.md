# Day-014: WORKDIR, ENV, ARG - Exercises

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi hoàn thành các bài tập này, bạn sẽ:
- Viết được Dockerfile với WORKDIR, ENV, ARG đúng cách
- Hiểu được sự khác biệt giữa ENV và ARG
- Biết khi nào dùng ENV, khi nào dùng ARG
- Tối ưu được Dockerfile với environment variables
- Debug được các vấn đề liên quan đến WORKDIR, ENV, ARG

---

## 📝 BÀI TẬP 1: WORKDIR Basics

### Yêu cầu

Tạo Dockerfile cho Node.js application với:
- Base image: `node:18-alpine`
- Set WORKDIR: `/app`
- Copy `package.json` và install dependencies
- Copy source code
- Run application

### Câu hỏi

1. **Tại sao cần WORKDIR?**
2. **So sánh với RUN cd**: WORKDIR vs RUN cd
3. **Test**: Verify WORKDIR được set đúng

### Deliverables

- Dockerfile
- Commands để test
- Giải thích

---

## 📝 BÀI TẬP 2: ENV Basics

### Yêu cầu

Tạo Dockerfile cho Python application với:
- Base image: `python:3.9-slim`
- Set ENV: `NODE_ENV=production`, `PORT=3000`
- Copy và run application

### Câu hỏi

1. **Tại sao dùng ENV?**
2. **Làm thế nào override ENV khi run?**
3. **Test**: Verify ENV variables trong container

### Deliverables

- Dockerfile
- Commands để test override
- Giải thích

---

## 📝 BÀI TẬP 3: ARG Basics

### Yêu cầu

Tạo Dockerfile với:
- Base image: `alpine:latest`
- Define ARG: `VERSION=latest`
- Use ARG trong RUN command

### Câu hỏi

1. **Tại sao dùng ARG?**
2. **Làm thế nào pass ARG khi build?**
3. **Test**: Verify ARG không available khi runtime

### Deliverables

- Dockerfile
- Commands để build với ARG
- Test results

---

## 📝 BÀI TẬP 4: ENV vs ARG

### Yêu cầu

Tạo 2 Dockerfiles:

**Dockerfile 1: ENV**
```dockerfile
FROM node:18-alpine
ENV NODE_ENV=production
```

**Dockerfile 2: ARG**
```dockerfile
FROM node:18-alpine
ARG NODE_ENV=production
```

### Câu hỏi

1. **So sánh behavior:**
   - Available khi runtime?
   - Available khi build?
   - Override cách nào?

2. **Test**: Build và run cả 2, so sánh

3. **Recommendation**: Khi nào dùng ENV? Khi nào dùng ARG?

### Deliverables

- 2 Dockerfiles
- Comparison table
- Recommendation

---

## 📝 BÀI TẬP 5: ENV từ ARG

### Yêu cầu

Tạo Dockerfile với:
- ARG: `VERSION=latest`
- ENV từ ARG: `APP_VERSION=$VERSION`
- Verify ENV available khi runtime

### Câu hỏi

1. **Tại sao set ENV từ ARG?**
2. **Use case**: Khi nào cần pattern này?
3. **Test**: Build với ARG, verify ENV khi runtime

### Deliverables

- Dockerfile
- Build commands
- Test results

---

## 📝 BÀI TẬP 6: Practical Dockerfile

### Yêu cầu

Tạo Dockerfile cho Node.js application với:

**Requirements:**
- Base image: `node:18-alpine`
- WORKDIR: `/app`
- ENV: `NODE_ENV=production`, `PORT=3000`
- ARG: `VERSION=latest`
- ENV từ ARG: `APP_VERSION=$VERSION`
- Copy và run application

### Câu hỏi

1. **Viết Dockerfile hoàn chỉnh**
2. **Test**: Build với ARG, run và verify ENV

### Deliverables

- Dockerfile
- Build và run commands
- Test results

---

## 📝 BÀI TẬP 7: Troubleshooting

### Scenario

**Dockerfile:**
```dockerfile
FROM node:18-alpine
ARG DB_HOST=localhost
COPY app.js .
CMD ["node", "app.js"]
```

**app.js:**
```javascript
console.log(process.env.DB_HOST);
```

**Problem:**
- Application log: `undefined`
- Root cause: ARG không available khi runtime

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
COPY package.json .
RUN npm install
COPY . .
CMD ["node", "index.js"]
```

**Requirements:**
- Set WORKDIR
- Set ENV defaults
- Support build-time version

### Câu hỏi

1. **Phân tích current Dockerfile:**
   - Ưu điểm?
   - Nhược điểm?

2. **Refactor Dockerfile:**
   - Add WORKDIR
   - Add ENV defaults
   - Add ARG for version

3. **Justification**: Giải thích changes

### Deliverables

- Analysis
- Refactored Dockerfile
- Justification

---

## 📝 BÀI TẬP 9: Advanced - Build Metadata

### Yêu cầu

Tạo Dockerfile với build metadata:

**Requirements:**
- ARG: `VERSION`, `BUILD_DATE`, `GIT_COMMIT`
- ENV từ ARG: `APP_VERSION`, `BUILD_DATE`, `GIT_COMMIT`
- Build script để pass ARG values

### Câu hỏi

1. **Viết Dockerfile**
2. **Viết build script** (bash) để pass ARG values
3. **Test**: Build và verify metadata trong container

### Deliverables

- Dockerfile
- Build script
- Test results

---

## 📝 BÀI TẬP 10: Advanced - Conditional Build

### Yêu cầu

**Scenario:** Build với different environments

**Requirements:**
- ARG: `NODE_ENV=production`
- Conditional install: `npm install --production` nếu production
- ENV: `NODE_ENV` từ ARG

### Câu hỏi

1. **Viết Dockerfile với conditional logic**
2. **Test**: Build với different NODE_ENV values
3. **Compare**: So sánh image size

### Deliverables

- Dockerfile
- Build commands
- Comparison results

---

## 🎯 CHECKLIST

Trước khi submit, đảm bảo:

- [ ] Đã viết Dockerfile với WORKDIR, ENV, ARG đúng syntax
- [ ] Đã hiểu ENV vs ARG
- [ ] Đã test override behavior
- [ ] Đã giải thích decisions

---

## 📚 HINTS

1. **WORKDIR:**
   - Set early
   - Use absolute paths
   - Persist qua layers

2. **ENV:**
   - Runtime variables
   - In image
   - Override với `-e`

3. **ARG:**
   - Build-time only
   - Not in image
   - Pass với `--build-arg`

4. **ENV từ ARG:**
   - Pattern: Build version → Runtime version
   - `ENV VAR=$ARG_VAR`

---

**Lưu ý:** Tất cả số liệu trong exercises là illustrative/approximate cho mục đích giáo dục.

