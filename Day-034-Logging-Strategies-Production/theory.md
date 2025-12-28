# Day-034: Logging Strategies cho Production

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được logging strategies trong production
- Biết cách configure logging drivers
- Hiểu được centralized logging
- Biết cách integrate với logging systems
- Áp dụng trong production

---

## 📝 PHẦN 1: LOGGING DRIVERS

### 1.1. Logging Drivers

**Docker logging drivers:**
- **json-file**: Default, JSON files
- **syslog**: Syslog
- **journald**: systemd journal
- **gelf**: Graylog Extended Log Format
- **fluentd**: Fluentd
- **awslogs**: AWS CloudWatch
- **splunk**: Splunk

### 1.2. Configure Logging Driver

**Default:**
```bash
$ docker run app
# Uses json-file driver
```

**Custom driver:**
```bash
$ docker run --log-driver=syslog app
$ docker run --log-driver=fluentd --log-opt fluentd-address=localhost:24224 app
```

### 1.3. Logging Options

**Options:**
- **max-size**: Max log file size
- **max-file**: Max number of log files
- **labels**: Add labels to logs

**Example:**
```bash
$ docker run \
  --log-driver=json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  app
```

---

## 🏭 PRODUCTION STORY: Centralized Logging

### Context

**Công ty:** SaaS, 600 employees
**Issue:** Logs scattered across containers
**Solution:** Centralized logging

### Implementation

**Solution: Fluentd**
```bash
$ docker run --log-driver=fluentd \
  --log-opt fluentd-address=localhost:24224 \
  app
```

**Results:**
- Centralized logs
- Easy search
- Better monitoring

---

## 🎓 TÓM TẮT

**Logging strategies:**
- Choose appropriate driver
- Centralized logging
- Log rotation
- Integration với logging systems

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-035)** sẽ đi sâu vào:
- Monitoring & Observability
- Metrics và alerts

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.

