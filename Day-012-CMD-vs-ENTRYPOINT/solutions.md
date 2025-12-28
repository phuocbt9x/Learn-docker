# Day-012: CMD vs ENTRYPOINT - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

Tài liệu này cung cấp giải pháp chi tiết cho tất cả các bài tập, bao gồm:
- Dockerfiles hoàn chỉnh
- Commands để test
- Giải thích "why" cho mỗi decision
- So sánh approaches
- Common errors và cách fix
- Production notes

---

## ✅ BÀI TẬP 1: CMD Basics

### Solution

**Dockerfile:**
```dockerfile
FROM python:3.9-slim
COPY app.py /app/app.py
CMD ["python", "/app/app.py"]
```

**Build và run:**
```bash
# Build
$ docker build -t my-python-app .

# Run (default)
$ docker run my-python-app
# Chạy: python /app/app.py

# Override CMD
$ docker run my-python-app python --version
# Chạy: python --version (override CMD)
```

### Giải thích

**1. Tại sao dùng exec form?**

**Exec form:**
```dockerfile
CMD ["python", "/app/app.py"]
```

**Ưu điểm:**
- **Signal handling**: Python process là PID 1 → handle SIGTERM tốt
- **No shell**: Không cần shell → smaller image
- **Production-ready**: Recommended cho production

**Shell form (không nên dùng):**
```dockerfile
CMD python /app/app.py
```

**Nhược điểm:**
- **Signal handling**: Shell là PID 1 → không forward signals tốt
- **Need shell**: Cần shell → larger image
- **Not production-ready**: Không recommended

**2. Làm thế nào override CMD?**

**Override bằng arguments:**
```bash
$ docker run my-python-app python --version
# Arguments sau image name override CMD
```

**Override bằng command:**
```bash
$ docker run my-python-app bash
# bash override CMD
```

**3. Test override:**

```bash
# Test default
$ docker run --rm my-python-app
# Output: Application output

# Test override
$ docker run --rm my-python-app python --version
# Output: Python 3.9.x
```

### Production Notes

- **Luôn dùng exec form** cho CMD
- **Document override behavior** trong README
- **Test override** trong CI/CD

---

## ✅ BÀI TẬP 2: ENTRYPOINT Basics

### Solution

**Dockerfile:**
```dockerfile
FROM nginx:alpine
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```

**Build và run:**
```bash
# Build
$ docker build -t my-nginx .

# Run (default)
$ docker run my-nginx
# Chạy: nginx -g daemon off;

# Override ENTRYPOINT
$ docker run --entrypoint bash my-nginx
# Chạy: bash (override ENTRYPOINT)
```

### Giải thích

**1. Tại sao dùng ENTRYPOINT?**

**ENTRYPOINT:**
- **Fixed command**: Command không đổi
- **Hard to override**: Khó override nhầm
- **Production safety**: Prevent accidental override

**CMD (không phù hợp):**
- **Easy to override**: Dễ override nhầm
- **Risk**: Developer có thể override trong production

**2. Làm thế nào override ENTRYPOINT?**

**Override bằng --entrypoint flag:**
```bash
$ docker run --entrypoint bash my-nginx
# Override ENTRYPOINT thành bash
```

**Không thể override bằng arguments:**
```bash
$ docker run my-nginx bash
# ❌ Không work
# Arguments được append vào ENTRYPOINT
# → nginx -g daemon off; bash (syntax error)
```

**3. Test override:**

```bash
# Test default
$ docker run --rm my-nginx nginx -v
# Output: nginx version: ...

# Test override
$ docker run --rm --entrypoint bash my-nginx -c "echo hello"
# Output: hello
```

### Production Notes

- **ENTRYPOINT cho fixed commands**: Khi không muốn override
- **Document override process**: Nếu cần override, document
- **Use --entrypoint carefully**: Chỉ dùng khi cần debug

---

## ✅ BÀI TẬP 3: CMD vs ENTRYPOINT Comparison

### Solution

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

### Comparison Table

| Aspect | CMD | ENTRYPOINT |
|--------|-----|------------|
| **Override** | Dễ: `docker run image bash` | Khó: `docker run --entrypoint bash image` |
| **Arguments** | Override toàn bộ | Arguments được append |
| **Use case** | Default command | Fixed command |
| **Production** | Flexible | Controlled |

