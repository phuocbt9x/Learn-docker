# Day-004: Container Runtime & Docker Architecture

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được Container Runtime là gì và tại sao quan trọng
- Hiểu được Docker Architecture (Client, Daemon, Registry)
- Biết các Container Runtimes khác (containerd, CRI-O, runc)
- Hiểu được Docker components hoạt động như thế nào
- Có thể giải thích Docker internals cho team

---

## 📖 PHẦN 1: CONTAINER RUNTIME LÀ GÌ?

### 1.1. Nó là gì?

**Container Runtime** là phần mềm chịu trách nhiệm **chạy containers**. Nó quản lý:

- **Container lifecycle**: Create, start, stop, delete containers
- **Image management**: Pull, push, store images
- **Resource management**: CPU, memory, I/O limits (cgroups)
- **Isolation**: Namespaces, security
- **Networking**: Container networks
- **Storage**: Volumes, bind mounts

**Ví dụ đơn giản:**

```
User: docker run nginx
    ↓
Container Runtime: Tạo container từ image nginx
    ↓
Linux Kernel: Chạy container (namespace, cgroup)
```

**Container Runtime không phải Docker:**
- Docker là một **container platform** (bao gồm runtime + tools)
- Container Runtime là **component** của Docker
- Có nhiều container runtimes khác (containerd, CRI-O, Podman)

### 1.2. Tại sao Container Runtime tồn tại?

**Vấn đề trước khi có Container Runtime:**

1. **Manual setup**: Phải tạo namespace, cgroup thủ công
2. **Complexity**: Rất phức tạp để quản lý containers
3. **No standardization**: Mỗi người làm khác nhau
4. **No tooling**: Không có tools để manage containers

**Container Runtime giải quyết:**
- **Automation**: Tự động tạo namespace, cgroup
- **Standardization**: Standard interface (OCI - Open Container Initiative)
- **Tooling**: CLI, API để manage containers
- **Abstraction**: Ẩn complexity của kernel features

### 1.3. Khi nào cần hiểu Container Runtime?

**Use cases:**

1. **Debug container issues:**
   - Container không start
   - Container crash
   - Performance issues

2. **Optimize performance:**
   - Tối ưu container startup time
   - Tối ưu resource usage

3. **Security:**
   - Hiểu container isolation
   - Đánh giá security risks

4. **Integration:**
   - Tích hợp với orchestration (Kubernetes)
   - Custom container runtime

5. **Troubleshooting:**
   - Debug runtime issues
   - Understand error messages

### 1.4. Hậu quả nếu không hiểu Container Runtime?

**Hậu quả:**

1. **Khó debug**: Không hiểu tại sao container fail
2. **Khó optimize**: Không biết cách tối ưu
3. **Security risks**: Không hiểu isolation → misconfiguration
4. **Dependency issues**: Phụ thuộc vào một runtime (vendor lock-in)

---

## 🐳 PHẦN 2: DOCKER ARCHITECTURE

### 2.1. Docker Components

Docker có **3 components chính**:

```
┌─────────────────────────────────────┐
│     Docker Client (CLI)             │
│     docker run, docker build, ...  │
├─────────────────────────────────────┤
│     Docker Daemon (dockerd)        │
│     - Container management         │
│     - Image management             │
│     - Network management           │
├─────────────────────────────────────┤
│     Container Runtime              │
│     - containerd                   │
│     - runc                         │
├─────────────────────────────────────┤
│     Linux Kernel                   │
│     - Namespaces                   │
│     - Cgroups                      │
└─────────────────────────────────────┘
```

### 2.2. Docker Client

**Nó là gì?**

**Docker Client** là command-line interface (CLI) để interact với Docker.

**Ví dụ:**
```bash
$ docker run nginx
$ docker build -t my-app .
$ docker ps
```

**Chức năng:**
- Parse commands từ user
- Send requests đến Docker Daemon
- Display results

**Architecture:**
```
User → docker run nginx
    ↓
Docker Client (CLI)
    ↓ (HTTP/Unix socket)
Docker Daemon
```

**Lưu ý:**
- Docker Client **không chạy containers**
- Chỉ là interface để communicate với Docker Daemon
- Có thể chạy trên máy khác (remote Docker)

### 2.3. Docker Daemon (dockerd)

**Nó là gì?**

**Docker Daemon** là service chạy trên host, chịu trách nhiệm:

1. **Container management:**
   - Create, start, stop, delete containers
   - Manage container lifecycle

2. **Image management:**
   - Pull, push, build images
   - Store images locally

3. **Network management:**
   - Create networks
   - Manage container networking

4. **Volume management:**
   - Create, manage volumes
   - Mount volumes vào containers

