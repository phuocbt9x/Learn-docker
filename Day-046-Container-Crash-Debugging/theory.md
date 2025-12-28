# Day-046: Container Crash Debugging

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được cách debug container crashes
- Biết cách inspect container state
- Hiểu được exit codes
- Biết cách analyze logs
- Biết cách debug common crash scenarios
- Debug như senior engineer

---

## 🔍 PHẦN 1: DEBUGGING BASICS

### 1.1. Container State

**Container states:**
- **Created**: Container created nhưng chưa start
- **Running**: Container đang chạy
- **Restarting**: Container đang restart
- **Exited**: Container đã exit
- **Dead**: Container dead (không thể start)

**Check state:**
```bash
$ docker ps -a
# Show all containers, including stopped
```

**Inspect state:**
```bash
$ docker inspect <container>
# Show detailed container information
```

### 1.2. Exit Codes

**Exit codes:**
- **0**: Success
- **1-255**: Error codes
- **125**: Docker daemon error
- **126**: Command không thể execute
- **127**: Command not found
- **128+N**: Killed by signal N

**Check exit code:**
```bash
$ docker inspect <container> --format='{{.State.ExitCode}}'
```

### 1.3. Logs Analysis

**View logs:**
```bash
$ docker logs <container>
# View container logs

$ docker logs --tail 100 <container>
# Last 100 lines

$ docker logs -f <container>
# Follow logs (real-time)
```

**Log timestamps:**
```bash
$ docker logs -t <container>
# Show timestamps
```

---

## 🐛 PHẦN 2: COMMON CRASH SCENARIOS

### 2.1. Application Error

**Symptoms:**
- Container exit với non-zero code
- Application error trong logs
- Container không restart

**Debug:**
```bash
$ docker logs <container>
# Check application logs

$ docker inspect <container>
# Check exit code và state
```

**Fix:**
- Fix application code
- Handle errors properly
- Add error handling

### 2.2. Missing Dependencies

**Symptoms:**
- Container exit immediately
- "Command not found" errors
- Missing files

**Debug:**
```bash
$ docker logs <container>
# Check for missing dependencies

$ docker exec <container> ls /path
# Check if files exist
```

**Fix:**
- Add missing dependencies
- Fix Dockerfile
- Check COPY commands

### 2.3. Configuration Errors

**Symptoms:**
- Container start nhưng crash sau đó
- Configuration errors trong logs
- Invalid config files

**Debug:**
```bash
$ docker logs <container>
# Check configuration errors

$ docker exec <container> cat /config/file
# Check config files
```

**Fix:**
- Fix configuration
- Validate config files
- Add config validation

---

## 🏭 PRODUCTION STORY #1: Container Crash Mystery

### Context

**Công ty:** E-commerce, 800 employees
**Hệ thống:** Python microservices
**Traffic:** 25M requests/day
**Team:** 40 backend engineers

### Problem

**Tháng 6/2024:**
- **Container crashes**: Containers crash không rõ lý do
- **No logs**: Không có logs trước khi crash
- **Intermittent**: Crashes không consistent
- **Root cause**: Application error không được log

**Timeline:**
- **2:00 PM**: Container crash
- **2:01 PM**: Auto-restart
- **2:02 PM**: Crash again
- **2:05 PM**: Team investigate
- **2:15 PM**: Found root cause

**Impact:**
- **Service disruption**: Service bị disrupt
- **User complaints**: Users report issues
- **Revenue loss**: $15K

### Investigation

**Root cause:**
```python
# app.py
def process_request():
    # Missing error handling
    result = external_api.call()  # ← Can fail
    return result.process()  # ← Crash if result is None
```

**Vấn đề:**
- **No error handling**: Không handle errors
- **No logging**: Không log errors trước khi crash
- **Silent failure**: Fail silently

**Debug process:**
```bash
# Step 1: Check container state
$ docker ps -a
# Container exited with code 1

# Step 2: Check logs
$ docker logs <container>
# No error messages (silent failure)

# Step 3: Inspect container
$ docker inspect <container>
# Exit code: 1, no error message

# Step 4: Run manually
$ docker run --rm my-app
# Reproduce crash

# Step 5: Add debugging
$ docker run -it --entrypoint sh my-app
# Interactive shell để debug
```

### Fix

**Solution: Add error handling và logging**
```python
# app.py
import logging

logger = logging.getLogger(__name__)

def process_request():
    try:
        result = external_api.call()
        if result is None:
            logger.error("External API returned None")
            raise ValueError("Invalid result")
        return result.process()
    except Exception as e:
        logger.error(f"Error processing request: {e}", exc_info=True)
        raise
```

