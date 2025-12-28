# Day-006: Giải Pháp & Giải Thích

## 🎯 LƯU Ý QUAN TRỌNG

Các giải pháp dưới đây là commands và explanations thực tế. Quan trọng là bạn:

- **Thực hành** các commands trên terminal
- **Hiểu** tại sao mỗi command hoạt động
- **Troubleshoot** nếu gặp lỗi
- **Experiment** với các options khác nhau

---

## 📝 BÀI TẬP 1: CÀI ĐẶT DOCKER

### 1.1. Check System Requirements

**Check kernel version:**
```bash
$ uname -r
5.4.0-74-generic  # Phải >= 3.10
```

**Check cgroup support:**
```bash
$ mount | grep cgroup
cgroup on /sys/fs/cgroup type cgroup2
# Hoặc
cgroup on /sys/fs/cgroup type cgroup
```

**Check namespace support:**
```bash
$ ls /proc/self/ns
cgroup  ipc  mnt  net  pid  pid_for_children  user  uts
# Phải có: pid, net, mnt, uts, ipc, user, cgroup
```

**Check architecture:**
```bash
$ uname -m
x86_64  # Phải là 64-bit
```

**Check disk space:**
```bash
$ df -h
# Phải có ít nhất 20GB free
```

### 1.2. Cài Đặt Docker

**Step 1: Update system**
```bash
$ sudo apt update
$ sudo apt upgrade -y
```

**Step 2: Install prerequisites**
```bash
$ sudo apt install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

**Step 3: Add Docker's GPG key**
```bash
$ sudo mkdir -p /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
    sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

**Step 4: Add Docker repository**
```bash
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

**Step 5: Install Docker**
```bash
$ sudo apt update
$ sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**Step 6: Verify installation**
```bash
$ sudo docker run hello-world
# Phải thấy "Hello from Docker!"
```

### 1.3. Post-Installation Setup

**Add user to docker group:**
```bash
$ sudo usermod -aG docker $USER
$ newgrp docker  # Hoặc logout/login lại
```

**Verify không cần sudo:**
```bash
$ docker run hello-world
# Nếu chạy được → OK
```

**Start và enable Docker daemon:**
```bash
$ sudo systemctl start docker
$ sudo systemctl enable docker  # Auto-start on boot
```

**Verify Docker daemon:**
```bash
$ sudo systemctl status docker
# Phải thấy "active (running)"
```

### 1.4. Troubleshooting

**Permission denied:**

**Root cause:**
- User không trong docker group
- Docker socket chỉ accessible bởi root và docker group

**Fix:**
```bash
$ sudo usermod -aG docker $USER
$ newgrp docker
# Hoặc logout/login lại
```

**Docker daemon not running:**

**Root cause:**
- Docker daemon chưa start

**Fix:**
```bash
$ sudo systemctl start docker
$ sudo systemctl status docker
```

**Cannot pull image:**

**Debug:**
```bash
# Check internet
$ ping 8.8.8.8

# Check DNS
$ nslookup registry-1.docker.io

# Check firewall
$ sudo ufw status

# Check Docker daemon logs
$ sudo journalctl -u docker -n 50
```

**Fix:**
- Fix internet connection
- Configure DNS nếu cần
- Allow Docker ports trong firewall
- Configure proxy nếu cần

---

## 📝 BÀI TẬP 2: FIRST CONTAINER

### 2.1. Hello World Container

**Command:**
```bash
$ docker run hello-world
```

**Output:**
```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
...
Hello from Docker!
This message shows that your installation appears to be working correctly.
...
```

**Giải thích:**
1. Docker không tìm thấy image locally → pull từ Docker Hub
2. Docker tạo và chạy container
3. Container in message và exit ngay

**Container có chạy lâu không?**

**❌ KHÔNG:**
- Container chạy, in message, và exit ngay
- Không có process nào chạy lâu
- Container stop ngay sau khi in message

**Verify:**
```bash
$ docker ps -a
# Container có status "Exited"
```

### 2.2. Nginx Container

**Command:**
```bash
$ docker run -d -p 8080:80 --name my-nginx nginx
```

**Giải thích:**
- `-d`: Detached mode (background)
- `-p 8080:80`: Map port 8080 (host) → 80 (container)
- `--name my-nginx`: Đặt tên container
- `nginx`: Image name

**Verify container running:**
```bash
$ docker ps
# Phải thấy my-nginx đang chạy
```

**Access web server:**
```bash
$ curl http://localhost:8080
# Hoặc mở browser: http://localhost:8080
# Phải thấy nginx welcome page
```

