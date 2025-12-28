# Day-003: Linux Kernel Basics cho Container

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được container hoạt động ở kernel level như thế nào
- Hiểu được namespace là gì và tại sao quan trọng
- Hiểu được cgroup là gì và cách nó limit resources
- Có thể giải thích container isolation ở mức kernel
- Hiểu được tại sao container chỉ chạy trên Linux (hoặc với Linux kernel)

---

## 📖 PHẦN 1: TẠI SAO CẦN HIỂU KERNEL?

### 1.1. Container không phải "magic"

Nhiều người nghĩ container là "magic" - chỉ cần `docker run` là xong. Nhưng thực tế:

**Container là một tập hợp các Linux kernel features:**
- **Namespaces**: Cô lập processes, network, filesystem
- **Cgroups**: Giới hạn và theo dõi resources
- **Chroot**: Cô lập filesystem
- **Capabilities**: Giới hạn quyền của processes

**Tại sao cần hiểu kernel?**

1. **Debug issues**: Khi container có vấn đề, cần hiểu kernel để debug
2. **Optimize performance**: Hiểu kernel giúp optimize container performance
3. **Security**: Hiểu isolation giúp đánh giá security risks
4. **Senior-level knowledge**: Senior engineers cần hiểu internals

### 1.2. Linux Kernel là gì?

**Linux Kernel** là core của Linux operating system:

```
┌─────────────────────────────────────┐
│     Applications (User Space)        │
│     (Docker, containers, apps)      │
├─────────────────────────────────────┤
│     System Calls Interface          │
├─────────────────────────────────────┤
│     Linux Kernel (Kernel Space)     │
│     - Process management            │
│     - Memory management              │
│     - Filesystem                     │
│     - Network stack                  │
│     - Device drivers                 │
├─────────────────────────────────────┤
│     Hardware                         │
└─────────────────────────────────────┘
```

**Kernel làm gì?**

1. **Process management**: Tạo, kill, schedule processes
2. **Memory management**: Allocate, free memory
3. **Filesystem**: Quản lý files, directories
4. **Network**: Handle network packets
5. **Device drivers**: Interface với hardware

**Container sử dụng kernel features:**
- Namespaces (process management)
- Cgroups (resource management)
- Chroot (filesystem isolation)

---

## 🔒 PHẦN 2: NAMESPACE - ISOLATION CỦA CONTAINER

### 2.1. Namespace là gì?

**Namespace** là một Linux kernel feature cho phép **cô lập (isolate)** các resources giữa các processes.

**Ví dụ đơn giản:**

**Không có namespace:**
```
Process A (PID 1)  ←→  Process B (PID 2)
     ↓                    ↓
  Same PID namespace
  Same network namespace
  Same filesystem
```

**Có namespace:**
```
Container A              Container B
Process A (PID 1)       Process B (PID 1)
     ↓                       ↓
Namespace A              Namespace B
(Isolated)               (Isolated)
```

**Kết quả:**
- Process A và Process B đều có PID 1 (trong namespace riêng)
- Chúng không "nhìn thấy" nhau
- Hoàn toàn isolated

### 2.2. Tại sao namespace tồn tại?

**Vấn đề trước khi có namespace:**

1. **PID conflicts**: Nhiều processes không thể có cùng PID
2. **Network conflicts**: Processes share cùng network stack
3. **Filesystem conflicts**: Processes share cùng filesystem view
4. **Security risks**: Một process có thể access resources của process khác

**Namespace giải quyết:**
- Mỗi namespace có **view riêng** của resources
- Processes trong namespace này không thấy processes trong namespace kia
- **Isolation** ở kernel level

### 2.3. Các loại Namespace

Linux kernel có **7 loại namespace**:

#### 1. **PID Namespace (Process ID)**

**Nó là gì?**
- Cô lập process IDs
- Mỗi namespace có PID numbering riêng

**Ví dụ:**
```bash
# Host
$ ps aux
PID 1: systemd
PID 2: kthreadd
PID 100: my-app

# Container A (PID namespace riêng)
$ ps aux
PID 1: my-app  # ← Cùng process nhưng PID khác!
PID 2: nginx

# Container B (PID namespace riêng)
$ ps aux
PID 1: another-app
PID 2: redis
```

