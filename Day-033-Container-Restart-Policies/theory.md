# Day-033: Container Restart Policies

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được restart policies là gì
- Biết được các restart policy types
- Hiểu được khi nào dùng policy nào
- Biết cách configure restart policies
- Debug restart issues
- Áp dụng trong production

---

## 🔄 PHẦN 1: RESTART POLICIES

### 1.1. Restart Policy là gì?

**Restart Policy** là **cơ chế Docker tự động restart** container khi container exit.

**Purpose:**
- **Automatic recovery**: Tự động recover khi container fail
- **High availability**: Đảm bảo service available
- **Production readiness**: Critical cho production

### 1.2. Restart Policy Types

**Types:**
- **no**: Không restart (default)
- **always**: Luôn restart
- **on-failure**: Restart khi exit code != 0
- **unless-stopped**: Restart trừ khi manually stopped

### 1.3. Configure Restart Policy

**Dockerfile:**
```dockerfile
# Không có restart policy trong Dockerfile
# Set khi run container
```

**Runtime:**
```bash
$ docker run --restart=always app
$ docker run --restart=on-failure:5 app
$ docker run --restart=unless-stopped app
```

---

## 🏭 PRODUCTION STORY: Container Not Restarting

### Context

**Công ty:** E-commerce, 800 employees
**Issue:** Container không restart sau khi crash
**Root cause:** No restart policy

### Fix

**Solution: Restart policy**
```bash
$ docker run -d --restart=always app
```

**Results:**
- Automatic recovery
- High availability
- Production-ready

---

## 🎓 TÓM TẮT

**Restart policies:**
- no: Không restart
- always: Luôn restart
- on-failure: Restart khi fail
- unless-stopped: Restart trừ khi stopped

**Best practices:**
- Use always cho production
- Use on-failure cho development
- Monitor restarts

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-034)** sẽ đi sâu vào:
- Logging Strategies cho Production
- Centralized logging

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.

