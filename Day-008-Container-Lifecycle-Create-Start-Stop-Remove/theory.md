# Day-008: Container Lifecycle - Create, Start, Stop, Remove

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được container lifecycle và các states
- Biết cách create containers (với và không start)
- Biết cách start, stop, restart containers
- Hiểu được sự khác biệt giữa stop và kill
- Biết cách remove containers và cleanup
- Hiểu được restart policies và cách chúng hoạt động

---

## 📖 PHẦN 1: CONTAINER LIFECYCLE

### 1.1. Container States

**Container có các states sau:**

1. **Created**: Container đã được tạo nhưng chưa start
2. **Running**: Container đang chạy
3. **Paused**: Container đã bị pause (tạm dừng)
4. **Restarting**: Container đang restart
5. **Exited**: Container đã stop (process exited)
6. **Dead**: Container failed và không thể recover

**State transitions:**
```
Created → Running → Exited
         ↓
      Paused
         ↓
      Running
         ↓
    Restarting
         ↓
      Running
```

### 1.2. Lifecycle Commands

**Các commands chính:**

- `docker create`: Tạo container (không start)
- `docker start`: Start container
- `docker run`: Create và start container (combined)
- `docker stop`: Stop container (graceful)
- `docker kill`: Kill container (force)
- `docker restart`: Restart container
- `docker pause`: Pause container
- `docker unpause`: Unpause container
- `docker rm`: Remove container

### 1.3. Tại sao cần hiểu Lifecycle?

**Lý do:**

1. **Debug issues**: Hiểu state giúp debug
2. **Resource management**: Quản lý resources hiệu quả
3. **Performance**: Optimize container operations
4. **Production**: Quan trọng cho production operations

---

## 🆕 PHẦN 2: CREATE CONTAINERS

### 2.1. docker create

**Command:**
```bash
$ docker create nginx
```

**Chức năng:**
- Tạo container từ image
- **KHÔNG start** container
- Container ở state **Created**

**Output:**
```
container-id
```

**Ví dụ:**
```bash
$ docker create --name my-nginx nginx
abc123def456...

$ docker ps -a
CONTAINER ID   IMAGE   COMMAND   CREATED   STATUS
abc123...      nginx   ...       1 min ago Created
```

### 2.2. docker create Options

**Common options:**

```bash
# Đặt tên container
$ docker create --name my-container nginx

# Set environment variables
$ docker create -e VAR1=value1 -e VAR2=value2 nginx

# Map ports
$ docker create -p 8080:80 nginx

# Mount volumes
$ docker create -v /host/path:/container/path nginx

# Set working directory
$ docker create -w /app nginx

# Set user
$ docker create -u 1000:1000 nginx

# Set restart policy
$ docker create --restart=always nginx
```

### 2.3. Khi nào dùng docker create?

**Use cases:**

1. **Pre-configure containers**: Tạo trước, start sau
2. **Batch operations**: Tạo nhiều containers, start sau
3. **Testing**: Test container config trước khi start
4. **Resource planning**: Plan resources trước khi start

**Ví dụ:**
```bash
# Tạo container với config phức tạp
$ docker create \
  --name my-app \
  -e DATABASE_URL=postgres://... \
  -e REDIS_URL=redis://... \
  -p 8080:3000 \
  -v app-data:/app/data \
  --restart=unless-stopped \
  my-app:v1.0

# Start sau khi verify config
$ docker start my-app
```

### 2.4. docker create vs docker run

**docker create:**
- Chỉ tạo container
- Không start
- Container ở state Created
- Phải start manually

**docker run:**
- Create và start container
- Container chạy ngay
- Convenient cho most cases

**Khi nào dùng create?**
- Khi cần tạo trước, start sau
- Khi config phức tạp, muốn verify trước
- Khi batch operations

**Khi nào dùng run?**
- Development
- Quick testing
- Most common use cases

---

## ▶️ PHẦN 3: START CONTAINERS

### 3.1. docker start

**Command:**
```bash
$ docker start <container-id>
# Hoặc
$ docker start <container-name>
```

