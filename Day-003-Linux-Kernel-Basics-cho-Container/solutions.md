# Day-003: Giải Pháp & Giải Thích

## 🎯 LƯU Ý QUAN TRỌNG

Các giải pháp dưới đây giải thích container ở **kernel level**. Đây là kiến thức nền tảng để hiểu sâu container, không chỉ dùng Docker commands.

Quan trọng là bạn hiểu được:
- **Tại sao** container hoạt động như vậy
- **Cách** namespace và cgroup hoạt động
- **Khi nào** cần hiểu kernel level để debug issues

---

## 📝 BÀI TẬP 1: HIỂU NAMESPACE

### 1.1. PID Namespace

**Tại sao process trong container có thể là PID 1?**

**Giải thích:**
- **PID namespace** tạo một **namespace riêng** cho process IDs
- Mỗi namespace có **PID numbering riêng**, bắt đầu từ 1
- Process đầu tiên trong namespace sẽ có PID 1 (trong namespace đó)
- Nhưng trên host, process đó có thể có PID khác (ví dụ: PID 1000)

**Ví dụ:**
```
Host PID Namespace:
PID 1: systemd
PID 100: process-a (trong container A)
PID 200: process-b (trong container B)

Container A PID Namespace:
PID 1: process-a  # ← Cùng process nhưng PID khác!

Container B PID Namespace:
PID 1: process-b  # ← Cùng process nhưng PID khác!
```

**Tại sao process trong container không thấy processes trên host?**

**Giải thích:**
- **PID namespace isolation**: Processes trong container chỉ thấy processes trong cùng namespace
- Kernel filter process list dựa trên namespace
- Process trong container A không thể thấy processes trong container B hoặc host

**Diagram:**
```
Host Namespace
├── PID 1: systemd
├── PID 100: container-a-process (trong Container A namespace)
└── PID 200: container-b-process (trong Container B namespace)

Container A Namespace (isolated view)
└── PID 1: container-a-process
    # Không thấy systemd, không thấy container-b-process

Container B Namespace (isolated view)
└── PID 1: container-b-process
    # Không thấy systemd, không thấy container-a-process
```

**Senior thinking:**
- PID namespace cho phép mỗi container "nghĩ" nó là root process
- Process có thể là PID 1 trong container nhưng PID 1000 trên host
- Isolation ở kernel level → không thể bypass (trừ khi có bug)

### 1.2. Network Namespace

**Tại sao 2 containers có thể bind cùng port 80?**

**Giải thích:**
- **Network namespace** tạo một **network stack riêng** cho mỗi container
- Mỗi namespace có:
  - Network interfaces riêng
  - IP addresses riêng
  - Port bindings riêng
  - Routing tables riêng
- Port 80 trong namespace A ≠ Port 80 trong namespace B

**Ví dụ:**
```
Host Network Namespace:
eth0: 192.168.1.100
Port 80: Not bound

Container A Network Namespace:
eth0: 172.17.0.2
Port 80: Bound to app-a  # ← OK

Container B Network Namespace:
eth0: 172.17.0.3
Port 80: Bound to app-b  # ← OK (khác namespace!)
```

**Container có thể access network của host không?**

**Có thể, nhưng phải được configure:**
- Mặc định, container có network namespace riêng → **không thể** access host network trực tiếp
- Nhưng có thể:
  - **Host network mode**: Container dùng host network namespace (mất isolation)
  - **Port mapping**: Map container port → host port
  - **Bridge network**: Container connect qua bridge network

**Diagram:**
```
Host Network Namespace
├── eth0: 192.168.1.100
└── docker0: 172.17.0.1 (bridge)

Container A Network Namespace
├── eth0: 172.17.0.2
└── Port 80 → Mapped to Host Port 8080
    # Container có thể access host network qua port mapping

Container B Network Namespace
├── eth0: 172.17.0.3
└── Port 80 → Mapped to Host Port 8081
    # Container có thể access host network qua port mapping
```

**Senior thinking:**
- Network namespace cho phép mỗi container có network stack riêng
- Port conflicts chỉ xảy ra trong cùng namespace
- Host network mode = mất isolation (không nên dùng trừ khi cần)

### 1.3. 7 Loại Namespace

**1. PID Namespace:**
- **Làm gì**: Cô lập process IDs
- **Kết quả**: Mỗi namespace có PID numbering riêng
- **Use case**: Process trong container có thể là PID 1

