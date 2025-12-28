# Day-056: Docker Interview Questions - Fundamentals - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Explain Container

**Explanation:**

Container là isolated environment chạy application, sử dụng Linux kernel features (namespace, cgroup) để tạo isolation.

**Key points:**
- Isolation: Process, network, filesystem
- Lightweight: Không cần full OS
- Portable: Consistent across environments
- Resource limits: CPU, memory via cgroup

**Production context:**
- Use cases: Microservices, CI/CD
- Benefits: Consistency, scalability
- Trade-offs: Security, resource limits

---

## ✅ BÀI TẬP 2: Dockerfile Review

**Issues:**
1. `ubuntu:latest` → Use specific tag
2. Multiple RUN → Combine
3. No .dockerignore
4. Root user
5. No multi-stage

**Improved:**
```dockerfile
FROM node:18-alpine

RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production && \
    npm cache clean --force

COPY --chown=appuser:appuser . .

USER appuser

CMD ["node", "app.js"]
```

**Changes:**
- Specific tag: `node:18-alpine`
- Combined RUN commands
- Non-root user
- Copy package.json first (layer caching)
- Production-only dependencies

---

## ✅ BÀI TẬP 3: Network Design

**Design:**
- `app-network`: Application services
- `db-network`: Database (isolated)
- `frontend-network`: Frontend services

**Implementation:**
```yaml
networks:
  app-network:
    driver: bridge
  db-network:
    driver: bridge
    internal: true  # No external access
  frontend-network:
    driver: bridge
```

**Justification:**
- Isolation: Database isolated
- Security: Internal network cho database
- Communication: Services communicate via app-network

---

## ✅ BÀI TẬP 4: Security Checklist

**Checklist:**
- [ ] Non-root user
- [ ] Minimal base images
- [ ] Image scanning
- [ ] Resource limits
- [ ] Secrets management
- [ ] No hardcoded secrets
- [ ] Health checks
- [ ] Regular updates

**Explanations:**
- Non-root: Reduce attack surface
- Minimal images: Smaller attack surface
- Scanning: Identify vulnerabilities
- Limits: Prevent resource exhaustion
- Secrets: Secure credential handling

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

