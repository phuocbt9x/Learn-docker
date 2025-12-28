# Day-004: Giải Pháp & Giải Thích

## 🎯 LƯU Ý QUAN TRỌNG

Các giải pháp dưới đây giải thích Docker Architecture và Container Runtime ở mức sâu. Quan trọng là bạn hiểu được:

- **Tại sao** Docker có architecture như vậy
- **Cách** các components tương tác với nhau
- **Khi nào** nên dùng runtime nào

---

## 📝 BÀI TẬP 1: HIỂU DOCKER ARCHITECTURE

### 1.1. Architecture Diagram

```
┌─────────────────────────────────────┐
│     User                              │
│     docker run nginx                  │
├─────────────────────────────────────┤
│     Docker Client (CLI)              │
│     - Parse commands                 │
│     - Send API requests             │
│     - Display results                │
├─────────────────────────────────────┤
│     Docker Daemon (dockerd)          │
│     - Container management           │
│     - Image management              │
│     - Network/Volume management     │
│     - API server (HTTP/Unix socket) │
├─────────────────────────────────────┤
│     containerd (High-level runtime) │
│     - Image management              │
│     - Container lifecycle           │
│     - gRPC API                      │
├─────────────────────────────────────┤
│     runc (Low-level runtime)         │
│     - Create namespaces             │
│     - Setup cgroups                 │
│     - Run container process         │
│     - OCI-compliant                 │
├─────────────────────────────────────┤
│     Linux Kernel                    │
│     - Namespaces                    │
│     - Cgroups                      │
│     - System calls                 │
└─────────────────────────────────────┘
```

**Key Points:**
- **Client**: User interface
- **Daemon**: Management layer
- **containerd**: High-level runtime
- **runc**: Low-level runtime
- **Kernel**: Actual execution

### 1.2. Flow: `docker run nginx`

**Step 1: User Command**
```bash
$ docker run nginx
```

**Step 2: Docker Client**
- Parse command: `docker run nginx`
- Send API request đến Docker Daemon:
  ```http
  POST /containers/create
  Body: { "Image": "nginx" }
  
  POST /containers/{id}/start
  ```

**Step 3: Docker Daemon**
- Receive API request
- Check if image `nginx` exists locally
- If not → pull from registry (Docker Hub)
- Create container configuration
- Call containerd API

**Step 4: containerd**
- Pull image (if not exists)
- Create container from image
- Setup container configuration
- Call runc to create container

**Step 5: runc**
- Create namespaces (PID, Network, Mount, etc.)
- Setup cgroups (CPU, Memory limits)
- Chroot vào container filesystem
- Start container process

**Step 6: Linux Kernel**
- Execute container process
- Apply namespaces isolation
- Apply cgroup limits
- Container running!

**Complete Flow:**
```
User: docker run nginx
  ↓
Client: POST /containers/create
  ↓
Daemon: Check image, create container
  ↓
containerd: Pull image (if needed), create container
  ↓
runc: Setup namespaces, cgroups, start process
  ↓
Kernel: Run container process
  ↓
Container: Running!
```

### 1.3. Docker Daemon là Single Point of Failure

**Tại sao?**

**Docker Daemon quản lý tất cả:**
- Container lifecycle (create, start, stop, delete)
- Image management
- Network, volumes
- **Nếu Daemon crash → không thể manage containers**

**Nếu Docker Daemon crash:**

1. **Containers vẫn chạy:**
   - Containers là processes trong kernel
   - Kernel vẫn chạy processes
   - **Nhưng**: Không thể control (start, stop, logs, exec)

2. **Không thể manage:**
   - `docker ps` → fail (không connect được)
   - `docker stop` → fail
   - `docker logs` → fail
   - `docker exec` → fail

3. **Impact:**
   - **High**: Nếu cần restart container → không thể
   - **High**: Nếu cần debug → không thể xem logs
   - **Medium**: Containers vẫn serve traffic (nếu không cần restart)

**Mitigation:**

1. **Monitor Docker Daemon:**
   ```bash
   # Monitor memory, CPU
   $ systemctl status docker
   $ journalctl -u docker -f
   ```

