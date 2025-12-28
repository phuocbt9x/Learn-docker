# 🐳 DOCKER & CONTAINER TRAINING ROADMAP
## Từ Zero → Senior Docker/DevOps Engineer

---

## 📋 TỔNG QUAN

Chương trình đào tạo này được thiết kế để biến một người chưa biết gì về container thành một **Senior Docker/DevOps Engineer** có thể:

- Hiểu sâu container internals (namespace, cgroup, overlayfs)
- Viết Dockerfile production-grade
- Debug container issues trong production
- Tối ưu image size & build time
- Thiết kế multi-stage builds
- Quản lý networking, volumes, permissions
- Pass Mid → Senior DevOps interviews

---

## 🎯 NGUYÊN TẮC THIẾT KẾ

### 4-Question Framework
Mỗi concept phải trả lời:
1. **Nó là gì?** - Định nghĩa rõ ràng
2. **Tại sao tồn tại?** - Vấn đề nó giải quyết
3. **Khi nào dùng trong production?** - Use cases thực tế
4. **Hậu quả nếu dùng sai?** - Security, performance, reliability risks

### Senior Thinking
- Luôn so sánh trade-offs
- Giải thích security impact
- Production consequences
- Alternative approaches

---

## 🗺️ CẤU TRÚC CHƯƠNG TRÌNH

### **PHASE 1: Container & Linux Foundations**
**Mục tiêu**: Hiểu container hoạt động ở kernel level

- Day-001: Vấn đề của deployment truyền thống & Container là gì?
- Day-002: Virtual Machine vs Container - So sánh sâu
- Day-003: Linux Kernel Basics cho Container (namespace, cgroup)
- Day-004: Container Runtime & Docker Architecture
- Day-005: Image vs Container - Layers & Filesystem

**Kết quả**: Hiểu tại sao container tồn tại, khác VM thế nào, hoạt động ở kernel level ra sao.

---

### **PHASE 2: Core Docker Usage**
**Mục tiêu**: Thành thạo Docker CLI và các khái niệm cơ bản

- Day-006: Docker Installation & First Container
- Day-007: Docker Images - Pull, Tag, Inspect
- Day-008: Container Lifecycle - Create, Start, Stop, Remove
- Day-009: Container Logs & Debugging
- Day-010: Docker Hub & Registry Basics

**Kết quả**: Có thể chạy, quản lý container và image cơ bản.

---

### **PHASE 3: Dockerfile Fundamentals**
**Mục tiêu**: Viết Dockerfile đúng, gọn, secure

- Day-011: Dockerfile Syntax - FROM, RUN, COPY
- Day-012: CMD vs ENTRYPOINT - Khi nào dùng gì?
- Day-013: COPY vs ADD - Trade-offs & Best Practices
- Day-014: WORKDIR, ENV, ARG - Environment Management
- Day-015: Multi-stage Builds - Tối ưu Image Size

**Kết quả**: Viết được Dockerfile production-ready, hiểu từng instruction.

---

### **PHASE 4: Image Optimization**
**Mục tiêu**: Tối ưu build time và image size

- Day-016: Layer Caching - Build Performance
- Day-017: .dockerignore & Build Context
- Day-018: Image Size Optimization Strategies
- Day-019: BuildKit & Advanced Build Features
- Day-020: Image Security Scanning

**Kết quả**: Build nhanh, image nhỏ, secure.

---

### **PHASE 5: Networking & Storage**
**Mục tiêu**: Quản lý network và data persistence

- Day-021: Docker Networking - Bridge, Host, None
- Day-022: Custom Networks & Container Communication
- Day-023: Docker Volumes - Named vs Anonymous
- Day-024: Bind Mounts vs Volumes - Production Use Cases
- Day-025: Volume Backup & Restore Strategies

**Kết quả**: Thiết kế network và storage cho production.

---

### **PHASE 6: Security & Hardening**
**Mục tiêu**: Secure containers trong production

- Day-026: Container Security Fundamentals
- Day-027: Non-root User & Capabilities
- Day-028: Secrets Management trong Docker
- Day-029: Image Vulnerabilities & Patching
- Day-030: Container Isolation & Resource Limits

**Kết quả**: Hiểu và áp dụng security best practices.

---

### **PHASE 7: Production Operations**
**Mục tiêu**: Vận hành Docker trong production