**Tại sao quan trọng?**
- Process trong container có thể là PID 1 (init process)
- Không conflict với host processes
- Container có thể "nghĩ" nó là root process

**Khi nào dùng:**
- Mọi container đều dùng PID namespace
- Cho phép process trong container là PID 1

#### 2. **Network Namespace**

**Nó là gì?**
- Cô lập network stack
- Mỗi namespace có network interfaces, routing tables, firewall rules riêng

**Ví dụ:**
```bash
# Host
$ ip addr
eth0: 192.168.1.100
lo: 127.0.0.1

# Container A (Network namespace riêng)
$ ip addr
eth0: 172.17.0.2  # ← IP riêng
lo: 127.0.0.1

# Container B (Network namespace riêng)
$ ip addr
eth0: 172.17.0.3  # ← IP riêng khác
lo: 127.0.0.1
```

**Tại sao quan trọng?**
- Mỗi container có IP riêng
- Có thể bind cùng port (ví dụ: cả 2 containers bind port 80)
- Network isolation hoàn toàn

**Khi nào dùng:**
- Mọi container đều dùng Network namespace
- Cho phép network isolation

#### 3. **Mount Namespace**

**Nó là gì?**
- Cô lập filesystem mount points
- Mỗi namespace có filesystem tree riêng

**Ví dụ:**
```bash
# Host
$ mount
/dev/sda1 on / type ext4
/dev/sdb1 on /data type ext4

# Container A (Mount namespace riêng)
$ mount
/dev/sda1 on / type ext4
/data on /app/data type bind  # ← Chỉ thấy mount này

# Container B (Mount namespace riêng)
$ mount
/dev/sda1 on / type ext4
# ← Không thấy /data mount
```

**Tại sao quan trọng?**
- Container có filesystem view riêng
- Có thể mount volumes riêng
- Không thấy mounts của containers khác

**Khi nào dùng:**
- Mọi container đều dùng Mount namespace
- Cho phép filesystem isolation

#### 4. **UTS Namespace (Unix Timesharing System)**

**Nó là gì?**
- Cô lập hostname và domain name
- Mỗi namespace có hostname riêng

**Ví dụ:**
```bash
# Host
$ hostname
server-01

# Container A (UTS namespace riêng)
$ hostname
app-container-1  # ← Hostname riêng

# Container B (UTS namespace riêng)
$ hostname
app-container-2  # ← Hostname riêng khác
```

**Tại sao quan trọng?**
- Mỗi container có thể có hostname riêng
- Useful cho service discovery
- Không conflict với host hostname

**Khi nào dùng:**
- Mọi container đều dùng UTS namespace
- Cho phép hostname isolation

#### 5. **IPC Namespace (Inter-Process Communication)**

**Nó là gì?**
- Cô lập IPC resources (shared memory, semaphores, message queues)
- Processes trong namespace này không thể communicate với processes trong namespace kia qua IPC

**Ví dụ:**
```bash
# Host
$ ipcs -m
# Shared memory segments

# Container A (IPC namespace riêng)
$ ipcs -m
# Chỉ thấy shared memory của container A

# Container B (IPC namespace riêng)
$ ipcs -m
# Chỉ thấy shared memory của container B
```

**Tại sao quan trọng?**
- IPC isolation
- Security: Processes không thể access shared memory của processes khác

**Khi nào dùng:**
- Mọi container đều dùng IPC namespace
- Cho phép IPC isolation

#### 6. **User Namespace**

**Nó là gì?**
- Map user IDs giữa namespace và host
- Process có thể là root (UID 0) trong namespace nhưng là non-root trên host

**Ví dụ:**
```bash
# Host
$ id
uid=1000(user) gid=1000(user)

# Container A (User namespace)
# Process chạy với UID 0 (root) trong container
$ id
uid=0(root) gid=0(root)

# Nhưng trên host, process này chạy với UID 1000
# (mapped bởi User namespace)
```

