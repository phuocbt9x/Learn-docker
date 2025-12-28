# Day-003: Bài Tập Thực Hành

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi làm các bài tập này, bạn sẽ:

- Hiểu được namespace hoạt động như thế nào
- Hiểu được cgroup hoạt động như thế nào
- Có thể debug container issues ở kernel level
- Có thể giải thích container isolation cho team
- Hiểu được tại sao container chỉ chạy trên Linux

---

## 📝 BÀI TẬP 1: HIỂU NAMESPACE

### Scenario

Bạn là DevOps Engineer và cần giải thích cho team về namespace.

### Câu hỏi

**1.1.** Giải thích PID namespace:
- Tại sao process trong container có thể là PID 1?
- Tại sao process trong container không thấy processes trên host?
- Vẽ diagram minh họa PID namespace.

**1.2.** Giải thích Network namespace:
- Tại sao 2 containers có thể bind cùng port 80?
- Container có thể access network của host không?
- Vẽ diagram minh họa Network namespace.

**1.3.** Liệt kê 7 loại namespace và giải thích mỗi loại làm gì.

**1.4.** Nếu một container KHÔNG có namespace, điều gì sẽ xảy ra? Liệt kê 3 vấn đề cụ thể.

---

## 📝 BÀI TẬP 2: HIỂU CGROUP

### Scenario

Bạn đang vận hành một server với 3 containers:

- Container A: Web app (cần 2GB RAM, 1 CPU)
- Container B: Database (cần 4GB RAM, 2 CPUs)
- Container C: Background worker (cần 1GB RAM, 0.5 CPU)

Server có: 8GB RAM, 4 CPUs

### Câu hỏi

**2.1.** Tính toán cgroup limits:
- Set memory limits cho mỗi container
- Set CPU limits cho mỗi container
- Đảm bảo tổng không vượt quá server resources

**2.2.** Nếu Container A vượt quá memory limit, điều gì sẽ xảy ra?
- Process nào sẽ bị kill?
- Containers khác có bị ảnh hưởng không?
- Host có bị ảnh hưởng không?

**2.3.** Nếu Container B vượt quá CPU limit, điều gì sẽ xảy ra?
- Container có bị kill không?
- Performance sẽ như thế nào?
- Containers khác có bị ảnh hưởng không?

**2.4.** Nếu KHÔNG set cgroup limits, điều gì sẽ xảy ra? Liệt kê 3 vấn đề cụ thể.

---

## 📝 BÀI TẬP 3: CONTAINER ESCAPE SCENARIO

### Scenario

Bạn phát hiện một container có thể access filesystem của host:

```bash
# Trong container
$ ls /host
bin  boot  dev  etc  home  lib  ...
# ❌ Có thể thấy host filesystem!
```

### Câu hỏi

**3.1.** Phân tích vấn đề:
- Namespace nào bị misconfigured?
- Tại sao container có thể access host filesystem?
- Security risk là gì?

**3.2.** Làm thế nào để fix vấn đề này?
- Cần configure namespace như thế nào?
- Cần thêm security measures gì?

**3.3.** Làm thế nào để prevent vấn đề này trong tương lai?
- Best practices là gì?
- Cần monitor gì?

**3.4.** Nếu attacker escape container và access host, họ có thể làm gì? Liệt kê 3 actions có thể.

---

## 📝 BÀI TẬP 4: OOM KILL TROUBLESHOOTING

### Scenario

Bạn đang vận hành production và containers bị OOM kill random:

```bash
$ docker ps -a
CONTAINER   STATUS
app-1       Exited (137) 2 minutes ago  # ← OOM kill
app-2       Running
app-3       Exited (137) 5 minutes ago  # ← OOM kill
```

**Server resources:**
- RAM: 16GB
- Containers: 10 containers, không có memory limits

### Câu hỏi

**4.1.** Phân tích root cause:
- Tại sao containers bị OOM kill?
- Tại sao OOM kill random (không chỉ container consume nhiều memory)?
- Cgroup nào bị thiếu?

**4.2.** Làm thế nào để fix?
- Cần set memory limits như thế nào?
- Làm thế nào để calculate limits cho mỗi container?

**4.3.** Sau khi fix, nếu một container vượt quá memory limit:
- Process nào sẽ bị kill?
- Containers khác có bị ảnh hưởng không?
- Có thể predict được không?

**4.4.** Best practices để prevent OOM kills:
- Làm thế nào để monitor memory usage?
- Làm thế nào để set limits đúng?

---

## 📝 BÀI TẬP 5: TẠO CONTAINER THỦ CÔNG

### Scenario

Bạn muốn hiểu container hoạt động như thế nào bằng cách tạo container thủ công (không dùng Docker).

### Câu hỏi