### Test Results

**CMD override:**
```bash
$ docker run my-app:cmd bash
# ✅ Work: Chạy bash

$ docker run my-app:cmd python --version
# ✅ Work: Chạy python --version
```

**ENTRYPOINT override:**
```bash
$ docker run my-app:entrypoint bash
# ❌ Không work: Syntax error

$ docker run --entrypoint bash my-app:entrypoint
# ✅ Work: Chạy bash
```

### Recommendation

**Dùng CMD khi:**
- Default command
- User cần override dễ
- Most common use case

**Dùng ENTRYPOINT khi:**
- Fixed command
- Không muốn override
- Production safety

**Production scenario:**
- **Prevent accidental override**: Dùng ENTRYPOINT
- **Need flexibility**: Dùng CMD
- **Best practice**: ENTRYPOINT cho production, CMD cho development

---

## ✅ BÀI TẬP 4: CMD + ENTRYPOINT Together

### Solution

**Dockerfile:**
```dockerfile
FROM alpine:latest
RUN apk add --no-cache curl
ENTRYPOINT ["curl"]
CMD ["-s", "https://example.com"]
```

**Build và run:**
```bash
# Build
$ docker build -t my-curl .

# Run (default)
$ docker run my-curl
# Chạy: curl -s https://example.com

# Override CMD
$ docker run my-curl -I https://google.com
# Chạy: curl -I https://google.com

# Override ENTRYPOINT
$ docker run --entrypoint bash my-curl
# Chạy: bash
```

### Giải thích

**1. Command cuối cùng:**

**Default:**
```bash
$ docker run my-curl
# Final command: curl -s https://example.com
# ENTRYPOINT + CMD
```

**Override CMD:**
```bash
$ docker run my-curl -I https://google.com
# Final command: curl -I https://google.com
# ENTRYPOINT + override CMD
```

**2. Override CMD:**

```bash
# Override bằng arguments
$ docker run my-curl -I https://google.com
# CMD bị override: -I https://google.com
# ENTRYPOINT giữ nguyên: curl
```

**3. Override ENTRYPOINT:**

```bash
# Override bằng --entrypoint flag
$ docker run --entrypoint bash my-curl
# ENTRYPOINT bị override: bash
# CMD không được dùng (vì ENTRYPOINT đã đổi)
```

### Behavior Explanation

**ENTRYPOINT + CMD:**
- **ENTRYPOINT**: Base command (không đổi)
- **CMD**: Default arguments (có thể override)
- **Final**: ENTRYPOINT + CMD (hoặc override CMD)

**Use case:**
- **Tool/executable**: ENTRYPOINT = tool
- **Default args**: CMD = default arguments
- **Flexible**: User có thể override arguments

---

## ✅ BÀI TẬP 5: Signal Handling

### Solution

**Dockerfile 1: Shell form**
```dockerfile
FROM python:3.9-slim
COPY app.py /app/app.py
CMD python /app/app.py
```

**Dockerfile 2: Exec form**
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

### Test Results

**Shell form:**
```bash
# Build
$ docker build -f Dockerfile.shell -t my-app:shell .

# Run
$ docker run -d --name test1 my-app:shell

# Stop
$ docker stop test1
# Wait 10 seconds → SIGKILL

# Check logs
$ docker logs test1
# Output: Application started
# ❌ Không có "Received SIGTERM"
# → Shell không forward signal
```

**Exec form:**
```bash
# Build
$ docker build -f Dockerfile.exec -t my-app:exec .

# Run
$ docker run -d --name test2 my-app:exec

# Stop
$ docker stop test2
# Wait < 5 seconds → Graceful shutdown

# Check logs
$ docker logs test2
# Output: Application started
# Output: Received SIGTERM, shutting down gracefully...
# ✅ Nhận SIGTERM và shutdown gracefully
```

### Analysis

**Shell form:**
- **PID 1**: Shell process (`/bin/sh -c python app.py`)
- **SIGTERM**: Shell nhận signal nhưng không forward đến Python
- **Result**: Container bị force kill (SIGKILL) sau 10 seconds

