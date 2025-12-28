# Day-012: CMD vs ENTRYPOINT - Khi nào dùng gì?

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được sự khác biệt giữa CMD và ENTRYPOINT
- Biết khi nào dùng CMD, khi nào dùng ENTRYPOINT
- Hiểu được cách CMD và ENTRYPOINT tương tác với nhau
- Biết cách override CMD và ENTRYPOINT khi run container
- Viết được Dockerfile với CMD/ENTRYPOINT đúng cách

---

## 📖 PHẦN 1: CMD INSTRUCTION

### 1.1. CMD là gì?

**CMD** là instruction chỉ định **default command** chạy khi container start.

**Syntax:**
```dockerfile
# Shell form
CMD command param1 param2

# Exec form (recommended)
CMD ["executable", "param1", "param2"]
```

**Ví dụ:**
```dockerfile
CMD ["nginx", "-g", "daemon off;"]
CMD ["python", "app.py"]
CMD echo "Hello World"
```

**Đặc điểm:**
- **Default command**: Chạy khi container start
- **Overrideable**: Có thể override khi `docker run`
- **Only one CMD**: Chỉ có một CMD (CMD cuối cùng được dùng)
- **Not executed during build**: Chỉ chạy khi container run

### 1.2. CMD Shell Form vs Exec Form

**Shell form:**
```dockerfile
CMD python app.py
```

**Đặc điểm:**
- Chạy trong shell (`/bin/sh -c`)
- Có thể dùng shell features (variables, pipes, etc.)
- **PID 1**: Shell process là PID 1 (không handle signals tốt)

**Exec form:**
```dockerfile
CMD ["python", "app.py"]
```

**Đặc điểm:**
- Chạy trực tiếp executable
- **Không có shell** → không dùng shell features
- **PID 1**: Application là PID 1 (handle signals tốt)

**Recommendation:**
- **Luôn dùng exec form** cho CMD
- **Better signal handling**: SIGTERM được handle đúng
- **Production-ready**: Recommended cho production

### 1.3. CMD Override

**CMD có thể override khi run container:**

```dockerfile
# Dockerfile
CMD ["python", "app.py"]
```

```bash
# Override CMD
$ docker run my-app bash
# Chạy bash thay vì python app.py

$ docker run my-app python --version
# Chạy python --version thay vì python app.py
```

**Khi nào override?**
- Debugging: Chạy shell để debug
- Testing: Chạy commands khác
- Development: Override để test

### 1.4. Multiple CMD

**Chỉ CMD cuối cùng được dùng:**

```dockerfile
CMD ["echo", "First"]
CMD ["echo", "Second"]  # ← Chỉ CMD này được dùng
```

**Kết quả:**
- Chỉ "Second" được in
- CMD đầu tiên bị ignore

**Best practice:**
- **Chỉ có một CMD** trong Dockerfile
- Nếu cần nhiều commands → dùng script

---

## 🚪 PHẦN 2: ENTRYPOINT INSTRUCTION

### 2.1. ENTRYPOINT là gì?

**ENTRYPOINT** là instruction chỉ định **main command** của container (khó override hơn CMD).

**Syntax:**
```dockerfile
# Shell form
ENTRYPOINT command param1 param2

# Exec form (recommended)
ENTRYPOINT ["executable", "param1", "param2"]
```

**Ví dụ:**
```dockerfile
ENTRYPOINT ["nginx", "-g", "daemon off;"]
ENTRYPOINT ["python"]
ENTRYPOINT ["/entrypoint.sh"]
```

**Đặc điểm:**
- **Main command**: Command chính của container
- **Hard to override**: Khó override hơn CMD
- **Only one ENTRYPOINT**: Chỉ có một ENTRYPOINT
- **Not executed during build**: Chỉ chạy khi container run

### 2.2. ENTRYPOINT Shell Form vs Exec Form

**Shell form:**
```dockerfile
ENTRYPOINT python app.py
```

**Đặc điểm:**
- Chạy trong shell
- **PID 1**: Shell là PID 1
- **Signal handling**: Không tốt

