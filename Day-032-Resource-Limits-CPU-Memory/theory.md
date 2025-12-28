# Day-032: Resource Limits (CPU, Memory) - Advanced

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu sâu về resource limits trong production
- Biết cách set và tune CPU limits
- Biết cách set và tune memory limits
- Hiểu được OOM (Out of Memory) handling
- Monitor và optimize resources
- Áp dụng trong production scenarios

---

## 💻 PHẦN 1: CPU LIMITS - ADVANCED

### 1.1. CPU Limits Types

**CPU limits:**
- **--cpus**: Limit số CPUs
- **--cpu-shares**: Relative CPU priority
- **--cpuset-cpus**: Pin to specific CPUs

**Examples:**
```bash
$ docker run --cpus="1.5" app
$ docker run --cpu-shares=512 app
$ docker run --cpuset-cpus="0,1" app
```

### 1.2. CPU Shares

**CPU shares** là **relative priority** giữa containers.

**Default:** 1024

**Examples:**
- Container A: 1024 shares
- Container B: 512 shares
- Container A nhận 2x CPU của Container B

### 1.3. CPU Pinning

**CPU pinning** là **pin container** to specific CPUs.

**Use cases:**
- **Performance**: Better performance
- **Isolation**: CPU isolation
- **NUMA**: NUMA-aware applications

---

## 🧠 PHẦN 2: MEMORY LIMITS - ADVANCED

### 2.1. Memory Limits Types

**Memory limits:**
- **-m / --memory**: Hard limit
- **--memory-reservation**: Soft limit
- **--memory-swap**: Swap limit

**Examples:**
```bash
$ docker run -m 512m app
$ docker run --memory-reservation=256m app
$ docker run -m 512m --memory-swap=1g app
```

### 2.2. OOM (Out of Memory) Handling

**OOM Killer:**
- Khi container exceed memory limit
- OOM killer kill process
- Container có thể bị kill

**Prevent OOM:**
- Set appropriate limits
- Monitor memory usage
- Tune application

### 2.3. Memory Swapping

**Swap:**
- **--memory-swap**: Total memory + swap
- **-m 512m --memory-swap=1g**: 512MB memory + 512MB swap

**Best practice:**
- **Disable swap**: Cho production (performance)
- **Monitor**: Monitor swap usage

---

## 📊 PHẦN 3: MONITORING RESOURCES

### 3.1. Docker Stats

**Real-time stats:**
```bash
$ docker stats
# Show CPU, memory usage cho all containers
```

**Specific container:**
```bash
$ docker stats <container>
```

### 3.2. Resource Usage Analysis

**Analyze usage:**
- **CPU**: Check CPU usage patterns
- **Memory**: Check memory usage và leaks
- **Trends**: Identify trends

---

## 🏭 PRODUCTION STORY: Resource Exhaustion

### Context

**Công ty:** SaaS, 700 employees
**Issue:** Containers consume all resources
**Root cause:** No resource limits

### Fix

**Solution: Resource limits**
```bash
$ docker run -m 1g --cpus="2" app
```

**Results:**
- No resource exhaustion
- Predictable performance
- Better stability

---

## 🎓 TÓM TẮT

**CPU limits:**
- --cpus: Limit CPUs
- --cpu-shares: Relative priority
- --cpuset-cpus: CPU pinning

**Memory limits:**
- -m: Hard limit
- --memory-reservation: Soft limit
- --memory-swap: Swap limit

**Monitoring:**
- docker stats: Real-time monitoring
- Analyze usage: Identify issues

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-033)** sẽ đi sâu vào:
- Container Restart Policies
- Automatic recovery

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.