**Exec form:**
- **PID 1**: Python process (`python app.py`)
- **SIGTERM**: Python nhận signal trực tiếp
- **Result**: Application handle signal và shutdown gracefully

### Production Impact

**Nếu không handle signals:**
- **Data loss**: Requests đang xử lý bị mất
- **Corrupted state**: Application không có cơ hội cleanup
- **Customer impact**: Incomplete transactions

**Best practice:**
- **Luôn dùng exec form**: Better signal handling
- **Handle SIGTERM**: Applications phải handle signals
- **Test graceful shutdown**: Verify trong CI/CD

---

## ✅ BÀI TẬP 6: Wrapper Script

### Solution

**entrypoint.sh:**
```bash
#!/bin/sh
echo "Starting application..."
echo "Environment: $ENV"
echo "Database: $DB_HOST"

# Initialization code
# - Wait for database
# - Run migrations
# - Setup environment
# ...

# Exec main application
exec python /app/app.py
```

**Dockerfile:**
```dockerfile
FROM python:3.9-slim
COPY entrypoint.sh /entrypoint.sh
COPY app.py /app/app.py
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
```

**Build và run:**
```bash
# Build
$ docker build -t my-app .

# Run
$ docker run -e ENV=production -e DB_HOST=db.example.com my-app
# Output: Starting application...
# Output: Environment: production
# Output: Database: db.example.com
# Output: Application output
```

### Giải thích

**1. Tại sao dùng wrapper script?**

**Use cases:**
- **Initialization**: Wait for dependencies, run migrations
- **Configuration**: Setup environment variables, config files
- **Logging**: Log startup information
- **Health checks**: Verify dependencies before start

**2. Tại sao dùng `exec`?**

**Without exec:**
```bash
python /app/app.py
# Shell process là PID 1
# Python process là child
# → Signal handling không tốt
```

**With exec:**
```bash
exec python /app/app.py
# Python process là PID 1
# Shell process được replace
# → Signal handling tốt
```

**3. Test:**

```bash
# Test initialization
$ docker run --rm -e ENV=production my-app
# Verify initialization output

# Test signal handling
$ docker run -d --name test my-app
$ docker stop test
# Verify graceful shutdown
```

### Production Notes

- **Dùng `exec`**: Để application là PID 1
- **Error handling**: Script phải handle errors
- **Logging**: Log initialization steps
- **Idempotent**: Initialization phải idempotent

---

## ✅ BÀI TẬP 7: Practical Dockerfile

### Solution

**Decision: Dùng CMD**

**Lý do:**
- **Default command**: `node app.js` là default
- **Override needed**: Developers cần override để debug
- **Flexibility**: Cần flexibility trong development

**Dockerfile:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install
COPY app.js ./
EXPOSE 3000
CMD ["node", "app.js"]
```

**Build và run:**
```bash
# Build
$ docker build -t my-node-app .

# Run
$ docker run -p 3000:3000 my-node-app
# Server running on port 3000

# Test
$ curl http://localhost:3000
# Output: Hello World

# Override để debug
$ docker run -it --entrypoint sh my-node-app
# Shell để debug
```

### Alternative: ENTRYPOINT

**Nếu muốn prevent override:**
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm install
COPY app.js ./
EXPOSE 3000
ENTRYPOINT ["node", "app.js"]
```

**Trade-off:**
- **Pros**: Prevent accidental override
- **Cons**: Khó debug (cần --entrypoint flag)

### Recommendation

**Development:**
- **CMD**: Flexible, dễ override

**Production:**
- **ENTRYPOINT**: Controlled, prevent override
- **Hoặc CMD**: Nếu cần flexibility

---

## ✅ BÀI TẬP 8: Troubleshooting

### Problem Analysis

**Root cause:**
```dockerfile
CMD python /app/app.py
```

**Vấn đề:**
- **Shell form**: Shell process là PID 1
- **Signal handling**: Shell không forward SIGTERM
- **Result**: Container bị force kill

### Fix

**Fixed Dockerfile:**
```dockerfile
FROM python:3.9-slim
COPY app.py /app/app.py
CMD ["python", "/app/app.py"]
```

**Changes:**
- **Exec form**: Python process là PID 1
- **Signal handling**: Python nhận SIGTERM trực tiếp
- **Result**: Graceful shutdown

