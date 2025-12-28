# Day-031: Container Health Checks

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được health checks là gì và tại sao cần
- Biết cách define health checks trong Dockerfile
- Biết cách define health checks khi run container
- Hiểu được health check states
- Debug được health check issues
- Áp dụng được trong production

---

## 🏥 PHẦN 1: HEALTH CHECKS LÀ GÌ?

### 1.1. Health Check là gì?

**Health Check** là **cơ chế Docker check** xem container có **healthy** hay không.

**Purpose:**
- **Detect failures**: Phát hiện khi container fail
- **Automatic recovery**: Tự động recover (với orchestrators)
- **Load balancer**: Load balancer biết container healthy
- **Monitoring**: Monitor container health

### 1.2. Health Check States

**Health states:**
- **starting**: Container đang start, chưa có health check result
- **healthy**: Container healthy (health check pass)
- **unhealthy**: Container unhealthy (health check fail)

**View health status:**
```bash
$ docker ps
# Show health status trong STATUS column
```

### 1.3. Tại sao Health Checks tồn tại?

**Vấn đề:**
- **Container running ≠ Application healthy**: Container có thể running nhưng application fail
- **No automatic detection**: Không tự động detect failures
- **Manual checks**: Phải manually check

**Health checks giải quyết:**
- **Automatic detection**: Tự động detect failures
- **Application-level check**: Check application level, không chỉ container
- **Integration**: Integrate với orchestrators, load balancers

---

## 📋 PHẦN 2: DEFINE HEALTH CHECKS

### 2.1. Health Check trong Dockerfile

**Syntax:**
```dockerfile
HEALTHCHECK [OPTIONS] CMD command
```

**Options:**
- `--interval=DURATION`: Interval giữa các checks (default: 30s)
- `--timeout=DURATION`: Timeout cho mỗi check (default: 30s)
- `--start-period=DURATION`: Start period (default: 0s)
- `--retries=N`: Số lần retry trước khi mark unhealthy (default: 3)

**Ví dụ:**
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

### 2.2. Health Check khi Run Container

**Define health check:**
```bash
$ docker run \
  --health-cmd="curl -f http://localhost:8080/health || exit 1" \
  --health-interval=30s \
  --health-timeout=3s \
  --health-start-period=5s \
  --health-retries=3 \
  my-app
```

**Override Dockerfile health check:**
```bash
$ docker run --health-cmd="echo ok" my-app
# Override health check từ Dockerfile
```

### 2.3. Health Check Examples

**HTTP endpoint:**
```dockerfile
HEALTHCHECK CMD curl -f http://localhost:8080/health || exit 1
```

**TCP check:**
```dockerfile
HEALTHCHECK CMD nc -z localhost 3306 || exit 1
```

**Custom script:**
```dockerfile
HEALTHCHECK CMD /app/healthcheck.sh || exit 1
```

**Database check:**
```dockerfile
HEALTHCHECK CMD pg_isready -U postgres || exit 1
```

---

## 🔍 PHẦN 3: HEALTH CHECK BEHAVIOR

### 3.1. Health Check Flow

**Flow:**
1. **Container start**: Container start
2. **Start period**: Wait start-period (cho application start)
3. **First check**: Run health check lần đầu
4. **Interval**: Wait interval, run check lại
5. **Retries**: Nếu fail, retry (retries times)
6. **State**: Mark healthy hoặc unhealthy

**Example:**
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

**Timeline:**
- **0s**: Container start
- **10s**: Start period end, first check
- **40s**: Second check (30s interval)
- **70s**: Third check
- ...

### 3.2. Health Check States

**Starting:**
- Container vừa start
- Chưa có health check result
- Trong start-period

**Healthy:**
- Health check pass
- Container ready

**Unhealthy:**
- Health check fail
- Retries exhausted
- Container có vấn đề

### 3.3. View Health Status

**Docker ps:**
```bash
$ docker ps
# STATUS column shows: (healthy), (unhealthy), (starting)
```

**Inspect:**
```bash
$ docker inspect --format='{{.State.Health.Status}}' <container>
# Output: healthy, unhealthy, hoặc starting
```

**Detailed health:**
```bash
$ docker inspect <container> | grep -A 10 Health
```

---

## 🏭 PRODUCTION STORY #1: Container Running but Application Down

### Context

**Công ty:** E-commerce, 800 employees
**Hệ thống:** Node.js microservices với Docker
**Traffic:** 25M requests/day
**Team:** 40 backend engineers

### Problem

**Tháng 3/2024:**
- **Application down**: Application crash nhưng container vẫn running
- **No detection**: Không detect được application down
- **User impact**: Users không thể access
- **Root cause**: Không có health checks

**Timeline:**
- **2:00 PM**: Application crash
- **2:05 PM**: Container vẫn running (process không exit)
- **2:10 PM**: Users report issues
- **2:15 PM**: Team investigate
- **2:20 PM**: Found application down

**Impact:**
- **15 minutes downtime**: 15 phút không detect được
- **User complaints**: 1000+ user complaints
- **Revenue loss**: $10K

