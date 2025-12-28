# Day-008: Giải Pháp & Giải Thích

## 🎯 LƯU Ý QUAN TRỌNG

Các giải pháp dưới đây là commands và explanations thực tế. Quan trọng là bạn:

- **Thực hành** các commands trên terminal
- **Hiểu** container states và transitions
- **Apply** best practices trong production
- **Troubleshoot** khi gặp issues

---

## 📝 BÀI TẬP 1: CONTAINER STATES

### 1.1. Container States

**Tất cả container states:**

1. **Created**: Container đã được tạo nhưng chưa start
2. **Running**: Container đang chạy
3. **Paused**: Container đã bị pause (freeze)
4. **Restarting**: Container đang restart
5. **Exited**: Container đã stop (process exited)
6. **Dead**: Container failed và không thể recover

**Mô tả:**

- **Created**: `docker create` → container ở state này
- **Running**: `docker start` → container chạy
- **Paused**: `docker pause` → container bị freeze
- **Restarting**: Container đang restart (transition state)
- **Exited**: `docker stop` hoặc process exit → container stop
- **Dead**: Container failed (rare)

### 1.2. State Transitions

**Diagram:**
```
Created → Running → Exited
         ↓    ↑
      Paused  |
         ↓    |
      Running |
         ↓
    Restarting
         ↓
      Running
         ↓
      Exited → (remove) → Deleted
```

**Transitions:**

- **Created → Running**: `docker start`
- **Running → Exited**: `docker stop` hoặc process exit
- **Running → Paused**: `docker pause`
- **Paused → Running**: `docker unpause`
- **Exited → Running**: `docker start`
- **Running → Restarting**: Auto-restart (với restart policy)
- **Restarting → Running**: Restart complete
- **Exited → Deleted**: `docker rm`

### 1.3. Check Container State

**Commands:**
```bash
# List running containers
$ docker ps

# List all containers (including stopped)
$ docker ps -a

# Check specific container
$ docker ps -a | grep my-container
```

**Output explanation:**

`docker ps`:
- Chỉ show containers ở state **Running**

`docker ps -a`:
- Show tất cả containers (mọi states)
- **STATUS column**: Hiển thị state
  - `Up X minutes`: Running
  - `Exited (0) X minutes ago`: Exited
  - `Created X minutes ago`: Created
  - `Up X minutes (Paused)`: Paused
  - `Restarting (X) X seconds ago`: Restarting

### 1.4. Practical Exercise

**Step 1: Create container**
```bash
$ docker create --name my-nginx nginx
abc123...

$ docker ps -a | grep my-nginx
abc123...   nginx   ...   Created
# State: Created
```

**Step 2: Start container**
```bash
$ docker start my-nginx
my-nginx

$ docker ps | grep my-nginx
abc123...   nginx   ...   Up 2 seconds
# State: Running
```

**Step 3: Stop container**
```bash
$ docker stop my-nginx
my-nginx

$ docker ps -a | grep my-nginx
abc123...   nginx   ...   Exited (0) 5 seconds ago
# State: Exited
```

**Step 4: Remove container**
```bash
$ docker rm my-nginx
my-nginx

$ docker ps -a | grep my-nginx
# Container không còn trong list
```

---

## 📝 BÀI TẬP 2: CREATE CONTAINERS

### 2.1. Basic Create

**Command:**
```bash
$ docker create --name my-nginx nginx
abc123def456...
```

**Verify:**
```bash
$ docker ps -a | grep my-nginx
abc123...   nginx   ...   Created
```

**Check state:**
```bash
$ docker ps -a --filter "name=my-nginx" --format "{{.Status}}"
Created
```

### 2.2. Create với Options

**Commands:**
```bash
# Environment variables
$ docker create \
  --name my-app \
  -e DATABASE_URL=postgres://... \
  -e API_KEY=secret \
  my-app:v1.0

# Port mapping
$ docker create \
  --name my-nginx \
  -p 8080:80 \
  nginx

# Volume mount
$ docker create \
  --name my-app \
  -v /host/data:/app/data \
  my-app:v1.0

# Restart policy
$ docker create \
  --name my-app \
  --restart=unless-stopped \
  my-app:v1.0

# Combined
$ docker create \
  --name my-app \
  -e ENV_VAR=value \
  -p 8080:3000 \
  -v app-data:/app/data \
  --restart=unless-stopped \
  my-app:v1.0
```

