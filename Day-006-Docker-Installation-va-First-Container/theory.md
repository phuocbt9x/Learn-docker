# Day-006: Docker Installation & First Container

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Cài đặt được Docker trên Linux
- Hiểu được Docker requirements và prerequisites
- Chạy được container đầu tiên
- Hiểu được các Docker commands cơ bản
- Verify được Docker installation

---

## 📖 PHẦN 1: DOCKER REQUIREMENTS

### 1.1. System Requirements

**Operating System:**

**Linux:**
- Ubuntu 20.04 LTS hoặc mới hơn (recommended)
- Debian 10 hoặc mới hơn
- CentOS 7 hoặc mới hơn
- Fedora 32 hoặc mới hơn
- RHEL 7 hoặc mới hơn

**macOS:**
- macOS 10.15 hoặc mới hơn
- Dùng Docker Desktop (không phải native Docker)

**Windows:**
- Windows 10/11 (64-bit)
- Dùng Docker Desktop (không phải native Docker)

**Lưu ý:**
- **Linux là recommended** cho production
- macOS và Windows dùng Linux VM bên trong (Docker Desktop)

### 1.2. Kernel Requirements

**Linux Kernel:**
- **Kernel version**: 3.10+ (64-bit)
- **cgroup version**: cgroup v1 hoặc v2
- **Namespace support**: PID, Network, Mount, UTS, IPC, User
- **Overlay filesystem**: overlay2 storage driver

**Check kernel version:**
```bash
$ uname -r
5.4.0-74-generic  # Phải >= 3.10
```

**Check kernel features:**
```bash
# Check cgroup support
$ mount | grep cgroup
cgroup on /sys/fs/cgroup type cgroup2

# Check namespace support
$ ls /proc/self/ns
cgroup  ipc  mnt  net  pid  pid_for_children  user  uts
```

### 1.3. Hardware Requirements

**Minimum:**
- **CPU**: 1 core
- **RAM**: 2GB
- **Disk**: 20GB free space

**Recommended (Production):**
- **CPU**: 2+ cores
- **RAM**: 4GB+
- **Disk**: 50GB+ free space
- **Network**: Stable internet connection (để pull images)

### 1.4. Prerequisites

**Linux:**

1. **Update system:**
   ```bash
   $ sudo apt update && sudo apt upgrade -y
   ```

2. **Install prerequisites:**
   ```bash
   $ sudo apt install -y \
       ca-certificates \
       curl \
       gnupg \
       lsb-release
   ```

3. **Remove old Docker (nếu có):**
   ```bash
   $ sudo apt remove docker docker-engine docker.io containerd runc
   ```

**macOS/Windows:**
- Download Docker Desktop từ docker.com
- Install như ứng dụng bình thường

---

## 📦 PHẦN 2: DOCKER INSTALLATION

### 2.1. Installation trên Ubuntu/Debian

**Method 1: Repository (Recommended)**

**Step 1: Add Docker's official GPG key**
```bash
$ sudo mkdir -p /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

**Step 2: Setup repository**
```bash
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

