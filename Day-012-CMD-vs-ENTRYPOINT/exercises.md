# Day-012: CMD vs ENTRYPOINT - Exercises

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi hoàn thành các bài tập này, bạn sẽ:
- Viết được Dockerfile với CMD/ENTRYPOINT đúng cách
- Hiểu được cách override CMD/ENTRYPOINT
- Phân tích được khi nào dùng CMD, khi nào dùng ENTRYPOINT
- Debug được các vấn đề liên quan đến CMD/ENTRYPOINT
- Tối ưu được Dockerfile với CMD/ENTRYPOINT

---

## 📝 BÀI TẬP 1: CMD Basics

### Yêu cầu

Tạo Dockerfile cho Python application với:
- Base image: `python:3.9-slim`
- Copy file `app.py` vào `/app/app.py`
- CMD chạy `python /app/app.py` (exec form)

### Câu hỏi

1. **Tại sao dùng exec form thay vì shell form?**
2. **Làm thế nào để override CMD khi run container?**
3. **Test override CMD**: Chạy container với command `python --version`

### Deliverables

- Dockerfile
- Commands để build và run
- Commands để override CMD

---

## 📝 BÀI TẬP 2: ENTRYPOINT Basics

### Yêu cầu

Tạo Dockerfile cho nginx với:
- Base image: `nginx:alpine`
- ENTRYPOINT: `nginx -g daemon off;` (exec form)
- Không có CMD

### Câu hỏi

1. **Tại sao dùng ENTRYPOINT thay vì CMD?**
2. **Làm thế nào để override ENTRYPOINT?**
3. **Test override ENTRYPOINT**: Chạy container với `bash` để debug

### Deliverables

- Dockerfile
- Commands để build và run
- Commands để override ENTRYPOINT

---

## 📝 BÀI TẬP 3: CMD vs ENTRYPOINT Comparison

### Yêu cầu

Tạo 2 Dockerfiles cho cùng một application (Python app):

**Dockerfile 1: CMD only**
```dockerfile
FROM python:3.9-slim
COPY app.py /app/app.py
CMD ["python", "/app/app.py"]
```

**Dockerfile 2: ENTRYPOINT only**
```dockerfile
FROM python:3.9-slim
COPY app.py /app/app.py
ENTRYPOINT ["python", "/app/app.py"]
```

### Câu hỏi

1. **So sánh cách override:**
   - Override CMD: `docker run my-app bash`
   - Override ENTRYPOINT: `docker run --entrypoint bash my-app`
   - Ghi lại kết quả

2. **Khi nào dùng CMD? Khi nào dùng ENTRYPOINT?**
3. **Production scenario**: Nếu muốn prevent accidental override, dùng gì?

### Deliverables

- 2 Dockerfiles
- Comparison table
- Production recommendation

---

## 📝 BÀI TẬP 4: CMD + ENTRYPOINT Together

### Yêu cầu

Tạo Dockerfile cho curl tool với:
- Base image: `alpine:latest`
- Install curl: `apk add --no-cache curl`
- ENTRYPOINT: `["curl"]`
- CMD: `["-s", "https://example.com"]`

### Câu hỏi

1. **Command cuối cùng khi run container là gì?**
2. **Làm thế nào để override CMD?**
   - Test: `docker run my-curl -I https://google.com`
3. **Làm thế nào để override ENTRYPOINT?**
   - Test: `docker run --entrypoint bash my-curl`

### Deliverables

- Dockerfile
- Commands để test override
- Giải thích behavior

---

## 📝 BÀI TẬP 5: Signal Handling

### Yêu cầu

Tạo 2 Dockerfiles cho Python app:

**Dockerfile 1: Shell form CMD**
```dockerfile
FROM python:3.9-slim
COPY app.py /app/app.py
CMD python /app/app.py
```

**Dockerfile 2: Exec form CMD**
```dockerfile
FROM python:3.9-slim
COPY app.py /app/app.py
CMD ["python", "/app/app.py"]
```

**app.py:**
```python
import signal
import sys
import time

def signal_handler(sig, frame):
    print('Received SIGTERM, shutting down gracefully...')
    sys.exit(0)

signal.signal(signal.SIGTERM, signal_handler)

print('Application started')
while True:
    time.sleep(1)
```

### Câu hỏi

1. **Test signal handling:**
   - Build cả 2 images
   - Run container: `docker run -d --name test1 my-app:shell`
   - Stop container: `docker stop test1`
   - Check logs: `docker logs test1`
   - Repeat với exec form

2. **So sánh kết quả:**
   - Shell form: Có nhận SIGTERM không?
   - Exec form: Có nhận SIGTERM không?
   - Giải thích sự khác biệt

3. **Production impact**: Nếu không handle signals, hậu quả là gì?

### Deliverables

- 2 Dockerfiles
- app.py với signal handling
- Test results và analysis

---

## 📝 BÀI TẬP 6: Wrapper Script

### Yêu cầu

Tạo Dockerfile với wrapper script:

