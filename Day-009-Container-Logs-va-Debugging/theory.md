# Day-009: Container Logs & Debugging

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được container logging mechanism
- Biết cách xem và quản lý container logs
- Biết cách debug container issues
- Hiểu được logging drivers và cách configure
- Biết cách troubleshoot common container problems
- Hiểu được best practices cho container logging

---

## 📖 PHẦN 1: CONTAINER LOGGING

### 1.1. Container Logs là gì?

**Container logs** là output từ processes chạy trong container:
- **stdout**: Standard output
- **stderr**: Standard error
- **Application logs**: Logs từ application

**Đặc điểm:**
- **Stored locally**: Logs được lưu trên host
- **JSON format**: Default format là JSON
- **Rotated**: Logs được rotate để prevent disk full
- **Accessible**: Có thể xem qua Docker CLI

### 1.2. Logging Mechanism

**Cách hoạt động:**

```
Application in Container
    ↓ (writes to stdout/stderr)
Container Runtime
    ↓ (captures output)
Logging Driver
    ↓ (processes logs)
Log File (on host)
```

**Log location:**
- Linux: `/var/lib/docker/containers/<container-id>/<container-id>-json.log`
- macOS/Windows: Docker Desktop manages logs

### 1.3. Tại sao Logs quan trọng?

**Lý do:**

1. **Debugging**: Debug issues trong containers
2. **Monitoring**: Monitor application behavior
3. **Troubleshooting**: Troubleshoot problems
4. **Auditing**: Audit và compliance
5. **Performance**: Analyze performance issues

---

## 📋 PHẦN 2: XEM LOGS

### 2.1. docker logs

**Basic command:**
```bash
$ docker logs <container-id>
# Hoặc
$ docker logs <container-name>
```

**Output:**
```
2023-01-15T10:00:00.123456789Z Starting application...
2023-01-15T10:00:01.123456789Z Server listening on port 3000
2023-01-15T10:00:02.123456789Z Database connected
```

### 2.2. docker logs Options

**Common options:**

```bash
# Follow logs (real-time)
$ docker logs -f my-container

# Follow với timestamps
$ docker logs -f -t my-container

# Last N lines
$ docker logs --tail 100 my-container

# Since timestamp
$ docker logs --since "2023-01-15T10:00:00" my-container

# Until timestamp
$ docker logs --until "2023-01-15T11:00:00" my-container

# Combined
$ docker logs -f -t --tail 50 my-container
```

### 2.3. Log Timestamps

**View với timestamps:**
```bash
$ docker logs -t my-container
```

**Output:**
```
2023-01-15T10:00:00.123456789Z Starting application...
2023-01-15T10:00:01.123456789Z Server listening on port 3000
```

**Format:**
- **ISO 8601**: `YYYY-MM-DDTHH:MM:SS.nnnnnnnnnZ`
- **UTC timezone**: Z suffix

### 2.4. Follow Logs

**Real-time logs:**
```bash
$ docker logs -f my-container
```

**Behavior:**
- Show existing logs
- Follow new logs in real-time
- **Ctrl+C** để exit

**Use case:**
- Debugging
- Monitoring
- Development

---

## 🔍 PHẦN 3: LOGGING DRIVERS

### 3.1. Logging Drivers là gì?

**Logging driver** quy định cách Docker lưu và xử lý logs.

**Default driver:**
- **json-file**: Lưu logs dưới dạng JSON files

**Các drivers khác:**
- **none**: Không lưu logs
- **syslog**: Gửi đến syslog
- **journald**: Gửi đến systemd journal
- **gelf**: Gửi đến Graylog
- **fluentd**: Gửi đến Fluentd
- **awslogs**: Gửi đến CloudWatch Logs
- **splunk**: Gửi đến Splunk
- **etwlogs**: Windows Event Tracing
- **gcplogs**: Gửi đến Google Cloud Logging
- **logentries**: Gửi đến Logentries
- **datadog**: Gửi đến Datadog

### 3.2. json-file Driver (Default)

**Đặc điểm:**
- Lưu logs trong JSON format
- Rotate logs automatically
- Accessible qua `docker logs`

**Configuration:**
```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3"
  }
}
```

**Options:**
- `max-size`: Max size của mỗi log file (default: -1, unlimited)
- `max-file`: Số lượng log files (default: 1)

### 3.3. Configure Logging Driver

**Per-container:**
```bash
$ docker run --log-driver json-file \
  --log-opt max-size=10m \
  --log-opt max-file=3 \
  nginx
```

**Daemon-level (default):**
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

**Restart Docker daemon:**
```bash
$ sudo systemctl restart docker
```

### 3.4. none Driver

**Disable logging:**
```bash
$ docker run --log-driver none nginx
```

**Use case:**
- Performance optimization
- Security (không lưu logs)
- **Not recommended** cho production

---