**Architecture:**
```
Docker Client
    ↓ (API requests)
Docker Daemon (dockerd)
    ↓
Container Runtime (containerd)
    ↓
Linux Kernel
```

**Lưu ý:**
- Docker Daemon chạy như một **background service**
- Listen trên Unix socket hoặc TCP port
- Có thể remote access (nhưng cần security)

### 2.4. Container Runtime trong Docker

**Docker sử dụng 2 runtimes:**

#### 1. **containerd**

**Nó là gì?**
- **High-level runtime**: Quản lý container lifecycle, images
- **Daemon**: Chạy như background service
- **API**: gRPC API

**Chức năng:**
- Pull, push, store images
- Create, start, stop containers
- Manage container state
- Interface với low-level runtime (runc)

**Architecture:**
```
Docker Daemon
    ↓
containerd (high-level runtime)
    ↓
runc (low-level runtime)
    ↓
Linux Kernel
```

#### 2. **runc**

**Nó là gì?**
- **Low-level runtime**: Chạy containers ở kernel level
- **OCI-compliant**: Tuân thủ OCI (Open Container Initiative) spec
- **Lightweight**: Chỉ chạy containers, không quản lý images

**Chức năng:**
- Tạo namespaces
- Setup cgroups
- Chạy container process
- Interface trực tiếp với kernel

**Architecture:**
```
containerd
    ↓
runc
    ↓ (system calls)
Linux Kernel (namespace, cgroup)
```

**Lưu ý:**
- runc là **reference implementation** của OCI
- Nhiều runtimes khác cũng dùng runc (CRI-O, Podman)

### 2.5. Docker Registry

**Nó là gì?**

**Docker Registry** là nơi **lưu trữ và phân phối** Docker images.

**Ví dụ:**
- **Docker Hub**: Public registry (docker.io)
- **Private Registry**: Self-hosted (Harbor, GitLab Registry)
- **Cloud Registries**: AWS ECR, GCP GCR, Azure ACR

**Architecture:**
```
Docker Client
    ↓ docker pull nginx
Docker Daemon
    ↓
Docker Registry (docker.io)
    ↓
Image layers
```

**Chức năng:**
- Store images
- Distribute images
- Version control (tags)
- Authentication, authorization

### 2.6. Docker Architecture Flow

**Ví dụ: `docker run nginx`**

```
1. User: docker run nginx
    ↓
2. Docker Client: Parse command, send API request
    ↓
3. Docker Daemon: Receive request
    ↓
4. Docker Daemon: Check if image exists locally
    ↓ (nếu không có)
5. Docker Daemon: Pull image from Registry
    ↓
6. containerd: Create container from image
    ↓
7. runc: Setup namespaces, cgroups
    ↓
8. Linux Kernel: Run container process
    ↓
9. Container: Running!
```

**Chi tiết từng bước:**

**Step 1-2: Client → Daemon**
```bash
$ docker run nginx
# Client gửi HTTP request đến Docker Daemon
POST /containers/create
POST /containers/{id}/start
```

**Step 3-4: Daemon checks image**
```bash
# Daemon check local image store
# Nếu không có → pull từ registry
```

**Step 5: Pull image**
```bash
# Daemon pull image từ Docker Hub
GET /v2/nginx/manifests/latest
GET /v2/nginx/blobs/{digest}
```

**Step 6-7: Create container**
```bash
# containerd tạo container
# runc setup namespaces, cgroups
```

**Step 8-9: Run container**
```bash
# Kernel chạy container process
# Container running!
```

---

## 🔧 PHẦN 3: CÁC CONTAINER RUNTIMES KHÁC

### 3.1. containerd (Standalone)

**Nó là gì?**

**containerd** có thể chạy **standalone** (không cần Docker).

**Architecture:**
```
User → ctr (containerd CLI)
    ↓
containerd daemon
    ↓
runc
    ↓
Linux Kernel
```

**So sánh với Docker:**

| Feature | Docker | containerd |
|---------|--------|------------|
| **CLI** | docker | ctr, crictl |
| **Image management** | ✅ | ✅ |
| **Network management** | ✅ | ⚠️ (limited) |
| **Volume management** | ✅ | ⚠️ (limited) |
| **Ecosystem** | Large | Smaller |
| **Use case** | Development, production | Kubernetes, CI/CD |

**Khi nào dùng containerd standalone:**
- Kubernetes (dùng containerd thay vì Docker)
- CI/CD pipelines (lightweight)
- Embedded systems

### 3.2. CRI-O

**Nó là gì?**

**CRI-O** là container runtime được thiết kế cho **Kubernetes**.

