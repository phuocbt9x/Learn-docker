# Day-025: Volume Backup & Restore Strategies

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được cách backup volumes
- Biết cách restore volumes
- Hiểu được backup strategies
- Biết cách automate backup
- Implement backup trong production

---

## 💾 PHẦN 1: VOLUME BACKUP

### 1.1. Backup Volume

**Basic backup:**
```bash
$ docker run --rm \
  -v my-volume:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/backup.tar.gz /data
```

**Steps:**
1. Mount volume vào container
2. Mount backup directory
3. Create archive

### 1.2. Backup Strategies

**Full backup:**
- Backup toàn bộ volume
- Regular schedule
- Keep multiple versions

**Incremental backup:**
- Backup changes only
- Faster
- Less storage

---

## 🔄 PHẦN 2: VOLUME RESTORE

### 2.1. Restore Volume

**Restore from backup:**
```bash
$ docker run --rm \
  -v my-volume:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/backup.tar.gz -C /
```

**Steps:**
1. Mount volume
2. Mount backup
3. Extract archive

### 2.2. Restore Strategies

**Full restore:**
- Restore toàn bộ volume
- From latest backup
- Verify data

---

## 🏭 PRODUCTION STORY: Backup Strategy

### Context

**Công ty:** Fintech, 500 employees
**Issue:** Data loss do no backup
**Solution:** Automated backup

### Implementation

**Automated backup:**
```bash
#!/bin/bash
# Backup script
docker run --rm \
  -v db-data:/data \
  -v /backups:/backup \
  alpine tar czf /backup/db-$(date +%Y%m%d).tar.gz /data
```

**Results:**
- Automated backups
- No data loss
- Easy restore

---

## 🎓 TÓM TẮT

**Backup:**
- Regular backups
- Multiple versions
- Automated

**Restore:**
- Test restore process
- Verify data
- Document process

---

## 🚀 BƯỚC TIẾP THEO

**Phase 5 hoàn thành!** Bạn đã nắm vững Networking & Storage.

**Phase tiếp theo (Phase 6)** sẽ đi sâu vào:
- Security & Hardening
- Container security

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.