### 2.3. Create vs Run

**docker create:**
- ✅ Chỉ tạo container
- ✅ Không start
- ✅ Có thể verify config trước
- ❌ Phải start manually

**docker run:**
- ✅ Create và start
- ✅ Convenient
- ❌ Không thể verify config trước
- ❌ Start ngay (có thể không mong muốn)

**Khi nào dùng create:**
- Config phức tạp, muốn verify trước
- Batch operations
- Pre-configure containers

**Khi nào dùng run:**
- Development
- Quick testing
- Most common cases

### 2.4. Batch Create

**Script:**
```bash
#!/bin/bash
for i in {1..5}; do
  docker create --name app-$i nginx
done

# Verify
$ docker ps -a | grep app-
```

**Or one-liner:**
```bash
$ for i in {1..5}; do docker create --name app-$i nginx; done
```

---

## 📝 BÀI TẬP 3: START CONTAINERS

### 3.1. Basic Start

**Command:**
```bash
$ docker start my-nginx
my-nginx
```

**Verify:**
```bash
$ docker ps | grep my-nginx
abc123...   nginx   ...   Up 5 seconds
```

**Check logs:**
```bash
$ docker logs my-nginx
```

**Start process:**
1. Check container state (phải Created hoặc Exited)
2. Restore container layer
3. Apply configurations
4. Setup networking
5. Mount volumes
6. Start main process

### 3.2. Start Options

**Start với attach:**
```bash
$ docker start -a my-nginx
# Xem output, block until container stop
```

**Start với interactive:**
```bash
$ docker start -ai my-nginx
# Interactive + attach
```

**Start detached:**
```bash
$ docker start -d my-nginx
# Background (default)
```

**So sánh:**
- `-a`: Xem output, block
- `-i`: Interactive
- `-d`: Background (default)

### 3.3. Start Multiple

**Start nhiều containers:**
```bash
$ docker start container1 container2 container3
```

**Start all stopped:**
```bash
$ docker start $(docker ps -a -q -f status=exited)
```

### 3.4. Start Behavior

**Khi start container:**

1. **Container layer restore**: ✅ Yes
   - Container layer được restore
   - Changes trong container layer vẫn còn

2. **Data trong volumes**: ✅ Yes
   - Volumes được mount lại
   - Data vẫn còn (volumes persistent)

3. **Network config**: ✅ Yes
   - Network được setup lại
   - Port mappings được restore

4. **Environment variables**: ✅ Yes
   - Environment variables được apply

---

## 📝 BÀI TẬP 4: STOP CONTAINERS

### 4.1. Basic Stop

**Command:**
```bash
$ docker stop my-nginx
my-nginx
```

**Verify:**
```bash
$ docker ps -a | grep my-nginx
abc123...   nginx   ...   Exited (0) 5 seconds ago
```

**Check exit code:**
```bash
$ docker inspect my-nginx --format='{{.State.ExitCode}}'
0
# 0 = success, != 0 = error
```

**Stop process:**
1. Send SIGTERM signal
2. Wait 10 seconds (default)
3. If not stopped → send SIGKILL
4. Container → Exited state

### 4.2. Stop Options

**Timeout 30 seconds:**
```bash
$ docker stop -t 30 my-nginx
# Wait 30 seconds before SIGKILL
```

**Timeout 5 seconds:**
```bash
$ docker stop -t 5 my-nginx
# Wait 5 seconds before SIGKILL
```

**So sánh:**
- Longer timeout: More time for graceful shutdown
- Shorter timeout: Faster stop (but may force kill)

**Khi nào increase timeout:**
- Application cần thời gian cleanup
- Database connections cần close
- Long-running operations

### 4.3. Stop Multiple

**Stop nhiều containers:**
```bash
$ docker stop container1 container2 container3
```

