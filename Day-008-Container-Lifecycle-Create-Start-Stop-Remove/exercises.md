# Day-008: Bài Tập Thực Hành

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi làm các bài tập này, bạn sẽ:

- Hiểu được container lifecycle và các states
- Thành thạo các commands: create, start, stop, restart, remove
- Hiểu được sự khác biệt giữa stop và kill
- Biết cách sử dụng restart policies
- Quản lý containers hiệu quả

---

## 📝 BÀI TẬP 1: CONTAINER STATES

### Scenario

Bạn cần hiểu các container states và transitions.

### Câu hỏi

**1.1.** Liệt kê tất cả container states:
- States nào có?
- Mô tả mỗi state
- Khi nào container ở mỗi state?

**1.2.** State transitions:
- Vẽ diagram state transitions
- Created → ? → ?
- Running → ? → ?
- Exited → ? → ?

**1.3.** Check container state:
- Command để check state của container
- Làm thế nào biết container đang ở state nào?
- Giải thích output của `docker ps` và `docker ps -a`

**1.4.** Practical exercise:
- Tạo container (không start) → check state
- Start container → check state
- Stop container → check state
- Remove container → verify removed

---

## 📝 BÀI TẬP 2: CREATE CONTAINERS

### Scenario

Bạn cần tạo containers với các configurations khác nhau.

### Câu hỏi

**2.1.** Basic create:
- Tạo container từ `nginx` image
- Đặt tên `my-nginx`
- Verify container đã được tạo
- Check state của container

**2.2.** Create với options:
- Tạo container với environment variables
- Tạo container với port mapping
- Tạo container với volume mount
- Tạo container với restart policy
- Viết commands

**2.3.** Create vs Run:
- So sánh `docker create` và `docker run`
- Khi nào nên dùng `create`?
- Khi nào nên dùng `run`?
- Trade-offs?

**2.4.** Batch create:
- Tạo 5 containers cùng lúc
- Tạo với names: `app-1`, `app-2`, `app-3`, `app-4`, `app-5`
- Verify tất cả đã được tạo
- Viết script để automate

---

## 📝 BÀI TẬP 3: START CONTAINERS

### Scenario

Bạn cần start containers đã được tạo.

### Câu hỏi

**3.1.** Basic start:
- Start container `my-nginx` đã tạo
- Verify container đang chạy
- Check logs
- Giải thích start process

**3.2.** Start options:
- Start với attach (xem output)
- Start với interactive
- Start detached (background)
- So sánh các options

**3.3.** Start multiple:
- Start nhiều containers cùng lúc
- Start tất cả stopped containers
- Viết commands

**3.4.** Start behavior:
- Khi start container, điều gì xảy ra?
- Container layer có được restore không?
- Data trong volumes có còn không?
- Network config có được restore không?

---

## 📝 BÀI TẬP 4: STOP CONTAINERS

### Scenario

Bạn cần stop containers đang chạy.

### Câu hỏi

**4.1.** Basic stop:
- Stop container `my-nginx` đang chạy
- Verify container đã stop
- Check exit code
- Giải thích stop process

**4.2.** Stop options:
- Stop với timeout 30 seconds
- Stop với timeout 5 seconds
- So sánh behavior
- Khi nào cần increase timeout?

**4.3.** Stop multiple:
- Stop nhiều containers cùng lúc
- Stop tất cả running containers
- Viết commands

**4.4.** Stop vs Kill:
- So sánh `docker stop` và `docker kill`
- Khi nào dùng stop?
- Khi nào dùng kill?
- Best practices?

---

## 📝 BÀI TẬP 5: RESTART CONTAINERS

### Scenario

Bạn cần restart containers.

### Câu hỏi

**5.1.** Basic restart:
- Restart container `my-nginx`
- Verify container đã restart
- Check uptime
- Giải thích restart process

**5.2.** Restart options:
- Restart với timeout
- Restart nhiều containers
- So sánh với stop + start

**5.3.** Restart vs Stop + Start:
- So sánh `docker restart` và `docker stop` + `docker start`
- Khi nào dùng restart?
- Khi nào dùng stop + start?
- Trade-offs?

**5.4.** Restart scenarios:
- Restart running container
- Restart stopped container
- Restart paused container
- Behavior khác nhau như thế nào?

---

## 📝 BÀI TẬP 6: REMOVE CONTAINERS

### Scenario

Bạn cần remove containers không còn dùng.

### Câu hỏi

**6.1.** Basic remove:
- Remove container `my-nginx` đã stop
- Verify container đã bị xóa
- Giải thích remove process

**6.2.** Remove options:
- Force remove running container
- Remove với volumes
- Remove nhiều containers
- So sánh các options

**6.3.** Remove all stopped:
- Remove tất cả stopped containers
- Verify cleanup
- Best practices?