**Stop và remove:**
```bash
$ docker stop my-nginx
$ docker rm my-nginx
```

### 2.3. Interactive Container

**Command:**
```bash
$ docker run -it ubuntu:20.04 bash
```

**Trong container:**
```bash
root@container-id:/# touch /tmp/test.txt
root@container-id:/# ls /tmp/test.txt
/tmp/test.txt
root@container-id:/# exit
```

**Container có còn chạy không?**

**❌ KHÔNG:**
- Khi exit → container stop
- Container có status "Exited"

**File có còn không?**

**⚠️ CÓ (trong container layer):**
- File vẫn còn trong container layer
- Nhưng container đã stop
- Nếu remove container → file mất

**Verify:**
```bash
$ docker ps -a
# Container có status "Exited"

$ docker start <container-id>
$ docker exec <container-id> ls /tmp/test.txt
# ✅ File vẫn còn

$ docker rm <container-id>
# File mất khi container bị xóa
```

### 2.4. Container Lifecycle

**Tạo container (không start):**
```bash
$ docker create --name my-container nginx
container-id
```

**Start container:**
```bash
$ docker start my-container
# Hoặc
$ docker start container-id
```

**Stop container:**
```bash
$ docker stop my-container
# Hoặc
$ docker stop container-id
```

**Remove container:**
```bash
$ docker rm my-container
# Hoặc
$ docker rm container-id
```

**Complete lifecycle:**
```bash
# Create
$ docker create --name test nginx

# Start
$ docker start test

# Check status
$ docker ps

# Stop
$ docker stop test

# Remove
$ docker rm test
```

---

## 📝 BÀI TẬP 3: DOCKER COMMANDS

### 3.1. List Containers

**List running containers:**
```bash
$ docker ps
```

**List all containers:**
```bash
$ docker ps -a
```

**List với format custom:**
```bash
$ docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"
```

**Giải thích output columns:**
- **CONTAINER ID**: Unique ID của container
- **IMAGE**: Image name
- **COMMAND**: Command đang chạy
- **CREATED**: Thời gian tạo
- **STATUS**: Trạng thái (running, exited, etc.)
- **PORTS**: Port mappings
- **NAMES**: Container name

### 3.2. List Images

**List images:**
```bash
$ docker images
```

**List với format custom:**
```bash
$ docker images --format "table {{.Repository}}\t{{.Tag}}\t{{.Size}}"
```

**Giải thích output columns:**
- **REPOSITORY**: Image repository name
- **TAG**: Image tag (version)
- **IMAGE ID**: Unique ID của image
- **CREATED**: Thời gian tạo
- **SIZE**: Image size

### 3.3. Container Management

**Stop container:**
```bash
$ docker stop <container-id>
# Hoặc
$ docker stop <container-name>
```

**Start stopped container:**
```bash
$ docker start <container-id>
# Hoặc
$ docker start <container-name>
```

**Restart container:**
```bash
$ docker restart <container-id>
# Hoặc
$ docker restart <container-name>
```

**Remove container:**
```bash
$ docker rm <container-id>
# Hoặc
$ docker rm <container-name>
```

**Remove all stopped containers:**
```bash
$ docker container prune
# Hoặc
$ docker rm $(docker ps -a -q -f status=exited)
```

### 3.4. Docker Info

**Check Docker version:**
```bash
$ docker --version
Docker version 24.0.0, build abc123
```

**Check detailed version:**
```bash
$ docker version
Client: Docker Engine - Community
 Version:           24.0.0
 ...
Server: Docker Engine - Community
 Version:           24.0.0
 ...
```

**Check system info:**
```bash
$ docker info
```

**Thông tin quan trọng:**
- **Containers**: Số containers (running, stopped)
- **Images**: Số images
- **Storage Driver**: overlay2 (recommended)
- **Operating System**: OS info
- **Kernel Version**: Kernel version
- **Total Memory**: RAM available
- **CPUs**: CPU cores

---

## 📝 BÀI TẬP 4: VERIFY INSTALLATION

### 4.1. Check Docker Version

**Command:**
```bash
$ docker --version
Docker version 24.0.0, build abc123
```

**Client vs Server version:**

**Có thể khác nhau:**
- Client version: Version của Docker CLI
- Server version: Version của Docker daemon
- **Nên giống nhau** để tránh compatibility issues

**Version nào quan trọng hơn?**

**Server version:**
- Server version quan trọng hơn
- Server quản lý containers
- Client chỉ là interface

**Check cả 2:**
```bash
$ docker version
# Xem cả Client và Server version
```