**Chức năng:**
- Start container từ state Created hoặc Exited
- Container chuyển sang state Running

**Ví dụ:**
```bash
$ docker create --name my-nginx nginx
abc123...

$ docker start my-nginx
my-nginx

$ docker ps
CONTAINER ID   STATUS
abc123...      Up 2 seconds
```

### 3.2. docker start Options

**Common options:**

```bash
# Start và attach (xem output)
$ docker start -a my-container

# Start và attach với interactive
$ docker start -ai my-container

# Start detached (background)
$ docker start -d my-container
```

### 3.3. Start Multiple Containers

**Start nhiều containers:**
```bash
$ docker start container1 container2 container3
```

**Start all stopped containers:**
```bash
$ docker start $(docker ps -a -q -f status=exited)
```

### 3.4. Start Behavior

**Khi start container:**

1. **Check state**: Container phải ở Created hoặc Exited
2. **Restore state**: Restore từ previous state (nếu có)
3. **Start process**: Start main process
4. **Apply config**: Apply all configurations
5. **Network setup**: Setup networking
6. **Volume mounts**: Mount volumes

**Lưu ý:**
- Container layer được restore
- Data trong volumes vẫn còn
- Network config được restore
- Environment variables được apply

---

## ⏹️ PHẦN 4: STOP CONTAINERS

### 4.1. docker stop

**Command:**
```bash
$ docker stop <container-id>
# Hoặc
$ docker stop <container-name>
```

**Chức năng:**
- Stop container gracefully
- Gửi SIGTERM signal
- Wait 10 seconds, sau đó SIGKILL nếu cần
- Container chuyển sang state Exited

**Ví dụ:**
```bash
$ docker stop my-nginx
my-nginx

$ docker ps -a
CONTAINER ID   STATUS
abc123...      Exited (0) 5 seconds ago
```

### 4.2. docker stop Options

**Timeout:**
```bash
# Set timeout (seconds)
$ docker stop -t 30 my-container
# Wait 30 seconds before SIGKILL
```

**Default timeout:**
- Default: 10 seconds
- Có thể customize với `-t` option

### 4.3. Stop Multiple Containers

**Stop nhiều containers:**
```bash
$ docker stop container1 container2 container3
```

**Stop all running containers:**
```bash
$ docker stop $(docker ps -q)
```

### 4.4. Stop vs Kill

**docker stop:**
- **Graceful shutdown**: Gửi SIGTERM
- **Wait**: Wait 10 seconds (default)
- **SIGKILL**: Nếu process không stop → SIGKILL
- **Use case**: Normal shutdown

**docker kill:**
- **Force kill**: Gửi SIGKILL ngay
- **No wait**: Không wait
- **Use case**: Container không respond

**Khi nào dùng stop?**
- Normal shutdown
- Allow application cleanup
- Most cases

**Khi nào dùng kill?**
- Container không respond
- Emergency shutdown
- Stop không work

---

## 🔄 PHẦN 5: RESTART CONTAINERS

### 5.1. docker restart

**Command:**
```bash
$ docker restart <container-id>
# Hoặc
$ docker restart <container-name>
```

**Chức năng:**
- Stop container (nếu đang chạy)
- Start container lại
- Equivalent to: `docker stop` + `docker start`

**Ví dụ:**
```bash
$ docker restart my-nginx
my-nginx

$ docker ps
CONTAINER ID   STATUS
abc123...      Up 2 seconds (Restarted 1 minute ago)
```

### 5.2. Restart Options

**Timeout:**
```bash
$ docker restart -t 30 my-container
# Stop với timeout 30 seconds
```

### 5.3. Restart Multiple Containers

**Restart nhiều containers:**
```bash
$ docker restart container1 container2 container3
```

**Restart all containers:**
```bash
$ docker restart $(docker ps -q)
```

### 5.4. Restart vs Stop + Start

**docker restart:**
- Convenient: One command
- Atomic: Stop + Start in one operation
- **Use case**: Quick restart