## 🐛 PHẦN 4: DEBUGGING CONTAINERS

### 4.1. docker exec

**Execute command trong running container:**
```bash
$ docker exec <container-id> <command>
```

**Ví dụ:**
```bash
# Execute shell
$ docker exec -it my-container bash

# Execute command
$ docker exec my-container ls /app
$ docker exec my-container ps aux
$ docker exec my-container env
```

### 4.2. docker exec Options

**Common options:**

```bash
# Interactive + TTY
$ docker exec -it my-container bash

# Run as specific user
$ docker exec -u root my-container bash

# Set working directory
$ docker exec -w /app my-container pwd

# Set environment variables
$ docker exec -e VAR=value my-container env
```

### 4.3. docker inspect

**Inspect container:**
```bash
$ docker inspect my-container
```

**Useful fields:**
```bash
# State
$ docker inspect --format='{{.State.Status}}' my-container

# Exit code
$ docker inspect --format='{{.State.ExitCode}}' my-container

# Started at
$ docker inspect --format='{{.State.StartedAt}}' my-container

# Finished at
$ docker inspect --format='{{.State.FinishedAt}}' my-container

# Error message
$ docker inspect --format='{{.State.Error}}' my-container

# Log path
$ docker inspect --format='{{.LogPath}}' my-container
```

### 4.4. docker top

**View running processes:**
```bash
$ docker top my-container
```

**Output:**
```
UID    PID    PPID   C   STIME   TTY   TIME       CMD
root   1234   0      0   10:00   ?    00:00:00   nginx: master process
www    1235   1234   0   10:00   ?    00:00:00   nginx: worker process
```

**Use case:**
- Debug processes
- Check resource usage
- Verify application running

---

## 🔧 PHẦN 5: TROUBLESHOOTING

### 5.1. Container Không Start

**Debug steps:**

1. **Check logs:**
   ```bash
   $ docker logs my-container
   ```

2. **Check exit code:**
   ```bash
   $ docker inspect --format='{{.State.ExitCode}}' my-container
   ```

3. **Check error:**
   ```bash
   $ docker inspect --format='{{.State.Error}}' my-container
   ```

4. **Check config:**
   ```bash
   $ docker inspect my-container
   ```

5. **Try run manually:**
   ```bash
   $ docker run -it my-image bash
   # Test manually
   ```

### 5.2. Container Crash

**Debug steps:**

1. **Check logs:**
   ```bash
   $ docker logs my-container
   ```

2. **Check exit code:**
   ```bash
   $ docker inspect --format='{{.State.ExitCode}}' my-container
   # != 0 = error
   ```

3. **Check last logs:**
   ```bash
   $ docker logs --tail 100 my-container
   ```

4. **Check system logs:**
   ```bash
   $ journalctl -u docker
   ```

### 5.3. Container Performance Issues

**Debug steps:**

1. **Check resource usage:**
   ```bash
   $ docker stats my-container
   ```

2. **Check processes:**
   ```bash
   $ docker top my-container
   ```

3. **Check logs for errors:**
   ```bash
   $ docker logs my-container | grep -i error
   ```

4. **Check network:**
   ```bash
   $ docker exec my-container netstat -tulpn
   ```

### 5.4. Container Network Issues

**Debug steps:**

1. **Check network:**
   ```bash
   $ docker network ls
   $ docker network inspect <network-name>
   ```

2. **Check container network:**
   ```bash
   $ docker inspect --format='{{.NetworkSettings}}' my-container
   ```

3. **Test connectivity:**
   ```bash
   $ docker exec my-container ping 8.8.8.8
   $ docker exec my-container curl http://localhost
   ```

4. **Check DNS:**
   ```bash
   $ docker exec my-container nslookup google.com
   ```

---

## 🏭 PRODUCTION STORY #1: Log Disk Full Issue

### Context

**Công ty:** SaaS platform, 500 employees
**Hệ thống:** 200 containers chạy trên 10 servers
**Traffic:** 5M requests/day
**Team:** 30 DevOps engineers

### Problem

**Tháng 6/2023:**
- **Server disk full** → containers không thể write logs
- **Containers crash** do không thể write
- **Root cause**: Logs không được rotate, accumulate

**Timeline:**
- **10:00 AM**: Disk usage 95%
- **10:30 AM**: Disk usage 100%
- **10:35 AM**: Containers bắt đầu crash
- **10:40 AM**: Alerts triggered
- **11:00 AM**: Team investigate
- **11:30 AM**: Fix và recover

**Impact:**
- **50 minutes downtime**
- **200K requests failed**
- **Customer complaints**

### Investigation

**Root cause:**
```bash
# Check disk usage
$ df -h
/dev/sda1   100G   100G   0B   100%  /var/lib/docker

# Check log files
$ du -sh /var/lib/docker/containers/*/
# Logs chiếm 80GB!
```

