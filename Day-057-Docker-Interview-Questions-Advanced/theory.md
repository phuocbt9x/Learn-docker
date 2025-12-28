# Day-057: Docker Interview Questions - Advanced

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Trả lời được advanced interview questions
- Hiểu được complex scenarios
- Biết cách troubleshoot advanced issues
- Áp dụng senior-level problem solving

---

## 💼 PHẦN 1: ADVANCED CONCEPTS

### 1.1. Image Layers Deep Dive

**Câu hỏi:** "Giải thích image layers và caching strategy."

**Senior Answer:**

**Image layers:**
- **Structure**: Each instruction creates a layer
- **Immutable**: Layers không thể thay đổi
- **Sharing**: Layers shared giữa images
- **Storage**: Stored in `/var/lib/docker/overlay2/`

**Caching strategy:**
```dockerfile
# Layer 1: Base image
FROM node:18-alpine

# Layer 2: Dependencies (cacheable)
COPY package*.json ./
RUN npm install

# Layer 3: Application code (changes frequently)
COPY . .
```

**Production optimization:**
- **Order matters**: Stable layers first
- **Minimize changes**: Reduce layer invalidation
- **Combine commands**: Reduce layer count

**Trade-offs:**
- **More layers**: Better caching, slower builds
- **Fewer layers**: Faster builds, less caching

### 1.2. Multi-stage Builds

**Câu hỏi:** "Khi nào dùng multi-stage builds? Giải thích benefits."

**Senior Answer:**

**Use cases:**
- **Build tools**: Compilers, build dependencies
- **Size reduction**: Remove build tools từ final image
- **Security**: Reduce attack surface

**Example:**
```dockerfile
# Build stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./
CMD ["node", "dist/index.js"]
```

**Benefits:**
- **Smaller images**: No build tools
- **Security**: Minimal runtime dependencies
- **Performance**: Faster deployments

**Production impact:**
- **Size**: 500MB → 50MB
- **Security**: Reduced attack surface
- **Deployment**: Faster image pulls

### 1.3. Container Lifecycle

**Câu hỏi:** "Giải thích container lifecycle và states."

**Senior Answer:**

**States:**
1. **Created**: Container created, not started
2. **Running**: Container running
3. **Paused**: Container paused (SIGSTOP)
4. **Restarting**: Container restarting
5. **Exited**: Container stopped
6. **Dead**: Container dead (cannot start)

**Transitions:**
```
Created → Running → Exited
         ↓
      Paused
         ↓
    Restarting
```

**Production considerations:**
- **Restart policies**: `always`, `on-failure`, `unless-stopped`
- **Health checks**: Detect unhealthy containers
- **Graceful shutdown**: Handle SIGTERM properly

---

## 💼 PHẦN 2: TROUBLESHOOTING QUESTIONS

### 2.1. Container Won't Start

**Câu hỏi:** "Container không start. Debug như thế nào?"

**Senior Answer:**

**Debug steps:**

1. **Check logs:**
```bash
$ docker logs <container>
```

2. **Check exit code:**
```bash
$ docker inspect <container> --format='{{.State.ExitCode}}'
```

3. **Run interactively:**
```bash
$ docker run -it --entrypoint sh <image>
```

4. **Check configuration:**
```bash
$ docker inspect <container>
```

5. **Test manually:**
```bash
$ docker exec <container> <command>
```

**Common issues:**
- **Missing dependencies**: Check Dockerfile
- **Wrong entrypoint**: Check CMD/ENTRYPOINT
- **Permission issues**: Check file permissions
- **Port conflicts**: Check port mappings

**Production debugging:**
- **Systematic approach**: Step-by-step debugging
- **Logs first**: Always check logs
- **Reproduce**: Reproduce issue locally

### 2.2. OOM Issues

**Câu hỏi:** "Container bị OOM kill. Làm thế nào?"

**Senior Answer:**

**Symptoms:**
- Exit code: 137 (128 + 9, SIGKILL)
- No logs before kill
- Memory usage high

**Debug:**
```bash
$ docker inspect <container> --format='{{.State.ExitCode}}'
# 137 = OOM kill

$ dmesg | grep -i oom
# System OOM messages

$ docker stats <container>
# Monitor memory usage
```

**Fix:**
1. **Set memory limits:**
```yaml
deploy:
  resources:
    limits:
      memory: 512M
```

2. **Fix memory leaks:**
- Profile application
- Fix leaks trong code
- Monitor memory usage

3. **Optimize application:**
- Reduce memory usage
- Use streaming
- Cache optimization

**Production prevention:**
- **Always set limits**: Prevent OOM
- **Monitor memory**: Track usage
- **Fix leaks**: Address root cause

---

## 💼 PHẦN 3: ARCHITECTURE QUESTIONS

### 3.1. Scaling Strategy

**Câu hỏi:** "Làm thế nào scale Docker containers?"

**Senior Answer:**

**Horizontal scaling:**
- **Multiple instances**: Run multiple containers
- **Load balancing**: Distribute traffic
- **Stateless**: Containers should be stateless

**Implementation:**
```yaml
services:
  api:
    deploy:
      replicas: 3
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost/health"]
```

**Considerations:**
- **Stateless**: No local state
- **Shared storage**: Use volumes cho state
- **Session management**: External session store
- **Database connections**: Connection pooling

**Production patterns:**
- **Auto-scaling**: Based on metrics
- **Health checks**: Remove unhealthy instances
- **Rolling updates**: Zero-downtime deployments

### 3.2. High Availability

**Câu hỏi:** "Design high availability với Docker."

**Senior Answer:**

**Components:**
1. **Multiple instances**: Run multiple containers
2. **Health checks**: Monitor health
3. **Load balancing**: Distribute traffic
4. **Restart policies**: Auto-restart
5. **Data persistence**: Volumes cho data

**Architecture:**
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
```

**Production requirements:**
- **Redundancy**: Multiple instances
- **Monitoring**: Health checks
- **Recovery**: Automatic recovery
- **Data backup**: Regular backups

---

## 🎓 TÓM TẮT

**Advanced topics:**
- Image layers và caching
- Multi-stage builds
- Container lifecycle
- Troubleshooting
- Scaling và HA

**Senior-level thinking:**
- Deep understanding
- Production context
- Trade-offs analysis
- Problem solving

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-058)** sẽ đi sâu vào:
- System Design với Docker
- Architecture design

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-058: System-Design-voi-Docker](../Day-058-System-Design-voi-Docker/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
