# Day-046: Container Crash Debugging - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Container State Inspection

**Commands:**
```bash
$ docker ps -a
# List all containers

$ docker inspect <container> --format='{{.State.Status}}'
# Check status

$ docker inspect <container> --format='{{.State.ExitCode}}'
# Check exit code
```

**Analysis:**
- Status: Exited
- Exit code: 1 (error)
- State: Container crashed

---

## ✅ BÀI TẬP 2: Logs Analysis

**Commands:**
```bash
$ docker logs <container>
# View logs

$ docker logs --tail 50 <container>
# Last 50 lines

$ docker logs -t <container>
# With timestamps
```

**Analysis:**
- Error: Application error
- Root cause: Missing dependency
- Fix: Add dependency

---

## ✅ BÀI TẬP 3: Debug Crash Scenario

**Debug process:**
1. Check exit code: 1
2. View logs: Application error
3. Inspect container: Missing file
4. Fix: Add missing file

**Root cause:** Missing configuration file

**Fix:** Add configuration file to Dockerfile

---

## ✅ BÀI TẬP 4: Exit Code Analysis

**Scenario 1: Exit code 0 but app down**
- Issue: Wrong exit code
- Fix: Use exit code 1 on error

**Scenario 2: Exit code 1**
- Issue: Application error
- Fix: Fix application code

**Scenario 3: Exit code 125**
- Issue: Docker daemon error
- Fix: Check Docker daemon

---

## ✅ BÀI TẬP 5: Production Debugging

**Debug process:**
1. Check logs: Intermittent errors
2. Pattern: Errors during high load
3. Root cause: Memory issues
4. Solution: Increase memory limit

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

