# Day-031: Container Health Checks - Exercises

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi hoàn thành các bài tập này, bạn sẽ:
- Define health checks trong Dockerfile
- Define health checks khi run container
- Debug health check issues
- Tune health check parameters
- Áp dụng trong production

---

## 📝 BÀI TẬP 1: Basic Health Check

### Yêu cầu

Tạo Dockerfile với health check:

**Requirements:**
- HTTP health check endpoint
- Interval: 30s
- Timeout: 3s
- Retries: 3

### Câu hỏi

1. **Create Dockerfile với health check**
2. **Test**: Build và run container
3. **Verify**: Check health status

### Deliverables

- Dockerfile
- Test results
- Health status verification

---

## 📝 BÀI TẬP 2: Health Check Options

### Yêu cầu

Tạo health check với different options:

**Scenarios:**
1. Fast detection (10s interval, 1 retry)
2. Stable detection (30s interval, 3 retries)
3. Startup period (10s start-period)

### Câu hỏi

1. **Create health checks với different options**
2. **Compare**: So sánh behavior
3. **Recommend**: Recommend cho different scenarios

### Deliverables

- Health check configurations
- Comparison
- Recommendations

---

## 📝 BÀI TẬP 3: Health Check Types

### Yêu cầu

Tạo health checks cho different types:

**Types:**
1. HTTP endpoint check
2. TCP port check
3. Custom script check

### Câu hỏi

1. **Create health checks cho mỗi type**
2. **Test**: Test mỗi type
3. **Compare**: So sánh use cases

### Deliverables

- Health check examples
- Test results
- Use case comparison

---

## 📝 BÀI TẬP 4: Debug Health Check

### Scenario

**Problem:** Health check luôn fail

**Dockerfile:**
```dockerfile
HEALTHCHECK CMD curl -f http://localhost:8080/health || exit 1
```

### Câu hỏi

1. **Debug**: Tìm root cause
2. **Fix**: Fix health check
3. **Test**: Verify fix

### Deliverables

- Debug analysis
- Fixed health check
- Test results

---

## 📝 BÀI TẬP 5: Production Health Check

### Yêu cầu

Tạo production-ready health check:

**Requirements:**
- Application: Node.js API
- Health endpoint: /health
- Startup time: 10s
- Tolerate network hiccups

### Câu hỏi

1. **Design health check**
2. **Justify**: Giải thích parameters
3. **Implement**: Create Dockerfile

### Deliverables

- Health check design
- Justification
- Dockerfile

---

## 🎯 CHECKLIST

- [ ] Đã define health checks
- [ ] Đã test health checks
- [ ] Đã debug health check issues
- [ ] Đã tune parameters

---

**Lưu ý:** Tất cả số liệu trong exercises là illustrative/approximate cho mục đích giáo dục.