**2. Network Namespace:**
- **Làm gì**: Cô lập network stack
- **Kết quả**: Mỗi namespace có IP, ports, routing riêng
- **Use case**: Containers có thể bind cùng port

**3. Mount Namespace:**
- **Làm gì**: Cô lập filesystem mount points
- **Kết quả**: Mỗi namespace có filesystem view riêng
- **Use case**: Container có filesystem riêng

**4. UTS Namespace:**
- **Làm gì**: Cô lập hostname và domain name
- **Kết quả**: Mỗi namespace có hostname riêng
- **Use case**: Mỗi container có hostname riêng

**5. IPC Namespace:**
- **Làm gì**: Cô lập IPC resources (shared memory, semaphores)
- **Kết quả**: Processes không thể IPC với processes ngoài namespace
- **Use case**: IPC isolation

**6. User Namespace:**
- **Làm gì**: Map user IDs giữa namespace và host
- **Kết quả**: Process có thể là root trong namespace nhưng non-root trên host
- **Use case**: Security (root processes an toàn hơn)

**7. Cgroup Namespace:**
- **Làm gì**: Cô lập cgroup view
- **Kết quả**: Process chỉ thấy cgroup của nó
- **Use case**: Cgroup isolation

### 1.4. Nếu Container Không Có Namespace

**Vấn đề 1: PID Conflicts**
- Nhiều processes không thể có cùng PID
- Container A: Process PID 1
- Container B: Process PID 1 → **Conflict!**
- **Kết quả**: Không thể chạy nhiều containers

**Vấn đề 2: Network Conflicts**
- Không thể bind cùng port
- Container A: Bind port 80
- Container B: Bind port 80 → **Error: Port already in use**
- **Kết quả**: Không thể chạy nhiều containers cùng port

**Vấn đề 3: Filesystem Conflicts**
- Processes share cùng filesystem view
- Container A: Mount /data
- Container B: Cũng thấy /data → **Conflict!**
- **Kết quả**: Không có filesystem isolation

**Vấn đề 4: Security Risks**
- Processes có thể access resources của processes khác
- Container A có thể kill processes của Container B
- **Kết quả**: Không có security isolation

**Kết luận:**
- **Không có namespace = không có container**
- Namespace là **foundation** của container isolation

---

## 📝 BÀI TẬP 2: HIỂU CGROUP

### 2.1. Tính Toán Cgroup Limits

**Given:**
- Server: 8GB RAM, 4 CPUs
- Container A: Web app (cần 2GB RAM, 1 CPU)
- Container B: Database (cần 4GB RAM, 2 CPUs)
- Container C: Background worker (cần 1GB RAM, 0.5 CPU)

**Calculation:**

**Memory Limits:**
- Container A: 2GB = 2147483648 bytes
- Container B: 4GB = 4294967296 bytes
- Container C: 1GB = 1073741824 bytes
- **Total: 7GB** (còn 1GB cho OS và overhead)

**CPU Limits:**
- Container A: 1 CPU = 100000 / 100000 (quota/period)
- Container B: 2 CPUs = 200000 / 100000
- Container C: 0.5 CPU = 50000 / 100000
- **Total: 3.5 CPUs** (còn 0.5 CPU cho OS và overhead)

**Commands:**
```bash
# Container A
echo 2147483648 > /sys/fs/cgroup/memory/docker/container-a/memory.limit_in_bytes
echo 100000 > /sys/fs/cgroup/cpu/docker/container-a/cpu.cfs_quota_us
echo 100000 > /sys/fs/cgroup/cpu/docker/container-a/cpu.cfs_period_us

# Container B
echo 4294967296 > /sys/fs/cgroup/memory/docker/container-b/memory.limit_in_bytes
echo 200000 > /sys/fs/cgroup/cpu/docker/container-b/cpu.cfs_quota_us
echo 100000 > /sys/fs/cgroup/cpu/docker/container-b/cpu.cfs_period_us

# Container C
echo 1073741824 > /sys/fs/cgroup/memory/docker/container-c/memory.limit_in_bytes
echo 50000 > /sys/fs/cgroup/cpu/docker/container-c/cpu.cfs_quota_us
echo 100000 > /sys/fs/cgroup/cpu/docker/container-c/cpu.cfs_period_us
```

### 2.2. Container A Vượt Quá Memory Limit

**Điều gì sẽ xảy ra:**

