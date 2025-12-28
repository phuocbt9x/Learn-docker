# Day-009: Giải Pháp & Giải Thích

## 🎯 LƯU Ý QUAN TRỌNG

Các giải pháp dưới đây là commands và explanations thực tế. Quan trọng là bạn:

- **Thực hành** các commands trên terminal
- **Hiểu** logging mechanism
- **Apply** best practices trong production
- **Troubleshoot** effectively

---

## 📝 BÀI TẬP 1: XEM LOGS

### 1.1. Basic Logs

**Command:**
```bash
$ docker logs my-nginx
```

**Output format:**
```
2023-01-15T10:00:00.123456789Z Starting nginx...
2023-01-15T10:00:01.123456789Z nginx started
```

**Format:**
- **ISO 8601 timestamp**: `YYYY-MM-DDTHH:MM:SS.nnnnnnnnnZ`
- **UTC timezone**: Z suffix
- **Log message**: Application output

**Log location:**
```bash
# Linux
/var/lib/docker/containers/<container-id>/<container-id>-json.log

# Check log path
$ docker inspect --format='{{.LogPath}}' my-nginx
```

### 1.2. Log Options

**Commands:**
```bash
# Last 50 lines
$ docker logs --tail 50 my-nginx

# Follow logs real-time
$ docker logs -f my-nginx

# With timestamps
$ docker logs -t my-nginx

# Since timestamp
$ docker logs --since "2023-01-15T10:00:00" my-nginx

# Until timestamp
$ docker logs --until "2023-01-15T11:00:00" my-nginx

# Combined
$ docker logs -f -t --tail 50 my-nginx
```

### 1.3. Log Filtering

**Commands:**
```bash
# Filter errors
$ docker logs my-nginx 2>&1 | grep -i error

# Filter warnings
$ docker logs my-nginx 2>&1 | grep -i warn

# Count lines
$ docker logs my-nginx | wc -l

# Count errors
$ docker logs my-nginx 2>&1 | grep -i error | wc -l

# Last 100 lines with errors
$ docker logs --tail 100 my-nginx 2>&1 | grep -i error
```

### 1.4. Multiple Containers

**Commands:**
```bash
# View logs của nhiều containers
$ docker logs container1
$ docker logs container2
$ docker logs container3

# Or script
for container in container1 container2 container3; do
  echo "=== $container ==="
  docker logs --tail 10 $container
done

# Compare logs
$ docker logs container1 > logs1.txt
$ docker logs container2 > logs2.txt
$ diff logs1.txt logs2.txt
```

---

## 📝 BÀI TẬP 2: LOGGING DRIVERS

### 2.1. Default Driver

**Check current driver:**
```bash
# Per-container
$ docker inspect --format='{{.HostConfig.LogConfig.Type}}' my-container
json-file

# Daemon default
$ docker info | grep "Logging Driver"
Logging Driver: json-file
```

**Log location:**
```bash
# Linux
/var/lib/docker/containers/<container-id>/<container-id>-json.log
```

### 2.2. Configure json-file Driver

**Per-container:**
```bash
$ docker run --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  nginx
```

**Daemon-level:**
```json
# /etc/docker/daemon.json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

**Restart Docker:**
```bash
$ sudo systemctl restart docker
```

**Verify:**
```bash
$ docker info | grep "Logging Driver"
Logging Driver: json-file
```

### 2.3. Other Drivers

**none driver:**
```bash
$ docker run --log-driver none nginx
# Logs không được lưu
```

**syslog driver:**
```bash
$ docker run --log-driver syslog \
  --log-opt syslog-address=udp://localhost:514 \
  nginx
```

**So sánh:**

| Driver | Use Case | Pros | Cons |
|--------|----------|------|------|
| **json-file** | Default, local | Simple, accessible | Local only |
| **none** | Performance | No overhead | No logs |
| **syslog** | Centralized | Centralized logs | Setup needed |
| **fluentd** | Production | Scalable | Complex setup |

**Khi nào dùng:**
- **json-file**: Development, small scale
- **syslog/fluentd**: Production, centralized logging
- **none**: Performance-critical (not recommended)

### 2.4. Log Rotation

**Tại sao cần log rotation?**
- Prevent disk full
- Manage log size
- Compliance (retention)

**Configure:**
```json
{
  "log-opts": {
    "max-size": "10m",  # Max size per file
    "max-file": "3"     # Number of files
  }
}
```

**Test:**
```bash
# Generate logs
$ docker run --log-opt max-size=1m --log-opt max-file=2 \
  alpine sh -c 'while true; do echo "test"; sleep 1; done'