**docker stop + start:**
- More control: Separate operations
- Can inspect between stop and start
- **Use case**: When need to inspect/config between

**Khi nào dùng restart?**
- Quick restart
- Most cases

**Khi nào dùng stop + start?**
- Need to inspect/config between
- Debugging
- Manual control

---

## 🗑️ PHẦN 6: REMOVE CONTAINERS

### 6.1. docker rm

**Command:**
```bash
$ docker rm <container-id>
# Hoặc
$ docker rm <container-name>
```

**Chức năng:**
- Remove container
- Container phải ở state Exited hoặc Created
- **Xóa container layer** (data mất, trừ volumes)

**Ví dụ:**
```bash
$ docker rm my-nginx
my-nginx

$ docker ps -a
# Container không còn trong list
```

### 6.2. docker rm Options

**Force remove:**
```bash
$ docker rm -f my-container
# Remove even if running (stop first)
```

**Remove volumes:**
```bash
$ docker rm -v my-container
# Remove associated volumes
```

**Remove multiple:**
```bash
$ docker rm container1 container2 container3
```

### 6.3. Remove All Stopped Containers

**Command:**
```bash
$ docker container prune
# Hoặc
$ docker rm $(docker ps -a -q -f status=exited)
```

**Lưu ý:**
- Chỉ remove stopped containers
- Running containers không bị remove
- Data trong container layer mất (trừ volumes)

### 6.4. Remove vs Stop

**docker stop:**
- Stop container
- Container vẫn còn (state Exited)
- Data vẫn còn (trong container layer)
- Có thể start lại

**docker rm:**
- Remove container
- Container bị xóa
- Data mất (trừ volumes)
- Không thể start lại

**Workflow:**
```bash
# Stop trước
$ docker stop my-container

# Remove sau
$ docker rm my-container

# Hoặc force remove (stop + remove)
$ docker rm -f my-container
```

---

## ⏸️ PHẦN 7: PAUSE/UNPAUSE

### 7.1. docker pause

**Command:**
```bash
$ docker pause <container-id>
# Hoặc
$ docker pause <container-name>
```

**Chức năng:**
- Pause container (freeze)
- Container ở state Paused
- **Processes bị freeze** (không chạy, nhưng không kill)
- **Memory vẫn được giữ**

**Ví dụ:**
```bash
$ docker pause my-nginx
my-nginx

$ docker ps
CONTAINER ID   STATUS
abc123...      Up 5 minutes (Paused)
```

### 7.2. docker unpause

**Command:**
```bash
$ docker unpause <container-id>
# Hoặc
$ docker unpause <container-name>
```

**Chức năng:**
- Unpause container
- Container chuyển từ Paused → Running
- **Processes resume** từ điểm pause

**Ví dụ:**
```bash
$ docker unpause my-nginx
my-nginx

$ docker ps
CONTAINER ID   STATUS
abc123...      Up 5 minutes
```

### 7.3. Pause vs Stop

**docker pause:**
- **Freeze processes**: Processes không chạy nhưng không kill
- **Memory kept**: Memory vẫn được giữ
- **Fast resume**: Resume nhanh (không cần restart)
- **Use case**: Temporary pause, debugging

**docker stop:**
- **Kill processes**: Processes bị kill
- **Memory freed**: Memory được free
- **Restart needed**: Cần restart để resume
- **Use case**: Normal shutdown

**Khi nào dùng pause?**
- Temporary pause
- Debugging
- Resource management

**Khi nào dùng stop?**
- Normal shutdown
- Most cases

---

## 🔁 PHẦN 8: RESTART POLICIES

### 8.1. Restart Policies là gì?

**Restart policy** quy định container có tự động restart không khi exit.

**Các policies:**

1. **no**: Không tự động restart (default)
2. **on-failure**: Restart chỉ khi exit với error code
3. **always**: Luôn restart (kể cả khi manually stop)
4. **unless-stopped**: Restart trừ khi manually stop

### 8.2. no (Default)

**Behavior:**
- Không tự động restart
- Container exit → stay exited

**Use case:**
- Development
- One-time tasks
- Manual control