1. **OOM Killer được trigger:**
   - Kernel phát hiện Container A vượt quá memory limit
   - OOM killer kill processes trong Container A's cgroup

2. **Process nào bị kill:**
   - Processes trong Container A (trong cgroup của Container A)
   - **KHÔNG** kill processes của Container B, C
   - **KHÔNG** kill host processes

3. **Containers khác:**
   - **KHÔNG bị ảnh hưởng**
   - Vẫn có memory riêng (2GB, 4GB, 1GB)
   - Vẫn chạy bình thường

4. **Host:**
   - **KHÔNG bị ảnh hưởng**
   - Host vẫn có memory (1GB cho OS)
   - Host processes không bị kill

**Ví dụ:**
```
Container A: Limit 2GB, Usage 2.5GB → OOM kill processes trong Container A
Container B: Limit 4GB, Usage 3GB → OK
Container C: Limit 1GB, Usage 0.8GB → OK
Host: 1GB available → OK
```

**Senior thinking:**
- Với cgroup, OOM kills **predictable** và **isolated**
- Chỉ processes trong cgroup bị kill
- Containers khác và host không bị ảnh hưởng

### 2.3. Container B Vượt Quá CPU Limit

**Điều gì sẽ xảy ra:**

1. **Container KHÔNG bị kill:**
   - CPU limit ≠ Memory limit
   - CPU limit chỉ **throttle** (giới hạn), không kill

2. **Performance:**
   - Container B chỉ được dùng tối đa 2 CPUs
   - Nếu cần nhiều hơn → **throttled** (chậm lại)
   - **Latency tăng**, throughput giảm

3. **Containers khác:**
   - **KHÔNG bị ảnh hưởng**
   - Vẫn có CPU riêng (1 CPU, 0.5 CPU)
   - Vẫn chạy bình thường

**Ví dụ:**
```
Container B: Limit 2 CPUs, Need 4 CPUs
→ Chỉ được dùng 2 CPUs
→ Performance giảm 50%
→ Nhưng không bị kill, vẫn chạy
```

**Senior thinking:**
- CPU limit = **throttle**, không kill
- Memory limit = **kill** nếu vượt quá
- Quan trọng để hiểu sự khác biệt

### 2.4. Nếu Không Có Cgroup Limits

**Vấn đề 1: Resource Starvation**
- Một container có thể consume hết resources
- Container B consume 8GB RAM → Containers A, C không có RAM
- **Kết quả**: Containers A, C bị OOM kill (unpredictable)

**Vấn đề 2: Unpredictable OOM Kills**
- Kernel OOM killer kill processes **random** (không chỉ container consume nhiều)
- Có thể kill important processes
- **Kết quả**: Unpredictable behavior, hard to debug

**Vấn đề 3: Performance Degradation**
- Containers ảnh hưởng lẫn nhau
- Container B consume hết CPU → Containers A, C chậm
- **Kết quả**: Performance không predictable

**Vấn đề 4: No Resource Guarantees**
- Không thể đảm bảo container có resources
- Container A cần 2GB nhưng không có guarantee
- **Kết quả**: Không thể plan capacity

**Kết luận:**
- **Không có cgroup = không có resource isolation**
- Cgroup là **bắt buộc** trong production

---

## 📝 BÀI TẬP 3: CONTAINER ESCAPE SCENARIO

### 3.1. Phân Tích Vấn Đề

**Namespace nào bị misconfigured?**

**Mount Namespace** bị misconfigured:
- Container có thể mount host filesystem
- Mount namespace không được isolate đúng cách
- Container có thể access `/host` (host filesystem)

**Tại sao container có thể access host filesystem?**

**Root cause:**
1. **Mount namespace không isolated:**
   - Container share mount namespace với host
   - Hoặc mount namespace được setup sai

2. **Host filesystem được mount vào container:**
   - Có thể do misconfiguration
   - Hoặc do security vulnerability

**Security Risk:**
- **Container escape**: Attacker có thể escape container và access host
- **Data breach**: Access sensitive data trên host
- **Privilege escalation**: Có thể gain root access trên host

### 3.2. Làm Thế Nào Để Fix

**Solution 1: Fix Mount Namespace**
```bash
# Đảm bảo container có Mount namespace riêng
# Docker tự động làm điều này, nhưng cần check config
```