**Tại sao quan trọng?**
- **Security**: Process có thể là root trong container nhưng không có quyền trên host
- Giảm security risk
- Cho phép root processes trong container

**Khi nào dùng:**
- Optional (nhưng recommended)
- Cho phép root processes an toàn hơn

#### 7. **Cgroup Namespace**

**Nó là gì?**
- Cô lập cgroup view
- Process trong namespace chỉ thấy cgroup của nó

**Ví dụ:**
```bash
# Host
$ cat /proc/self/cgroup
0::/system.slice/docker.service

# Container A (Cgroup namespace riêng)
$ cat /proc/self/cgroup
0::/  # ← Chỉ thấy cgroup của container
```

**Tại sao quan trọng?**
- Cgroup isolation
- Process không biết nó bị limit bởi cgroup

**Khi nào dùng:**
- Optional
- Cho phép cgroup isolation

### 2.4. Namespace trong Container

**Container sử dụng tất cả các namespaces:**

```
Container
├── PID Namespace      → Process isolation
├── Network Namespace  → Network isolation
├── Mount Namespace    → Filesystem isolation
├── UTS Namespace      → Hostname isolation
├── IPC Namespace      → IPC isolation
├── User Namespace     → User ID mapping (optional)
└── Cgroup Namespace   → Cgroup isolation (optional)
```

**Kết quả:**
- Container hoàn toàn isolated
- Processes trong container không thấy processes ngoài container
- Network, filesystem, IPC đều isolated

### 2.5. Hậu quả nếu không có Namespace?

**Không có namespace:**

1. **PID conflicts**: Nhiều processes không thể có cùng PID
2. **Network conflicts**: Không thể bind cùng port
3. **Filesystem conflicts**: Processes share cùng filesystem view
4. **Security risks**: Process có thể access resources của process khác
5. **Không có isolation**: Container không thể hoạt động

**Ví dụ:**
```bash
# Không có namespace
Container A: Process bind port 80
Container B: Process bind port 80  # ❌ Error: Port already in use

# Có namespace
Container A: Process bind port 80 (trong Network namespace A)
Container B: Process bind port 80 (trong Network namespace B)
# ✅ OK: Mỗi namespace có network stack riêng
```

---

## ⚙️ PHẦN 3: CGROUP - RESOURCE LIMITS

### 3.1. Cgroup là gì?

**Cgroup (Control Group)** là một Linux kernel feature cho phép **giới hạn và theo dõi** resources (CPU, memory, I/O) của processes.

**Ví dụ đơn giản:**

**Không có cgroup:**
```
Process A: Consume 8GB RAM (toàn bộ server)
Process B: Không có RAM → OOM (Out of Memory)
```

**Có cgroup:**
```
Process A (cgroup A): Limit 2GB RAM
Process B (cgroup B): Limit 2GB RAM
# Cả 2 đều có RAM, không process nào consume hết
```

### 3.2. Tại sao cgroup tồn tại?

**Vấn đề trước khi có cgroup:**

1. **Resource starvation**: Một process có thể consume hết resources
2. **No limits**: Không thể giới hạn CPU, memory của process
3. **No monitoring**: Không biết process dùng bao nhiêu resources
4. **OOM kills**: Kernel kill processes khi hết memory (không predictable)

**Cgroup giải quyết:**
- **Limit resources**: Set CPU, memory limits
- **Monitor usage**: Track resource usage
- **Prevent starvation**: Đảm bảo mọi process có resources
- **Predictable behavior**: OOM kills chỉ affect processes trong cgroup

### 3.3. Các loại Cgroup

#### 1. **CPU Cgroup**

**Nó là gì?**
- Giới hạn và theo dõi CPU usage

**Ví dụ:**
```bash
# Limit container A dùng tối đa 1 CPU core
$ echo 100000 > /sys/fs/cgroup/cpu/docker/container-a/cpu.cfs_quota_us
$ echo 100000 > /sys/fs/cgroup/cpu/docker/container-a/cpu.cfs_period_us
# = 1 CPU core

# Limit container B dùng tối đa 0.5 CPU core
$ echo 50000 > /sys/fs/cgroup/cpu/docker/container-b/cpu.cfs_quota_us
$ echo 100000 > /sys/fs/cgroup/cpu/docker/container-b/cpu.cfs_period_us
# = 0.5 CPU core
```