2. **Auto-restart:**
   ```bash
   # systemd auto-restart
   [Service]
   Restart=always
   RestartSec=5
   ```

3. **Use containerd directly:**
   - Bypass Docker Daemon
   - Dùng containerd (lightweight hơn, ít crash hơn)

4. **High availability:**
   - Multiple Docker hosts
   - Load balancer
   - Container orchestration (Kubernetes)

### 1.4. Docker Client vs Docker Daemon

**Docker Client:**

**Chức năng:**
- Parse commands từ user
- Send API requests đến Docker Daemon
- Display results
- **KHÔNG chạy containers**

**Ví dụ:**
```bash
$ docker run nginx
# Client: Parse "run nginx"
# Client: Send POST /containers/create
# Client: Display container ID
```

**Docker Daemon:**

**Chức năng:**
- Receive API requests
- Manage containers (create, start, stop)
- Manage images (pull, push, build)
- Manage networks, volumes
- **CHẠY containers** (thông qua containerd/runc)

**Ví dụ:**
```bash
# Daemon receive: POST /containers/create
# Daemon: Create container
# Daemon: Call containerd
# Daemon: Return container ID
```

**Communication:**

**Local (default):**
```bash
# Unix socket
/var/run/docker.sock
```

**Remote:**
```bash
# TCP
docker -H tcp://remote-host:2376 run nginx

# SSH
docker -H ssh://user@remote-host run nginx
```

**Có thể chạy Client và Daemon trên máy khác nhau:**

**✅ CÓ THỂ:**
- Docker Client có thể connect đến remote Docker Daemon
- Dùng `-H` flag hoặc `DOCKER_HOST` environment variable

**Ví dụ:**
```bash
# Client trên máy A
$ docker -H tcp://192.168.1.100:2376 run nginx
# Daemon trên máy B (192.168.1.100)
```

**Security:**
- Remote access cần TLS/authentication
- Không nên expose Docker Daemon public (security risk)

---

## 📝 BÀI TẬP 2: CONTAINER RUNTIME

### 2.1. High-level vs Low-level Runtime

**High-level Runtime (containerd):**

**Chức năng:**
- **Image management**: Pull, push, store images
- **Container lifecycle**: Create, start, stop, delete
- **State management**: Track container state
- **API**: gRPC API cho clients

**Ví dụ:**
```bash
# containerd quản lý images
$ ctr images pull nginx
$ ctr containers create nginx my-container
$ ctr containers start my-container
```

**Low-level Runtime (runc):**

**Chức năng:**
- **Kernel interface**: Tạo namespaces, cgroups
- **Process execution**: Chạy container process
- **OCI-compliant**: Tuân thủ OCI spec
- **Lightweight**: Chỉ chạy containers, không quản lý images

**Ví dụ:**
```bash
# runc chỉ chạy container
$ runc create my-container
$ runc start my-container
# runc không pull images, không quản lý state
```

**So sánh:**

| Feature | High-level (containerd) | Low-level (runc) |
|---------|------------------------|------------------|
| **Image management** | ✅ | ❌ |
| **Container lifecycle** | ✅ | ✅ |
| **State management** | ✅ | ❌ |
| **API** | ✅ (gRPC) | ❌ (CLI only) |
| **Kernel interface** | ❌ | ✅ |
| **OCI-compliant** | ⚠️ (uses runc) | ✅ |

### 2.2. Tại sao Docker dùng cả containerd và runc?

**Tại sao không chỉ dùng runc?**

**Runc limitations:**
- **Không quản lý images**: Phải tự pull, store images
- **Không có API**: Chỉ CLI, khó integrate
- **Không quản lý state**: Phải tự track container state
- **Không có networking**: Phải tự setup network

**Ví dụ:**
```bash
# Chỉ dùng runc
$ # Phải tự pull image
$ # Phải tự extract image
$ # Phải tự setup network
$ runc create container
# Phức tạp, không practical
```

**Tại sao không chỉ dùng containerd?**

**Containerd cần low-level runtime:**
- containerd là **high-level**, không interface trực tiếp với kernel
- Cần **low-level runtime** (runc) để:
  - Tạo namespaces
  - Setup cgroups
  - Chạy processes