**Example:**
```bash
$ docker run --restart=no nginx
```

### 8.3. on-failure

**Behavior:**
- Restart chỉ khi exit với error code (non-zero)
- Exit code 0 → không restart
- Exit code != 0 → restart

**Options:**
```bash
# Restart với max retries
$ docker run --restart=on-failure:5 nginx
# Restart tối đa 5 lần
```

**Use case:**
- Applications có thể crash
- Want restart on errors only

**Example:**
```bash
$ docker run --restart=on-failure nginx
```

### 8.4. always

**Behavior:**
- Luôn restart khi container exit
- Kể cả khi manually stop → restart khi Docker daemon start

**Use case:**
- Long-running services
- Critical services
- Production services

**Example:**
```bash
$ docker run --restart=always nginx
```

**Lưu ý:**
- **Restart cả khi manually stop**: Có thể không mong muốn
- **unless-stopped** thường tốt hơn

### 8.5. unless-stopped

**Behavior:**
- Restart trừ khi manually stop
- Manually stop → không restart
- Docker daemon restart → restart containers (trừ khi manually stopped)

**Use case:**
- Production services
- **Recommended** cho most cases

**Example:**
```bash
$ docker run --restart=unless-stopped nginx
```

### 8.6. Set Restart Policy

**Khi create:**
```bash
$ docker create --restart=unless-stopped nginx
```

**Khi run:**
```bash
$ docker run --restart=unless-stopped nginx
```

**Update existing container:**
```bash
# Không thể update restart policy của existing container
# Phải recreate container
$ docker stop my-container
$ docker rm my-container
$ docker run --restart=unless-stopped --name my-container nginx
```

---

## 🏭 PRODUCTION STORY #1: Container Không Restart sau Server Reboot

### Context

**Công ty:** SaaS platform, 400 employees
**Hệ thống:** Production servers với Docker
**Team:** 25 DevOps engineers
**Issue:** Containers không tự động restart sau server reboot

### Problem

**Tháng 5/2023:**
- Server reboot (maintenance)
- **Containers không tự động start**
- **Services down** cho đến khi manual start
- **Root cause**: Không set restart policy

**Timeline:**
- **2:00 AM**: Server reboot
- **2:05 AM**: Server up, nhưng containers không start
- **2:10 AM**: Alerts triggered
- **2:15 AM**: Team investigate
- **2:30 AM**: Manual start containers
- **2:35 AM**: Services restored

**Impact:**
- **30 minutes downtime**
- **50K requests failed**
- **Customer complaints**

### Investigation

**Root cause:**
```bash
# Containers được tạo không có restart policy
$ docker ps -a
CONTAINER ID   STATUS
abc123...      Exited (0) 10 minutes ago
def456...      Exited (0) 10 minutes ago
```

**Vấn đề:**
- Containers có restart policy `no` (default)
- Server reboot → containers không tự động start
- Phải manual start

### Fix

**Solution: Set restart policy**

**Option 1: Recreate containers với restart policy**
```bash
# Stop và remove containers
$ docker stop my-container
$ docker rm my-container

# Recreate với restart policy
$ docker run -d \
  --name my-container \
  --restart=unless-stopped \
  my-app:v1.0
```

**Option 2: Update docker-compose.yml**
```yaml
services:
  app:
    image: my-app:v1.0
    restart: unless-stopped  # ← Add restart policy
```

**Option 3: Systemd service (alternative)**
```bash
# Create systemd service để start containers
# Nhưng restart policy đơn giản hơn
```

### Result

**Trước:**
- Containers không tự động restart
- **Manual intervention** cần thiết sau mỗi reboot
- **3-4 incidents** mỗi tháng

**Sau:**
- Containers tự động restart với `unless-stopped`
- **Zero manual intervention** trong 6 tháng
- **Faster recovery** sau reboots

### Lesson Learned

1. **Always set restart policy**: Quan trọng cho production
2. **unless-stopped recommended**: Tốt hơn `always` (không restart khi manually stop)
3. **Test after reboot**: Verify containers restart
4. **Document**: Document restart policies

