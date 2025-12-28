# Day-047: OOM (Out of Memory) Issues

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được OOM (Out of Memory) là gì
- Biết cách detect OOM issues
- Hiểu được OOM killer
- Biết cách prevent OOM
- Debug OOM issues
- Áp dụng trong production

---

## 💾 PHẦN 1: OOM OVERVIEW

### 1.1. OOM là gì?

**OOM (Out of Memory)** xảy ra khi container **exceed memory limit** và bị **OOM killer** kill.

**Symptoms:**
- Container bị kill đột ngột
- Exit code: 137 (128 + 9, SIGKILL)
- "Out of memory" messages
- No logs trước khi kill

### 1.2. OOM Killer

**OOM Killer** là **Linux kernel mechanism** kill processes khi system out of memory.

**Behavior:**
- **Kill process**: Kill process sử dụng nhiều memory
- **SIGKILL**: Send SIGKILL (không thể catch)
- **No cleanup**: Process không có cơ hội cleanup

### 1.3. Detect OOM

**Check exit code:**
```bash
$ docker inspect <container> --format='{{.State.ExitCode}}'
# Exit code 137 = OOM kill
```

**Check logs:**
```bash
$ docker logs <container>
# May show OOM messages
```

**Check system logs:**
```bash
$ dmesg | grep -i oom
# System OOM messages
```

---

## 🔍 PHẦN 2: DEBUGGING OOM

### 2.1. Memory Usage Analysis

**Monitor memory:**
```bash
$ docker stats <container>
# Real-time memory usage
```

**Check memory limit:**
```bash
$ docker inspect <container> --format='{{.HostConfig.Memory}}'
# Memory limit in bytes
```

**Check memory usage:**
```bash
$ docker exec <container> free -m
# Memory usage trong container
```

### 2.2. Memory Leaks

**Symptoms:**
- Memory usage tăng dần
- Container OOM sau một thời gian
- Memory không được release

**Debug:**
```bash
$ docker stats <container>
# Monitor memory over time
```

**Fix:**
- Fix memory leaks trong code
- Add memory limits
- Monitor memory usage

---

## 🛡️ PHẦN 3: PREVENT OOM

### 3.1. Set Memory Limits

**Set limit:**
```bash
$ docker run -m 512m app
# Limit to 512MB
```

**In Compose:**
```yaml
services:
  app:
    deploy:
      resources:
        limits:
          memory: 512M
```

### 3.2. Monitor Memory

**Monitor:**
```bash
$ docker stats
# Monitor all containers
```

**Alerts:**
- Set up alerts cho high memory usage
- Monitor trends
- Proactive management

---

## 🏭 PRODUCTION STORY: OOM Kill Issues

### Context

**Công ty:** Fintech, 500 employees
**Issue:** Containers bị OOM kill
**Root cause:** No memory limits

### Fix

**Solution: Memory limits**
```bash
$ docker run -m 1g app
```

**Results:**
- No OOM kills
- Predictable memory usage
- Better stability

---

## 🎓 TÓM TẮT

**OOM:**
- Out of Memory
- Exit code 137
- OOM killer

**Prevention:**
- Set memory limits
- Monitor memory
- Fix memory leaks

**Debugging:**
- Check exit code
- Monitor memory
- Analyze usage

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-048)** sẽ đi sâu vào:
- Network Connectivity Problems
- Network debugging

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.

