# Day-025: Volume Backup & Restore - Solutions

## 🎯 GIẢI PHÁP CHI TIẾT

---

## ✅ BÀI TẬP 1: Basic Backup

**Commands:**
```bash
# Create volume và data
$ docker volume create my-data
$ docker run --rm -v my-data:/data alpine sh -c "echo 'test' > /data/file.txt"

# Backup
$ docker run --rm \
  -v my-data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/backup.tar.gz /data

# Verify
$ ls -lh backup.tar.gz
```

---

## ✅ BÀI TẬP 2: Restore

**Commands:**
```bash
# Create new volume
$ docker volume create restored-data

# Restore
$ docker run --rm \
  -v restored-data:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/backup.tar.gz -C /

# Verify
$ docker run --rm -v restored-data:/data alpine cat /data/file.txt
# Output: test
```

---

## ✅ BÀI TẬP 3: Automated Backup

**Script:**
```bash
#!/bin/bash
BACKUP_DIR="/backups"
VOLUME_NAME="db-data"
KEEP_DAYS=7

# Backup
docker run --rm \
  -v $VOLUME_NAME:/data \
  -v $BACKUP_DIR:/backup \
  alpine tar czf /backup/backup-$(date +%Y%m%d).tar.gz /data

# Cleanup old backups
find $BACKUP_DIR -name "backup-*.tar.gz" -mtime +$KEEP_DAYS -delete
```

**Schedule:**
```bash
# Add to crontab
0 2 * * * /path/to/backup.sh
```

---

**Lưu ý:** Tất cả số liệu trong solutions là illustrative/approximate cho mục đích giáo dục.

