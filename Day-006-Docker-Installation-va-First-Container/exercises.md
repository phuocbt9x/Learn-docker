# Day-006: Bài Tập Thực Hành

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi làm các bài tập này, bạn sẽ:

- Cài đặt được Docker trên Linux
- Verify được Docker installation
- Chạy được containers cơ bản
- Hiểu được các Docker commands cơ bản
- Troubleshoot được các vấn đề cơ bản

---

## 📝 BÀI TẬP 1: CÀI ĐẶT DOCKER

### Scenario

Bạn cần cài Docker trên Ubuntu 20.04 server.

### Câu hỏi

**1.1.** Check system requirements:
- Kernel version có đủ không? (>= 3.10)
- Có cgroup support không?
- Có namespace support không?
- Viết commands để check.

**1.2.** Cài đặt Docker:
- Add Docker repository
- Install Docker Engine
- Verify installation
- Viết từng bước commands.

**1.3.** Post-installation setup:
- Add user vào docker group
- Start và enable Docker daemon
- Verify không cần sudo
- Viết commands.

**1.4.** Troubleshooting:
- Nếu gặp "permission denied", làm thế nào fix?
- Nếu Docker daemon không chạy, làm thế nào fix?
- Nếu không pull được image, làm thế nào debug?

---

## 📝 BÀI TẬP 2: FIRST CONTAINER

### Scenario

Bạn muốn chạy container đầu tiên để test Docker.

### Câu hỏi

**2.1.** Chạy hello-world container:
- Command để chạy hello-world
- Giải thích output
- Container có chạy lâu không? Tại sao?

**2.2.** Chạy nginx container:
- Command để chạy nginx ở background
- Map port 8080 (host) → 80 (container)
- Đặt tên container là "my-nginx"
- Verify container đang chạy
- Access web server từ browser/curl

**2.3.** Interactive container:
- Chạy ubuntu container interactive
- Trong container, tạo file `/tmp/test.txt`
- Exit container
- Container có còn chạy không? Tại sao?
- File `/tmp/test.txt` có còn không? Tại sao?

**2.4.** Container lifecycle:
- Tạo container (không start)
- Start container
- Stop container
- Remove container
- Viết commands cho từng bước

---

## 📝 BÀI TẬP 3: DOCKER COMMANDS

### Scenario

Bạn cần quản lý containers và images.

### Câu hỏi

**3.1.** List containers:
- Command để list running containers
- Command để list all containers (including stopped)
- Command để list containers với format custom
- Giải thích output columns

**3.2.** List images:
- Command để list images
- Command để list images với format custom
- Giải thích output columns

**3.3.** Container management:
- Command để stop container
- Command để start stopped container
- Command để restart container
- Command để remove container
- Command để remove all stopped containers

**3.4.** Docker info:
- Command để check Docker version
- Command để check Docker system info
- Giải thích các thông tin quan trọng trong output

---

## 📝 BÀI TẬP 4: VERIFY INSTALLATION

### Scenario

Bạn cần verify Docker installation đúng cách.

### Câu hỏi

**4.1.** Check Docker version:
- Command để check version
- Client và Server version có giống nhau không? Tại sao?
- Version nào quan trọng hơn?

**4.2.** Check Docker info:
- Command để check system info
- Thông tin nào quan trọng?
- Storage driver là gì? Tại sao quan trọng?

**4.3.** Test Docker:
- Test với hello-world
- Test với nginx
- Test với interactive container
- Tất cả tests phải pass

**4.4.** Troubleshooting checklist:
- Tạo checklist để verify installation
- Các bước nào cần check?
- Làm thế nào để verify mọi thứ OK?

---

## 📝 BÀI TẬP 5: TROUBLESHOOTING

### Scenario

Bạn gặp các lỗi sau khi dùng Docker.

### Câu hỏi

**5.1.** Permission denied:
```bash
$ docker run hello-world
permission denied while trying to connect to the Docker daemon socket
```
- Root cause là gì?
- Làm thế nào fix?
- Security implications?

**5.2.** Docker daemon not running:
```bash
$ docker run hello-world
Cannot connect to the Docker daemon. Is the docker daemon running?
```
- Root cause là gì?
- Làm thế nào fix?
- Làm thế nào prevent?