**Step 3: Install Docker Engine**
```bash
$ sudo apt update
$ sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

**Step 4: Verify installation**
```bash
$ sudo docker run hello-world
```

**Method 2: Convenience Script (Không recommended cho production)**

```bash
$ curl -fsSL https://get.docker.com -o get-docker.sh
$ sudo sh get-docker.sh
```

**Lưu ý:**
- Convenience script không phù hợp production
- Không thể customize installation
- Security concerns

### 2.2. Post-Installation Setup

**Add user to docker group (để chạy Docker không cần sudo):**

```bash
$ sudo usermod -aG docker $USER
$ newgrp docker  # Hoặc logout/login lại
```

**Verify không cần sudo:**
```bash
$ docker run hello-world
# Nếu chạy được → OK
```

**Lưu ý bảo mật:**
- User trong docker group có quyền tương đương root
- Chỉ add users đáng tin cậy
- Production: Cân nhắc dùng sudo

**Start Docker daemon:**
```bash
$ sudo systemctl start docker
$ sudo systemctl enable docker  # Auto-start on boot
```

**Verify Docker daemon:**
```bash
$ sudo systemctl status docker
# Phải thấy "active (running)"
```

### 2.3. Installation trên macOS

**Method: Docker Desktop**

1. **Download Docker Desktop:**
   - Truy cập: https://www.docker.com/products/docker-desktop
   - Download cho macOS

2. **Install:**
   - Mở file .dmg
   - Kéo Docker vào Applications
   - Chạy Docker Desktop

3. **Verify:**
   ```bash
   $ docker run hello-world
   ```

**Lưu ý:**
- Docker Desktop trên macOS chạy Linux VM bên trong
- Performance overhead (~5-10%)
- Phù hợp development, không phù hợp production

### 2.4. Installation trên Windows

**Method: Docker Desktop**

1. **Requirements:**
   - Windows 10/11 (64-bit)
   - WSL 2 enabled
   - Virtualization enabled trong BIOS

2. **Install WSL 2 (nếu chưa có):**
   ```powershell
   wsl --install
   ```

3. **Download Docker Desktop:**
   - Truy cập: https://www.docker.com/products/docker-desktop
   - Download cho Windows

4. **Install và verify:**
   - Install Docker Desktop
   - Verify: `docker run hello-world`

**Lưu ý:**
- Docker Desktop trên Windows cũng chạy Linux VM
- Cần WSL 2
- Phù hợp development

---

## 🐳 PHẦN 3: FIRST CONTAINER

### 3.1. Hello World Container

**Chạy container đầu tiên:**

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
1. Docker không tìm thấy image `hello-world` locally
2. Docker pull image từ Docker Hub
3. Docker tạo và chạy container
4. Container in ra message và exit

### 3.2. Các Commands Cơ Bản

**docker run:**
```bash
# Chạy container từ image
$ docker run <image-name>

# Chạy với options
$ docker run -d <image-name>        # Detached mode (background)
$ docker run -it <image-name>       # Interactive + TTY
$ docker run --name my-container <image-name>  # Đặt tên
```

**docker ps:**
```bash
# List running containers
$ docker ps

# List all containers (including stopped)
$ docker ps -a
```

**docker images:**
```bash
# List images
$ docker images
```

**docker stop:**
```bash
# Stop running container
$ docker stop <container-id>
```

**docker rm:**
```bash
# Remove container
$ docker rm <container-id>

# Remove all stopped containers
$ docker container prune
```

### 3.3. Chạy Nginx Container

**Chạy web server:**

```bash
$ docker run -d -p 8080:80 --name my-nginx nginx
```

**Giải thích:**
- `-d`: Detached mode (chạy background)
- `-p 8080:80`: Map port 8080 (host) → 80 (container)
- `--name my-nginx`: Đặt tên container
- `nginx`: Image name

**Verify:**
```bash
# Check container running
$ docker ps

# Access web server
$ curl http://localhost:8080
# Hoặc mở browser: http://localhost:8080
```

**Stop và remove:**
```bash
$ docker stop my-nginx
$ docker rm my-nginx
```

### 3.4. Interactive Container

**Chạy container interactive:**

```bash
$ docker run -it ubuntu:20.04 bash
```

**Giải thích:**
- `-i`: Interactive (giữ STDIN open)
- `-t`: TTY (allocate pseudo-TTY)
- `ubuntu:20.04`: Image
- `bash`: Command to run

**Trong container:**
```bash
# Bạn đang trong container
root@container-id:/# ls
root@container-id:/# pwd
root@container-id:/# exit  # Thoát container
```

**Lưu ý:**
- Khi exit → container stop
- Muốn chạy background → dùng `-d`

---

## 🔍 PHẦN 4: VERIFY INSTALLATION

### 4.1. Check Docker Version

```bash
$ docker --version
Docker version 24.0.0, build abc123

$ docker version
Client: Docker Engine - Community
 Version:           24.0.0
 ...
Server: Docker Engine - Community
 Version:           24.0.0
 ...