**Additional: Health check**
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD python healthcheck.py || exit 1
```

**Kết quả:**
- **Error logging**: Errors được log
- **Debugging**: Dễ debug hơn
- **Crashes**: Giảm 90%

### Result

**Trước:**
- No error handling
- **Debugging**: ❌ Khó debug
- **Crashes**: Frequent

**Sau:**
- Error handling và logging
- **Debugging**: ✅ Dễ debug
- **Crashes**: Rare

### Lesson Learned

1. **Always log errors**: Log errors trước khi crash
2. **Error handling**: Handle errors properly
3. **Health checks**: Use health checks
4. **Debugging tools**: Know debugging tools

---

## 🏭 PRODUCTION STORY #2: Exit Code Confusion

### Context

**Công ty:** SaaS platform, 600 employees
**Hệ thống:** Node.js applications
**Traffic:** 15M requests/day
**Team:** 30 backend engineers

### Problem

**Tháng 8/2024:**
- **Container exits**: Container exit với code 0
- **Application down**: Application không chạy
- **No errors**: Không có errors trong logs
- **Root cause**: Application exit với code 0 (success) nhưng không chạy

**Timeline:**
- **10:00 AM**: Container exit
- **10:01 AM**: Team check logs
- **10:05 AM**: No errors, exit code 0
- **10:10 AM**: Team confused
- **10:15 AM**: Found root cause

**Impact:**
- **Service down**: Service down nhưng không detect được
- **Monitoring**: Monitoring không alert (exit code 0)
- **User impact**: Users không thể access

### Investigation

**Root cause:**
```javascript
// app.js
async function start() {
  try {
    await connectDB();
    await startServer();
  } catch (error) {
    console.error(error);
    process.exit(0);  // ← Exit với code 0 (wrong!)
  }
}
```

**Vấn đề:**
- **Exit code 0**: Exit với code 0 (success)
- **Application down**: Application không chạy
- **Monitoring**: Monitoring không detect (code 0 = success)

**Debug process:**
```bash
# Step 1: Check exit code
$ docker inspect <container> --format='{{.State.ExitCode}}'
# Output: 0 (success - confusing!)

# Step 2: Check logs
$ docker logs <container>
# Error: Database connection failed
# But exit code is 0!

# Step 3: Check application code
# Found: process.exit(0) on error
```

### Fix

**Solution: Use correct exit codes**
```javascript
// app.js
async function start() {
  try {
    await connectDB();
    await startServer();
  } catch (error) {
    console.error(error);
    process.exit(1);  // ← Exit với code 1 (error)
  }
}
```

**Additional: Health check**
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD node healthcheck.js || exit 1
```

**Kết quả:**
- **Correct exit codes**: Exit codes đúng
- **Monitoring**: Monitoring detect failures
- **Debugging**: Dễ debug hơn

### Result

**Trước:**
- Exit code 0 on error
- **Monitoring**: ❌ Không detect
- **Debugging**: ❌ Confusing

**Sau:**
- Correct exit codes
- **Monitoring**: ✅ Detect failures
- **Debugging**: ✅ Clear

### Lesson Learned

1. **Use correct exit codes**: Exit code 0 = success, non-zero = error
2. **Health checks**: Use health checks để detect issues
3. **Logging**: Log errors với proper context
4. **Monitoring**: Monitor exit codes và health

---

## 🎓 TÓM TẮT

### Debugging Tools

**Container inspection:**
- `docker ps -a`: List all containers
- `docker inspect`: Detailed container info
- `docker logs`: View logs

**Debugging techniques:**
- Check exit codes
- Analyze logs
- Inspect container state
- Run interactively

### Common Crash Scenarios

**1. Application errors:**
- Missing error handling
- Unhandled exceptions
- Fix: Add error handling

**2. Missing dependencies:**
- Missing files
- Missing packages
- Fix: Add dependencies

**3. Configuration errors:**
- Invalid config
- Missing config
- Fix: Validate config

**4. Exit code issues:**
- Wrong exit codes
- Silent failures
- Fix: Use correct exit codes

### Best Practices

**1. Always log errors:**
- Log trước khi crash
- Include context
- Use proper log levels

**2. Use correct exit codes:**
- 0 = success
- Non-zero = error
- Consistent exit codes

**3. Health checks:**
- Detect issues early
- Monitor health
- Automatic recovery

**4. Debugging tools:**
- Know tools
- Use effectively
- Document process

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ bạn đã:
- ✅ Hiểu cách debug container crashes
- ✅ Biết debugging tools
- ✅ Debug common scenarios

**Day tiếp theo (Day-047)** sẽ đi sâu vào:
- OOM (Out of Memory) Issues
- Memory debugging
- OOM prevention

---

## 📚 TÀI LIỆU THAM KHẢO

- Docker debugging: https://docs.docker.com/config/containers/logging/
- Container inspection: https://docs.docker.com/engine/reference/commandline/inspect/

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.