**Solution 2: Security Hardening**
```dockerfile
# Dockerfile
# 1. Read-only root filesystem
FROM alpine
RUN ...
# Read-only rootfs
```

```yaml
# docker-compose.yml hoặc Kubernetes
securityContext:
  readOnlyRootFilesystem: true
  runAsNonRoot: true
  capabilities:
    drop:
      - ALL
    add:
      - NET_BIND_SERVICE  # Chỉ capabilities cần thiết
```

**Solution 3: Don't Mount Host Filesystem**
- Không mount host filesystem vào container
- Dùng volumes thay vì bind mounts
- Hoặc dùng read-only mounts nếu cần

### 3.3. Prevention

**Best Practices:**

1. **Read-only root filesystem:**
   - Container không thể modify filesystem
   - Giảm attack surface

2. **Non-root user:**
   - Chạy với non-root user
   - Giảm privilege

3. **Drop capabilities:**
   - Chỉ giữ capabilities cần thiết
   - Drop tất cả capabilities không cần

4. **Regular security audits:**
   - Check namespace configuration
   - Check mount points
   - Check capabilities

5. **Monitoring:**
   - Alert khi có container escape attempts
   - Track filesystem access patterns

### 3.4. Nếu Attacker Escape Container

**Actions có thể:**

1. **Access sensitive data:**
   - Read files trên host
   - Access databases, secrets
   - Steal credentials

2. **Modify host system:**
   - Install backdoors
   - Modify system files
   - Gain persistence

3. **Lateral movement:**
   - Access other containers
   - Access other servers
   - Expand attack surface

**Mitigation:**
- Defense in depth
- Network segmentation
- Regular security audits
- Incident response plan

---

## 📝 BÀI TẬP 4: OOM KILL TROUBLESHOOTING

### 4.1. Root Cause

**Tại sao containers bị OOM kill?**

**Root cause:**
- **Không có memory limits (cgroup)**
- 10 containers, mỗi container có thể consume unlimited memory
- Server chỉ có 16GB RAM
- Total memory usage > 16GB → OOM!

**Tại sao OOM kill random?**

**Giải thích:**
- **Không có cgroup** → Kernel OOM killer kill processes **random**
- OOM killer không biết container nào consume nhiều memory
- Kill processes dựa trên heuristics (không predictable)
- **Kết quả**: Có thể kill important processes, không chỉ container consume nhiều

**Cgroup nào bị thiếu?**

**Memory cgroup** bị thiếu:
- Không có `memory.limit_in_bytes`
- Containers không bị limit memory
- → OOM kills unpredictable

### 4.2. Làm Thế Nào Để Fix

**Solution: Set Memory Limits**

**Calculate limits:**
- Server: 16GB RAM
- OS overhead: 2GB
- Available: 14GB
- 10 containers → ~1.4GB mỗi container

**Set limits:**
```yaml
# Kubernetes
resources:
  limits:
    memory: "1.5Gi"  # 1.5GB limit
  requests:
    memory: "1Gi"     # 1GB request
```

**Hoặc Docker:**
```bash
$ docker run --memory="1.5g" my-app
```

**Best practices:**
- Set limits dựa trên:
  - Application requirements
  - Historical usage
  - Buffer (20-30% overhead)

### 4.3. Sau Khi Fix

**Nếu một container vượt quá memory limit:**

1. **Process nào bị kill:**
   - **Chỉ** processes trong container đó (trong cgroup của container)
   - **KHÔNG** kill processes của containers khác
   - **KHÔNG** kill host processes

2. **Containers khác:**
   - **KHÔNG bị ảnh hưởng**
   - Vẫn có memory riêng (1.5GB mỗi container)
   - Vẫn chạy bình thường

3. **Có thể predict được:**
   - **CÓ**: OOM kill chỉ affect container vượt quá limit
   - Predictable behavior
   - Dễ debug

**Ví dụ:**
```
Container A: Limit 1.5GB, Usage 1.8GB → OOM kill processes trong Container A
Container B: Limit 1.5GB, Usage 1.2GB → OK
Container C: Limit 1.5GB, Usage 1.4GB → OK
...
# Chỉ Container A bị ảnh hưởng
```

### 4.4. Best Practices

**Monitor Memory Usage:**
```bash
# Docker
$ docker stats

# Kubernetes
$ kubectl top pods
```

**Set Limits Đúng:**
1. **Measure first:**
   - Monitor memory usage trong 1-2 tuần
   - Identify peak usage
   - Set limit = peak usage + 20-30% buffer

