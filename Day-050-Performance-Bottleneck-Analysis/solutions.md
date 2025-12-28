# Day-050: Performance Bottleneck Analysis - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Performance Monitoring

**Commands:**
```bash
$ docker stats
# Monitor all containers

$ docker stats <container>
# Monitor specific container
```

**Analysis:**
- CPU: 80% (high)
- Memory: 400MB/512MB
- I/O: Normal

---

## ✅ BÀI TẬP 2: Bottleneck Identification

**Bottleneck:** CPU

**Root cause:** Inefficient algorithm

**Fix:** Optimize algorithm

---

## ✅ BÀI TẬP 3: Performance Optimization

**Optimizations:**
- Optimize code
- Increase CPU limit
- Scale containers

**Results:**
- CPU usage: 80% → 40%
- Response time: 2s → 0.5s

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