**6.4.** Remove vs Stop:
- So sánh `docker rm` và `docker stop`
- Khi nào dùng remove?
- Khi nào dùng stop?
- Data có mất không?

---

## 📝 BÀI TẬP 7: PAUSE/UNPAUSE

### Scenario

Bạn cần pause và unpause containers.

### Câu hỏi

**7.1.** Pause container:
- Pause container đang chạy
- Verify container đã pause
- Check state
- Giải thích pause behavior

**7.2.** Unpause container:
- Unpause container đã pause
- Verify container đã resume
- Check state
- Processes có resume không?

**7.3.** Pause vs Stop:
- So sánh `docker pause` và `docker stop`
- Khi nào dùng pause?
- Khi nào dùng stop?
- Memory usage khác nhau như thế nào?

**7.4.** Pause use cases:
- Khi nào nên pause container?
- Use cases cho pause?
- Best practices?

---

## 📝 BÀI TẬP 8: RESTART POLICIES

### Scenario

Bạn cần setup restart policies cho production.

### Câu hỏi

**8.1.** Restart policies:
- Liệt kê tất cả restart policies
- Giải thích mỗi policy
- Khi nào dùng policy nào?

**8.2.** no policy:
- Tạo container với `--restart=no`
- Stop container
- Container có tự động restart không?
- Use cases?

**8.3.** on-failure policy:
- Tạo container với `--restart=on-failure`
- Test với exit code 0
- Test với exit code != 0
- Behavior khác nhau như thế nào?

**8.4.** always policy:
- Tạo container với `--restart=always`
- Stop container manually
- Container có tự động restart không?
- Use cases và limitations?

**8.5.** unless-stopped policy:
- Tạo container với `--restart=unless-stopped`
- Stop container manually
- Restart Docker daemon
- Behavior như thế nào?
- Tại sao recommended cho production?

**8.6.** Update restart policy:
- Làm thế nào update restart policy của existing container?
- Có thể update trực tiếp không?
- Nếu không, làm thế nào?

---

## 📝 BÀI TẬP 9: PRACTICAL SCENARIOS

### Scenario 1: Development Workflow

Bạn là developer và cần quản lý containers cho development.

**Requirements:**
- Tạo containers với config phức tạp
- Start/stop containers khi cần
- Cleanup khi không dùng

### Câu hỏi

**9.1.** Setup development containers:
- Tạo nginx container với port mapping
- Tạo MySQL container với environment variables
- Tạo app container với volumes
- Setup restart policies

**9.2.** Daily workflow:
- Start containers vào buổi sáng
- Stop containers vào buổi tối
- Restart containers khi cần
- Cleanup containers không dùng

### Scenario 2: Production Deployment

Bạn là DevOps và cần deploy containers lên production.

**Requirements:**
- Containers phải tự động restart
- Graceful shutdown
- Fast deployments

### Câu hỏi

**9.3.** Production setup:
- Setup restart policy cho production
- Test graceful shutdown
- Test auto-restart sau reboot
- Verify deployment process

**9.4.** Deployment process:
- Stop old container (graceful)
- Start new container
- Verify new container running
- Remove old container
- Viết script để automate

---

## 📝 BÀI TẬP 10: TROUBLESHOOTING

### Scenario

Bạn gặp các vấn đề với container lifecycle.

### Câu hỏi

**10.1.** Container không start:
```bash
$ docker start my-container
Error response from daemon: ...
```
- Root cause là gì?
- Làm thế nào debug?
- Làm thế nào fix?

**10.2.** Container không stop:
```bash
$ docker stop my-container
# Container vẫn running sau 10 seconds
```
- Root cause là gì?
- Làm thế nào debug?
- Làm thế nào fix?

**10.3.** Container không restart:
- Container exit nhưng không tự động restart
- Root cause là gì?
- Làm thế nào fix?

**10.4.** Container remove fail:
```bash
$ docker rm my-container
Error response from daemon: cannot remove a running container
```
- Root cause là gì?
- Làm thế nào fix?

---

## ✅ CHECKLIST HOÀN THÀNH

Trước khi xem solutions, đảm bảo bạn đã:

- [ ] Hiểu được container states
- [ ] Thực hành create, start, stop, restart, remove
- [ ] Hiểu được restart policies
- [ ] Thực hành pause/unpause
- [ ] Làm tất cả các bài tập trên
- [ ] Thực hành các commands trên terminal

---

## 💡 GỢI Ý

- **Practice**: Thực hành tất cả commands nhiều lần
- **Experiment**: Thử các options khác nhau
- **Observe**: Quan sát state changes
- **Document**: Document workflows cho team

---

**Chúc bạn làm bài tốt! Sau khi hoàn thành, hãy xem `solutions.md` để so sánh với đáp án.**