2. **Start conservative:**
   - Set limit thấp hơn một chút
   - Monitor, adjust nếu cần
   - Increase nếu thường xuyên OOM

3. **Review regularly:**
   - Review limits mỗi tháng
   - Adjust dựa trên usage patterns
   - Optimize applications nếu cần

---

## 📝 BÀI TẬP 5: TẠO CONTAINER THỦ CÔNG

### 5.1. Các Bước Tạo Container Thủ Công

**Steps:**

1. **Tạo namespaces:**
   - PID namespace
   - Network namespace
   - Mount namespace
   - UTS namespace
   - IPC namespace

2. **Setup cgroups:**
   - Memory limit
   - CPU limit

3. **Chroot:**
   - Chroot vào container filesystem

4. **Mount filesystems:**
   - Mount /proc, /sys, /dev

5. **Start process:**
   - Start application trong namespace

### 5.2. Commands

```bash
# 1. Tạo namespaces
$ unshare --pid --net --mount --uts --ipc --user --fork bash

# 2. Tạo memory cgroup và set limit
$ mkdir -p /sys/fs/cgroup/memory/my-container
$ echo 2147483648 > /sys/fs/cgroup/memory/my-container/memory.limit_in_bytes
$ echo $$ > /sys/fs/cgroup/memory/my-container/cgroup.procs

# 3. Tạo CPU cgroup và set limit
$ mkdir -p /sys/fs/cgroup/cpu/my-container
$ echo 100000 > /sys/fs/cgroup/cpu/my-container/cpu.cfs_quota_us
$ echo 100000 > /sys/fs/cgroup/cpu/my-container/cpu.cfs_period_us
$ echo $$ > /sys/fs/cgroup/cpu/my-container/cgroup.procs

# 4. Chroot vào container filesystem
$ chroot /path/to/container/rootfs /bin/bash

# 5. Mount proc, sys, dev
$ mount -t proc proc /proc
$ mount -t sysfs sysfs /sys
$ mount -t devtmpfs devtmpfs /dev

# Bây giờ bạn đã có một "container"!
```

### 5.3. So Sánh Với Docker

**Docker làm gì tự động:**

1. **Tạo tất cả namespaces:**
   - Docker tự động tạo 7 namespaces
   - Không cần manual setup

2. **Setup cgroups:**
   - Docker tự động tạo cgroups
   - Set limits từ Docker config

3. **Manage filesystem:**
   - Docker tự động setup filesystem
   - Mount volumes, bind mounts

4. **Network setup:**
   - Docker tự động setup network
   - Create bridge, veth pairs

5. **Security:**
   - Docker tự động setup security
   - Capabilities, user namespace

**Tại sao nên dùng Docker:**

1. **Automation:**
   - Tự động làm tất cả
   - Không cần manual setup

2. **Standardization:**
   - Standard format (OCI)
   - Portable

3. **Ecosystem:**
   - Docker Hub, registries
   - Tools, plugins

4. **Security:**
   - Security best practices
   - Regular updates

### 5.4. Nếu Quên Setup Namespace

**Ví dụ: Quên PID namespace**

**Vấn đề:**
- Container không có PID namespace riêng
- Processes trong container share PID namespace với host
- **Kết quả**: PID conflicts, không có isolation

**Ví dụ:**
```
Host: PID 1 = systemd
Container: PID 1 = my-app  # ❌ Conflict! Không thể có 2 PID 1
```

**Ví dụ: Quên Network namespace**

**Vấn đề:**
- Container không có Network namespace riêng
- Container share network với host
- **Kết quả**: Port conflicts, network không isolated

**Ví dụ:**
```
Host: Port 80 = nginx
Container: Port 80 = my-app  # ❌ Conflict! Port already in use
```

**Kết luận:**
- **Mỗi namespace đều quan trọng**
- Thiếu một namespace → mất isolation
- Container không hoạt động đúng

---

## 📝 BÀI TẬP 6: TẠI SAO CONTAINER CHỈ CHẠY TRÊN LINUX?

### 6.1. Giải Thích Ở Kernel Level

**Container cần kernel features:**

1. **Namespaces:**
   - Linux kernel feature (từ kernel 2.6.24+)
   - Windows kernel không có namespaces (có features tương tự nhưng khác)