### 4.2. Check Docker Info

**Command:**
```bash
$ docker info
```

**Thông tin quan trọng:**

1. **Containers:**
   - Running: Số containers đang chạy
   - Paused: Số containers paused
   - Stopped: Số containers stopped

2. **Images:**
   - Số images local

3. **Storage Driver:**
   - overlay2 (recommended)
   - Quan trọng cho performance

4. **Operating System:**
   - OS info
   - Kernel version

5. **Resources:**
   - Total Memory
   - CPUs
   - Disk space

### 4.3. Test Docker

**Test với hello-world:**
```bash
$ docker run hello-world
# Phải thấy "Hello from Docker!"
```

**Test với nginx:**
```bash
$ docker run -d -p 8080:80 nginx
$ curl http://localhost:8080
# Phải thấy HTML response
$ docker stop $(docker ps -q)
```

**Test với interactive container:**
```bash
$ docker run -it ubuntu:20.04 bash
# Phải vào được container
# Type commands, exit
```

### 4.4. Troubleshooting Checklist

**Checklist:**

1. **Docker installed:**
   ```bash
   $ docker --version
   ```

2. **Docker daemon running:**
   ```bash
   $ sudo systemctl status docker
   ```

3. **User in docker group:**
   ```bash
   $ groups | grep docker
   ```

4. **Can run containers:**
   ```bash
   $ docker run hello-world
   ```

5. **Can pull images:**
   ```bash
   $ docker pull nginx
   ```

6. **Network access:**
   ```bash
   $ docker run -d -p 8080:80 nginx
   $ curl http://localhost:8080
   ```

**All tests pass → Installation OK!**

---

## 📝 BÀI TẬP 5: TROUBLESHOOTING

### 5.1. Permission Denied

**Root cause:**
- User không trong docker group
- Docker socket (`/var/run/docker.sock`) chỉ accessible bởi root và docker group

**Fix:**
```bash
$ sudo usermod -aG docker $USER
$ newgrp docker
# Hoặc logout/login lại
```

**Security implications:**
- Users trong docker group có quyền tương đương root
- Chỉ add users đáng tin cậy
- Production: Cân nhắc dùng sudo

### 5.2. Docker Daemon Not Running

**Root cause:**
- Docker daemon chưa start hoặc crash

**Fix:**
```bash
$ sudo systemctl start docker
$ sudo systemctl status docker
```

**Prevent:**
```bash
$ sudo systemctl enable docker  # Auto-start on boot
```

**Debug nếu vẫn fail:**
```bash
$ sudo journalctl -u docker -n 50
# Xem logs để tìm lỗi
```

### 5.3. Cannot Pull Image

**Root cause:**
- Network issues
- Firewall blocking
- DNS issues
- Proxy configuration

**Debug:**
```bash
# Check internet
$ ping 8.8.8.8

# Check DNS
$ nslookup registry-1.docker.io

# Check firewall
$ sudo ufw status

# Check Docker logs
$ sudo journalctl -u docker -n 50
```

**Fix:**
- Fix internet connection
- Configure DNS
- Allow Docker ports trong firewall
- Configure proxy nếu cần

### 5.4. Port Already in Use

**Root cause:**
- Port 8080 đã được sử dụng bởi process khác

**Check port đang dùng:**
```bash
$ sudo lsof -i :8080
# Hoặc
$ sudo netstat -tulpn | grep 8080
```

**Fix:**
```bash
# Option 1: Dùng port khác
$ docker run -d -p 8081:80 nginx

# Option 2: Stop process đang dùng port
$ sudo kill <pid>

# Option 3: Stop container đang dùng port
$ docker stop <container-id>
```

---

## 📝 BÀI TẬP 6: PRACTICAL SCENARIOS

### Scenario 1: Development Environment

**6.1. Setup nginx:**
```bash
$ docker run -d -p 8080:80 --name dev-nginx nginx
$ curl http://localhost:8080
# Verify OK
```

**6.2. Setup MySQL:**
```bash
$ docker run -d \
  -p 3306:3306 \
  --name dev-mysql \
  -e MYSQL_ROOT_PASSWORD=rootpassword \
  mysql:8.0

# Verify
$ docker ps | grep mysql
```

**6.3. Manage containers:**
```bash
# List all
$ docker ps -a

# Stop when not needed
$ docker stop dev-nginx dev-mysql

# Start when needed
$ docker start dev-nginx dev-mysql

# Remove when not needed
$ docker rm dev-nginx dev-mysql
```

### Scenario 2: Production Server

