# Day-036: Docker Compose Basics

## 🎯 MỤC TIÊU NGÀY HÔM NAY

Sau khi hoàn thành day này, bạn sẽ:

- Hiểu được Docker Compose là gì và tại sao cần
- Biết cách tạo và sử dụng docker-compose.yml
- Hiểu được Compose file format (v2, v3)
- Biết các commands cơ bản (up, down, ps, logs)
- Viết được docker-compose.yml cơ bản
- Áp dụng được trong development và production

---

## 📋 PHẦN 1: DOCKER COMPOSE LÀ GÌ?

### 1.1. Docker Compose là gì?

**Docker Compose** là **tool để define và run multi-container Docker applications**.

**Purpose:**
- **Multi-container**: Quản lý nhiều containers cùng lúc
- **Configuration as code**: Define configuration trong file
- **Easy management**: Dễ dàng start, stop, manage
- **Development**: Rất hữu ích cho development

### 1.2. Tại sao Docker Compose tồn tại?

**Vấn đề:**
- **Multiple containers**: Phải run nhiều containers manually
- **Complex commands**: Commands phức tạp với nhiều options
- **No coordination**: Containers không coordinate với nhau
- **Repetitive**: Lặp lại commands mỗi lần

**Docker Compose giải quyết:**
- **Single file**: Define tất cả trong một file
- **Simple commands**: Commands đơn giản (up, down)
- **Coordination**: Containers coordinate với nhau
- **Reproducible**: Reproducible setup

### 1.3. Docker Compose vs Docker CLI

**Docker CLI:**
```bash
$ docker network create app-network
$ docker run -d --network app-network --name db postgres
$ docker run -d --network app-network --name web nginx
# Phức tạp, nhiều commands
```

**Docker Compose:**
```yaml
# docker-compose.yml
services:
  db:
    image: postgres
  web:
    image: nginx
```

```bash
$ docker compose up
# Đơn giản, một command
```

---

## 📝 PHẦN 2: DOCKER-COMPOSE.YML

### 2.1. Compose File Format

**Basic structure:**
```yaml
version: '3.8'

services:
  service-name:
    image: image-name
    # service configuration

networks:
  network-name:
    # network configuration

volumes:
  volume-name:
    # volume configuration
```

### 2.2. Services

**Service definition:**
```yaml
services:
  web:
    image: nginx:alpine
    ports:
      - "8080:80"
    environment:
      - NODE_ENV=production
```

**Multiple services:**
```yaml
services:
  web:
    image: nginx
  db:
    image: postgres
  api:
    image: node:18-alpine
```

### 2.3. Compose File Versions

**Version 2:**
```yaml
version: '2'
services:
  web:
    image: nginx
```

**Version 3:**
```yaml
version: '3.8'
services:
  web:
    image: nginx
```

**Version 3.8 (Recommended):**
- **Latest stable**: Latest stable version
- **Feature-rich**: Nhiều features
- **Production-ready**: Recommended cho production

**Note:** Từ Docker Compose v2, `version` field không còn required.

---

## 🚀 PHẦN 3: COMPOSE COMMANDS

### 3.1. Basic Commands

**Start services:**
```bash
$ docker compose up
# Start tất cả services
```

**Start in background:**
```bash
$ docker compose up -d
# Start trong background (detached mode)
```

**Stop services:**
```bash
$ docker compose down
# Stop và remove containers
```

**View services:**
```bash
$ docker compose ps
# List running services
```

**View logs:**
```bash
$ docker compose logs
# View logs của tất cả services
$ docker compose logs web
# View logs của service cụ thể
```

### 3.2. Build và Run

**Build images:**
```bash
$ docker compose build
# Build images từ Dockerfiles
```

**Build và start:**
```bash
$ docker compose up --build
# Build images và start services
```

**Rebuild:**
```bash
$ docker compose up --build --force-recreate
# Force rebuild và recreate containers
```

### 3.3. Other Commands

**Execute command:**
```bash
$ docker compose exec web sh
# Execute command trong running container
```

**Scale services:**
```bash
$ docker compose up --scale web=3
# Scale web service to 3 instances
```

**Stop without remove:**
```bash
$ docker compose stop
# Stop containers nhưng không remove
```

---

## 🏭 PRODUCTION STORY #1: Multi-container Setup Complexity

### Context

**Công ty:** SaaS platform, 500 employees
**Hệ thống:** Microservices với multiple containers
**Traffic:** 12M requests/day
**Team:** 30 backend engineers

### Problem

**Tháng 4/2024:**
- **Setup phức tạp**: Phải run 10+ containers manually
- **Time-consuming**: Mất 30 phút để setup environment
- **Error-prone**: Dễ sai khi run nhiều commands
- **Root cause**: Không dùng Docker Compose

**Timeline:**
- **9:00 AM**: Developer cần setup local environment
- **9:05 AM**: Start run containers manually
- **9:30 AM**: Setup complete (25 phút)
- **9:35 AM**: Developer report issues (wrong network, wrong ports)

**Impact:**
- **Setup time**: 30 phút per developer
- **Errors**: 20% setup errors
- **Productivity**: Lost productivity

### Investigation

**Root cause:**
```bash
# Manual setup
$ docker network create app-network
$ docker run -d --network app-network --name db postgres
$ docker run -d --network app-network --name redis redis
$ docker run -d --network app-network --name api -e DB_HOST=db api
$ docker run -d --network app-network --name web -e API_URL=http://api web
# ... 10+ more commands
```