**Stop all running:**
```bash
$ docker stop $(docker ps -q)
```

### 4.4. Stop vs Kill

**docker stop:**
- ✅ Graceful shutdown (SIGTERM)
- ✅ Allow cleanup
- ✅ Wait timeout
- ❌ Slower (wait timeout)

**docker kill:**
- ✅ Fast (SIGKILL ngay)
- ❌ Force kill (no cleanup)
- ❌ Potential data loss

**Khi nào dùng stop:**
- Normal shutdown
- Most cases
- Allow cleanup

**Khi nào dùng kill:**
- Container không respond
- Emergency
- Stop không work

**Best practices:**
- **Always use stop first**
- Chỉ dùng kill khi stop không work
- Handle SIGTERM trong applications

---

## 📝 BÀI TẬP 5: RESTART CONTAINERS

### 5.1. Basic Restart

**Command:**
```bash
$ docker restart my-nginx
my-nginx
```

**Verify:**
```bash
$ docker ps | grep my-nginx
abc123...   nginx   ...   Up 2 seconds (Restarted 1 minute ago)
```

**Check uptime:**
```bash
$ docker ps --format "table {{.Names}}\t{{.Status}}"
NAMES       STATUS
my-nginx    Up 2 seconds (Restarted 1 minute ago)
```

**Restart process:**
1. Stop container (nếu đang chạy)
2. Start container
3. Equivalent to: `docker stop` + `docker start`

### 5.2. Restart Options

**Restart với timeout:**
```bash
$ docker restart -t 30 my-nginx
# Stop với timeout 30 seconds
```

**Restart nhiều:**
```bash
$ docker restart container1 container2 container3
```

**So sánh với stop + start:**
- **restart**: One command, atomic
- **stop + start**: Two commands, more control

### 5.3. Restart vs Stop + Start

**docker restart:**
- ✅ Convenient (one command)
- ✅ Atomic operation
- ❌ Less control

**docker stop + start:**
- ✅ More control
- ✅ Can inspect/config between
- ❌ Two commands

**Khi nào dùng restart:**
- Quick restart
- Most cases

**Khi nào dùng stop + start:**
- Need to inspect/config between
- Debugging
- Manual control

### 5.4. Restart Scenarios

**Restart running container:**
```bash
$ docker restart my-nginx
# Stop → Start
```

**Restart stopped container:**
```bash
$ docker restart my-nginx
# Start (no stop needed)
```

**Restart paused container:**
```bash
$ docker restart my-nginx
# Unpause → Stop → Start
```

**Behavior:**
- **Running**: Stop → Start
- **Stopped**: Start (no stop)
- **Paused**: Unpause → Stop → Start

---

## 📝 BÀI TẬP 6: REMOVE CONTAINERS

### 6.1. Basic Remove

**Command:**
```bash
$ docker rm my-nginx
my-nginx
```

**Verify:**
```bash
$ docker ps -a | grep my-nginx
# Container không còn
```

**Remove process:**
1. Check container state (phải Exited hoặc Created)
2. Remove container layer
3. Remove container metadata
4. **Data mất** (trừ volumes)

### 6.2. Remove Options

**Force remove:**
```bash
$ docker rm -f my-nginx
# Stop container first (if running), then remove
```

**Remove với volumes:**
```bash
$ docker rm -v my-nginx
# Remove associated volumes
```

**Remove nhiều:**
```bash
$ docker rm container1 container2 container3
```

**So sánh:**
- `-f`: Force (stop + remove)
- `-v`: Remove volumes
- Multiple: Remove nhiều containers

### 6.3. Remove All Stopped

**Command:**
```bash
$ docker container prune
# Hoặc
$ docker rm $(docker ps -a -q -f status=exited)
```

**Verify:**
```bash
$ docker ps -a
# Chỉ còn running containers
```

**Best practices:**
- Regular cleanup
- Remove containers không dùng
- Keep running containers

### 6.4. Remove vs Stop

**docker stop:**
- Stop container
- Container vẫn còn (Exited)
- Data vẫn còn
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