**Architecture:**
```
containerd (high-level)
    ↓ (calls)
runc (low-level)
    ↓ (system calls)
Kernel
```

**Kết luận:**
- **containerd**: Quản lý images, lifecycle, state
- **runc**: Interface với kernel, chạy containers
- **Cả 2 cần thiết**: containerd quản lý, runc thực thi

### 2.3. OCI (Open Container Initiative)

**OCI là gì?**

**Open Container Initiative** là một standard specification cho containers:
- **Runtime spec**: Cách chạy containers
- **Image spec**: Format của container images
- **Distribution spec**: Cách distribute images

**Tại sao quan trọng?**

1. **Interoperability:**
   - Images có thể chạy trên bất kỳ OCI-compliant runtime
   - Không bị vendor lock-in

2. **Standardization:**
   - Một format cho tất cả
   - Dễ develop tools

3. **Ecosystem:**
   - Nhiều runtimes, tools tuân thủ OCI
   - Dễ integrate

**Runtimes nào tuân thủ OCI?**

**✅ OCI-compliant:**
- **runc**: Reference implementation
- **containerd**: Dùng runc (OCI-compliant)
- **CRI-O**: Dùng runc (OCI-compliant)
- **Podman**: Dùng runc (OCI-compliant)

**❌ Không OCI-compliant:**
- **Docker (old)**: Trước khi OCI ra đời
- **LXC**: Legacy, không OCI

**Ví dụ:**
```bash
# Image build với Docker
$ docker build -t my-app .

# Image có thể chạy trên bất kỳ OCI runtime
$ podman run my-app      # ✅
$ containerd run my-app  # ✅
$ runc run my-app        # ✅
```

### 2.4. Tạo Container Runtime Riêng

**OCI Spec Requirements:**

1. **Runtime Spec:**
   - Create container (namespaces, cgroups)
   - Start container process
   - Stop, delete container
   - State management

2. **Image Spec:**
   - Image format (layers, manifest)
   - Image distribution

3. **Kernel Features:**
   - Namespaces (PID, Network, Mount, etc.)
   - Cgroups (CPU, Memory, I/O)
   - Chroot
   - System calls

**Implementation:**

**Step 1: Implement OCI Runtime Spec**
```go
// Pseudo-code
func CreateContainer(config OCIConfig) {
    // Create namespaces
    unshare(CLONE_NEWPID | CLONE_NEWNET | ...)
    
    // Setup cgroups
    setupCgroup(config.Resources)
    
    // Chroot
    chroot(config.Rootfs)
    
    // Start process
    exec(config.Process.Args)
}
```

**Step 2: Implement Image Management**
- Pull images từ registry
- Store images locally
- Extract image layers

**Step 3: Implement API**
- gRPC API (như containerd)
- Hoặc REST API (như Docker)

**Step 4: Testing**
- Test với OCI test suite
- Verify OCI compliance

**Challenges:**
- **Complexity**: Rất phức tạp
- **Security**: Phải handle security đúng cách
- **Performance**: Phải optimize
- **Maintenance**: Phải maintain

**Recommendation:**
- **Không nên** tạo runtime riêng trừ khi có lý do đặc biệt
- **Nên dùng** existing runtimes (runc, containerd)
- **Nếu cần customize**: Fork và modify existing runtime

---

## 📝 BÀI TẬP 3: SO SÁNH CÁC RUNTIMES

### Use Case 1: Development Environment

**Recommendation: Docker hoặc Podman**

**Lý do:**
1. **Docker-compatible commands**: Developers quen với Docker
2. **Full features**: Network, volumes, etc.
3. **Ecosystem**: Nhiều tools, plugins

**Lý do chọn Docker:**
1. ✅ Docker-compatible commands
2. ✅ Full features (network, volumes)
3. ✅ Large ecosystem

**Lý do chọn Podman:**
1. ✅ Rootless (không cần root)
2. ✅ Daemonless (nhẹ hơn)
3. ✅ Docker-compatible (alias)

**Limitations:**