```

**Giải thích:**
- `docker --version`: Version ngắn gọn
- `docker version`: Chi tiết Client và Server

### 4.2. Check Docker Info

```bash
$ docker info
```

**Output:**
```
Client:
 Context:    default
 ...

Server:
 Containers: 1
  Running: 1
  Paused: 0
  Stopped: 0
 Images: 2
 ...
 Storage Driver: overlay2
 ...
```

**Thông tin quan trọng:**
- Containers: Số containers
- Images: Số images
- Storage Driver: overlay2 (recommended)
- Operating System: OS info
- Kernel Version: Kernel version

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

### 4.4. Troubleshooting

**Vấn đề 1: Permission denied**

**Error:**
```bash
$ docker run hello-world
permission denied while trying to connect to the Docker daemon socket
```

**Fix:**
```bash
# Add user to docker group
$ sudo usermod -aG docker $USER
$ newgrp docker

# Hoặc dùng sudo
$ sudo docker run hello-world
```

**Vấn đề 2: Docker daemon not running**

**Error:**
```bash
$ docker run hello-world
Cannot connect to the Docker daemon. Is the docker daemon running?
```

**Fix:**
```bash
# Start Docker daemon
$ sudo systemctl start docker

# Check status
$ sudo systemctl status docker
```

**Vấn đề 3: Cannot pull image**

**Error:**
```bash
$ docker run nginx
Error response from daemon: Get "https://registry-1.docker.io/v2/": net/http: request canceled
```

**Fix:**
- Check internet connection
- Check firewall
- Check Docker Hub access
- Configure proxy nếu cần

---

## 🏭 PRODUCTION STORY #1: Docker Installation Issues tại Production

### Context

**Công ty:** Startup, 50 employees
**Hệ thống:** Production servers cần cài Docker
**Team:** 5 DevOps engineers
**Timeline:** Cần deploy trong 1 ngày

### Problem

**Tháng 2/2023:**
- Cài Docker trên 10 production servers
- **3 servers fail** installation
- **Root cause**: Kernel version cũ, không support Docker

**Timeline:**
- **9:00 AM**: Bắt đầu cài Docker
- **10:00 AM**: 7 servers OK
- **10:30 AM**: 3 servers fail
- **11:00 AM**: Debug issues
- **12:00 PM**: Fix issues

**Impact:**
- **Delay deployment**: 3 giờ
- **Production risk**: Servers không ready

### Investigation

**Server 1:**
```bash
$ uname -r
3.8.0-44-generic  # ← Kernel quá cũ (< 3.10)
```

**Server 2:**
```bash
$ mount | grep cgroup
# ← Không có cgroup support
```

**Server 3:**
```bash
$ ls /proc/self/ns
# ← Thiếu namespace support
```

**Root cause:**
- **Kernel version cũ**: < 3.10
- **Thiếu kernel features**: cgroup, namespace
- **Không check requirements** trước khi cài

### Fix

**Solution 1: Upgrade Kernel (Server 1)**
```bash
$ sudo apt update
$ sudo apt install -y linux-generic
$ sudo reboot
# Sau reboot, kernel mới → cài Docker OK
```

**Solution 2: Enable cgroup (Server 2)**
```bash
# Check kernel config
$ zcat /proc/config.gz | grep CGROUP
# Enable cgroup trong kernel config
# Rebuild kernel (phức tạp) hoặc upgrade kernel
```

**Solution 3: Upgrade OS (Server 3)**
```bash
# Upgrade Ubuntu lên version mới hơn
$ sudo do-release-upgrade
```

### Result

**Trước:**
- 3 servers không cài được Docker
- Delay 3 giờ
- Production risk

**Sau:**
- Tất cả servers cài Docker thành công
- **Lesson learned**: Check requirements trước

**Best Practices:**
1. **Check requirements** trước khi cài
2. **Document requirements** cho team
3. **Test trên staging** trước production
4. **Automate installation** (Ansible, Terraform)

### Lesson Learned

1. **Check requirements trước**: Quan trọng để prevent issues
2. **Kernel version**: Phải >= 3.10
3. **Kernel features**: Phải có cgroup, namespace
4. **Automation**: Dùng tools để automate installation

---

## 🏭 PRODUCTION STORY #2: Docker Permission Issues

### Context

**Công ty:** E-commerce, 200 employees
**Hệ thống:** Development servers
**Team:** 20 developers
**Issue:** Developers không chạy được Docker

### Problem

**Tháng 4/2023:**
- Developers cần chạy Docker để develop
- **Permission denied** errors
- **Workaround**: Dùng sudo (không ideal)

**Timeline:**
- **Day 1**: Developers report permission issues
- **Day 2**: Team investigate
- **Day 3**: Fix và document

**Impact:**
- **Developer productivity**: Giảm (phải dùng sudo)
- **Security risk**: Sudo có thể gây issues

### Investigation

**Error:**
```bash
$ docker run hello-world
permission denied while trying to connect to the Docker daemon socket
```

**Root cause:**
- Developers không trong `docker` group
- Docker socket (`/var/run/docker.sock`) chỉ accessible bởi root và docker group

**Check:**
```bash
$ groups
user docker  # ← Không có docker group
$ ls -la /var/run/docker.sock
srw-rw---- 1 root docker 0 ... /var/run/docker.sock
# ← Chỉ root và docker group có quyền
```

### Fix

**Solution: Add users to docker group**
```bash
# Add user to docker group
$ sudo usermod -aG docker $USER