# Hoặc force (stop + remove)
$ docker rm -f my-container
```

---

## 📝 BÀI TẬP 7: PAUSE/UNPAUSE

### 7.1. Pause Container

**Command:**
```bash
$ docker pause my-nginx
my-nginx
```

**Verify:**
```bash
$ docker ps | grep my-nginx
abc123...   nginx   ...   Up 5 minutes (Paused)
```

**Check state:**
```bash
$ docker inspect my-nginx --format='{{.State.Status}}'
paused
```

**Pause behavior:**
- Processes bị freeze (không chạy)
- Memory vẫn được giữ
- Fast resume (không cần restart)

### 7.2. Unpause Container

**Command:**
```bash
$ docker unpause my-nginx
my-nginx
```

**Verify:**
```bash
$ docker ps | grep my-nginx
abc123...   nginx   ...   Up 5 minutes
```

**Processes resume:**
- ✅ Yes, từ điểm pause
- Không cần restart
- State được restore

### 7.3. Pause vs Stop

**docker pause:**
- ✅ Freeze processes (không kill)
- ✅ Memory kept
- ✅ Fast resume
- ❌ Memory không được free

**docker stop:**
- ✅ Kill processes
- ✅ Memory freed
- ❌ Cần restart để resume
- ❌ Slower

**Khi nào dùng pause:**
- Temporary pause
- Debugging
- Resource management

**Khi nào dùng stop:**
- Normal shutdown
- Most cases

### 7.4. Pause Use Cases

**Use cases:**
- **Debugging**: Pause để inspect
- **Resource management**: Freeze containers khi cần resources
- **Testing**: Test pause/resume behavior

**Best practices:**
- Dùng pause cho temporary freeze
- Dùng stop cho normal shutdown
- Resume sau khi debug

---

## 📝 BÀI TẬP 8: RESTART POLICIES

### 8.1. Restart Policies

**Tất cả policies:**

1. **no**: Không tự động restart (default)
2. **on-failure**: Restart chỉ khi error
3. **always**: Luôn restart
4. **unless-stopped**: Restart trừ khi manually stop

### 8.2. no Policy

**Command:**
```bash
$ docker run --restart=no --name test nginx
```

**Test:**
```bash
$ docker stop test
$ docker ps -a | grep test
# Container ở Exited, không tự động restart
```

**Use cases:**
- Development
- One-time tasks
- Manual control

### 8.3. on-failure Policy

**Command:**
```bash
$ docker run --restart=on-failure --name test nginx
```

**Test exit code 0:**
```bash
$ docker exec test exit 0
$ docker ps -a | grep test
# Container ở Exited, không restart (exit code 0)
```

**Test exit code != 0:**
```bash
$ docker exec test exit 1
$ docker ps | grep test
# Container restart (exit code != 0)
```

**With max retries:**
```bash
$ docker run --restart=on-failure:5 nginx
# Restart tối đa 5 lần
```

### 8.4. always Policy

**Command:**
```bash
$ docker run --restart=always --name test nginx
```

**Test:**
```bash
$ docker stop test
$ docker ps | grep test
# Container tự động restart (even after manual stop)
```

**Use cases:**
- Long-running services
- Critical services

**Limitations:**
- Restart cả khi manually stop (có thể không mong muốn)
- **unless-stopped** thường tốt hơn

### 8.5. unless-stopped Policy

**Command:**
```bash
$ docker run --restart=unless-stopped --name test nginx
```

**Test manual stop:**
```bash
$ docker stop test
$ docker ps -a | grep test
# Container ở Exited, không restart (manually stopped)
```

**Test Docker daemon restart:**
```bash
$ sudo systemctl restart docker
$ docker ps | grep test
# Container tự động restart (nếu không manually stopped)
```

**Tại sao recommended:**
- ✅ Restart sau reboot
- ✅ Không restart khi manually stop
- ✅ Best of both worlds

### 8.6. Update Restart Policy

**Không thể update trực tiếp:**
```bash
# Không có command để update restart policy
# Phải recreate container
```

**Solution:**
```bash
# Stop và remove
$ docker stop my-container
$ docker rm my-container