**Vấn đề:**
- **Complex**: Nhiều commands phức tạp
- **Error-prone**: Dễ sai (network name, environment variables)
- **No coordination**: Containers không coordinate
- **Repetitive**: Lặp lại mỗi lần

### Fix

**Solution: Docker Compose**
```yaml
# docker-compose.yml
version: '3.8'

services:
  db:
    image: postgres:14
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password

  redis:
    image: redis:alpine

  api:
    build: ./api
    environment:
      DB_HOST: db
      REDIS_HOST: redis
    depends_on:
      - db
      - redis

  web:
    build: ./web
    ports:
      - "8080:80"
    environment:
      API_URL: http://api:3000
    depends_on:
      - api
```

**Commands:**
```bash
$ docker compose up -d
# Setup complete trong 2 phút
```

**Kết quả:**
- **Setup time**: 30 phút → 2 phút (93% reduction)
- **Errors**: 20% → 0% (100% reduction)
- **Reproducible**: Cùng setup mỗi lần

### Result

**Trước:**
- Manual setup
- **Setup time**: 30 phút
- **Errors**: 20%
- **Reproducible**: ❌

**Sau:**
- Docker Compose
- **Setup time**: 2 phút
- **Errors**: 0%
- **Reproducible**: ✅

### Lesson Learned

1. **Always use Compose**: Cho multi-container applications
2. **Configuration as code**: Define trong file
3. **Reproducible**: Cùng setup mỗi lần
4. **Time-saving**: Tiết kiệm thời gian đáng kể

---

## 🏭 PRODUCTION STORY #2: Environment Inconsistency

### Context

**Công ty:** E-commerce, 700 employees
**Hệ thống:** Development environments
**Team:** 50 developers

### Problem

**Tháng 6/2024:**
- **Environment inconsistency**: Mỗi developer có environment khác nhau
- **"Works on my machine"**: Code work trên máy này nhưng không work trên máy khác
- **Setup issues**: Mỗi developer setup khác nhau
- **Root cause**: Không có standardized setup

**Timeline:**
- **10:00 AM**: Developer A push code
- **10:05 AM**: Developer B pull và test
- **10:10 AM**: Code không work (environment khác)
- **10:15 AM**: Developer B phải setup lại environment

**Impact:**
- **Productivity loss**: 2-3 giờ per developer per week
- **Frustration**: Developer frustration
- **Delays**: Project delays

### Investigation

**Root cause:**
- **No standard setup**: Mỗi developer setup khác nhau
- **Different versions**: Different versions của services
- **Different configurations**: Different configurations

**Example:**
- Developer A: PostgreSQL 13, Redis 6
- Developer B: PostgreSQL 14, Redis 7
- Developer C: PostgreSQL 12, Redis 5

### Fix

**Solution: Docker Compose với version pinning**
```yaml
# docker-compose.yml
version: '3.8'

services:
  db:
    image: postgres:14-alpine
    # Pin version

  redis:
    image: redis:7-alpine
    # Pin version

  api:
    build: ./api
    environment:
      DB_HOST: db
      REDIS_HOST: redis
```

**Kết quả:**
- **Consistent environments**: Tất cả developers có cùng environment
- **Version control**: Versions được track trong Git
- **Easy onboarding**: New developers setup nhanh

### Result

**Trước:**
- No standard setup
- **Consistency**: ❌
- **Setup time**: 2-3 giờ

**Sau:**
- Docker Compose
- **Consistency**: ✅
- **Setup time**: 5 phút

### Lesson Learned

1. **Standardize setup**: Docker Compose standardize setup
2. **Version control**: Track versions trong Git
3. **Easy onboarding**: New developers onboard nhanh
4. **Consistency**: Đảm bảo consistency

---

## 🎓 TÓM TẮT

### Docker Compose

**Purpose:**
- Manage multi-container applications
- Configuration as code
- Easy management
- Reproducible setup

**Benefits:**
- **Simplified commands**: Commands đơn giản
- **Coordination**: Containers coordinate
- **Reproducible**: Cùng setup mỗi lần
- **Time-saving**: Tiết kiệm thời gian

### Compose File

**Structure:**
- **version**: Compose file version (optional từ v2)
- **services**: Service definitions
- **networks**: Network definitions
- **volumes**: Volume definitions

**Best practices:**
- Use version 3.8 (hoặc latest)
- Pin image versions
- Use environment variables
- Document services

### Commands

**Basic:**
- `docker compose up`: Start services
- `docker compose down`: Stop services
- `docker compose ps`: List services
- `docker compose logs`: View logs

**Advanced:**
- `docker compose build`: Build images
- `docker compose exec`: Execute commands
- `docker compose scale`: Scale services

---

## 🚀 BƯỚC TIẾP THEO

Bây giờ bạn đã:
- ✅ Hiểu Docker Compose
- ✅ Biết cách tạo docker-compose.yml
- ✅ Sử dụng Compose commands

**Day tiếp theo (Day-037)** sẽ đi sâu vào:
- Compose Networks & Volumes
- Service dependencies
- Advanced configurations

---

## 📚 TÀI LIỆU THAM KHẢO

- Docker Compose: https://docs.docker.com/compose/
- Compose file reference: https://docs.docker.com/compose/compose-file/

---

**Lưu ý:** Tất cả số liệu performance, incidents trong production stories là illustrative/approximate cho mục đích giáo dục.


---

## 📚 NAVIGATION

[→ Day-037: Compose-Networks-Volumes](../Day-037-Compose-Networks-Volumes/theory.md)

**Hoặc quay lại:** [← ROADMAP](../ROADMAP.md)
