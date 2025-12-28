# Day-060: Mock Interview & Review - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Mock Interview

**Question 1: Docker Architecture**

**Answer:**
Docker architecture gồm Client, Daemon, containerd, runc. Client sends commands to Daemon, Daemon manages containers via containerd, containerd uses runc to create containers.

**Question 2: Optimize Images**

**Answer:**
- Multi-stage builds
- Minimal base images
- Layer caching
- Combine RUN commands
- .dockerignore

**Question 3: Scalable System**

**Answer:**
- Microservices architecture
- Horizontal scaling
- Load balancing
- Health checks
- Auto-scaling

**Question 4: Debug Crash**

**Answer:**
- Check logs
- Check exit code
- Check state
- Reproduce locally
- Fix và test

---

## ✅ BÀI TẬP 2: Knowledge Review

**Key Topics Summary:**

**Fundamentals:**
- Container: Isolated environment
- Image: Read-only template
- Dockerfile: Build instructions

**Production:**
- Security: Non-root, scanning
- Monitoring: Health checks
- Scaling: Horizontal scaling

**Advanced:**
- BuildKit: Advanced features
- Multi-arch: Multiple architectures
- Internals: Deep dive

---

## ✅ BÀI TẬP 3: Final Project

**Project Structure:**
```
project/
├── docker-compose.yml
├── services/
│   ├── api/
│   │   ├── Dockerfile
│   │   └── app/
│   └── web/
│       ├── Dockerfile
│       └── app/
└── README.md
```

**Features:**
- Multi-service architecture
- Health checks
- Resource limits
- Security best practices
- Documentation

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