**Tại sao quan trọng?**
- Prevent một container consume hết CPU
- Đảm bảo fair sharing
- Predictable performance

**Khi nào dùng:**
- Mọi container nên có CPU limits
- Đặc biệt quan trọng trong multi-tenant environments

#### 2. **Memory Cgroup**

**Nó là gì?**
- Giới hạn và theo dõi memory usage

**Ví dụ:**
```bash
# Limit container A dùng tối đa 2GB RAM
$ echo 2147483648 > /sys/fs/cgroup/memory/docker/container-a/memory.limit_in_bytes
# 2GB = 2 * 1024 * 1024 * 1024 bytes

# Nếu container A vượt quá 2GB → OOM kill
```

**Tại sao quan trọng?**
- Prevent một container consume hết memory
- OOM kills chỉ affect container đó (không affect host)
- Predictable memory usage

**Khi nào dùng:**
- Mọi container nên có memory limits
- Critical trong production

#### 3. **I/O Cgroup**

**Nó là gì?**
- Giới hạn và theo dõi disk I/O (read/write bandwidth)

**Ví dụ:**
```bash
# Limit container A: 10MB/s read, 5MB/s write
$ echo "8:0 10485760" > /sys/fs/cgroup/blkio/docker/container-a/blkio.throttle.read_bps_device
$ echo "8:0 5242880" > /sys/fs/cgroup/blkio/docker/container-a/blkio.throttle.write_bps_device
```

**Tại sao quan trọng?**
- Prevent một container consume hết disk I/O
- Đảm bảo fair I/O sharing
- Predictable I/O performance

**Khi nào dùng:**
- Quan trọng cho I/O-intensive applications
- Database containers
- Log processing containers

#### 4. **PIDs Cgroup**

**Nó là gì?**
- Giới hạn số processes trong cgroup

**Ví dụ:**
```bash
# Limit container A có tối đa 100 processes
$ echo 100 > /sys/fs/cgroup/pids/docker/container-a/pids.max
```

**Tại sao quan trọng?**
- Prevent fork bomb
- Limit resource usage (mỗi process tốn memory)

**Khi nào dùng:**
- Optional nhưng recommended
- Đặc biệt quan trọng cho untrusted code

### 3.4. Cgroup trong Container

**Container sử dụng cgroups để limit resources:**

```dockerfile
# Dockerfile
# Limit: 1 CPU, 2GB RAM
```

```bash
# Docker run
$ docker run --cpus="1.0" --memory="2g" my-app
```

**Docker tạo cgroup:**
```
/sys/fs/cgroup/
├── cpu/docker/container-id/
│   ├── cpu.cfs_quota_us = 100000  # 1 CPU
│   └── cpu.cfs_period_us = 100000
├── memory/docker/container-id/
│   ├── memory.limit_in_bytes = 2147483648  # 2GB
│   └── memory.usage_in_bytes = 1073741824  # Current usage
└── blkio/docker/container-id/
    └── blkio.throttle.read_bps_device = ...
```

**Kết quả:**
- Container bị limit CPU, memory, I/O
- Nếu vượt quá limit → throttled hoặc OOM kill
- Host và containers khác không bị ảnh hưởng

### 3.5. Hậu quả nếu không có Cgroup?

**Không có cgroup:**

1. **Resource starvation**: Một container có thể consume hết CPU, memory
2. **No limits**: Không thể predict resource usage
3. **OOM kills unpredictable**: Kernel kill processes random (có thể kill important processes)
4. **Performance degradation**: Containers ảnh hưởng lẫn nhau