**Exec form:**
```dockerfile
ENTRYPOINT ["python", "app.py"]
```

**Đặc điểm:**
- Chạy trực tiếp
- **PID 1**: Application là PID 1
- **Signal handling**: Tốt

**Recommendation:**
- **Luôn dùng exec form** cho ENTRYPOINT
- **Better signal handling**
- **Production-ready**

### 2.3. ENTRYPOINT Override

**ENTRYPOINT khó override hơn CMD:**

```dockerfile
# Dockerfile
ENTRYPOINT ["python"]
```

```bash
# Override ENTRYPOINT (cần --entrypoint flag)
$ docker run --entrypoint bash my-app
# Override ENTRYPOINT thành bash

# Không thể override bằng arguments
$ docker run my-app --version
# Arguments được append vào ENTRYPOINT
# → python --version (không override)
```

**Khi nào override?**
- Debugging: Cần shell để debug
- Special cases: Cần chạy command khác

### 2.4. ENTRYPOINT Use Cases

**Use cases cho ENTRYPOINT:**

1. **Wrapper scripts:**
   ```dockerfile
   ENTRYPOINT ["/entrypoint.sh"]
   ```
   - Script xử lý initialization
   - Sau đó exec main application

2. **Fixed command:**
   ```dockerfile
   ENTRYPOINT ["nginx", "-g", "daemon off;"]
   ```
   - Command không đổi
   - Không muốn override

3. **Base command:**
   ```dockerfile
   ENTRYPOINT ["python"]
   CMD ["app.py"]
   ```
   - ENTRYPOINT = base command
   - CMD = arguments (có thể override)

---

## 🔄 PHẦN 3: CMD vs ENTRYPOINT

### 3.1. So Sánh

| Tiêu chí | CMD | ENTRYPOINT |
|----------|-----|------------|
| **Override** | Dễ (chỉ cần arguments) | Khó (cần --entrypoint) |
| **Purpose** | Default command | Main command |
| **Arguments** | Có thể override | Arguments được append |
| **Use case** | Default behavior | Fixed behavior |
| **Multiple** | Chỉ CMD cuối cùng | Chỉ ENTRYPOINT cuối cùng |

### 3.2. Khi Nào Dùng CMD?

**Dùng CMD khi:**
- **Default command**: Muốn có default command nhưng có thể override
- **Flexible**: Muốn user có thể override dễ dàng
- **Most cases**: 90% use cases

**Ví dụ:**
```dockerfile
FROM python:3.9
COPY app.py /app/
CMD ["python", "/app/app.py"]
```

**User có thể override:**
```bash
$ docker run my-app
# Chạy: python /app/app.py

$ docker run my-app python --version
# Override: python --version
```

### 3.3. Khi Nào Dùng ENTRYPOINT?

**Dùng ENTRYPOINT khi:**
- **Fixed command**: Command không đổi, không muốn override
- **Wrapper script**: Cần script xử lý initialization
- **Base command**: ENTRYPOINT = base, CMD = arguments

**Ví dụ:**
```dockerfile
FROM nginx:alpine
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```

**User không thể override dễ:**
```bash
$ docker run my-nginx
# Chạy: nginx -g daemon off;

$ docker run my-nginx bash
# ❌ Không work (cần --entrypoint)
```

### 3.4. CMD + ENTRYPOINT Together

**CMD và ENTRYPOINT có thể dùng cùng nhau:**

```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```

**Kết quả khi run:**
```bash
$ docker run my-app
# Chạy: python app.py

$ docker run my-app --version
# Chạy: python --version (CMD bị override)
```

**Behavior:**
- **ENTRYPOINT**: Base command (không đổi)
- **CMD**: Arguments (có thể override)
- **Final command**: ENTRYPOINT + CMD (hoặc override CMD)

**Use case:**
- **ENTRYPOINT**: Tool/executable cố định
- **CMD**: Default arguments, có thể override

**Ví dụ:**
```dockerfile
# curl image
ENTRYPOINT ["curl"]
CMD ["-s", "https://example.com"]

# User có thể override CMD
$ docker run my-curl -I https://google.com
# Chạy: curl -I https://google.com
```