**Architecture:**
```
Kubernetes
    ↓ (CRI - Container Runtime Interface)
CRI-O
    ↓
runc
    ↓
Linux Kernel
```

**Đặc điểm:**
- **Kubernetes-native**: Designed cho Kubernetes
- **Lightweight**: Chỉ những gì Kubernetes cần
- **OCI-compliant**: Tuân thủ OCI spec
- **Security-focused**: Security best practices

**So sánh với Docker:**

| Feature | Docker | CRI-O |
|---------|--------|-------|
| **Kubernetes support** | ⚠️ (deprecated) | ✅ Native |
| **CLI** | docker | crictl |
| **Image management** | ✅ | ✅ |
| **Use case** | General purpose | Kubernetes only |

**Khi nào dùng CRI-O:**
- Kubernetes clusters
- Khi không cần Docker features
- Security-focused environments

### 3.3. Podman

**Nó là gì?**

**Podman** là **daemonless** container runtime (không cần daemon).

**Architecture:**
```
User → podman run nginx
    ↓ (direct)
runc
    ↓
Linux Kernel
```

**Đặc điểm:**
- **No daemon**: Chạy trực tiếp, không cần background service
- **Rootless**: Có thể chạy không cần root
- **Docker-compatible**: Có thể dùng Docker commands (alias)

**So sánh với Docker:**

| Feature | Docker | Podman |
|---------|--------|--------|
| **Daemon** | ✅ Required | ❌ No daemon |
| **Rootless** | ⚠️ (new) | ✅ Native |
| **Docker-compatible** | ✅ | ✅ (alias) |
| **Use case** | General purpose | Development, rootless |

**Khi nào dùng Podman:**
- Development environments
- Rootless containers
- Khi không muốn daemon

### 3.4. So Sánh Các Runtimes

| Runtime | Daemon | Rootless | Kubernetes | Use Case |
|---------|--------|----------|-------------|----------|
| **Docker** | ✅ | ⚠️ | ⚠️ (deprecated) | General purpose |
| **containerd** | ✅ | ⚠️ | ✅ | Kubernetes, CI/CD |
| **CRI-O** | ✅ | ⚠️ | ✅ | Kubernetes only |
| **Podman** | ❌ | ✅ | ⚠️ | Development, rootless |

**Recommendation:**
- **Development**: Docker hoặc Podman
- **Kubernetes**: containerd hoặc CRI-O
- **CI/CD**: containerd (lightweight)
- **Rootless**: Podman

---

## 🏭 PRODUCTION STORY #1: Docker Daemon Crash tại Production

### Context

**Công ty:** SaaS platform, 300 employees
**Hệ thống:** 200 containers chạy trên 10 servers
**Traffic:** 2M requests/day
**Team:** 20 DevOps engineers

### Problem

**Tháng 5/2023:**
- Production server #5: Docker Daemon crash
- **Tất cả containers trên server đó down**
- Zero downtime requirement → Critical incident

**Timeline:**
- **10:00 AM**: Docker Daemon crash
- **10:01 AM**: All containers down
- **10:02 AM**: Alerts triggered
- **10:05 AM**: Team investigate
- **10:15 AM**: Restart Docker Daemon
- **10:20 AM**: Containers restart
- **10:25 AM**: Service restored

**Impact:**
- **25 minutes downtime**
- **50,000 requests failed**
- **Customer complaints**

### Investigation

**Root cause:**
```bash
# Docker Daemon logs
$ journalctl -u docker
...
Out of memory: Killed process dockerd
```

**Phân tích:**
- Docker Daemon consume hết memory
- Server có 16GB RAM
- 200 containers metadata trong memory
- **Memory leak** trong Docker Daemon

**Why all containers down?**
- **Docker Daemon quản lý tất cả containers**
- Daemon crash → không thể manage containers
- Containers vẫn chạy nhưng không thể control (start, stop, logs)

### Fix

**Solution 1: Immediate (Restart Daemon)**
```bash
$ systemctl restart docker
# Containers tự động restart (nếu có restart policy)
```

**Solution 2: Long-term (Prevent)**
1. **Monitor Docker Daemon:**
   - Track memory usage
   - Alert khi > 80%

2. **Limit Daemon resources:**
   - Set memory limit cho Docker Daemon
   - Restart policy

3. **Upgrade Docker:**
   - Fix memory leak trong newer version

4. **Use containerd directly:**
   - Bypass Docker Daemon
   - Dùng containerd (lightweight hơn)

### Result

**Trước:**
- Docker Daemon không được monitor
- Memory leak không được detect
- Single point of failure

**Sau:**
- Monitor Docker Daemon memory
- Alert khi memory > 80%
- Auto-restart policy
- Zero incidents trong 6 tháng

### Lesson Learned