### Investigation

**Root cause:**
```dockerfile
FROM node:18-alpine
COPY . .
CMD ["node", "app.js"]
# Không có health check
```

**Vấn đề:**
- **No health check**: Không có health check
- **Container running**: Container process vẫn running
- **Application down**: Application crash nhưng process không exit
- **No detection**: Không detect được

**Test:**
```bash
$ docker ps
# Container status: Up 10 minutes
# Nhưng application không respond
```

### Fix

**Solution: Add health check**
```dockerfile
FROM node:18-alpine
COPY . .
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {process.exit(r.statusCode === 200 ? 0 : 1)})"
CMD ["node", "app.js"]
```

**Kết quả:**
- **Automatic detection**: Tự động detect application down
- **Fast detection**: Detect trong 30-90 giây
- **Integration**: Integrate với orchestrators

### Result

**Trước:**
- No health check
- **Detection time**: 15 phút (manual)
- **User impact**: High

**Sau:**
- Health check
- **Detection time**: 30-90 giây (automatic)
- **User impact**: Minimal

### Lesson Learned

1. **Always use health checks**: Best practice cho production
2. **Application-level checks**: Check application, không chỉ container
3. **Fast detection**: Detect failures nhanh
4. **Integration**: Integrate với orchestrators

---

## 🏭 PRODUCTION STORY #2: Health Check False Positives

### Context

**Công ty:** SaaS platform, 600 employees
**Hệ thống:** Python applications với Docker
**Traffic:** 15M requests/day
**Team:** 30 backend engineers

### Problem

**Tháng 5/2024:**
- **False positives**: Health checks fail không đúng
- **Container restart**: Containers restart không cần thiết
- **Service disruption**: Service bị disrupt
- **Root cause**: Health check quá strict

**Timeline:**
- **10:00 AM**: Health check fail
- **10:00:30 AM**: Container mark unhealthy
- **10:01 AM**: Orchestrator restart container
- **10:01:30 AM**: Application restart, service disrupt
- **10:02 AM**: Team investigate

**Impact:**
- **Service disruption**: Service bị disrupt
- **User complaints**: Users report issues
- **False alarms**: False alarms trong monitoring

### Investigation

**Root cause:**
```dockerfile
HEALTHCHECK --interval=10s --timeout=2s --retries=1 \
  CMD curl -f http://localhost:8080/health || exit 1
```

**Vấn đề:**
- **Too frequent**: Check quá thường xuyên (10s)
- **Too strict**: Retries = 1 (fail ngay)
- **Network issues**: Network hiccup → false positive

**Test:**
```bash
# Network hiccup → health check fail
# Container mark unhealthy → restart
# Application healthy → false positive
```

### Fix

**Solution: Adjust health check**
```dockerfile
HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD curl -f http://localhost:8080/health || exit 1
```

**Changes:**
- **Longer interval**: 30s thay vì 10s
- **More retries**: 3 retries thay vì 1
- **Start period**: 10s cho application start
- **Longer timeout**: 5s timeout

**Kết quả:**
- **Fewer false positives**: Ít false positives hơn
- **Stable**: Health checks stable hơn
- **Accurate**: Accurate hơn

### Result

**Trước:**
- Strict health check
- **False positives**: 10-15 per day
- **Service disruption**: Frequent

**Sau:**
- Adjusted health check
- **False positives**: 0-1 per day
- **Service disruption**: Minimal

### Lesson Learned

1. **Tune health checks**: Tune health check parameters
2. **Balance**: Balance giữa fast detection và false positives
3. **Start period**: Use start period cho application startup
4. **Retries**: Use retries để avoid false positives

---

## 🎓 TÓM TẮT

### Health Checks

**Purpose:**
- Detect application failures
- Automatic recovery
- Load balancer integration
- Monitoring

**Definition:**
- **Dockerfile**: HEALTHCHECK instruction
- **Runtime**: --health-cmd option
- **Options**: interval, timeout, start-period, retries

**Best Practices:**
- **Always use**: Best practice cho production
- **Application-level**: Check application, không chỉ container
- **Tune parameters**: Tune để balance detection và false positives
- **Start period**: Use start period cho startup
- **Retries**: Use retries để avoid false positives

### Health Check States

**States:**
- **starting**: Container starting
- **healthy**: Container healthy
- **unhealthy**: Container unhealthy

**View:**
- `docker ps`: Show health status
- `docker inspect`: Detailed health info

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ bạn đã:
- ✅ Hiểu health checks
- ✅ Biết cách define health checks
- ✅ Debug health check issues

**Day tiếp theo (Day-032)** sẽ đi sâu vào:
- Resource Limits (CPU, Memory) - Advanced
- Production resource management

---

## 📚 TÀI LIỆU THAM KHẢO

- Health checks: https://docs.docker.com/engine/reference/builder/#healthcheck
- Health check options: https://docs.docker.com/engine/reference/run/#healthcheck

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.