**Ví dụ:**
```bash
# Không có cgroup
Container A: Consume 8GB RAM (toàn bộ server)
Container B: Không có RAM → OOM kill
Container C: Không có RAM → OOM kill
# ❌ Tất cả containers bị ảnh hưởng

# Có cgroup
Container A: Limit 2GB RAM → Consume 2GB → Stop
Container B: Limit 2GB RAM → Consume 2GB → OK
Container C: Limit 2GB RAM → Consume 2GB → OK
# ✅ Mỗi container có resources riêng
```

---

## 🔗 PHẦN 4: NAMESPACE + CGROUP = CONTAINER

### 4.1. Container là gì ở kernel level?

**Container = Namespaces + Cgroups + Chroot + Capabilities**

```
Container
├── Namespaces (Isolation)
│   ├── PID Namespace
│   ├── Network Namespace
│   ├── Mount Namespace
│   ├── UTS Namespace
│   ├── IPC Namespace
│   ├── User Namespace (optional)
│   └── Cgroup Namespace (optional)
├── Cgroups (Resource Limits)
│   ├── CPU limit
│   ├── Memory limit
│   ├── I/O limit
│   └── PIDs limit
├── Chroot (Filesystem isolation)
└── Capabilities (Security)
```

**Ví dụ: Tạo container thủ công (không dùng Docker):**

```bash
# 1. Tạo namespaces
$ unshare --pid --net --mount --uts --ipc --user --fork bash

# 2. Tạo cgroup và set limits
$ mkdir /sys/fs/cgroup/memory/my-container
$ echo 2147483648 > /sys/fs/cgroup/memory/my-container/memory.limit_in_bytes
$ echo $$ > /sys/fs/cgroup/memory/my-container/cgroup.procs

# 3. Chroot vào filesystem
$ chroot /path/to/container/rootfs /bin/bash

# 4. Mount proc, sys, dev
$ mount -t proc proc /proc
$ mount -t sysfs sysfs /sys
$ mount -t devtmpfs devtmpfs /dev

# Bây giờ bạn đã có một "container"!
```

**Docker làm tất cả những điều này tự động.**

### 4.2. Tại sao Container chỉ chạy trên Linux?

**Container cần Linux kernel features:**
- Namespaces: Linux kernel feature (từ kernel 2.6.24+)
- Cgroups: Linux kernel feature (từ kernel 2.6.24+)
- Chroot: Linux feature

**Windows containers:**
- Windows có namespace và cgroup riêng (không phải Linux)
- Windows containers chỉ chạy trên Windows host
- Không thể chạy Windows container trên Linux host (vì kernel khác)

**macOS containers:**
- macOS không có namespace, cgroup
- Docker Desktop trên macOS chạy Linux VM bên trong
- Containers thực sự chạy trong Linux VM, không phải macOS

**Kết luận:**
- **Linux containers** chỉ chạy trên Linux kernel
- **Windows containers** chỉ chạy trên Windows kernel
- **macOS**: Phải dùng Linux VM

---

## 🏭 PRODUCTION STORY #1: Container Escape do Namespace Misconfiguration

### Context

**Công ty:** SaaS platform, 200 employees
**Hệ thống:** 100 containers chạy trên Kubernetes
**Traffic:** 500K requests/day
**Team:** 15 DevOps engineers

### Problem

**Tháng 4/2023:**
- Security team phát hiện một container có thể access filesystem của host
- Container escape vulnerability
- Risk: Attacker có thể escape container và access host

**Investigation:**

```bash
# Trong container
$ ls /host
# ❌ Có thể thấy filesystem của host!

# Check namespaces
$ lsns
# Mount namespace không được setup đúng
```

**Root cause:**
- Mount namespace không được isolate đúng cách
- Container có thể mount host filesystem
- Security misconfiguration

### Fix

**Solution:**
1. **Review namespace configuration**
   - Đảm bảo Mount namespace được setup đúng
   - Không mount host filesystem vào container

2. **Security hardening:**
   - Dùng read-only root filesystem
   - Drop capabilities không cần thiết
   - Dùng non-root user

3. **Monitoring:**
   - Setup alerts cho container escape attempts
   - Regular security audits

### Result

**Trước:**
- Container có thể access host filesystem
- Security risk cao

