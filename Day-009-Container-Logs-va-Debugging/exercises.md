# Day-009: Bài Tập Thực Hành

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi làm các bài tập này, bạn sẽ:

- Xem và quản lý container logs hiệu quả
- Debug container issues
- Configure logging drivers
- Troubleshoot common problems
- Apply best practices cho logging

---

## 📝 BÀI TẬP 1: XEM LOGS

### Scenario

Bạn cần xem logs của container để debug issues.

### Câu hỏi

**1.1.** Basic logs:
- Xem logs của container `my-nginx`
- Giải thích output format
- Logs được lưu ở đâu?

**1.2.** Log options:
- Xem last 50 lines
- Follow logs real-time
- Xem logs với timestamps
- Xem logs từ một thời điểm cụ thể
- Viết commands

**1.3.** Log filtering:
- Filter logs chỉ show errors
- Filter logs chỉ show warnings
- Count số dòng logs
- Viết commands

**1.4.** Multiple containers:
- Xem logs của nhiều containers cùng lúc
- So sánh logs giữa containers
- Viết commands

---

## 📝 BÀI TẬP 2: LOGGING DRIVERS

### Scenario

Bạn cần configure logging drivers cho production.

### Câu hỏi

**2.1.** Default driver:
- Logging driver mặc định là gì?
- Làm thế nào check current driver?
- Logs được lưu ở đâu?

**2.2.** Configure json-file driver:
- Configure log rotation (max-size: 10m, max-file: 3)
- Configure per-container
- Configure daemon-level
- Viết configs

**2.3.** Other drivers:
- Disable logging (none driver)
- Send logs to syslog
- So sánh các drivers
- Khi nào dùng driver nào?

**2.4.** Log rotation:
- Tại sao cần log rotation?
- Configure log rotation
- Test log rotation
- Best practices?

---

## 📝 BÀI TẬP 3: DEBUGGING CONTAINERS

### Scenario

Bạn cần debug container issues.

### Câu hỏi

**3.1.** docker exec:
- Execute shell trong container
- Execute commands trong container
- Check processes
- Check environment variables
- Viết commands

**3.2.** docker inspect:
- Inspect container state
- Check exit code
- Check error message
- Check log path
- Viết commands với format

**3.3.** docker top:
- View running processes
- Check process tree
- Check resource usage
- So sánh với `ps aux` trong container

**3.4.** docker stats:
- Monitor resource usage
- Monitor multiple containers
- Check CPU, memory, I/O
- Use cases?

---

## 📝 BÀI TẬP 4: TROUBLESHOOTING SCENARIOS

### Scenario 1: Container Không Start

Container `my-app` không start được.

### Câu hỏi

**4.1.** Debug steps:
- Làm thế nào debug?
- Commands nào cần chạy?
- Check gì?
- Viết debug script

**4.2.** Common causes:
- Liệt kê 5 common causes
- Làm thế nào identify mỗi cause?
- Làm thế nào fix?

### Scenario 2: Container Crash

Container `my-app` crash mỗi vài giờ.

### Câu hỏi

**4.3.** Debug steps:
- Làm thế nào debug crash?
- Check logs như thế nào?
- Check exit code
- Check resources
- Viết debug process

**4.4.** Root cause analysis:
- Exit code 0 vs != 0
- OOM kill (exit code 137)
- Application errors
- Làm thế nào identify root cause?

### Scenario 3: Performance Issues

Container `my-app` chạy chậm.

### Câu hỏi

**4.5.** Debug steps:
- Check resource usage
- Check processes
- Check logs for errors
- Check network
- Viết commands

**4.6.** Optimization:
- Identify bottlenecks
- Optimize resources
- Best practices?

---

## 📝 BÀI TẬP 5: LOG MANAGEMENT

### Scenario

Bạn cần quản lý logs cho 100+ containers.

### Câu hỏi

**5.1.** Log storage:
- Tính toán log storage cho 100 containers
- Mỗi container: 1GB logs/day
- Làm thế nào manage?
- Log rotation strategy?

**5.2.** Log aggregation:
- Tại sao cần log aggregation?
- Tools nào có thể dùng?
- Setup log aggregation
- Best practices?

**5.3.** Log analysis:
- Parse logs để tìm errors
- Count errors
- Analyze patterns
- Tools và commands?

**5.4.** Log retention:
- Log retention policy
- Làm thế nào implement?
- Compliance requirements?
- Best practices?

---

## 📝 BÀI TẬP 6: PRACTICAL DEBUGGING

### Scenario

Bạn gặp các issues sau và cần debug.

### Câu hỏi

**6.1.** Issue: Container exit immediately
```bash
$ docker run my-app
# Container exit ngay
```
- Debug steps?
- Commands?
- Common causes?

**6.2.** Issue: Container không respond
```bash
$ docker exec my-app curl http://localhost
# Timeout
```
- Debug steps?
- Check gì?
- Fix như thế nào?

**6.3.** Issue: High memory usage
```bash
$ docker stats my-app
# Memory: 95%
```
- Debug steps?
- Identify memory leaks?
- Fix như thế nào?

**6.4.** Issue: Network connectivity
```bash
$ docker exec my-app ping 8.8.8.8
# No response
```
- Debug steps?
- Check network config?
- Fix như thế nào?

---

## 📝 BÀI TẬP 7: LOGGING BEST PRACTICES

### Scenario

Bạn cần setup logging cho production.

### Câu hỏi

**7.1.** Structured logging:
- Tại sao cần structured logging?
- Format nào nên dùng? (JSON, key-value, etc.)
- Làm thế nào implement?
- Examples?

**7.2.** Log levels:
- Log levels nào nên dùng? (DEBUG, INFO, WARN, ERROR)
- Khi nào dùng level nào?
- Configure log levels
- Best practices?

**7.3.** Sensitive data:
- Làm thế nào prevent log sensitive data?
- Passwords, API keys, tokens
- Best practices?
- Tools?

**7.4.** Log monitoring:
- Setup log monitoring
- Alerts cho errors
- Dashboards
- Tools?

---

## 📝 BÀI TẬP 8: ADVANCED DEBUGGING

### Scenario

Bạn cần debug complex issues.

### Câu hỏi

**8.1.** Debug với docker exec:
- Access container filesystem
- Check application configs
- Test connectivity
- Run diagnostic commands
- Viết commands

**8.2.** Debug với docker inspect:
- Inspect container config
- Inspect network settings
- Inspect volume mounts
- Inspect environment variables
- Viết commands với format

**8.3.** Compare containers:
- So sánh 2 containers
- Tìm differences
- Identify issues
- Tools và commands?

**8.4.** Debug production issues:
- Debug without affecting production
- Read-only debugging
- Best practices?
- Safety measures?

---

## ✅ CHECKLIST HOÀN THÀNH

Trước khi xem solutions, đảm bảo bạn đã:

- [ ] Xem logs của containers
- [ ] Debug container issues
- [ ] Configure logging drivers
- [ ] Troubleshoot common problems
- [ ] Làm tất cả các bài tập trên
- [ ] Thực hành các commands trên terminal

---

## 💡 GỢI Ý

- **Practice**: Thực hành debug với real containers
- **Read logs**: Đọc logs để hiểu application behavior
- **Experiment**: Thử các logging drivers khác nhau
- **Document**: Document debugging processes cho team

---

**Chúc bạn làm bài tốt! Sau khi hoàn thành, hãy xem `solutions.md` để so sánh với đáp án.**