**entrypoint.sh:**
```bash
#!/bin/sh
echo "Starting application..."
echo "Environment: $ENV"
echo "Database: $DB_HOST"

# Initialization code
# ...

# Exec main application
exec python /app/app.py
```

**Dockerfile:**
- Base image: `python:3.9-slim`
- Copy `entrypoint.sh` và `app.py`
- ENTRYPOINT: `["/entrypoint.sh"]`
- Không có CMD

### Câu hỏi

1. **Tại sao dùng wrapper script?**
2. **Tại sao dùng `exec` trong script?**
3. **Test**: Chạy container và verify initialization

### Deliverables

- Dockerfile
- entrypoint.sh
- Commands để test

---

## 📝 BÀI TẬP 7: Practical Dockerfile

### Yêu cầu

Tạo Dockerfile cho Node.js application với:

**Requirements:**
- Base image: `node:18-alpine`
- Copy `package.json` và `package-lock.json`
- Run `npm install`
- Copy source code
- Expose port 3000
- **Decision**: Dùng CMD hay ENTRYPOINT? Giải thích

**app.js:**
```javascript
const http = require('http');
const server = http.createServer((req, res) => {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello World\n');
});
server.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### Câu hỏi

1. **Chọn CMD hay ENTRYPOINT? Tại sao?**
2. **Viết Dockerfile hoàn chỉnh**
3. **Test**: Build, run, và verify application

### Deliverables

- Dockerfile
- Giải thích decision
- Commands để test

---

## 📝 BÀI TẬP 8: Troubleshooting

### Scenario

**Dockerfile:**
```dockerfile
FROM python:3.9-slim
COPY app.py /app/app.py
CMD python /app/app.py
```

**Problem:**
- Container không shutdown gracefully
- `docker stop` mất > 10 seconds
- Application không nhận SIGTERM

### Câu hỏi

1. **Root cause là gì?**
2. **Fix Dockerfile**
3. **Test fix**: Verify graceful shutdown

### Deliverables

- Fixed Dockerfile
- Explanation
- Test results

---

## 📝 BÀI TẬP 9: Production Analysis

### Scenario

**Company:** E-commerce platform
**Application:** Python microservice
**Current Dockerfile:**
```dockerfile
FROM python:3.9-slim
COPY requirements.txt /app/
RUN pip install -r /app/requirements.txt
COPY app.py /app/app.py
CMD ["python", "/app/app.py"]
```

**Requirements:**
- Application phải shutdown gracefully
- Developers cần có thể debug (override command)
- Production không được chạy wrong command

### Câu hỏi

1. **Phân tích current Dockerfile:**
   - Ưu điểm?
   - Nhược điểm?
   - Production risks?

2. **Recommendation:**
   - Giữ CMD hay đổi ENTRYPOINT?
   - Cần wrapper script không?
   - Trade-offs?

3. **Refactor Dockerfile** theo recommendation

### Deliverables

- Analysis document
- Refactored Dockerfile
- Justification

---

## 📝 BÀI TẬP 10: Advanced - Multiple Commands

### Yêu cầu

**Scenario:** Application cần chạy migration trước khi start

**Options:**
1. **Option 1:** CMD với shell script
   ```dockerfile
   CMD ["sh", "-c", "python migrate.py && python app.py"]
   ```

2. **Option 2:** ENTRYPOINT với wrapper script
   ```dockerfile
   ENTRYPOINT ["/entrypoint.sh"]
   ```

3. **Option 3:** CMD với separate script
   ```dockerfile
   CMD ["/start.sh"]
   ```

### Câu hỏi

1. **So sánh 3 options:**
   - Signal handling?
   - Override behavior?
   - Maintainability?
   - Production readiness?

2. **Recommendation:** Option nào tốt nhất? Tại sao?

3. **Implement** option được recommend

### Deliverables

- Comparison table
- Recommended Dockerfile
- Justification

---

## 🎯 CHECKLIST

Trước khi submit, đảm bảo:

- [ ] Đã viết Dockerfile với CMD/ENTRYPOINT đúng syntax
- [ ] Đã test override behavior
- [ ] Đã test signal handling
- [ ] Đã phân tích trade-offs
- [ ] Đã giải thích decisions
- [ ] Đã test trong production-like scenario

---

## 📚 HINTS

1. **Exec form vs Shell form:**
   - Exec form: `["command", "arg1"]`
   - Shell form: `command arg1`
   - Luôn dùng exec form cho production

2. **Override:**
   - CMD: `docker run image new-command`
   - ENTRYPOINT: `docker run --entrypoint new-command image`

3. **Signal handling:**
   - Exec form → Application là PID 1 → Handle signals tốt
   - Shell form → Shell là PID 1 → Không handle signals tốt

4. **Wrapper scripts:**
   - Dùng `exec` để application là PID 1
   - `exec python app.py` thay vì `python app.py`

---

**Lưu ý:** Tất cả số liệu trong exercises là illustrative/approximate cho mục đích giáo dục.