1. **Docker Daemon là single point of failure**: Nếu crash → tất cả containers affected
2. **Monitor Daemon**: Quan trọng như monitor containers
3. **Consider alternatives**: containerd standalone có thể tốt hơn cho production

---

## 🏭 PRODUCTION STORY #2: Migration từ Docker sang containerd

### Context

**Công ty:** E-commerce platform, 500 employees
**Hệ thống:** Kubernetes cluster với Docker runtime
**Traffic:** 5M requests/day
**Team:** 30 DevOps engineers

### Problem

**Tháng 8/2023:**
- Kubernetes deprecate Docker runtime
- Cần migrate sang containerd hoặc CRI-O
- **Timeline**: 3 tháng

**Challenges:**
1. **200+ nodes** cần migrate
2. **Zero downtime** requirement
3. **Team không có kinh nghiệm** với containerd

### Investigation

**Options:**

**Option 1: CRI-O**
- ✅ Kubernetes-native
- ✅ Lightweight
- ❌ Team không có kinh nghiệm
- ❌ Migration phức tạp hơn

**Option 2: containerd**
- ✅ Similar to Docker (dùng cùng runc)
- ✅ Easier migration
- ✅ Team có thể học nhanh
- ✅ Production-ready

**Decision: containerd**

### Fix

**Migration Plan:**

**Phase 1: Testing (Week 1-2)**
- Setup test cluster với containerd
- Test applications
- Train team

**Phase 2: Pilot (Week 3-4)**
- Migrate 10 non-critical nodes
- Monitor, fix issues
- Document learnings

**Phase 3: Gradual Migration (Week 5-10)**
- Migrate 20-30 nodes mỗi tuần
- Monitor closely
- Rollback plan

**Phase 4: Completion (Week 11-12)**
- Migrate remaining nodes
- Remove Docker
- Documentation

**Implementation:**
```bash
# 1. Install containerd
$ apt install containerd

# 2. Configure containerd
$ containerd config default > /etc/containerd/config.toml

# 3. Update Kubernetes
$ kubeadm upgrade node --runtime=containerd

# 4. Verify
$ crictl ps
```

### Result

**Trước (Docker):**
- Docker runtime (deprecated)
- Heavier (Docker Daemon overhead)
- Maintenance phức tạp

**Sau (containerd):**
- containerd runtime (supported)
- Lighter (no Docker Daemon)
- Easier maintenance
- **Zero downtime migration**

**Benefits:**
- **Performance**: 5-10% better (lighter runtime)
- **Resource usage**: 20% less memory
- **Maintenance**: Easier (simpler architecture)

### Lesson Learned

1. **Docker deprecation**: Kubernetes không support Docker runtime nữa
2. **containerd là lựa chọn tốt**: Similar to Docker, easier migration
3. **Gradual migration**: Phân chia phases giảm risk
4. **Training quan trọng**: Team cần học containerd trước

---

## 🎓 TÓM TẮT

### Container Runtime

**Là gì:**
- Phần mềm chạy containers
- Quản lý lifecycle, resources, isolation

**Tại sao quan trọng:**
- Automation
- Standardization
- Tooling

**Các runtimes:**
- Docker (general purpose)
- containerd (Kubernetes, CI/CD)
- CRI-O (Kubernetes only)
- Podman (daemonless, rootless)

### Docker Architecture

**Components:**
1. **Docker Client**: CLI interface
2. **Docker Daemon**: Background service
3. **Container Runtime**: containerd + runc
4. **Docker Registry**: Image storage

**Flow:**
```
Client → Daemon → containerd → runc → Kernel
```

**Lưu ý:**
- Docker Daemon là single point of failure
- containerd có thể chạy standalone
- runc là low-level runtime (OCI-compliant)

### Khi Nào Dùng Runtime Nào?

**Docker:**
- Development
- General purpose
- Khi cần full features

**containerd:**
- Kubernetes
- CI/CD
- Production (lightweight)

**CRI-O:**
- Kubernetes only
- Security-focused

**Podman:**
- Development
- Rootless containers
- Daemonless

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ bạn đã hiểu:
- ✅ Container Runtime là gì
- ✅ Docker Architecture
- ✅ Các runtimes khác

**Day tiếp theo (Day-005)** sẽ đi sâu vào:
- Image vs Container
- Layers & Filesystem
- Image layering mechanism

---

## 📚 TÀI LIỆU THAM KHẢO

- Docker Architecture: https://docs.docker.com/get-started/overview/
- containerd: https://containerd.io/
- OCI Specification: https://opencontainers.org/

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-005: Image-vs-Container-Layers-va-Filesystem](../Day-005-Image-vs-Container-Layers-va-Filesystem/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