**6.4. Installation:**
```bash
# Follow installation steps từ bài tập 1.2
# Security: Không add users to docker group (dùng sudo)
# Document process
```

**6.5. Verification:**
```bash
# Run all tests từ bài tập 4.3
# Document results
# Create runbook
```

---

## 📝 BÀI TẬP 7: DOCKER COMMANDS DEEP DIVE

### 7.1. docker run Options

**`-d` (Detached mode):**
- Chạy container ở background
- Container không block terminal
- **Use case**: Long-running services

**`-it` (Interactive + TTY):**
- `-i`: Interactive (giữ STDIN open)
- `-t`: TTY (allocate pseudo-TTY)
- **Use case**: Interactive shells

**`-p` (Port mapping):**
- Syntax: `-p host-port:container-port`
- Example: `-p 8080:80`
- **Use case**: Expose container ports

**`--name` (Container name):**
- Đặt tên cho container
- Dễ reference hơn container ID
- **Use case**: Named containers

**`--rm` (Auto-remove):**
- Tự động remove container khi stop
- **Use case**: Temporary containers

### 7.2. docker ps Options

**`-a` (All containers):**
- List cả stopped containers
- **Use case**: Xem tất cả containers

**`-q` (Quiet mode):**
- Chỉ output container IDs
- **Use case**: Scripts, pipes

**`--format` (Custom format):**
```bash
$ docker ps --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"
```

**`--filter` (Filter containers):**
```bash
$ docker ps --filter "status=running"
$ docker ps --filter "name=nginx"
```

### 7.3. docker stop vs docker kill

**docker stop:**
- Gửi SIGTERM signal
- Graceful shutdown
- Wait 10 seconds, then SIGKILL
- **Use case**: Normal shutdown

**docker kill:**
- Gửi SIGKILL signal ngay
- Force kill
- **Use case**: Container không respond

**Best practices:**
- Dùng `stop` trước
- Chỉ dùng `kill` khi `stop` không work

### 7.4. docker rm Options

**`-f` (Force remove):**
- Remove running container
- **Use case**: Force remove

**`-v` (Remove volumes):**
- Remove associated volumes
- **Use case**: Cleanup volumes

**`docker container prune`:**
- Remove all stopped containers
- **Use case**: Cleanup

---

## 📝 BÀI TẬP 8: DOCKER SYSTEM MANAGEMENT

### 8.1. Cleanup Containers

**Remove stopped containers:**
```bash
$ docker container prune
```

**Remove all containers:**
```bash
$ docker rm -f $(docker ps -aq)
```

**Best practices:**
- Regular cleanup
- Use `--rm` flag cho temporary containers
- Document cleanup process

### 8.2. Cleanup Images

**Remove unused images:**
```bash
$ docker image prune
```

**Remove all images:**
```bash
$ docker rmi -f $(docker images -q)
```

**Best practices:**
- Keep images đang dùng
- Remove old/unused images
- Regular cleanup

### 8.3. System Cleanup

**Cleanup everything:**
```bash
$ docker system prune -a
```

**Cái gì bị xóa:**
- Stopped containers
- Unused images
- Unused networks
- Unused volumes (nếu dùng `-v`)

**Cái gì không bị xóa:**
- Running containers
- Images đang dùng
- Volumes (trừ khi dùng `-v`)

**Khi nào nên cleanup:**
- Disk space low
- Regular maintenance
- After testing

### 8.4. Disk Usage

**Check disk usage:**
```bash
$ docker system df
```

**Giảm disk usage:**
```bash
# Remove unused images
$ docker image prune -a

# Remove stopped containers
$ docker container prune

# Remove unused volumes
$ docker volume prune

# Full cleanup
$ docker system prune -a --volumes
```

**Best practices:**
- Monitor disk usage
- Regular cleanup
- Use multi-stage builds
- Optimize images

---

## ✅ TỔNG KẾT

Các bài tập này giúp bạn:

1. **Cài đặt Docker**: Đúng cách, verify installation
2. **Chạy containers**: Hello-world, nginx, interactive
3. **Quản lý containers**: Start, stop, remove
4. **Troubleshoot**: Permission, daemon, network issues
5. **System management**: Cleanup, disk usage

**Key takeaways:**
- **Installation**: Check requirements, follow steps
- **Verification**: Test với hello-world, nginx
- **Commands**: Practice các commands cơ bản
- **Troubleshooting**: Understand common issues
- **Management**: Regular cleanup, monitor resources

---

**Chúc bạn học tốt! Tiếp tục với Day-007 để học Docker Images - Pull, Tag, Inspect.**