- Day-031: Container Health Checks
- Day-032: Resource Limits (CPU, Memory)
- Day-033: Container Restart Policies
- Day-034: Logging Strategies cho Production
- Day-035: Monitoring & Observability

**Kết quả**: Vận hành container ổn định trong production.

---

### **PHASE 8: Docker Compose**
**Mục tiêu**: Quản lý multi-container applications

- Day-036: Docker Compose Basics
- Day-037: Compose Networks & Volumes
- Day-038: Compose Environment Variables
- Day-039: Compose Scaling & Dependencies
- Day-040: Compose Production Patterns

**Kết quả**: Thiết kế và quản lý multi-container apps.

---

### **PHASE 9: CI/CD Integration**
**Mục tiêu**: Tích hợp Docker vào CI/CD pipelines

- Day-041: Docker trong CI/CD - Overview
- Day-042: Build & Push Images trong CI
- Day-043: Docker Layer Caching trong CI
- Day-044: Multi-arch Builds
- Day-045: CI/CD Best Practices

**Kết quả**: Tích hợp Docker vào CI/CD hiệu quả.

---

### **PHASE 10: Troubleshooting & Debugging**
**Mục tiêu**: Debug production issues như senior engineer

- Day-046: Container Crash Debugging
- Day-047: OOM (Out of Memory) Issues
- Day-048: Network Connectivity Problems
- Day-049: Permission & Filesystem Issues
- Day-050: Performance Bottleneck Analysis

**Kết quả**: Debug nhanh các vấn đề production.

---

### **PHASE 11: Advanced Topics**
**Mục tiêu**: Kiến thức nâng cao cho senior level

- Day-051: Docker BuildKit Advanced Features
- Day-052: Image Manifest & Multi-arch
- Day-053: Container Runtime Interface (CRI)
- Day-054: Docker Internals - Deep Dive
- Day-055: Production Architecture Patterns

**Kết quả**: Hiểu sâu Docker internals và advanced patterns.

---

### **PHASE 12: Interview Preparation**
**Mục tiêu**: Pass Mid → Senior DevOps interviews

- Day-056: Docker Interview Questions - Fundamentals
- Day-057: Docker Interview Questions - Advanced
- Day-058: System Design với Docker
- Day-059: Case Studies & Real-world Scenarios
- Day-060: Mock Interview & Review

**Kết quả**: Sẵn sàng cho DevOps interviews.

---

## 📁 CẤU TRÚC FILE

Mỗi ngày có cấu trúc:

```
Day-XXX-[Topic]/
├── theory.md      # Lý thuyết + Production stories
├── exercises.md   # Bài tập thực hành
└── solutions.md   # Giải pháp + Giải thích
```

---

## 🎓 TIÊU CHUẨN CHẤT LƯỢNG

Mỗi day phải có:

✅ **Theory**:
- Giải thích bằng tiếng Việt
- 4-question framework
- Production stories (context, problem, investigation, fix, lesson)
- Trade-offs & alternatives
- Security considerations

✅ **Exercises**:
- Viết Dockerfile
- Debug issues
- Optimize performance
- Security analysis
- Refactor code

✅ **Solutions**:
- Giải thích tại sao
- So sánh với alternatives
- Common mistakes
- Production notes

---

## 🚀 CÁCH SỬ DỤNG

1. **Bắt đầu**: Đọc roadmap này
2. **Học từng ngày**: Bắt đầu từ Day-001
3. **Làm bài tập**: Không xem solution trước
4. **Review solution**: So sánh với cách làm của bạn
5. **Tiếp tục**: Nói "NEXT" hoặc "DAY-XXX" để tiếp tục

---

## ⚠️ LƯU Ý

- **KHÔNG** cloud provider (AWS/GCP/Azure)
- **KHÔNG** Kubernetes (cho đến khi được phép)
- **KHÔNG** phụ thuộc ngôn ngữ lập trình cụ thể
- **TẬP TRUNG** vào Docker & container core

---

## 📊 TIẾN ĐỘ HỌC TẬP

- **Beginner** (Day 001-010): Hiểu container basics
- **Intermediate** (Day 011-030): Viết Dockerfile, optimize
- **Advanced** (Day 031-050): Production operations
- **Senior** (Day 051-060): Advanced topics & interviews

---

**Sẵn sàng bắt đầu? Gõ `DAY-001` để bắt đầu ngày đầu tiên!**