### Test

**Before fix:**
```bash
$ docker run -d --name test my-app:bad
$ docker stop test
# Wait 10 seconds → SIGKILL
$ docker logs test
# ❌ Không có graceful shutdown message
```

**After fix:**
```bash
$ docker run -d --name test my-app:fixed
$ docker stop test
# Wait < 5 seconds → Graceful shutdown
$ docker logs test
# ✅ Có graceful shutdown message
```

### Common Errors

**Error 1: Shell form**
```dockerfile
CMD python app.py  # ❌
```
**Fix:** Dùng exec form
```dockerfile
CMD ["python", "app.py"]  # ✅
```

**Error 2: Missing signal handler**
```python
# app.py không handle SIGTERM
```
**Fix:** Add signal handler
```python
import signal
signal.signal(signal.SIGTERM, signal_handler)
```

---

## ✅ BÀI TẬP 9: Production Analysis

### Current Dockerfile Analysis

**Current:**
```dockerfile
FROM python:3.9-slim
COPY requirements.txt /app/
RUN pip install -r /app/requirements.txt
COPY app.py /app/app.py
CMD ["python", "/app/app.py"]
```

**Ưu điểm:**
- ✅ Exec form → Good signal handling
- ✅ Flexible → Developers có thể override
- ✅ Simple → Dễ maintain

**Nhược điểm:**
- ⚠️ Easy to override → Risk accidental override
- ⚠️ No initialization → Không có initialization logic

**Production risks:**
- **Accidental override**: Developer override nhầm trong production
- **No initialization**: Không có logic để wait dependencies

### Recommendation

**Option 1: Keep CMD (Recommended)**
```dockerfile
FROM python:3.9-slim
COPY requirements.txt /app/
RUN pip install -r /app/requirements.txt
COPY app.py /app/app.py
CMD ["python", "/app/app.py"]
```

**Pros:**
- Flexible
- Simple
- Good signal handling

**Cons:**
- Easy to override

**Mitigation:**
- **CI/CD validation**: Validate commands trong CI/CD
- **Documentation**: Document override process
- **Code review**: Review Docker commands

**Option 2: Use ENTRYPOINT**
```dockerfile
FROM python:3.9-slim
COPY requirements.txt /app/
RUN pip install -r /app/requirements.txt
COPY app.py /app/app.py
ENTRYPOINT ["python", "/app/app.py"]
```

**Pros:**
- Prevent accidental override
- Controlled

**Cons:**
- Khó debug (cần --entrypoint)
- Less flexible

**Option 3: Wrapper Script**
```dockerfile
FROM python:3.9-slim
COPY requirements.txt /app/
RUN pip install -r /app/requirements.txt
COPY entrypoint.sh /entrypoint.sh
COPY app.py /app/app.py
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
```

**entrypoint.sh:**
```bash
#!/bin/sh
# Wait for database
# Run migrations
# Setup environment
exec python /app/app.py
```

**Pros:**
- Initialization logic
- Prevent override
- Better control

**Cons:**
- More complex
- Need to maintain script

### Refactored Dockerfile (Recommended)

**For most cases: Keep CMD**
```dockerfile
FROM python:3.9-slim
COPY requirements.txt /app/
RUN pip install -r /app/requirements.txt
COPY app.py /app/app.py
CMD ["python", "/app/app.py"]
```

**If need initialization: Use wrapper script**
```dockerfile
FROM python:3.9-slim
COPY requirements.txt /app/
RUN pip install -r /app/requirements.txt
COPY entrypoint.sh /entrypoint.sh
COPY app.py /app/app.py
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
```

### Justification

**Keep CMD:**
- **Flexibility**: Developers cần override để debug
- **Simplicity**: Không cần wrapper script
- **Signal handling**: Exec form đã đủ tốt

**Use wrapper script nếu:**
- **Initialization needed**: Wait dependencies, migrations
- **Complex setup**: Cần nhiều setup steps
- **Production control**: Cần control tốt hơn

---

## ✅ BÀI TẬP 10: Advanced - Multiple Commands

### Comparison