---

## 🎯 PHẦN 4: DECISION MATRIX

### 4.1. Khi Nào Dùng Gì?

**Chỉ dùng CMD:**
```dockerfile
CMD ["python", "app.py"]
```
- ✅ Default command
- ✅ User có thể override dễ
- ✅ Most common use case

**Chỉ dùng ENTRYPOINT:**
```dockerfile
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```
- ✅ Fixed command
- ✅ Không muốn override
- ✅ Wrapper scripts

**Dùng cả CMD và ENTRYPOINT:**
```dockerfile
ENTRYPOINT ["python"]
CMD ["app.py"]
```
- ✅ Base command cố định
- ✅ Default arguments có thể override
- ✅ Flexible nhưng controlled

### 4.2. Examples

**Example 1: Web Server (CMD only)**
```dockerfile
FROM nginx:alpine
COPY nginx.conf /etc/nginx/nginx.conf
CMD ["nginx", "-g", "daemon off;"]
```
- **Lý do**: User có thể override để debug

**Example 2: Web Server (ENTRYPOINT only)**
```dockerfile
FROM nginx:alpine
COPY nginx.conf /etc/nginx/nginx.conf
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```
- **Lý do**: Command cố định, không muốn override

**Example 3: Python App (CMD only)**
```dockerfile
FROM python:3.9-slim
COPY app.py /app/
CMD ["python", "/app/app.py"]
```
- **Lý do**: Default command, có thể override

**Example 4: Tool (ENTRYPOINT + CMD)**
```dockerfile
FROM alpine:latest
RUN apk add --no-cache curl
ENTRYPOINT ["curl"]
CMD ["-s", "https://example.com"]
```
- **Lý do**: Base command cố định, arguments có thể override

---

## 🏭 PRODUCTION STORY #1: Container Không Handle Signals

### Context

**Công ty:** SaaS platform, 500 employees
**Hệ thống:** Python applications với Docker
**Traffic:** 5M requests/day
**Team:** 30 backend engineers

### Problem

**Tháng 6/2023:**
- **Containers không shutdown gracefully**
- **Data loss**: Requests đang xử lý bị mất
- **Root cause**: CMD dùng shell form → không handle SIGTERM

**Timeline:**
- **10:00 AM**: Deploy new version
- **10:01 AM**: Stop old containers
- **10:01:10 AM**: Containers bị force kill (không graceful)
- **10:01:15 AM**: Data loss reported
- **10:02 AM**: Team investigate

**Impact:**
- **Data loss**: 100+ requests
- **Customer complaints**: Incomplete transactions
- **Revenue loss**: $5K

### Investigation

**Root cause:**
```dockerfile
# Bad: Shell form
CMD python app.py
```

**Vấn đề:**
- Shell form → `/bin/sh -c python app.py`
- **PID 1**: Shell process là PID 1
- **SIGTERM**: Shell không forward signal đến Python
- **Result**: Container bị force kill (SIGKILL)

**Test:**
```bash
$ docker stop my-container
# Wait 10 seconds → SIGKILL
# Application không có cơ hội cleanup
```

### Fix

**Solution: Exec form**
```dockerfile
# Good: Exec form
CMD ["python", "app.py"]
```

**Kết quả:**
- **PID 1**: Python process là PID 1
- **SIGTERM**: Python nhận signal
- **Graceful shutdown**: Application có thể cleanup

**Add signal handling:**
```python
# app.py
import signal
import sys

def signal_handler(sig, frame):
    print('Shutting down gracefully...')
    # Cleanup code
    sys.exit(0)

signal.signal(signal.SIGTERM, signal_handler)
```

### Result

**Trước:**
- Shell form CMD
- **Force kill** thường xuyên
- **Data loss**: 100+ requests

**Sau:**
- Exec form CMD
- **Graceful shutdown**: < 5 seconds
- **Zero data loss** trong 6 tháng

### Lesson Learned

1. **Luôn dùng exec form**: Better signal handling
2. **Handle SIGTERM**: Applications phải handle signals
3. **Test graceful shutdown**: Verify shutdown behavior
4. **Production requirement**: Graceful shutdown là must-have

