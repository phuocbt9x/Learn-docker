# Day-004: Bài Tập Thực Hành

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi làm các bài tập này, bạn sẽ:

- Hiểu được Docker Architecture và các components
- Hiểu được Container Runtime và vai trò của nó
- So sánh được các container runtimes khác nhau
- Có thể troubleshoot Docker Daemon issues
- Quyết định được khi nào dùng runtime nào

---

## 📝 BÀI TẬP 1: HIỂU DOCKER ARCHITECTURE

### Scenario

Bạn là DevOps Engineer và cần giải thích cho team về Docker Architecture.

### Câu hỏi

**1.1.** Vẽ architecture diagram cho Docker, bao gồm:
- Docker Client
- Docker Daemon
- containerd
- runc
- Linux Kernel

**1.2.** Giải thích flow khi user chạy `docker run nginx`:
- Từng bước từ client đến kernel
- Component nào làm gì ở mỗi bước

**1.3.** Tại sao Docker Daemon là single point of failure?
- Nếu Docker Daemon crash, điều gì sẽ xảy ra?
- Containers có còn chạy không?
- Làm thế nào để mitigate risk này?

**1.4.** So sánh Docker Client và Docker Daemon:
- Chức năng của mỗi component
- Chúng communicate như thế nào?
- Có thể chạy Client và Daemon trên máy khác nhau không?

---

## 📝 BÀI TẬP 2: CONTAINER RUNTIME

### Scenario

Bạn đang thiết kế infrastructure và cần quyết định dùng container runtime nào.

### Câu hỏi

**2.1.** Giải thích sự khác biệt giữa:
- High-level runtime (containerd)
- Low-level runtime (runc)

**2.2.** Tại sao Docker dùng cả containerd và runc?
- Tại sao không chỉ dùng runc?
- Tại sao không chỉ dùng containerd?

**2.3.** OCI (Open Container Initiative) là gì?
- Tại sao quan trọng?
- Runtimes nào tuân thủ OCI?

**2.4.** Nếu bạn muốn tạo container runtime riêng, bạn cần implement những gì?
- OCI spec requirements
- Kernel features cần dùng

---

## 📝 BÀI TẬP 3: SO SÁNH CÁC RUNTIMES

### Scenario

Bạn cần quyết định dùng container runtime nào cho các use cases sau:

**Use Case 1: Development Environment**
- Developers cần chạy containers local
- Cần Docker-compatible commands
- Không cần root access

**Use Case 2: Kubernetes Production Cluster**
- 100+ nodes
- Cần performance tốt
- Cần security

**Use Case 3: CI/CD Pipeline**
- Chạy builds, tests trong containers
- Cần lightweight
- Cần fast startup

**Use Case 4: Embedded System**
- Limited resources
- Không cần daemon
- Rootless

### Câu hỏi

Với mỗi use case, hãy:

**3.1.** Recommend container runtime nào? Giải thích.

**3.2.** Liệt kê 3 lý do tại sao chọn runtime đó.

**3.3.** Liệt kê 2 limitations của runtime đó trong use case này.

**3.4.** So sánh với alternative runtime và giải thích trade-offs.

---

## 📝 BÀI TẬP 4: DOCKER DAEMON TROUBLESHOOTING

### Scenario

Bạn đang vận hành production và gặp vấn đề:

```bash
$ docker ps
Cannot connect to the Docker daemon. Is the docker daemon running?
```

**Server:**
- 50 containers đang chạy
- Docker Daemon không respond
- Containers vẫn chạy (check bằng `ps aux`)

### Câu hỏi

**4.1.** Phân tích vấn đề:
- Tại sao Docker Client không connect được?
- Containers có còn chạy không? Tại sao?
- Có thể control containers không?

**4.2.** Làm thế nào để debug?
- Check Docker Daemon status
- Check logs
- Check resources (CPU, memory)

**4.3.** Làm thế nào để fix?
- Restart Docker Daemon
- Verify containers sau khi restart
- Prevent trong tương lai

**4.4.** Nếu Docker Daemon crash và containers down:
- Làm thế nào để restore?
- Có thể recover containers không?
- Best practices để prevent?

---

## 📝 BÀI TẬP 5: MIGRATION SCENARIO

### Scenario