**Docker:**
1. ❌ Cần root (trừ rootless mode - mới)
2. ❌ Cần daemon (overhead)

**Podman:**
1. ❌ Ecosystem nhỏ hơn Docker
2. ❌ Một số features không có

**Trade-offs:**
- **Docker**: Full features nhưng cần root/daemon
- **Podman**: Rootless/daemonless nhưng features ít hơn

### Use Case 2: Kubernetes Production

**Recommendation: containerd hoặc CRI-O**

**Lý do:**
1. **Kubernetes-native**: Designed cho Kubernetes
2. **Performance**: Lightweight, fast
3. **Security**: Security-focused

**Lý do chọn containerd:**
1. ✅ Similar to Docker (easier migration)
2. ✅ Production-ready
3. ✅ Good performance

**Lý do chọn CRI-O:**
1. ✅ Kubernetes-native
2. ✅ Lightweight
3. ✅ Security-focused

**Limitations:**

**containerd:**
1. ⚠️ CLI khác Docker (ctr, crictl)
2. ⚠️ Một số Docker features không có

**CRI-O:**
1. ❌ Chỉ cho Kubernetes (không dùng standalone)
2. ❌ Ecosystem nhỏ hơn

**Trade-offs:**
- **containerd**: Easier migration từ Docker
- **CRI-O**: More Kubernetes-native nhưng ít flexible

### Use Case 3: CI/CD Pipeline

**Recommendation: containerd**

**Lý do:**
1. **Lightweight**: Không cần Docker Daemon overhead
2. **Fast startup**: Nhanh hơn Docker
3. **OCI-compliant**: Standard format

**Lý do:**
1. ✅ Lightweight (no Docker Daemon)
2. ✅ Fast startup
3. ✅ OCI-compliant

**Limitations:**
1. ⚠️ CLI khác Docker (ctr)
2. ⚠️ Một số features không có

**Trade-offs:**
- **containerd**: Lightweight, fast
- **Docker**: Full features nhưng nặng hơn

### Use Case 4: Embedded System

**Recommendation: Podman**

**Lý do:**
1. **Daemonless**: Không cần background service
2. **Rootless**: Không cần root
3. **Lightweight**: Ít resources

**Lý do:**
1. ✅ Daemonless (no overhead)
2. ✅ Rootless (security)
3. ✅ Lightweight

**Limitations:**
1. ⚠️ Ecosystem nhỏ hơn Docker
2. ⚠️ Một số features không có

**Trade-offs:**
- **Podman**: Lightweight, rootless
- **Docker**: Full features nhưng nặng, cần root

---

## 📝 BÀI TẬP 4: DOCKER DAEMON TROUBLESHOOTING

### 4.1. Phân Tích Vấn Đề

**Tại sao Docker Client không connect được?**

**Root cause:**
- **Docker Daemon không chạy** hoặc **crash**
- Client không thể connect đến Daemon socket

**Containers có còn chạy không?**

**✅ CÓ:**
- Containers là **processes trong kernel**
- Kernel vẫn chạy processes
- **Nhưng**: Không thể control (start, stop, logs)

**Có thể control containers không?**

**❌ KHÔNG:**
- `docker ps` → fail
- `docker stop` → fail
- `docker logs` → fail
- `docker exec` → fail

**Vì sao?**
- Tất cả commands cần Docker Daemon
- Daemon không chạy → không thể control

### 4.2. Debug

**Check Docker Daemon Status:**
```bash
# systemd
$ systemctl status docker

# Check if running
$ ps aux | grep dockerd
```

**Check Logs:**
```bash
# systemd logs
$ journalctl -u docker -n 100

# Docker logs
$ cat /var/log/docker.log
```

**Check Resources:**
```bash
# Memory
$ free -h

# CPU
$ top

# Disk
$ df -h
```

**Check Socket:**
```bash
# Unix socket
$ ls -la /var/run/docker.sock

# Test connection
$ curl --unix-socket /var/run/docker.sock http://localhost/version
```

### 4.3. Fix

**Restart Docker Daemon:**
```bash
# Restart
$ systemctl restart docker

# Verify
$ systemctl status docker
$ docker ps
```