**Sau:**
- Mount namespace isolated đúng cách
- Container không thể access host
- Zero security incidents trong 6 tháng

### Lesson Learned

1. **Namespace configuration quan trọng**: Misconfiguration → security risk
2. **Security audit cần thiết**: Regular check namespace setup
3. **Defense in depth**: Không chỉ dựa vào namespace, cần thêm security layers

---

## 🏭 PRODUCTION STORY #2: OOM Kill do thiếu Memory Cgroup Limits

### Context

**Công ty:** E-commerce platform, 300 employees
**Hệ thống:** 50 containers chạy trên 5 servers
**Traffic:** 1M requests/day
**Team:** 20 backend engineers

### Problem

**Tháng 6/2023:**
- Production incidents: Containers bị OOM kill random
- Một container crash → ảnh hưởng containers khác
- Unpredictable behavior

**Investigation:**

```bash
# Check memory usage
$ docker stats
CONTAINER   MEM USAGE   MEM LIMIT
app-1       8.5GB       /      # ← Không có limit!
app-2       6.2GB       /
app-3       4.1GB       /

# Server có 16GB RAM
# Total: 18.8GB > 16GB → OOM!
```

**Root cause:**
- Containers không có memory limits (cgroup)
- Một container consume hết memory
- Kernel OOM killer kill processes random (không chỉ container đó)
- Ảnh hưởng toàn bộ server

### Fix

**Solution:**
1. **Set memory limits cho tất cả containers:**
   ```yaml
   resources:
     limits:
       memory: "2Gi"
     requests:
       memory: "1Gi"
   ```

2. **Monitor memory usage:**
   - Setup alerts khi memory usage > 80%
   - Track memory trends

3. **Optimize applications:**
   - Fix memory leaks
   - Optimize memory usage

### Result

**Trước:**
- Containers không có memory limits
- OOM kills unpredictable
- 3-4 incidents mỗi tháng

**Sau:**
- Tất cả containers có memory limits
- OOM kills chỉ affect container đó
- Zero incidents trong 3 tháng

### Lesson Learned

1. **Cgroup limits bắt buộc**: Mọi container phải có limits
2. **OOM kills predictable**: Với cgroup, OOM chỉ kill processes trong cgroup
3. **Monitor resources**: Track memory usage để prevent issues

---

## 🎓 TÓM TẮT

### Namespace

**Là gì:**
- Linux kernel feature để isolate resources
- 7 loại: PID, Network, Mount, UTS, IPC, User, Cgroup

**Tại sao quan trọng:**
- Container isolation
- Security
- Prevent conflicts

**Khi nào dùng:**
- Mọi container đều dùng namespaces
- Cho phép isolation hoàn toàn

### Cgroup

**Là gì:**
- Linux kernel feature để limit và monitor resources
- CPU, Memory, I/O, PIDs limits

**Tại sao quan trọng:**
- Prevent resource starvation
- Predictable behavior
- Fair resource sharing

**Khi nào dùng:**
- Mọi container nên có cgroup limits
- Critical trong production

### Container = Namespace + Cgroup

**Container ở kernel level:**
- Namespaces: Isolation
- Cgroups: Resource limits
- Chroot: Filesystem isolation
- Capabilities: Security

**Kết quả:**
- Isolated, limited, secure processes
- Có thể chạy nhiều containers trên một host
- Performance gần như native

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ bạn đã hiểu:
- ✅ Container hoạt động ở kernel level như thế nào
- ✅ Namespace là gì và tại sao quan trọng
- ✅ Cgroup là gì và cách nó limit resources

**Day tiếp theo (Day-004)** sẽ đi sâu vào:
- Container Runtime là gì?
- Docker Architecture (Client, Daemon, Registry)
- Các container runtimes khác (containerd, CRI-O)

---

## 📚 TÀI LIỆU THAM KHẢO

- Linux Namespaces: https://man7.org/linux/man-pages/man7/namespaces.7.html
- Linux Cgroups: https://www.kernel.org/doc/Documentation/cgroup-v1/cgroups.txt
- "Containers from Scratch" - Liz Rice

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.

