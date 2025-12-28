# Day-057: Docker Interview Questions - Advanced - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Image Optimization

**Optimized Dockerfile:**
```dockerfile
# Multi-stage build
FROM python:3.11-slim AS builder

WORKDIR /app

# Install dependencies
COPY requirements.txt .
RUN pip install --user --no-cache-dir -r requirements.txt

# Production stage
FROM python:3.11-slim

# Non-root user
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser

WORKDIR /app

# Copy dependencies from builder
COPY --from=builder /root/.local /home/appuser/.local
COPY --chown=appuser:appuser . .

# Add local bin to PATH
ENV PATH=/home/appuser/.local/bin:$PATH

USER appuser

CMD ["python3", "app.py"]
```

**Optimizations:**
- Multi-stage: Remove build tools
- Specific tag: `python:3.11-slim`
- Combined RUN: Reduce layers
- Non-root user: Security
- Layer caching: Copy requirements.txt first

**Results:**
- Size: 800MB → 120MB
- Security: Non-root, minimal image
- Build time: Faster với caching

---

## ✅ BÀI TẬP 2: Troubleshooting Scenario

**Debug process:**

1. **Check exit code:**
```bash
$ docker inspect <container> --format='{{.State.ExitCode}}'
# 137 = OOM kill
```

2. **Check memory:**
```bash
$ docker stats <container>
# High memory usage
```

3. **Check system logs:**
```bash
$ dmesg | grep -i oom
# OOM kill messages
```

**Root cause:** OOM (Out of Memory) kill

**Fix:**
1. Set memory limits
2. Fix memory leaks
3. Optimize application

---

## ✅ BÀI TẬP 3: Scaling Design

**Design:**
- Horizontal scaling: Multiple instances
- Load balancing: Distribute traffic
- Auto-scaling: Based on metrics
- Health checks: Remove unhealthy instances

**Implementation:**
```yaml
services:
  api:
    deploy:
      replicas: 5
      update_config:
        parallelism: 2
        delay: 10s
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 3s
      retries: 3
```

**Results:**
- Handle 10x traffic
- Zero-downtime updates
- Automatic scaling

---

## ✅ BÀI TẬP 4: High Availability

**Design:**
- Multiple instances: 3+ replicas
- Health checks: Monitor health
- Restart policies: Auto-restart
- Data persistence: Volumes
- Load balancing: Distribute traffic

**Implementation:**
```yaml
services:
  app:
    deploy:
      replicas: 3
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
      interval: 30s
      timeout: 3s
      retries: 3
    restart: always
    volumes:
      - app-data:/app/data

volumes:
  app-data:
```

**Results:**
- 99.9% uptime
- Automatic recovery
- Data persistence

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