2. **Cgroups:**
   - Linux kernel feature (từ kernel 2.6.24+)
   - Windows kernel không có cgroups (có Job Objects tương tự)

3. **System calls:**
   - Linux: `unshare()`, `setns()`, `clone()`
   - Windows: Khác hoàn toàn

**Windows kernel không có những features này:**
- Windows kernel architecture khác Linux
- Windows có virtualization features riêng (Hyper-V)
- **Kết quả**: Không thể chạy Linux container trên Windows kernel

**Ví dụ:**
```bash
# Linux container cần Linux system calls
unshare(CLONE_NEWPID | CLONE_NEWNET | ...)  # Linux system call
# Windows kernel không có unshare()
```

### 6.2. Docker Desktop Trên macOS

**Containers thực sự chạy ở đâu?**

**Trong Linux VM:**
- Docker Desktop tạo một **Linux VM** (HyperKit, VirtualBox, hoặc QEMU)
- Containers chạy **trong Linux VM**, không phải macOS
- macOS chỉ là host cho VM

**Tại sao cần Linux VM?**

**macOS không có:**
- Namespaces (Linux kernel feature)
- Cgroups (Linux kernel feature)
- System calls cần thiết

**Solution:**
- Chạy Linux VM trên macOS
- Containers chạy trong Linux VM
- Docker Desktop quản lý VM

**Performance Impact:**
- **Overhead**: VM overhead (~5-10%)
- **Slower**: Chậm hơn native Linux
- **But acceptable**: Cho development

### 6.3. Windows Containers

**Windows containers chạy trên đâu?**

**Trên Windows host:**
- Windows containers cần **Windows kernel**
- Chạy trực tiếp trên Windows (không cần VM)
- Dùng Windows kernel features (Job Objects, etc.)

**Có thể chạy Windows container trên Linux host không?**

**KHÔNG:**
- Windows container cần Windows kernel
- Linux kernel không có Windows kernel features
- **Kết quả**: Không thể chạy Windows container trên Linux

**Workaround:**
- Chạy Windows VM trên Linux host
- Windows container chạy trong Windows VM
- Nhưng không practical

### 6.4. Giải Pháp Chạy Linux Container Trên Windows

**Solution: WSL2 (Windows Subsystem for Linux 2)**

**WSL2 là gì:**
- Linux kernel chạy trong VM trên Windows
- Linux containers chạy trong WSL2 Linux kernel
- **Kết quả**: Có thể chạy Linux containers trên Windows

**Trade-offs:**
- ✅ Có thể chạy Linux containers
- ❌ Performance overhead (VM)
- ❌ Phức tạp hơn native Linux

**Alternative: Docker Desktop:**
- Tương tự WSL2
- Linux VM trên Windows
- Containers chạy trong VM

---

## 📝 BÀI TẬP 7: RESOURCE CONTENTION

### 7.1. Phân Tích Vấn Đề

**Container nào gây ra vấn đề:**

**Container B (Database):**
- CPU: 180% (vượt quá 4 CPUs available)
- Memory: 6.5GB (vượt quá 8GB total)
- **Kết quả**: Consume hết resources

**Tại sao containers khác bị ảnh hưởng:**

**Không có cgroup limits:**
- Container B không bị limit
- Consume hết CPU, memory
- Containers A, C không có resources
- **Kết quả**: Performance degradation, timeouts

**Cgroup nào cần được set:**

1. **Memory cgroup:**
   - Limit Container B memory
   - Đảm bảo containers khác có memory

2. **CPU cgroup:**
   - Limit Container B CPU
   - Đảm bảo containers khác có CPU

### 7.2. Làm Thế Nào Để Fix

**Calculate Limits:**

**Server: 8GB RAM, 4 CPUs**

**Allocation:**
- Container A: 2GB RAM, 1 CPU
- Container B: 4GB RAM, 2 CPUs (limit để không consume hết)
- Container C: 1GB RAM, 0.5 CPU
- OS: 1GB RAM, 0.5 CPU

**Set Limits:**
```yaml
# Container B
resources:
  limits:
    memory: "4Gi"
    cpu: "2"
  requests:
    memory: "3Gi"
    cpu: "1.5"
```

**Kết quả:**
- Container B bị limit 4GB RAM, 2 CPUs
- Không thể consume hết resources
- Containers A, C có resources riêng

### 7.3. Sau Khi Fix

**Container B vẫn consume nhiều nhưng không ảnh hưởng containers khác:**