# Verify
$ groups
user docker  # ← Có docker group

# Test
$ docker run hello-world
# ✅ Works!
```

**Automation:**
```bash
# Script để add multiple users
for user in user1 user2 user3; do
    sudo usermod -aG docker $user
done
```

**Security considerations:**
- Users trong docker group có quyền tương đương root
- Chỉ add users đáng tin cậy
- Document security implications

### Result

**Trước:**
- Developers phải dùng sudo
- Security risk
- Inconvenient

**Sau:**
- Developers chạy Docker không cần sudo
- **Zero permission issues** trong 6 tháng
- Better developer experience

### Lesson Learned

1. **Add users to docker group**: Quan trọng cho developers
2. **Security awareness**: Users trong docker group = root privileges
3. **Documentation**: Document setup process
4. **Automation**: Automate user setup

---

## 🎓 TÓM TẮT

### Docker Requirements

**System:**
- Linux (recommended), macOS, Windows
- Kernel >= 3.10
- cgroup, namespace support

**Hardware:**
- CPU: 1+ cores
- RAM: 2GB+ (4GB+ recommended)
- Disk: 20GB+ (50GB+ recommended)

### Docker Installation

**Linux (Ubuntu/Debian):**
- Add Docker repository
- Install docker-ce, containerd
- Post-installation: Add user to docker group

**macOS/Windows:**
- Docker Desktop
- Chạy Linux VM bên trong

### First Container

**Basic commands:**
- `docker run`: Chạy container
- `docker ps`: List containers
- `docker images`: List images
- `docker stop`: Stop container
- `docker rm`: Remove container

**Hello World:**
```bash
$ docker run hello-world
```

**Nginx:**
```bash
$ docker run -d -p 8080:80 nginx
```

### Verify Installation

**Commands:**
- `docker --version`: Check version
- `docker info`: System info
- `docker run hello-world`: Test

**Troubleshooting:**
- Permission denied → Add user to docker group
- Daemon not running → Start docker service
- Cannot pull → Check network, firewall

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ bạn đã:
- ✅ Cài đặt Docker
- ✅ Chạy container đầu tiên
- ✅ Hiểu các commands cơ bản

**Day tiếp theo (Day-007)** sẽ đi sâu vào:
- Docker Images: Pull, Tag, Inspect
- Image management
- Image registry

---

## 📚 TÀI LIỆU THAM KHẢO

- Docker Installation: https://docs.docker.com/engine/install/
- Docker Get Started: https://docs.docker.com/get-started/
- Docker CLI: https://docs.docker.com/engine/reference/commandline/cli/

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-007: Docker-Images-Pull-Tag-Inspect](../Day-007-Docker-Images-Pull-Tag-Inspect/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