# Recreate với restart policy mới
$ docker run --restart=unless-stopped --name my-container nginx
```

---

## 📝 BÀI TẬP 9: PRACTICAL SCENARIOS

### Scenario 1: Development

**9.1. Setup containers:**
```bash
# Nginx
$ docker create --name dev-nginx -p 8080:80 nginx

# MySQL
$ docker create --name dev-mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  -p 3306:3306 \
  mysql:8.0

# App
$ docker create --name dev-app \
  -v app-data:/app/data \
  -p 3000:3000 \
  my-app:latest
```

**9.2. Daily workflow:**
```bash
# Morning: Start containers
$ docker start dev-nginx dev-mysql dev-app

# Evening: Stop containers
$ docker stop dev-nginx dev-mysql dev-app

# Restart when needed
$ docker restart dev-app

# Cleanup
$ docker container prune
```

### Scenario 2: Production

**9.3. Production setup:**
```bash
# Create với restart policy
$ docker run -d \
  --name prod-app \
  --restart=unless-stopped \
  -p 80:3000 \
  my-app:v1.0.0

# Test graceful shutdown
$ docker stop prod-app
# Check application logs for cleanup

# Test auto-restart
$ sudo systemctl restart docker
$ docker ps | grep prod-app
# Container tự động restart
```

**9.4. Deployment script:**
```bash
#!/bin/bash
# Stop old container
docker stop old-app

# Start new container
docker run -d \
  --name new-app \
  --restart=unless-stopped \
  -p 80:3000 \
  my-app:v1.1.0

# Verify
sleep 5
docker ps | grep new-app

# Remove old container
docker rm old-app
```

---

## 📝 BÀI TẬP 10: TROUBLESHOOTING

### 10.1. Container Không Start

**Debug:**
```bash
# Check container state
$ docker ps -a | grep my-container

# Check logs
$ docker logs my-container

# Inspect container
$ docker inspect my-container

# Check Docker daemon
$ sudo systemctl status docker
```

**Common causes:**
- Container đã ở Running state
- Image không tồn tại
- Config errors
- Resource constraints

**Fix:**
- Check state trước khi start
- Verify image exists
- Check config
- Check resources

### 10.2. Container Không Stop

**Debug:**
```bash
# Check container
$ docker ps | grep my-container

# Check processes
$ docker top my-container

# Check logs
$ docker logs my-container

# Force kill
$ docker kill my-container
```

**Common causes:**
- Application không handle SIGTERM
- Long-running operations
- Deadlock

**Fix:**
- Increase timeout: `docker stop -t 30`
- Fix application (handle SIGTERM)
- Force kill: `docker kill`

### 10.3. Container Không Restart

**Debug:**
```bash
# Check restart policy
$ docker inspect my-container --format='{{.HostConfig.RestartPolicy.Name}}'

# Check exit code
$ docker inspect my-container --format='{{.State.ExitCode}}'
```

**Common causes:**
- Restart policy = `no`
- Exit code 0 với `on-failure` policy
- Manually stopped với `unless-stopped`

**Fix:**
- Update restart policy (recreate container)
- Check exit code
- Verify policy behavior

### 10.4. Container Remove Fail

**Error:**
```bash
$ docker rm my-container
Error: cannot remove a running container
```

**Fix:**
```bash
# Option 1: Stop first
$ docker stop my-container
$ docker rm my-container

# Option 2: Force remove
$ docker rm -f my-container
```

---

## ✅ TỔNG KẾT

Các bài tập này giúp bạn:

1. **Understand states**: Container states và transitions
2. **Master commands**: create, start, stop, restart, remove
3. **Restart policies**: Setup cho production
4. **Troubleshoot**: Debug lifecycle issues

**Key takeaways:**
- **States**: Created → Running → Exited
- **Commands**: create, start, stop, restart, remove
- **Restart policies**: `unless-stopped` recommended cho production
- **Best practices**: Graceful shutdown, auto-restart

---

**Chúc bạn học tốt! Tiếp tục với Day-009 để học Container Logs & Debugging.**