| Aspect | Option 1: CMD shell | Option 2: ENTRYPOINT wrapper | Option 3: CMD script |
|--------|---------------------|------------------------------|----------------------|
| **Signal handling** | ❌ Shell là PID 1 | ✅ App là PID 1 (exec) | ✅ App là PID 1 (exec) |
| **Override** | ✅ Easy | ⚠️ Need --entrypoint | ✅ Easy |
| **Maintainability** | ⚠️ Inline script | ✅ Separate script | ✅ Separate script |
| **Production** | ❌ Not recommended | ✅ Recommended | ✅ Good |

### Analysis

**Option 1: CMD shell**
```dockerfile
CMD ["sh", "-c", "python migrate.py && python app.py"]
```

**Pros:**
- Simple
- Inline

**Cons:**
- **Signal handling**: Shell là PID 1 → không tốt
- **Error handling**: Khó handle errors
- **Not production-ready**

**Option 2: ENTRYPOINT wrapper**
```dockerfile
ENTRYPOINT ["/entrypoint.sh"]
```

**entrypoint.sh:**
```bash
#!/bin/sh
python migrate.py
exec python app.py
```

**Pros:**
- **Signal handling**: App là PID 1 (exec)
- **Error handling**: Có thể handle errors
- **Maintainability**: Separate script
- **Production-ready**

**Cons:**
- Khó override (cần --entrypoint)

**Option 3: CMD script**
```dockerfile
CMD ["/start.sh"]
```

**start.sh:**
```bash
#!/bin/sh
python migrate.py
exec python app.py
```

**Pros:**
- **Signal handling**: App là PID 1 (exec)
- **Override**: Dễ override
- **Maintainability**: Separate script
- **Production-ready**

**Cons:**
- Dễ override (có thể là con nếu muốn prevent)

### Recommendation

**Option 2: ENTRYPOINT wrapper (Recommended)**

**Lý do:**
- **Production-ready**: Signal handling tốt
- **Error handling**: Có thể handle errors
- **Maintainability**: Separate script
- **Control**: Prevent accidental override

**Implementation:**
```dockerfile
FROM python:3.9-slim
COPY requirements.txt /app/
RUN pip install -r /app/requirements.txt
COPY entrypoint.sh /entrypoint.sh
COPY migrate.py /app/
COPY app.py /app/
RUN chmod +x /entrypoint.sh
ENTRYPOINT ["/entrypoint.sh"]
```

**entrypoint.sh:**
```bash
#!/bin/sh
set -e

echo "Running migrations..."
python /app/migrate.py

echo "Starting application..."
exec python /app/app.py
```

**Features:**
- **set -e**: Exit on error
- **exec**: App là PID 1
- **Error handling**: Handle migration errors
- **Logging**: Log steps

### Alternative: Option 3

**Nếu cần flexibility:**
```dockerfile
CMD ["/start.sh"]
```

**Trade-off:**
- **Pros**: Dễ override
- **Cons**: Có thể override nhầm

---

## 🎓 TÓM TẮT BEST PRACTICES

### CMD

**Dùng khi:**
- Default command
- User cần override
- Most common use case

**Best practices:**
- ✅ Luôn dùng exec form
- ✅ Chỉ có một CMD
- ✅ Document override behavior

### ENTRYPOINT

**Dùng khi:**
- Fixed command
- Wrapper scripts
- Không muốn override

**Best practices:**
- ✅ Luôn dùng exec form
- ✅ Dùng cho wrapper scripts
- ✅ Document override process

### CMD + ENTRYPOINT

**Dùng khi:**
- Base command + arguments
- Flexible nhưng controlled

**Best practices:**
- ✅ ENTRYPOINT = base command
- ✅ CMD = default arguments
- ✅ User có thể override CMD

### Signal Handling

**Best practices:**
- ✅ Luôn dùng exec form
- ✅ Applications handle SIGTERM
- ✅ Wrapper scripts dùng `exec`
- ✅ Test graceful shutdown

---

## 🚀 PRODUCTION CHECKLIST

Trước khi deploy:

- [ ] CMD/ENTRYPOINT dùng exec form
- [ ] Application handle SIGTERM
- [ ] Test graceful shutdown
- [ ] Document override behavior
- [ ] CI/CD validate commands
- [ ] Wrapper scripts dùng `exec`

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