---

## 🏭 PRODUCTION STORY #2: Container Stop Timeout Issues

### Context

**Công ty:** E-commerce, 600 employees
**Hệ thống:** Microservices với Docker
**Traffic:** 10M requests/day
**Team:** 35 backend engineers

### Problem

**Tháng 7/2023:**
- **Container stop mất quá nhiều thời gian** (30+ seconds)
- **Deployment chậm** (phải wait container stop)
- **Root cause**: Application không handle SIGTERM đúng cách

**Timeline:**
- **10:00 AM**: Deploy new version
- **10:01 AM**: Stop old container
- **10:01:30 AM**: Container vẫn chưa stop (timeout 10s)
- **10:01:40 AM**: Docker send SIGKILL
- **10:01:45 AM**: Container stop
- **10:02 AM**: Start new container
- **Total**: 2 minutes (should be < 30 seconds)

**Impact:**
- **Slow deployments**: 2 minutes thay vì 30 seconds
- **User experience**: Brief service interruption
- **Team velocity**: Deploy chậm hơn

### Investigation

**Root cause:**
```bash
# Application không handle SIGTERM
# Docker send SIGTERM → application không respond
# Wait 10 seconds → Docker send SIGKILL
```

**Vấn đề:**
- Application không có graceful shutdown
- Không handle SIGTERM signal
- Force kill → potential data loss

### Fix

**Solution 1: Fix Application**

**Add signal handling:**
```python
# Python example
import signal
import sys

def signal_handler(sig, frame):
    print('Shutting down gracefully...')
    # Cleanup code
    sys.exit(0)

signal.signal(signal.SIGTERM, signal_handler)
```

**Solution 2: Increase Timeout**

**Temporary fix:**
```bash
$ docker stop -t 30 my-container
# Increase timeout to 30 seconds
```

**Solution 3: Health Checks**

**Add health check:**
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost/health || exit 1
```

### Result

**Trước:**
- Stop timeout: 10 seconds (default)
- **Force kill** thường xuyên
- **Slow deployments**: 2 minutes

**Sau:**
- Application handle SIGTERM
- **Graceful shutdown**: < 5 seconds
- **Fast deployments**: 30 seconds

### Lesson Learned

1. **Handle SIGTERM**: Applications phải handle signals
2. **Graceful shutdown**: Quan trọng cho data integrity
3. **Timeout tuning**: Có thể increase timeout nếu cần
4. **Health checks**: Monitor application health

---

## 🎓 TÓM TẮT

### Container Lifecycle

**States:**
- Created → Running → Exited
- Paused, Restarting, Dead

**Commands:**
- `docker create`: Tạo container
- `docker start`: Start container
- `docker stop`: Stop container (graceful)
- `docker kill`: Kill container (force)
- `docker restart`: Restart container
- `docker pause/unpause`: Pause/resume
- `docker rm`: Remove container

### Best Practices

**Restart Policies:**
- `unless-stopped`: Recommended cho production
- `always`: Có thể restart khi manually stop (không ideal)
- `on-failure`: Restart chỉ khi error
- `no`: Development, manual control

**Stop vs Kill:**
- **stop**: Graceful shutdown (recommended)
- **kill**: Force kill (emergency only)

**Pause vs Stop:**
- **pause**: Temporary freeze (debugging)
- **stop**: Normal shutdown (most cases)

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ bạn đã:
- ✅ Hiểu container lifecycle
- ✅ Biết cách create, start, stop, remove containers
- ✅ Hiểu restart policies

**Day tiếp theo (Day-009)** sẽ đi sâu vào:
- Container Logs & Debugging
- Cách xem và quản lý logs
- Debug container issues

---

## 📚 TÀI LIỆU THAM KHẢO

- Docker Container Lifecycle: https://docs.docker.com/engine/reference/commandline/container/
- Restart Policies: https://docs.docker.com/config/containers/start-containers-automatically/

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-009: Container-Logs-va-Debugging](../Day-009-Container-Logs-va-Debugging/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