---

## 🏭 PRODUCTION STORY #2: CMD Override Issues

### Context

**Công ty:** E-commerce, 700 employees
**Hệ thống:** Microservices với Docker
**Traffic:** 15M requests/day
**Team:** 40 backend engineers

### Problem

**Tháng 8/2023:**
- **Developers override CMD** để debug
- **Forget to restore**: CMD bị override trong production
- **Wrong command**: Production chạy debug command
- **Root cause**: CMD quá dễ override

**Timeline:**
- **2:00 PM**: Developer debug container
- **2:30 PM**: Override CMD: `docker run my-app bash`
- **3:00 PM**: Deploy lên production (forgot restore CMD)
- **3:05 PM**: Production issues (container không chạy app)
- **3:10 PM**: Team investigate
- **3:15 PM**: Fix và restore

**Impact:**
- **15 minutes downtime**
- **50K requests failed**
- **Customer complaints**

### Investigation

**Root cause:**
```dockerfile
# Dockerfile
CMD ["python", "app.py"]
```

**Vấn đề:**
- CMD quá dễ override
- Developer override → forget restore
- Production chạy wrong command

**Test:**
```bash
# Developer override
$ docker run my-app bash
# Container chạy bash thay vì app

# Deploy với override
$ docker run my-app bash  # ← Wrong!
# Production không chạy app
```

### Fix

**Solution 1: Use ENTRYPOINT**
```dockerfile
# Dockerfile
ENTRYPOINT ["python", "app.py"]
```

**Kết quả:**
- **Khó override**: Cần --entrypoint flag
- **Prevent mistakes**: Khó override nhầm
- **Fixed command**: Command cố định

**Solution 2: Wrapper Script**
```dockerfile
# entrypoint.sh
#!/bin/sh
# Initialization code
exec python app.py

# Dockerfile
COPY entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
```

### Result

**Trước:**
- CMD dễ override
- **3-4 incidents** mỗi tháng do override nhầm

**Sau:**
- ENTRYPOINT khó override
- **Zero incidents** trong 6 tháng
- Better control

### Lesson Learned

1. **ENTRYPOINT cho production**: Prevent accidental override
2. **Document override process**: Nếu cần override, document
3. **CI/CD validation**: Validate commands trong CI/CD
4. **Use wrapper scripts**: Cho complex initialization

---

## 🎓 TÓM TẮT

### CMD

**Chức năng:**
- Default command
- Dễ override
- Most common use case

**Best practices:**
- Luôn dùng exec form
- Chỉ có một CMD
- Document override behavior

### ENTRYPOINT

**Chức năng:**
- Main command
- Khó override
- Fixed behavior

**Best practices:**
- Luôn dùng exec form
- Dùng cho wrapper scripts
- Dùng khi không muốn override

### CMD + ENTRYPOINT

**Chức năng:**
- ENTRYPOINT = base command
- CMD = default arguments
- Flexible nhưng controlled

**Best practices:**
- ENTRYPOINT cho tool/executable
- CMD cho default arguments
- User có thể override CMD

### Decision Matrix

**Chỉ CMD:**
- Default command
- User có thể override
- Most cases

**Chỉ ENTRYPOINT:**
- Fixed command
- Wrapper scripts
- Không muốn override

**Cả hai:**
- Base command + arguments
- Flexible nhưng controlled

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ bạn đã:
- ✅ Hiểu CMD vs ENTRYPOINT
- ✅ Biết khi nào dùng gì
- ✅ Hiểu signal handling

**Day tiếp theo (Day-013)** sẽ đi sâu vào:
- COPY vs ADD
- Trade-offs và best practices
- Khi nào dùng COPY? Khi nào dùng ADD?

---

## 📚 TÀI LIỆU THAM KHẢO

- CMD: https://docs.docker.com/engine/reference/builder/#cmd
- ENTRYPOINT: https://docs.docker.com/engine/reference/builder/#entrypoint

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-013: COPY-vs-ADD](../Day-013-COPY-vs-ADD/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
