# Day-021: Docker Networks - Bridge, Host, None - Exercises

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi hoàn thành các bài tập này, bạn sẽ:
- Tạo và quản lý networks
- So sánh các network drivers
- Debug network issues
- Chọn network driver phù hợp

---

## 📝 BÀI TẬP 1: Bridge Network Basics

### Yêu cầu

Tạo container với bridge network:

**Task:**
1. Tạo container với default bridge
2. Inspect network
3. Test port mapping

### Câu hỏi

1. **Tạo container**
2. **Inspect**: Inspect bridge network
3. **Test**: Test port mapping và connectivity

### Deliverables

- Commands
- Network inspection results
- Test results

---

## 📝 BÀI TẬP 2: Custom Bridge Network

### Yêu cầu

Tạo custom bridge network:

**Task:**
1. Tạo custom bridge network
2. Connect containers vào network
3. Test DNS resolution

### Câu hỏi

1. **Create network**
2. **Connect containers**
3. **Test**: Verify DNS resolution

### Deliverables

- Network creation commands
- Container connection commands
- DNS test results

---

## 📝 BÀI TẬP 3: Host Network

### Yêu cầu

Test host network (Linux only):

**Task:**
1. Tạo container với host network
2. Test direct port access
3. Compare với bridge network

### Câu hỏi

1. **Create container với host network**
2. **Test**: Verify direct access
3. **Compare**: So sánh với bridge

### Deliverables

- Commands
- Test results
- Comparison

---

## 📝 BÀI TẬP 4: None Network

### Yêu cầu

Test none network:

**Task:**
1. Tạo container với none network
2. Verify no network connectivity
3. Test use cases

### Câu hỏi

1. **Create container với none network**
2. **Verify**: Verify no network
3. **Use cases**: Identify use cases

### Deliverables

- Commands
- Verification results
- Use case analysis

---

## 📝 BÀI TẬP 5: Network Comparison

### Yêu cầu

So sánh các network drivers:

**Task:**
1. Test bridge, host, none networks
2. Measure performance (nếu có thể)
3. Compare characteristics

### Câu hỏi

1. **Test all networks**
2. **Compare**: Create comparison table
3. **Recommend**: Recommend cho different scenarios

### Deliverables

- Test results
- Comparison table
- Recommendations

---

## 📝 BÀI TẬP 6: Network Troubleshooting

### Scenario

**Problem:** Container không thể access từ host

**Container:**
```bash
$ docker run -d --name web nginx
```

### Câu hỏi

1. **Root cause**: Tại sao không access được?
2. **Fix**: Fix network configuration
3. **Test**: Verify fix

### Deliverables

- Analysis
- Fix commands
- Test results

---

## 📝 BÀI TẬP 7: Production Scenario

### Scenario

**Requirements:**
- High performance application
- Linux environment
- Direct port access needed

### Câu hỏi

1. **Choose network driver**
2. **Justify**: Giải thích choice
3. **Implement**: Create container

### Deliverables

- Network choice
- Justification
- Implementation

---

## 🎯 CHECKLIST

- [ ] Đã tạo và test các network types
- [ ] Đã so sánh network drivers
- [ ] Đã debug network issues
- [ ] Đã chọn network driver phù hợp

---

**Lưu ý:** Tất cả số liệu trong exercises là illustrative/approximate cho mục đích giáo dục.

