# Day-035: Monitoring & Observability - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Basic Monitoring

**Commands:**
```bash
$ docker stats
# Monitor all containers

$ docker stats app
# Monitor specific container
```

**Analysis:**
- CPU usage: 50%
- Memory usage: 400MB/512MB
- Network: 1MB/s

---

## ✅ BÀI TẬP 2: Monitoring Setup

**Architecture:**
- Prometheus: Metrics collection
- Grafana: Visualization
- cAdvisor: Container metrics

**Implementation:**
```bash
# Run cAdvisor
$ docker run -d --name=cadvisor \
  -p 8080:8080 \
  -v /:/rootfs:ro \
  -v /var/run:/var/run:ro \
  -v /sys:/sys:ro \
  -v /var/lib/docker/:/var/lib/docker:ro \
  google/cadvisor
```

**Results:**
- Metrics collected
- Dashboards created
- Alerts configured

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

