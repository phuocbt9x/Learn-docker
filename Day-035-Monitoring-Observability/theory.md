# Day-035: Monitoring & Observability

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được monitoring và observability
- Biết cách monitor containers
- Hiểu được metrics collection
- Biết cách setup alerts
- Áp dụng trong production

---

## 📊 PHẦN 1: MONITORING

### 1.1. Monitoring là gì?

**Monitoring** là **theo dõi** containers và applications để detect issues.

**Key metrics:**
- **CPU usage**: CPU utilization
- **Memory usage**: Memory consumption
- **Network I/O**: Network traffic
- **Disk I/O**: Disk usage

### 1.2. Docker Stats

**Real-time monitoring:**
```bash
$ docker stats
# Show real-time stats cho all containers
```

**Specific container:**
```bash
$ docker stats <container>
```

### 1.3. Monitoring Tools

**Tools:**
- **Prometheus**: Metrics collection
- **Grafana**: Visualization
- **cAdvisor**: Container metrics
- **Datadog**: Commercial monitoring

---

## 🏭 PRODUCTION STORY: Monitoring Setup

### Context

**Công ty:** Fintech, 500 employees
**Issue:** No visibility into container performance
**Solution:** Monitoring setup

### Implementation

**Solution: Prometheus + Grafana**
- Prometheus: Metrics collection
- Grafana: Dashboards
- Alerts: Alert on thresholds

**Results:**
- Full visibility
- Proactive alerts
- Better performance

---

## 🎓 TÓM TẮT

**Monitoring:**
- Real-time metrics
- Historical data
- Alerts
- Dashboards

**Observability:**
- Metrics
- Logs
- Traces

---

## 🚀 BƯỚC TIẾP THEO

**Phase 7 hoàn thành!** Bạn đã nắm vững Production Operations.

**Phase tiếp theo (Phase 8)** sẽ đi sâu vào:
- Docker Compose
- Multi-container applications

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.

