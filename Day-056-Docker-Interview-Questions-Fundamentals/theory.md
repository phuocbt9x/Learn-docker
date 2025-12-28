# Day-056: Docker Interview Questions - Fundamentals

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Trả lời được các câu hỏi interview fundamentals
- Hiểu được cách trình bày technical concepts
- Biết cách explain Docker concepts clearly
- Áp dụng senior-level thinking trong interviews

---

## 💼 PHẦN 1: FUNDAMENTAL QUESTIONS

### 1.1. Container là gì?

**Câu hỏi:** "Container là gì? Giải thích như bạn đang nói với junior engineer."

**Senior Answer:**

Container là **isolated environment** chạy application, sử dụng **Linux kernel features** (namespace, cgroup) để tạo isolation.

**Key points:**
- **Isolation**: Process, network, filesystem isolation
- **Lightweight**: Không cần full OS như VM
- **Portable**: Chạy consistent trên mọi environment
- **Resource limits**: CPU, memory limits via cgroup

**Production context:**
- **Use case**: Microservices, CI/CD, development environments
- **Benefits**: Consistency, scalability, resource efficiency
- **Trade-offs**: Security considerations, resource limits

### 1.2. Docker vs Virtual Machine

**Câu hỏi:** "So sánh Docker và Virtual Machine."

**Senior Answer:**

**Docker (Container):**
- **Architecture**: Share host kernel
- **Isolation**: Process-level isolation
- **Resource usage**: Lightweight, fast startup
- **Use case**: Application containers, microservices

**Virtual Machine:**
- **Architecture**: Full OS, hypervisor
- **Isolation**: Hardware-level isolation
- **Resource usage**: Heavy, slower startup
- **Use case**: Full OS requirements, strong isolation

**Production decision:**
- **Choose Docker**: Application containers, microservices, CI/CD
- **Choose VM**: Strong isolation needed, different OS requirements

### 1.3. Image vs Container

**Câu hỏi:** "Image và Container khác nhau như thế nào?"

**Senior Answer:**

**Image:**
- **Definition**: Read-only template
- **Structure**: Layers, immutable
- **Storage**: Stored in registry
- **Use**: Create containers

**Container:**
- **Definition**: Running instance của image
- **Structure**: Image layers + writable layer
- **Storage**: Runtime state
- **Use**: Run applications

**Production context:**
- **Images**: Versioned, shared, immutable
- **Containers**: Ephemeral, stateful (via volumes)

---

## 💼 PHẦN 2: DOCKERFILE QUESTIONS

### 2.1. Dockerfile Best Practices

**Câu hỏi:** "Kể tên Dockerfile best practices."

**Senior Answer:**

**1. Use specific tags:**
```dockerfile
FROM node:18-alpine  # ✅ Specific
# NOT: FROM node:latest  # ❌
```

**2. Combine RUN commands:**
```dockerfile
RUN apk update && \
    apk add --no-cache git && \
    rm -rf /var/cache/apk/*  # ✅
```

**3. Use .dockerignore:**
- Reduce build context
- Faster builds
- Smaller images

**4. Non-root user:**
```dockerfile
RUN adduser -D appuser
USER appuser  # ✅
```

**5. Multi-stage builds:**
- Smaller final images
- Security benefits

**Production impact:**
- **Security**: Non-root, minimal images
- **Performance**: Faster builds, smaller images
- **Maintainability**: Clear, readable Dockerfiles

### 2.2. CMD vs ENTRYPOINT

**Câu hỏi:** "CMD và ENTRYPOINT khác nhau như thế nào?"

**Senior Answer:**

**CMD:**
- **Purpose**: Default command, can be overridden
- **Use case**: Default parameters
- **Override**: `docker run <image> <new-command>`

**ENTRYPOINT:**
- **Purpose**: Fixed command, cannot be overridden
- **Use case**: Application executable
- **Override**: Only via `--entrypoint` flag

**Production pattern:**
```dockerfile
ENTRYPOINT ["/app/entrypoint.sh"]
CMD ["--help"]
# docker run app → /app/entrypoint.sh --help
# docker run app --version → /app/entrypoint.sh --version
```

---

## 💼 PHẦN 3: NETWORKING & STORAGE

### 3.1. Docker Networks

**Câu hỏi:** "Giải thích Docker network types."

**Senior Answer:**

**Bridge (default):**
- **Isolation**: Containers isolated
- **Use case**: Single host, development
- **Limitation**: No external access by default

**Host:**
- **Isolation**: Share host network
- **Use case**: Performance-critical, single container
- **Limitation**: No port mapping

**None:**
- **Isolation**: No network
- **Use case**: Security, custom networking
- **Limitation**: No network access

**Custom bridge:**
- **Isolation**: Isolated network
- **Use case**: Production, service communication
- **Benefit**: DNS resolution, isolation

**Production choice:**
- **Custom bridge**: Production services
- **Host**: Performance-critical (rare)
- **None**: Security-sensitive (rare)

### 3.2. Volumes vs Bind Mounts

**Câu hỏi:** "Khi nào dùng volumes, khi nào dùng bind mounts?"

**Senior Answer:**

**Volumes:**
- **Storage**: Managed by Docker
- **Use case**: Production data, persistent storage
- **Benefits**: Portable, backup-friendly, performance
- **Example**: Database data, application logs

**Bind Mounts:**
- **Storage**: Host filesystem
- **Use case**: Development, config files
- **Benefits**: Direct access, easy debugging
- **Example**: Development code, config files

**Production decision:**
- **Volumes**: Production data persistence
- **Bind mounts**: Config files (careful với security)

---

## 💼 PHẦN 4: SECURITY QUESTIONS

### 4.1. Container Security

**Câu hỏi:** "Làm thế nào để secure containers?"

**Senior Answer:**

**1. Non-root user:**
```dockerfile
RUN adduser -D appuser
USER appuser
```

**2. Minimal base images:**
```dockerfile
FROM alpine:latest  # ✅ Minimal
# NOT: FROM ubuntu:latest  # ❌ Large attack surface
```

**3. Scan images:**
- Use Trivy, Snyk
- Regular scanning
- Fix vulnerabilities

**4. Resource limits:**
```yaml
deploy:
  resources:
    limits:
      memory: 512M
      cpus: '0.5'
```

**5. Secrets management:**
- Docker secrets
- External secret managers
- Never hardcode secrets

**Production checklist:**
- ✅ Non-root user
- ✅ Minimal images
- ✅ Image scanning
- ✅ Resource limits
- ✅ Secrets management

---

## 🎓 TÓM TẮT

**Interview fundamentals:**
- Clear explanations
- Production context
- Trade-offs analysis
- Best practices

**Key topics:**
- Container basics
- Dockerfile practices
- Networking & storage
- Security

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-057)** sẽ đi sâu vào:
- Advanced interview questions
- Complex scenarios

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.

