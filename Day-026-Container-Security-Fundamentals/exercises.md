# Day-026: Container Security Fundamentals - Exercises

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi hoàn thành các bài tập này, bạn sẽ:
- Phân tích security risks
- Áp dụng security best practices
- Secure Dockerfiles
- Scan và fix vulnerabilities

---

## 📝 BÀI TẬP 1: Security Analysis

### Yêu cầu

Phân tích Dockerfile cho security issues:

**Dockerfile:**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y curl wget vim git
COPY . .
CMD ["app"]
```

### Câu hỏi

1. **Identify security issues**
2. **Recommend fixes**
3. **Refactor Dockerfile**

### Deliverables

- Security analysis
- Recommendations
- Refactored Dockerfile

---

## 📝 BÀI TẬP 2: Minimal Base Image

### Yêu cầu

Refactor Dockerfile với minimal base image:

**Current:**
```dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y python3
```

### Câu hỏi

1. **Choose minimal base image**
2. **Refactor Dockerfile**
3. **Compare**: So sánh image size và security

### Deliverables

- Refactored Dockerfile
- Comparison
- Analysis

---

## 📝 BÀI TẬP 3: Security Scanning

### Yêu cầu

Scan image cho vulnerabilities:

**Task:**
1. Build image
2. Scan với Docker Scout hoặc Trivy
3. Fix vulnerabilities

### Câu hỏi

1. **Scan image**
2. **Identify vulnerabilities**
3. **Fix**: Update Dockerfile

### Deliverables

- Scan results
- Vulnerability list
- Fixed Dockerfile

---

## 📝 BÀI TẬP 4: Production Security

### Yêu cầu

Create production-ready secure Dockerfile:

**Requirements:**
- Minimal base image
- Non-root user
- No unnecessary packages
- Security best practices

### Câu hỏi

1. **Create secure Dockerfile**
2. **Justify**: Giải thích security measures
3. **Test**: Verify security

### Deliverables

- Secure Dockerfile
- Security justification
- Test results

---

## 🎯 CHECKLIST

- [ ] Đã phân tích security risks
- [ ] Đã áp dụng best practices
- [ ] Đã scan images
- [ ] Đã fix vulnerabilities

---

**Lưu ý:** Tất cả số liệu trong exercises là illustrative/approximate cho mục đích giáo dục.

