# Day-049: Permission & Filesystem Issues

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được permission issues
- Biết cách debug filesystem problems
- Hiểu được volume permissions
- Biết cách fix permission issues
- Debug filesystem issues
- Áp dụng trong production

---

## 🔐 PHẦN 1: PERMISSION ISSUES

### 1.1. Permission Problems

**Common issues:**
- **Permission denied**: Cannot access files
- **Wrong ownership**: Files owned by wrong user
- **Volume permissions**: Volume permission issues

### 1.2. Debug Permissions

**Check permissions:**
```bash
$ docker exec <container> ls -la /path
# Check file permissions
```

**Check ownership:**
```bash
$ docker exec <container> stat /path
# Check file ownership
```

**Check user:**
```bash
$ docker exec <container> whoami
# Check current user
```

### 1.3. Fix Permissions

**Fix ownership:**
```dockerfile
COPY --chown=user:group file /path
```

**Fix trong container:**
```bash
$ docker exec <container> chown user:group /path
```

---

## 🏭 PRODUCTION STORY: Permission Issues

### Context

**Công ty:** E-commerce, 800 employees
**Issue:** Application cannot write files
**Root cause:** Permission issues

### Fix

**Solution: Fix permissions**
```dockerfile
RUN addgroup -g 1000 appuser && \
    adduser -D -u 1000 -G appuser appuser
USER appuser
COPY --chown=appuser:appuser . /app
```

**Results:**
- Permissions correct
- Application works
- No permission errors

---

## 🎓 TÓM TẮT

**Permission issues:**
- Check permissions
- Fix ownership
- Use correct user

**Filesystem issues:**
- Volume permissions
- File ownership
- Access rights

---

## 🚀 BƯỚC TIẾP THEO

**Day tiếp theo (Day-050)** sẽ đi sâu vào:
- Performance Bottleneck Analysis
- Performance debugging

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.