# Check log files
$ ls -lh /var/lib/docker/containers/*/*-json.log
# Should see rotation when size > 1m
```

**Best practices:**
- Set max-size (10m-100m)
- Set max-file (3-10)
- Monitor disk usage
- External logging cho production

---

## 📝 BÀI TẬP 3: DEBUGGING CONTAINERS

### 3.1. docker exec

**Commands:**
```bash
# Execute shell
$ docker exec -it my-container bash

# Execute command
$ docker exec my-container ls /app
$ docker exec my-container ps aux
$ docker exec my-container env

# As specific user
$ docker exec -u root my-container whoami

# Set working directory
$ docker exec -w /app my-container pwd
```

### 3.2. docker inspect

**Commands:**
```bash
# State
$ docker inspect --format='{{.State.Status}}' my-container
running

# Exit code
$ docker inspect --format='{{.State.ExitCode}}' my-container
0

# Started at
$ docker inspect --format='{{.State.StartedAt}}' my-container
2023-01-15T10:00:00.123456789Z

# Finished at
$ docker inspect --format='{{.State.FinishedAt}}' my-container
0001-01-01T00:00:00Z  # Zero if running

# Error message
$ docker inspect --format='{{.State.Error}}' my-container

# Log path
$ docker inspect --format='{{.LogPath}}' my-container
/var/lib/docker/containers/abc123.../abc123...-json.log
```

### 3.3. docker top

**Command:**
```bash
$ docker top my-container
```

**Output:**
```
UID    PID    PPID   C   STIME   TTY   TIME       CMD
root   1234   0      0   10:00   ?    00:00:00   nginx: master process
www    1235   1234   0   10:00   ?    00:00:00   nginx: worker process
```

**So sánh với ps aux:**
```bash
# docker top: Processes từ host perspective
# ps aux trong container: Processes từ container perspective

$ docker exec my-container ps aux
# PID khác nhau (namespace isolation)
```

### 3.4. docker stats

**Command:**
```bash
# Single container
$ docker stats my-container

# Multiple containers
$ docker stats container1 container2 container3

# All containers
$ docker stats $(docker ps -q)

# No stream (one-time)
$ docker stats --no-stream my-container
```

**Output:**
```
CONTAINER   CPU %   MEM USAGE / LIMIT   MEM %   NET I/O   BLOCK I/O
my-app      2.5%    512MiB / 2GiB       25%     1.2MB/...  10MB/5MB
```

**Use cases:**
- Monitor resource usage
- Identify resource bottlenecks
- Debug performance issues

---

## 📝 BÀI TẬP 4: TROUBLESHOOTING SCENARIOS

### Scenario 1: Container Không Start

**4.1. Debug steps:**

```bash
# Step 1: Check logs
$ docker logs my-app

# Step 2: Check exit code
$ docker inspect --format='{{.State.ExitCode}}' my-app

# Step 3: Check error
$ docker inspect --format='{{.State.Error}}' my-app

# Step 4: Check config
$ docker inspect my-app

# Step 5: Try run manually
$ docker run -it my-image bash
# Test manually
```

**Debug script:**
```bash
#!/bin/bash
CONTAINER="my-app"

echo "=== Logs ==="
docker logs $CONTAINER

echo "=== Exit Code ==="
docker inspect --format='{{.State.ExitCode}}' $CONTAINER

echo "=== Error ==="
docker inspect --format='{{.State.Error}}' $CONTAINER

echo "=== Config ==="
docker inspect $CONTAINER | jq '.[0].Config'
```

**4.2. Common causes:**

1. **Image không tồn tại**
   - Check: `docker images`
   - Fix: Pull image

2. **Port conflict**
   - Check: `netstat -tulpn | grep <port>`
   - Fix: Dùng port khác

3. **Volume mount fail**
   - Check: `docker inspect --format='{{.Mounts}}'`
   - Fix: Check paths

4. **Resource constraints**
   - Check: `docker stats`
   - Fix: Increase limits

5. **Config errors**
   - Check: `docker inspect`
   - Fix: Correct config

### Scenario 2: Container Crash

**4.3. Debug steps:**

```bash
# Step 1: Check logs
$ docker logs my-app

# Step 2: Check exit code
$ docker inspect --format='{{.State.ExitCode}}' my-app
# 0 = success, != 0 = error, 137 = OOM kill

# Step 3: Check last logs
$ docker logs --tail 100 my-app

# Step 4: Check resources
$ docker stats --no-stream my-app

# Step 5: Check system logs
$ journalctl -u docker -n 50
```

**4.4. Root cause analysis:**

**Exit code 0:**
- Normal exit
- Application completed
- **Not an error**

**Exit code != 0:**
- Application error
- Check logs for error messages
- **Debug application**

**Exit code 137:**
- **OOM kill** (Out of Memory)
- Container exceeded memory limit
- **Fix**: Increase memory limit hoặc fix memory leak

**Application errors:**
- Check logs for exceptions
- Check application config
- Check dependencies

### Scenario 3: Performance Issues

**4.5. Debug steps:**

```bash
# Step 1: Check resource usage
$ docker stats my-app

# Step 2: Check processes
$ docker top my-app

# Step 3: Check logs for errors
$ docker logs my-app | grep -i error

# Step 4: Check network
$ docker exec my-app netstat -tulpn
$ docker exec my-app ping 8.8.8.8
```

**4.6. Optimization:**

**Identify bottlenecks:**
- High CPU → CPU-bound process
- High memory → Memory leak hoặc insufficient limit
- High I/O → Disk I/O bottleneck
- High network → Network bottleneck

**Optimize:**
- Set resource limits
- Fix memory leaks
- Optimize application
- Use appropriate base images

---

## 📝 BÀI TẬP 5: LOG MANAGEMENT

### 5.1. Log Storage

**Calculation:**
```
100 containers × 1GB/day = 100GB/day
100GB/day × 30 days = 3TB/month
```

**Management:**
- **Log rotation**: Max 10m per file, 3 files = 30MB per container
- **Retention**: 7-30 days
- **External logging**: Send to centralized system
- **Compression**: Compress old logs

**Log rotation strategy:**
```json
{
  "max-size": "10m",
  "max-file": "3"
}
# Max: 30MB per container
# 100 containers × 30MB = 3GB total
```

### 5.2. Log Aggregation

**Tại sao cần?**
- Centralized logs
- Easier analysis
- Better monitoring
- Compliance

**Tools:**
- **ELK Stack**: Elasticsearch, Logstash, Kibana
- **Fluentd**: Log collector
- **Splunk**: Enterprise logging
- **CloudWatch**: AWS logging
- **Datadog**: Monitoring và logging

**Setup:**
```bash
# Use fluentd driver
$ docker run --log-driver fluentd \
  --log-opt fluentd-address=localhost:24224 \
  nginx
```

### 5.3. Log Analysis

**Commands:**
```bash
# Parse logs for errors
$ docker logs my-app 2>&1 | grep -i error

# Count errors
$ docker logs my-app 2>&1 | grep -i error | wc -l

# Analyze patterns
$ docker logs my-app | awk '{print $1}' | sort | uniq -c

# Find most common errors
$ docker logs my-app 2>&1 | grep -i error | sort | uniq -c | sort -rn
```

**Tools:**
- **grep**: Basic filtering
- **awk**: Text processing
- **jq**: JSON parsing
- **ELK**: Advanced analysis

### 5.4. Log Retention

**Policy:**
- **Development**: 7 days
- **Staging**: 14 days
- **Production**: 30-90 days
- **Compliance**: As required

**Implementation:**
```bash
# Cleanup old logs
$ find /var/lib/docker/containers -name "*-json.log" -mtime +30 -delete

# Or use log rotation
# Max 3 files × 10MB = 30MB, auto cleanup
```

---

## 📝 BÀI TẬP 6: PRACTICAL DEBUGGING

### 6.1. Container Exit Immediately

**Debug:**
```bash
# Check logs
$ docker logs my-app

# Check exit code
$ docker inspect --format='{{.State.ExitCode}}' my-app

# Run manually
$ docker run -it my-image bash
# Test commands manually

# Check CMD/ENTRYPOINT
$ docker inspect --format='{{.Config.Cmd}}' my-app
```

**Common causes:**
- CMD/ENTRYPOINT sai
- Application crash ngay
- Missing dependencies
- Config errors

### 6.2. Container Không Respond

**Debug:**
```bash
# Check if running
$ docker ps | grep my-app

# Check processes
$ docker top my-app

# Test connectivity
$ docker exec my-app curl http://localhost

# Check logs
$ docker logs my-app

# Check network
$ docker inspect --format='{{.NetworkSettings}}' my-app
```

**Fix:**
- Check application health
- Check network config
- Check firewall
- Restart container

### 6.3. High Memory Usage

**Debug:**
```bash
# Check memory
$ docker stats my-app

# Check processes
$ docker top my-app

# Check memory in container
$ docker exec my-app free -h

# Check for leaks
$ docker logs my-app | grep -i memory
```

**Fix:**
- Increase memory limit
- Fix memory leaks
- Optimize application
- Use memory-efficient base images

### 6.4. Network Connectivity

**Debug:**
```bash
# Test ping
$ docker exec my-app ping 8.8.8.8

# Check DNS
$ docker exec my-app nslookup google.com

# Check network config
$ docker inspect --format='{{.NetworkSettings}}' my-app

# Check network mode
$ docker inspect --format='{{.HostConfig.NetworkMode}}' my-app
```

**Fix:**
- Check network mode
- Check DNS config
- Check firewall
- Check network driver

---

## 📝 BÀI TẬP 7: LOGGING BEST PRACTICES

### 7.1. Structured Logging

**Tại sao cần?**
- Easier parsing
- Better analysis
- Machine-readable
- Consistent format

**Format:**
```json
{
  "timestamp": "2023-01-15T10:00:00Z",
  "level": "ERROR",
  "message": "Database connection failed",
  "service": "payment-service",
  "request_id": "abc123"
}
```

**Implementation:**
```python
# Python example
import json
import logging

class StructuredFormatter(logging.Formatter):
    def format(self, record):
        log_data = {
            "timestamp": self.formatTime(record),
            "level": record.levelname,
            "message": record.getMessage(),
            "service": "my-app"
        }
        return json.dumps(log_data)

logger = logging.getLogger()
handler = logging.StreamHandler()
handler.setFormatter(StructuredFormatter())
logger.addHandler(handler)
```

### 7.2. Log Levels

**Levels:**
- **DEBUG**: Detailed info (development)
- **INFO**: General info
- **WARN**: Warnings
- **ERROR**: Errors
- **FATAL**: Critical errors

**When to use:**
- **DEBUG**: Development, troubleshooting
- **INFO**: Normal operations
- **WARN**: Potential issues
- **ERROR**: Errors that need attention
- **FATAL**: Critical failures

**Configure:**
```python
# Python
logging.basicConfig(level=logging.INFO)
# Production: INFO, Development: DEBUG
```

### 7.3. Sensitive Data

**Prevent logging sensitive data:**
- **Don't log**: Passwords, API keys, tokens
- **Mask**: Partial data (e.g., `password: ****`)
- **Sanitize**: Remove sensitive fields
- **Validate**: Check logs before commit

**Best practices:**
- Use environment variables
- Don't log request bodies với sensitive data
- Mask sensitive fields
- Regular audit logs

### 7.4. Log Monitoring

**Setup:**
- **Alerts**: Alert on errors
- **Dashboards**: Visualize logs
- **Metrics**: Track error rates
- **Trends**: Analyze patterns

**Tools:**
- **ELK**: Dashboards, alerts
- **Datadog**: Monitoring, alerts
- **Splunk**: Enterprise monitoring
- **Prometheus + Grafana**: Metrics, dashboards

---

## 📝 BÀI TẬP 8: ADVANCED DEBUGGING

### 8.1. Debug với docker exec

**Commands:**
```bash
# Access filesystem
$ docker exec -it my-app bash
$ ls /app
$ cat /app/config.json

# Check configs
$ docker exec my-app cat /etc/nginx/nginx.conf

# Test connectivity
$ docker exec my-app curl http://localhost
$ docker exec my-app ping 8.8.8.8

# Run diagnostics
$ docker exec my-app netstat -tulpn
$ docker exec my-app ps aux
$ docker exec my-app env
```

### 8.2. Debug với docker inspect

**Commands:**
```bash
# Config
$ docker inspect --format='{{.Config}}' my-app | jq

# Network
$ docker inspect --format='{{.NetworkSettings}}' my-app | jq

# Volumes
$ docker inspect --format='{{.Mounts}}' my-app | jq

# Environment
$ docker inspect --format='{{.Config.Env}}' my-app
```

### 8.3. Compare Containers

**Commands:**
```bash
# Compare configs
$ docker inspect container1 > config1.json
$ docker inspect container2 > config2.json
$ diff config1.json config2.json

# Compare networks
$ docker inspect --format='{{.NetworkSettings}}' container1
$ docker inspect --format='{{.NetworkSettings}}' container2

# Compare environments
$ docker inspect --format='{{.Config.Env}}' container1
$ docker inspect --format='{{.Config.Env}}' container2
```

### 8.4. Debug Production Issues

**Best practices:**
- **Read-only**: Use `docker exec` (read-only when possible)
- **Non-intrusive**: Don't modify production containers
- **Logging**: Enable comprehensive logging
- **Monitoring**: Use monitoring tools
- **Backup**: Backup before debugging

**Safety measures:**
- Test commands trên staging first
- Document debugging process
- Use read-only commands when possible
- Don't restart containers without approval

---

## ✅ TỔNG KẾT

Các bài tập này giúp bạn:

1. **View logs**: Xem và quản lý logs hiệu quả
2. **Configure logging**: Setup logging drivers
3. **Debug issues**: Troubleshoot container problems
4. **Best practices**: Apply production best practices

**Key takeaways:**
- **Logs**: Essential cho debugging và monitoring
- **Log rotation**: Quan trọng để prevent disk full
- **Structured logging**: Easier analysis
- **Debugging**: Systematic approach
- **Best practices**: Security, monitoring, retention

---

**Chúc bạn học tốt! Tiếp tục với Day-010 để học Docker Hub & Registry Basics.**

