# Day-047: OOM (Out of Memory) Issues - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Detect OOM

**Commands:**
```bash
$ docker inspect <container> --format='{{.State.ExitCode}}'
# Exit code 137 = OOM

$ dmesg | grep -i oom
# System OOM messages
```

**Detection:**
- Exit code: 137
- OOM messages in dmesg
- Container killed

---

## ✅ BÀI TẬP 2: Memory Analysis

**Commands:**
```bash
$ docker stats <container>
# Monitor memory usage
```

**Analysis:**
- Memory usage tăng dần
- Memory leak detected
- Fix: Fix memory leak trong code

---

## ✅ BÀI TẬP 3: Prevent OOM

**Configuration:**
```bash
$ docker run -m 512m app
# Set memory limit
```

**Monitoring:**
```bash
$ docker stats
# Monitor memory
```

**Results:**
- Memory limit enforced
- No OOM kills
- Predictable usage

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