**5.3.** Cannot pull image:
```bash
$ docker run nginx
Error response from daemon: Get "https://registry-1.docker.io/v2/": net/http: request canceled
```
- Root cause là gì?
- Làm thế nào debug?
- Làm thế nào fix?

**5.4.** Port already in use:
```bash
$ docker run -p 8080:80 nginx
Error: bind: address already in use
```
- Root cause là gì?
- Làm thế nào fix?
- Làm thế nào check port đang dùng?

---

## 📝 BÀI TẬP 6: PRACTICAL SCENARIOS

### Scenario 1: Development Environment

Bạn là developer và cần setup Docker để develop.

**Requirements:**
- Chạy nginx để test frontend
- Chạy MySQL để test database
- Containers phải chạy ở background
- Có thể access từ host

### Câu hỏi

**6.1.** Setup nginx:
- Command để chạy nginx
- Map port nào?
- Verify nginx đang chạy

**6.2.** Setup MySQL:
- Command để chạy MySQL
- Map port nào?
- Set root password
- Verify MySQL đang chạy

**6.3.** Manage containers:
- List tất cả containers
- Stop containers khi không dùng
- Start lại khi cần
- Remove containers khi không cần

### Scenario 2: Production Server

Bạn là DevOps và cần cài Docker trên production server.

**Requirements:**
- Cài Docker đúng cách
- Security best practices
- Verify installation
- Document process

### Câu hỏi

**6.4.** Installation:
- Cài Docker như thế nào?
- Security considerations?
- Post-installation steps?

**6.5.** Verification:
- Verify installation như thế nào?
- Test cases nào cần cover?
- Documentation?

---

## 📝 BÀI TẬP 7: DOCKER COMMANDS DEEP DIVE

### Scenario

Bạn muốn hiểu sâu hơn về Docker commands.

### Câu hỏi

**7.1.** docker run options:
- `-d`: Detached mode - làm gì?
- `-it`: Interactive + TTY - làm gì?
- `-p`: Port mapping - syntax như thế nào?
- `--name`: Container name - tại sao quan trọng?
- `--rm`: Auto-remove - khi nào dùng?

**7.2.** docker ps options:
- `-a`: All containers - tại sao cần?
- `-q`: Quiet mode - dùng khi nào?
- `--format`: Custom format - syntax như thế nào?
- `--filter`: Filter containers - filter theo gì?

**7.3.** docker stop vs docker kill:
- Sự khác biệt?
- Khi nào dùng stop?
- Khi nào dùng kill?
- Best practices?

**7.4.** docker rm options:
- `-f`: Force remove - khi nào dùng?
- `-v`: Remove volumes - tại sao quan trọng?
- `docker container prune`: Làm gì?

---

## 📝 BÀI TẬP 8: DOCKER SYSTEM MANAGEMENT

### Scenario

Bạn cần quản lý Docker system (cleanup, maintenance).

### Câu hỏi

**8.1.** Cleanup containers:
- Command để remove stopped containers
- Command để remove all containers
- Best practices?

**8.2.** Cleanup images:
- Command để remove unused images
- Command để remove all images
- Best practices?

**8.3.** System cleanup:
- Command để cleanup toàn bộ system
- Cái gì bị xóa?
- Cái gì không bị xóa?
- Khi nào nên cleanup?

**8.4.** Disk usage:
- Command để check disk usage
- Làm thế nào để giảm disk usage?
- Best practices để manage disk space?

---

## ✅ CHECKLIST HOÀN THÀNH

Trước khi xem solutions, đảm bảo bạn đã:

- [ ] Cài đặt Docker trên máy của bạn
- [ ] Chạy hello-world container thành công
- [ ] Chạy nginx container và access được
- [ ] Hiểu các Docker commands cơ bản
- [ ] Làm tất cả các bài tập trên
- [ ] Thực hành các commands trên terminal

---

## 💡 GỢI Ý

- **Practice makes perfect**: Thực hành các commands nhiều lần
- **Read error messages**: Error messages thường có thông tin hữu ích
- **Use --help**: `docker <command> --help` để xem options
- **Experiment**: Thử các options khác nhau để hiểu rõ hơn

---

**Chúc bạn làm bài tốt! Sau khi hoàn thành, hãy xem `solutions.md` để so sánh với đáp án.**

