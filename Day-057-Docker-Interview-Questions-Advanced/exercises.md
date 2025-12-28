# Day-057: Docker Interview Questions - Advanced - Exercises

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi hoàn thành các bài tập này, bạn sẽ:
- Trả lời advanced questions
- Troubleshoot complex issues
- Design architectures
- Apply senior-level thinking

---

## 📝 BÀI TẬP 1: Image Optimization

### Yêu cầu

Optimize Dockerfile cho production:

**Current Dockerfile:**
```dockerfile
FROM ubuntu:latest
RUN apt-get update
RUN apt-get install -y python3 python3-pip
COPY . .
RUN pip3 install -r requirements.txt
CMD ["python3", "app.py"]
```

### Câu hỏi

1. **Optimize**: Optimize Dockerfile
2. **Explain**: Explain optimizations
3. **Measure**: Measure improvements

### Deliverables

- Optimized Dockerfile
- Explanations
- Measurements

---

## 📝 BÀI TẬP 2: Troubleshooting Scenario

### Yêu cầu

Debug production issue:

**Scenario:**
- Container crashes intermittently
- No error logs
- Exit code 137
- High memory usage

### Câu hỏi

1. **Debug**: Debug issue
2. **Root cause**: Identify root cause
3. **Fix**: Propose fix

### Deliverables

- Debug process
- Root cause analysis
- Fix proposal

---

## 📝 BÀI TẬP 3: Scaling Design

### Yêu cầu

Design scaling strategy:

**Requirements:**
- Handle 10x traffic increase
- Zero-downtime
- Cost-effective

### Câu hỏi

1. **Design**: Design scaling strategy
2. **Implement**: Implement solution
3. **Test**: Test scaling

### Deliverables

- Scaling design
- Implementation
- Test results

---

## 📝 BÀI TẬP 4: High Availability

### Yêu cầu

Design high availability:

**Requirements:**
- 99.9% uptime
- Automatic recovery
- Data persistence

### Câu hỏi

1. **Design**: Design HA architecture
2. **Implement**: Implement solution
3. **Document**: Document architecture

### Deliverables

- HA design
- Implementation
- Documentation

---

## 🎯 CHECKLIST

- [ ] Đã optimize Dockerfiles
- [ ] Đã troubleshoot issues
- [ ] Đã design scaling
- [ ] Đã design HA

---

**Lưu ý:** Tất cả số liệu trong exercises là illustrative/approximate cho mục đích giáo dục.

