# Day-054: Docker Internals - Deep Dive - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Docker Architecture

**Components:**
- Docker Client: CLI
- Docker Daemon: Background service
- containerd: Runtime
- runc: OCI runtime

**Flow:**
Client → Daemon → containerd → runc → Container

---

## ✅ BÀI TẬP 2: Storage Analysis

**Storage location:**
- `/var/lib/docker/`: Default
- `/var/lib/docker/overlay2/`: overlay2 layers

**Structure:**
- Layers: Image layers
- Containers: Container layers
- Metadata: Image metadata

---

## ✅ BÀI TẬP 3: Internal Debugging

**Bottleneck:** Storage driver

**Fix:** Switch to overlay2

**Results:**
- Performance improved
- Operations faster

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

