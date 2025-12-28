# Day-019: BuildKit & Advanced Build - Exercises

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi hoàn thành các bài tập này, bạn sẽ:
- Enable và sử dụng BuildKit
- Sử dụng build secrets
- Sử dụng cache mounts
- Measure build performance improvements

---

## 📝 BÀI TẬP 1: Enable BuildKit

### Yêu cầu

Enable BuildKit và build image:

**Task:**
1. Enable BuildKit
2. Build image
3. Compare build time với và không có BuildKit

### Câu hỏi

1. **Enable BuildKit**
2. **Measure**: Compare build times
3. **Document**: Document improvements

### Deliverables

- Build commands
- Comparison results
- Documentation

---

## 📝 BÀI TẬP 2: Build Secrets

### Yêu cầu

Sử dụng build secrets trong Dockerfile:

**Requirements:**
- Use secret trong build
- Don't expose secret in image

### Câu hỏi

1. **Create Dockerfile với secrets**
2. **Build**: Build với secrets
3. **Verify**: Verify secret không có trong image

### Deliverables

- Dockerfile
- Build commands
- Verification results

---

## 📝 BÀI TẬP 3: Cache Mounts

### Yêu cầu

Sử dụng cache mounts cho npm install:

**Requirements:**
- Cache npm cache directory
- Faster npm installs

### Câu hỏi

1. **Create Dockerfile với cache mounts**
2. **Test**: Build và measure improvements
3. **Compare**: With và without cache mounts

### Deliverables

- Dockerfile
- Test results
- Comparison

---

## 🎯 CHECKLIST

- [ ] Đã enable BuildKit
- [ ] Đã sử dụng build secrets
- [ ] Đã sử dụng cache mounts
- [ ] Đã measure improvements

---

**Lưu ý:** Tất cả số liệu trong exercises là illustrative/approximate cho mục đích giáo dục.