**Vấn đề:**
- **Không có log rotation**: Logs không được rotate
- **Unlimited log size**: Default config không limit
- **200 containers**: Nhiều containers → nhiều logs

### Fix

**Solution 1: Configure Log Rotation**

**Daemon config:**
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

**Per-container:**
```bash
$ docker run --log-opt max-size=10m --log-opt max-file=3 nginx
```

**Solution 2: Cleanup Old Logs**
```bash
# Cleanup old logs
$ docker system prune -a --volumes

# Or manually
$ truncate -s 0 /var/lib/docker/containers/*/*-json.log
```

**Solution 3: External Logging**
```bash
# Use external logging (syslog, fluentd, etc.)
$ docker run --log-driver syslog nginx
```

### Result

**Trước:**
- Logs không được rotate
- **Disk full** mỗi tuần
- **Manual cleanup** cần thiết

**Sau:**
- Log rotation configured
- **Max 30MB per container** (10m × 3 files)
- **Zero disk full issues** trong 6 tháng

### Lesson Learned

1. **Always configure log rotation**: Quan trọng cho production
2. **Monitor disk usage**: Track disk usage
3. **External logging**: Consider external logging cho production
4. **Regular cleanup**: Cleanup old logs

---

## 🏭 PRODUCTION STORY #2: Debug Container Crash

### Context

**Công ty:** E-commerce, 800 employees
**Hệ thống:** Microservices với Docker
**Traffic:** 20M requests/day
**Team:** 40 backend engineers

### Problem

**Tháng 9/2023:**
- **Payment service container crash** mỗi 2-3 giờ
- **Root cause unknown**: Không có logs đầy đủ
- **Impact**: Payment failures

**Timeline:**
- **Day 1**: Container crash, team investigate
- **Day 2**: Still crashing, no clear cause
- **Day 3**: Enable detailed logging
- **Day 4**: Found root cause (memory leak)

**Impact:**
- **Payment failures**: 100+ failures/day
- **Revenue loss**: $10K/day
- **Customer complaints**: High

### Investigation

**Initial investigation:**
```bash
# Check logs
$ docker logs payment-service
# Chỉ thấy "Container exited"

# Check exit code
$ docker inspect --format='{{.State.ExitCode}}' payment-service
137  # OOM kill
```

**Vấn đề:**
- **Logs không đầy đủ**: Application không log trước khi crash
- **OOM kill**: Container bị kill do out of memory
- **No warning**: Không có warning trước khi crash

### Fix

**Solution 1: Enable Detailed Logging**

**Application logging:**
```python
# Add memory monitoring
import logging
import psutil

logging.info(f"Memory usage: {psutil.virtual_memory().percent}%")
```

**Solution 2: Monitor Resources**
```bash
# Monitor memory
$ docker stats payment-service

# Set memory limits
$ docker run --memory="2g" payment-service
```

**Solution 3: Health Checks**
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/health || exit 1
```

### Result

**Trước:**
- Logs không đầy đủ
- **Hard to debug** crashes
- **3-4 days** để find root cause

**Sau:**
- Detailed logging enabled
- **Memory monitoring** in place
- **Debug time**: 1-2 hours thay vì 3-4 days

### Lesson Learned

1. **Comprehensive logging**: Logs đầy đủ quan trọng
2. **Resource monitoring**: Monitor memory, CPU
3. **Health checks**: Early detection
4. **Structured logging**: Easier to parse và analyze

---

## 🎓 TÓM TẮT

### Container Logging

**Commands:**
- `docker logs`: Xem logs
- `docker logs -f`: Follow logs
- `docker logs -t`: Với timestamps
- `docker logs --tail N`: Last N lines

**Logging Drivers:**
- `json-file`: Default (local files)
- `syslog`: System logs
- `none`: Disable logging
- External: fluentd, splunk, etc.

### Debugging

**Commands:**
- `docker exec`: Execute commands
- `docker inspect`: Inspect container
- `docker top`: View processes
- `docker stats`: Resource usage

**Debug Process:**
1. Check logs
2. Check exit code
3. Check resources
4. Check network
5. Execute commands

### Best Practices

**Logging:**
- Configure log rotation
- Use structured logging
- External logging cho production
- Monitor log size

**Debugging:**
- Comprehensive logging
- Resource monitoring
- Health checks
- Regular debugging practice

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ bạn đã:
- ✅ Hiểu container logging
- ✅ Biết cách xem và quản lý logs
- ✅ Biết cách debug container issues

**Day tiếp theo (Day-010)** sẽ đi sâu vào:
- Docker Hub & Registry Basics
- Push/pull images
- Private registries

---

## 📚 TÀI LIỆU THAM KHẢO

- Docker Logging: https://docs.docker.com/config/containers/logging/
- Docker Debugging: https://docs.docker.com/config/containers/logging/

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.