**Verify Containers:**
```bash
# Check containers
$ docker ps -a

# Containers có thể đã stop (nếu không có restart policy)
# Cần restart containers
$ docker start <container-id>
```

**Prevent trong tương lai:**

1. **Monitor Daemon:**
   ```bash
   # Setup monitoring
   - Track memory, CPU usage
   - Alert khi Daemon down
   ```

2. **Auto-restart:**
   ```bash
   # systemd auto-restart
   [Service]
   Restart=always
   RestartSec=5
   ```

3. **Resource limits:**
   ```bash
   # Limit Daemon resources
   # (nếu có memory leak)
   ```

4. **Use containerd:**
   - Bypass Docker Daemon
   - Dùng containerd (lightweight hơn)

### 4.4. Nếu Containers Down

**Restore:**

1. **Restart Docker Daemon:**
   ```bash
   $ systemctl restart docker
   ```

2. **Restart Containers:**
   ```bash
   # Nếu có restart policy
   $ docker ps -a
   $ docker start <container-id>
   ```

3. **Recover State:**
   - Containers state có thể mất
   - Cần recreate nếu cần

**Có thể recover containers không?**

**⚠️ CÓ THỂ (một phần):**
- Containers processes đã stop → không thể recover
- **Nhưng**: Có thể restart containers (nếu có restart policy)
- **Nhưng**: State có thể mất (files, data)

**Best Practices:**

1. **Restart Policy:**
   ```yaml
   restart: unless-stopped
   ```

2. **Persistent Storage:**
   - Dùng volumes cho data
   - Data không mất khi container restart

3. **High Availability:**
   - Multiple Docker hosts
   - Load balancer
   - Container orchestration

4. **Backup:**
   - Backup container configurations
   - Backup volumes

---

## 📝 BÀI TẬP 5: MIGRATION SCENARIO

### 5.1. Migration Plan

**Phase 1: Preparation (Week 1-2)**
- Research containerd
- Setup test cluster
- Train team
- Create migration scripts

**Phase 2: Testing (Week 3-4)**
- Test applications trên test cluster
- Verify functionality
- Performance testing
- Fix issues

**Phase 3: Pilot (Week 5-6)**
- Migrate 5 non-critical nodes
- Monitor closely
- Document learnings
- Adjust plan

**Phase 4: Gradual Migration (Week 7-10)**
- Migrate 10-15 nodes mỗi tuần
- Monitor, verify
- Rollback nếu cần

**Phase 5: Completion (Week 11-12)**
- Migrate remaining nodes
- Remove Docker
- Documentation
- Post-migration review

**Timeline: 12 tuần (3 tháng)**

### 5.2. Risk Analysis

**Risk 1: Application Incompatibility**
- **Risk**: Apps không chạy trên containerd
- **Mitigation**: Test kỹ trên test cluster

**Risk 2: Team Skills**
- **Risk**: Team không có kinh nghiệm containerd
- **Mitigation**: Training, documentation

**Risk 3: Performance Issues**
- **Risk**: Performance degradation
- **Mitigation**: Benchmark before/after

**Risk 4: Downtime**
- **Risk**: Migration gây downtime
- **Mitigation**: Gradual migration, rollback plan

**Risk 5: Data Loss**
- **Risk**: Containers state mất
- **Mitigation**: Backup, persistent storage

### 5.3. Testing Strategy

**Test Cases:**

1. **Basic Functionality:**
   - Container start/stop
   - Image pull/push
   - Network connectivity
   - Volume mounts

2. **Application Testing:**
   - Test tất cả applications
   - Verify functionality
   - Performance testing

3. **Edge Cases:**
   - Container crash
   - Resource limits
   - Network issues

4. **Integration:**
   - Kubernetes integration
   - CI/CD pipelines
   - Monitoring tools

### 5.4. Rollback Plan

**Nếu Migration Fail:**

1. **Immediate Rollback:**
   - Revert node về Docker
   - Restart containers
   - Verify functionality

2. **Zero Downtime:**
   - Migrate từng node
   - Rollback node nếu có issues
   - Không affect toàn bộ cluster