**Giải thích:**
- **Cgroup limits**: Container B bị limit 4GB RAM, 2 CPUs
- Nếu cần nhiều hơn → **throttled** (CPU) hoặc **OOM kill** (Memory)
- **Nhưng**: Containers A, C có resources riêng (2GB, 1GB RAM)
- **Kết quả**: Containers A, C không bị ảnh hưởng

**Ví dụ:**
```
Container B: Limit 4GB, Need 6GB
→ Chỉ được dùng 4GB
→ OOM kill nếu vượt quá
→ Nhưng Container A (2GB) và C (1GB) vẫn OK
```

### 7.4. Best Practices

**Monitor Resources:**
```bash
# Docker
$ docker stats

# Kubernetes
$ kubectl top pods
$ kubectl describe pod <pod-name>
```

**Set Limits Đúng:**
1. **Measure first**: Monitor usage trước
2. **Set based on requirements**: Dựa trên app requirements
3. **Add buffer**: 20-30% overhead
4. **Review regularly**: Adjust khi cần

---

## 📝 BÀI TẬP 8: SECURITY ANALYSIS

### 8.1. Security Risks

**Risk 1: Root User**
- Container chạy với root (UID 0)
- Nếu bị hack → attacker có root trong container
- **Risk**: High

**Risk 2: No User Namespace**
- Process là root trong container = root trên host
- **Risk**: Container escape → root trên host

**Risk 3: No Capabilities Restrictions**
- Container có tất cả capabilities
- Có thể mount filesystems, modify kernel
- **Risk**: High

**Risk 4: Host Filesystem Access**
- Container có thể mount host filesystem
- **Risk**: Data breach, system compromise

**Risk 5: Network Access**
- Container có thể access host network
- **Risk**: Lateral movement

**Nếu Container Bị Hack:**

1. **Container escape:**
   - Escape container → access host
   - Gain root trên host

2. **Data breach:**
   - Access sensitive data
   - Steal credentials

3. **Lateral movement:**
   - Access other containers
   - Access other servers

### 8.2. Hardening Security

**Solution 1: User Namespace**
```yaml
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
```

**Solution 2: Drop Capabilities**
```yaml
securityContext:
  capabilities:
    drop:
      - ALL
    add:
      - NET_BIND_SERVICE  # Chỉ capabilities cần thiết
```

**Solution 3: Read-only Root Filesystem**
```yaml
securityContext:
  readOnlyRootFilesystem: true
```

**Solution 4: Don't Mount Host Filesystem**
- Dùng volumes thay vì bind mounts
- Hoặc read-only mounts

### 8.3. User Namespace

**User Namespace làm gì:**
- Map user IDs giữa namespace và host
- Process có thể là root (UID 0) trong namespace nhưng non-root trên host

**Tại sao quan trọng:**
- **Security**: Root trong container ≠ root trên host
- Giảm risk nếu container bị hack
- Defense in depth

**Làm thế nào enable:**
```yaml
# Kubernetes
securityContext:
  runAsNonRoot: true
  runAsUser: 1000
```

### 8.4. Capabilities

**Capabilities là gì:**
- Fine-grained permissions
- Thay vì all-or-nothing root

**Drop Capabilities:**
```yaml
securityContext:
  capabilities:
    drop:
      - ALL
    add:
      - NET_BIND_SERVICE  # Chỉ cần bind ports
```

**Ví dụ:**
- Container chỉ cần network → drop `CAP_SYS_ADMIN` (mount filesystems)
- Container chỉ cần bind ports → chỉ add `NET_BIND_SERVICE`
- **Kết quả**: Giảm attack surface

---

## ✅ TỔNG KẾT

Các bài tập này giúp bạn:

1. **Hiểu namespace**: Isolation ở kernel level
2. **Hiểu cgroup**: Resource limits ở kernel level
3. **Debug issues**: Troubleshoot ở kernel level
4. **Security**: Hardening containers
5. **Kernel knowledge**: Hiểu container internals

**Key takeaways:**
- **Container = Namespaces + Cgroups + Chroot**
- **Namespace**: Isolation
- **Cgroup**: Resource limits
- **Security**: Defense in depth
- **Kernel level**: Hiểu internals giúp debug và optimize

---

**Chúc bạn học tốt! Tiếp tục với Day-004 để hiểu Container Runtime & Docker Architecture.**

