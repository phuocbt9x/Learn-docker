# Day-034: Logging Strategies cho Production - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Logging Drivers

**Test:**
```bash
# json-file
$ docker run --log-driver=json-file app

# syslog
$ docker run --log-driver=syslog app

# fluentd
$ docker run --log-driver=fluentd --log-opt fluentd-address=localhost:24224 app
```

**Recommendations:**
- json-file: Development
- syslog: Linux systems
- fluentd: Centralized logging

---

## ✅ BÀI TẬP 2: Centralized Logging

**Architecture:**
- Fluentd collector
- Elasticsearch storage
- Kibana visualization

**Implementation:**
```bash
$ docker run --log-driver=fluentd \
  --log-opt fluentd-address=localhost:24224 \
  app
```

**Results:**
- Centralized logs
- Easy search
- Production-ready

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