**5.1.** Liệt kê các bước để tạo container thủ công:
- Cần tạo namespaces nào?
- Cần setup cgroups như thế nào?
- Cần chroot như thế nào?

**5.2.** Viết commands để:
- Tạo PID namespace
- Tạo Network namespace
- Tạo Mount namespace
- Set memory limit (2GB)
- Set CPU limit (1 core)

**5.3.** So sánh container thủ công với Docker:
- Docker làm gì tự động?
- Tại sao nên dùng Docker thay vì tạo thủ công?

**5.4.** Nếu bạn tạo container thủ công nhưng quên setup một namespace, điều gì sẽ xảy ra? Ví dụ cụ thể.

---

## 📝 BÀI TẬP 6: TẠI SAO CONTAINER CHỈ CHẠY TRÊN LINUX?

### Scenario

Một developer hỏi: "Tại sao Linux container không chạy được trên Windows host?"

### Câu hỏi

**6.1.** Giải thích ở kernel level:
- Container cần kernel features gì?
- Windows kernel có những features này không?
- Tại sao không thể chạy Linux container trên Windows?

**6.2.** Docker Desktop trên macOS hoạt động như thế nào?
- Containers thực sự chạy ở đâu?
- Tại sao cần Linux VM?
- Performance impact là gì?

**6.3.** Windows containers:
- Windows containers chạy trên đâu?
- Có thể chạy Windows container trên Linux host không?
- Tại sao?

**6.4.** Nếu bạn cần chạy Linux container trên Windows host, giải pháp là gì?
- Có thể làm được không?
- Trade-offs là gì?

---

## 📝 BÀI TẬP 7: RESOURCE CONTENTION

### Scenario

Bạn có 5 containers chạy trên một server:

- Server: 8GB RAM, 4 CPUs
- Containers: Không có resource limits

**Vấn đề:**
- Container A (web app) đột nhiên chậm
- Container B (database) queries timeout
- Container C (background worker) không chạy

**Investigation:**
```bash
$ docker stats
CONTAINER   CPU %   MEM USAGE   MEM %
app-a       15%     1.5GB       19%
db-b        180%    6.5GB       81%  # ← Consume hết CPU và memory!
worker-c    5%      0.5GB       6%
...
```

### Câu hỏi

**7.1.** Phân tích vấn đề:
- Container nào gây ra vấn đề?
- Tại sao containers khác bị ảnh hưởng?
- Cgroup nào cần được set?

**7.2.** Làm thế nào để fix?
- Set limits cho Container B
- Đảm bảo containers khác có resources
- Tính toán limits như thế nào?

**7.3.** Sau khi fix:
- Container B vẫn consume nhiều resources nhưng không ảnh hưởng containers khác
- Giải thích tại sao?

**7.4.** Best practices để prevent resource contention:
- Làm thế nào để monitor resources?
- Làm thế nào để set limits đúng?

---

## 📝 BÀI TẬP 8: SECURITY ANALYSIS

### Scenario

Bạn đang review security của container setup:

**Current setup:**
- Containers chạy với root user (UID 0)
- Không có User namespace
- Không có capabilities restrictions
- Mount namespace cho phép mount host filesystem

### Câu hỏi

**8.1.** Phân tích security risks:
- Liệt kê 5 security risks của setup này
- Nếu container bị hack, attacker có thể làm gì?

**8.2.** Làm thế nào để harden security?
- Cần setup namespaces như thế nào?
- Cần setup cgroups như thế nào?
- Cần thêm security measures gì?

**8.3.** User namespace:
- User namespace làm gì?
- Tại sao quan trọng cho security?
- Làm thế nào để enable?

**8.4.** Capabilities:
- Capabilities là gì?
- Làm thế nào để drop capabilities không cần thiết?
- Ví dụ: Container chỉ cần network, không cần mount → drop CAP_SYS_ADMIN

---

## ✅ CHECKLIST HOÀN THÀNH

Trước khi xem solutions, đảm bảo bạn đã:

- [ ] Đọc kỹ `theory.md`
- [ ] Hiểu được namespace là gì và tại sao quan trọng
- [ ] Hiểu được cgroup là gì và tại sao quan trọng
- [ ] Hiểu được container hoạt động ở kernel level như thế nào
- [ ] Làm tất cả các bài tập trên
- [ ] Viết câu trả lời chi tiết (không chỉ đáp án ngắn gọn)

---

## 💡 GỢI Ý

- **Think like kernel developer**: Hiểu container ở mức kernel, không chỉ user level
- **Security mindset**: Luôn nghĩ về security implications
- **Practical examples**: Dùng ví dụ cụ thể để giải thích
- **Trade-offs**: Hiểu được trade-offs của mỗi approach

---

**Chúc bạn làm bài tốt! Sau khi hoàn thành, hãy xem `solutions.md` để so sánh với đáp án.**