3. **Communication:**
   - Alert team
   - Document issues
   - Adjust plan

### 5.5. containerd vs CRI-O

**containerd:**

**Pros:**
- ✅ Similar to Docker (easier migration)
- ✅ Production-ready
- ✅ Good performance
- ✅ Can use standalone

**Cons:**
- ⚠️ Not Kubernetes-native
- ⚠️ CLI khác Docker

**CRI-O:**

**Pros:**
- ✅ Kubernetes-native
- ✅ Lightweight
- ✅ Security-focused

**Cons:**
- ❌ Chỉ cho Kubernetes
- ❌ Ecosystem nhỏ hơn

**Recommendation: containerd**

**Lý do:**
- Easier migration từ Docker
- Production-ready
- Can use standalone (flexible)
- Good performance

---

## 📝 BÀI TẬP 6: DOCKER REGISTRY

### 6.1. Registry Architecture

**Self-hosted vs Cloud:**

**Self-hosted (Harbor, GitLab Registry):**
- ✅ Full control
- ✅ No vendor lock-in
- ❌ Maintenance overhead
- ❌ Need infrastructure

**Cloud (AWS ECR, GCP GCR):**
- ✅ Managed service
- ✅ High availability
- ❌ Vendor lock-in
- ❌ Cost

**Recommendation:**
- **Small team**: Cloud (AWS ECR)
- **Large team**: Self-hosted (Harbor)

**Single vs Multiple:**

**Single Registry:**
- ✅ Simple
- ❌ Single point of failure

**Multiple Registries:**
- ✅ High availability
- ✅ Geographic distribution
- ❌ Complexity

**Recommendation:**
- Start với single registry
- Scale to multiple nếu cần

**Backup Strategy:**
- Regular backups
- Replication to secondary registry
- Disaster recovery plan

### 6.2. Security

**Authentication:**
- Username/password
- Token-based (JWT)
- OAuth2

**Authorization:**
- Role-based access control (RBAC)
- Who can push/pull
- Image-level permissions

**Example (Harbor):**
```yaml
# Users
- admin: full access
- developer: push/pull
- ci: pull only

# Projects
- project-a: team-a only
- project-b: team-b only
```

### 6.3. Performance

**Optimize Pull/Push:**

1. **Caching:**
   - Registry cache
   - CDN for images

2. **Compression:**
   - Image compression
   - Layer deduplication

3. **Geographic Distribution:**
   - Multiple registries
   - CDN

**Caching Strategy:**
- Local registry cache
- CDN for popular images
- Pre-pull images

### 6.4. So Sánh Options

**Docker Hub (Private):**
- ✅ Easy setup
- ❌ Limited features
- ❌ Cost

**Harbor:**
- ✅ Full features
- ✅ Self-hosted
- ❌ Maintenance

**GitLab Registry:**
- ✅ Integrated với GitLab
- ✅ Good features
- ❌ GitLab only

**AWS ECR:**
- ✅ Managed service
- ✅ High availability
- ❌ Vendor lock-in

**Recommendation:**
- **Small team**: AWS ECR
- **Large team**: Harbor

---

## 📝 BÀI TẬP 7: CONTAINERD STANDALONE

### 7.1. Setup containerd

**Installation:**
```bash
# Ubuntu/Debian
$ apt install containerd

# Or from source
$ wget https://github.com/containerd/containerd/releases/...
```

**Configuration:**
```bash
# Generate default config
$ containerd config default > /etc/containerd/config.toml

# Edit config
$ vi /etc/containerd/config.toml
```

**CLI Tools:**
```bash
# ctr (containerd CLI)
$ ctr images pull nginx
$ ctr containers create nginx my-container
$ ctr containers start my-container

# crictl (Kubernetes CLI)
$ crictl pull nginx
$ crictl run nginx my-container
```

### 7.2. So Sánh với Docker

**Commands:**

| Docker | containerd (ctr) |
|-------|------------------|
| `docker pull nginx` | `ctr images pull nginx` |
| `docker run nginx` | `ctr containers create nginx c1`<br>`ctr containers start c1` |
| `docker ps` | `ctr containers list` |
| `docker logs` | `ctr containers logs c1` |