Công ty bạn đang dùng Docker runtime trong Kubernetes và cần migrate sang containerd (vì Kubernetes deprecate Docker).

**Current setup:**
- 50 Kubernetes nodes
- Docker runtime
- 500+ pods
- Zero downtime requirement

### Câu hỏi

**5.1.** Thiết kế migration plan:
- Chia thành phases
- Mỗi phase nên làm gì?
- Timeline estimate

**5.2.** Risk analysis:
- Liệt kê 5 risks khi migrate
- Cách mitigate mỗi risk

**5.3.** Testing strategy:
- Làm thế nào để test trước khi migrate?
- Test cases nào cần cover?

**5.4.** Rollback plan:
- Nếu migration fail, làm thế nào rollback?
- Làm thế nào để đảm bảo zero downtime?

**5.5.** So sánh containerd vs CRI-O:
- Nên chọn runtime nào?
- Trade-offs của mỗi option?

---

## 📝 BÀI TẬP 6: DOCKER REGISTRY

### Scenario

Bạn cần setup Docker Registry cho công ty.

**Requirements:**
- Private registry (không public)
- Authentication, authorization
- 100+ developers
- 1000+ images
- High availability

### Câu hỏi

**6.1.** Thiết kế registry architecture:
- Self-hosted hay cloud?
- Single registry hay multiple?
- Backup strategy?

**6.2.** Security:
- Làm thế nào để secure registry?
- Authentication mechanism?
- Authorization (who can push/pull)?

**6.3.** Performance:
- Làm thế nào để optimize pull/push speed?
- Caching strategy?
- CDN cho images?

**6.4.** So sánh các options:
- Docker Hub (private)
- Harbor
- GitLab Registry
- AWS ECR
- Trade-offs của mỗi option?

---

## 📝 BÀI TẬP 7: CONTAINERD STANDALONE

### Scenario

Bạn muốn dùng containerd standalone (không dùng Docker) cho CI/CD pipeline.

**Requirements:**
- Lightweight
- Fast startup
- OCI-compliant
- CLI tools

### Câu hỏi

**7.1.** Làm thế nào để setup containerd standalone?
- Installation
- Configuration
- CLI tools (ctr, crictl)

**7.2.** So sánh với Docker:
- Commands khác nhau như thế nào?
- Features nào có/không có?
- Performance so sánh?

**7.3.** Use cases:
- Khi nào nên dùng containerd standalone?
- Khi nào nên dùng Docker?
- Trade-offs?

**7.4.** Migration từ Docker sang containerd:
- Làm thế nào migrate?
- Scripts, tools cần thiết?
- Challenges?

---

## 📝 BÀI TẬP 8: ROOTLESS CONTAINERS

### Scenario

Bạn muốn chạy containers không cần root (rootless) cho security.

**Requirements:**
- Không cần root để chạy containers
- Security tốt hơn
- Development environment

### Câu hỏi

**8.1.** Rootless containers là gì?
- Tại sao quan trọng?
- Security benefits?

**8.2.** Runtimes nào support rootless?
- Docker (rootless mode)
- Podman
- containerd (rootless)
- So sánh?

**8.3.** Setup rootless containers:
- Làm thế nào setup?
- Requirements?
- Limitations?

**8.4.** Production use cases:
- Có nên dùng rootless trong production không?
- Trade-offs?
- Best practices?

---

## ✅ CHECKLIST HOÀN THÀNH

Trước khi xem solutions, đảm bảo bạn đã:

- [ ] Đọc kỹ `theory.md`
- [ ] Hiểu được Docker Architecture
- [ ] Hiểu được Container Runtime và vai trò
- [ ] Hiểu được các runtimes khác nhau
- [ ] Làm tất cả các bài tập trên
- [ ] Viết câu trả lời chi tiết (không chỉ đáp án ngắn gọn)

---

## 💡 GỢI Ý

- **Think like architect**: Hiểu architecture giúp troubleshoot và optimize
- **Compare runtimes**: Mỗi runtime có use case riêng, không có "best" runtime
- **Production mindset**: Nghĩ về production scenarios, not just development
- **Security**: Luôn consider security implications

---

**Chúc bạn làm bài tốt! Sau khi hoàn thành, hãy xem `solutions.md` để so sánh với đáp án.**