**Features:**

| Feature | Docker | containerd |
|---------|--------|-----------|
| **Image management** | ✅ | ✅ |
| **Network management** | ✅ | ⚠️ (limited) |
| **Volume management** | ✅ | ⚠️ (limited) |
| **CLI** | ✅ (docker) | ⚠️ (ctr, different) |

**Performance:**
- containerd: **5-10% faster** (no Docker Daemon overhead)
- containerd: **20% less memory**

### 7.3. Use Cases

**Dùng containerd khi:**
- Kubernetes
- CI/CD (lightweight)
- Embedded systems
- Khi không cần Docker features

**Dùng Docker khi:**
- Development
- Khi cần full features
- Khi team quen Docker

### 7.4. Migration

**Steps:**

1. **Install containerd:**
   ```bash
   $ apt install containerd
   ```

2. **Migrate images:**
   ```bash
   # Export từ Docker
   $ docker save nginx > nginx.tar
   
   # Import vào containerd
   $ ctr images import nginx.tar
   ```

3. **Update scripts:**
   - Replace Docker commands với ctr
   - Update CI/CD pipelines

**Challenges:**
- CLI khác Docker
- Một số features không có
- Team cần training

---

## 📝 BÀI TẬP 8: ROOTLESS CONTAINERS

### 8.1. Rootless Containers

**Là gì:**
- Containers chạy **không cần root**
- User namespace mapping
- Security tốt hơn

**Tại sao quan trọng:**
- **Security**: Nếu container bị hack → không có root
- **Compliance**: Một số environments yêu cầu rootless
- **Best practice**: Defense in depth

### 8.2. Runtimes Support Rootless

**Docker (rootless mode):**
- ✅ Support (new)
- ⚠️ Setup phức tạp
- ⚠️ Một số features không có

**Podman:**
- ✅ Native rootless
- ✅ Easy setup
- ✅ Full features

**containerd (rootless):**
- ✅ Support
- ⚠️ Setup phức tạp

**So sánh:**
- **Podman**: Best cho rootless (native)
- **Docker**: Có thể nhưng phức tạp
- **containerd**: Có thể nhưng phức tạp

### 8.3. Setup Rootless

**Podman:**
```bash
# Install
$ apt install podman

# Run (không cần root)
$ podman run nginx
```

**Docker (rootless):**
```bash
# Install rootless Docker
$ curl -fsSL https://get.docker.com/rootless | sh

# Run
$ docker run nginx
```

**Requirements:**
- User namespace support
- cgroup v2 (cho một số features)
- Kernel >= 4.18

### 8.4. Production Use Cases

**Có nên dùng rootless trong production?**

**✅ NÊN (trong một số cases):**
- Multi-tenant environments
- Untrusted code
- Compliance requirements

**⚠️ KHÔNG NÊN (trong một số cases):**
- Khi cần full kernel features
- Khi performance critical
- Khi có compatibility issues

**Trade-offs:**
- ✅ Security tốt hơn
- ❌ Một số features không có
- ❌ Performance overhead nhỏ

**Best Practices:**
- Dùng rootless khi có thể
- Test kỹ trước khi production
- Monitor performance

---

## ✅ TỔNG KẾT

Các bài tập này giúp bạn:

1. **Hiểu Docker Architecture**: Components và flow
2. **Hiểu Container Runtime**: High-level vs low-level
3. **So sánh Runtimes**: Chọn đúng runtime cho use case
4. **Troubleshoot**: Debug Docker Daemon issues
5. **Migration**: Plan migration từ Docker sang containerd

**Key takeaways:**
- **Docker Architecture**: Client → Daemon → containerd → runc → Kernel
- **Container Runtime**: High-level (containerd) + Low-level (runc)
- **Runtimes**: Docker, containerd, CRI-O, Podman - mỗi cái có use case riêng
- **Production**: Monitor Daemon, consider alternatives

---

**Chúc bạn học tốt! Tiếp tục với Day-005 để hiểu Image vs Container - Layers & Filesystem.**

