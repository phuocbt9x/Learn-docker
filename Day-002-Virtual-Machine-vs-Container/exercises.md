# Day-002: Bài Tập Thực Hành

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi làm các bài tập này, bạn sẽ:

- So sánh được VM và Container một cách chi tiết
- Quyết định được khi nào nên dùng VM, khi nào nên dùng Container
- Hiểu được trade-offs và đưa ra quyết định đúng trong production
- Giải thích được lựa chọn của mình cho team/client

---

## 📝 BÀI TẬP 1: PHÂN TÍCH ARCHITECTURE

### Scenario

Bạn là DevOps Engineer và cần giải thích cho CTO về sự khác biệt giữa VM và Container.

### Câu hỏi

**1.1.** Vẽ architecture diagram cho:
- Virtual Machine (có Guest OS, Hypervisor)
- Container (chia sẻ Host OS kernel)

**1.2.** Giải thích tại sao Container nhẹ hơn VM? Liệt kê 3 lý do cụ thể.

**1.3.** Tại sao Container không thể chạy Windows container trên Linux host? Giải thích ở mức kernel level.

**1.4.** Nếu một application cần kernel module đặc biệt (ví dụ: custom driver), nên dùng VM hay Container? Tại sao?

---

## 📝 BÀI TẬP 2: SO SÁNH RESOURCE USAGE

### Scenario

Công ty bạn đang có 20 applications cần deploy. Mỗi application:
- Cần 500 MB RAM
- Cần 2 GB disk
- Cần 0.5 CPU core

Bạn cần quyết định: Deploy trên VM hay Container?

### Câu hỏi

**2.1.** Tính toán resources nếu dùng VM:
- Giả sử mỗi VM tốn 2 GB RAM cho OS
- Giả sử mỗi VM tốn 10 GB disk cho OS
- Mỗi VM có thể chạy tối đa 5 applications (do resource limits)
- Tính tổng RAM, Disk, số VMs cần

**2.2.** Tính toán resources nếu dùng Container:
- Giả sử mỗi container overhead 50 MB RAM
- Giả sử mỗi container overhead 200 MB disk
- Mỗi server có thể chạy tối đa 20 containers
- Tính tổng RAM, Disk, số servers cần

**2.3.** So sánh 2 approaches:
- Resource usage (RAM, Disk)
- Cost (giả sử mỗi server/VM = $100/month)
- Startup time (giả sử VM boot 3 phút, container start 2 giây)

**2.4.** Trong trường hợp nào thì VM approach tốt hơn? Trong trường hợp nào thì Container approach tốt hơn?

---

## 📝 BÀI TẬP 3: ISOLATION & SECURITY

### Scenario

Bạn đang thiết kế infrastructure cho một hệ thống xử lý payment (PCI-DSS compliant).

**Requirements:**
- Mỗi merchant (khách hàng) phải hoàn toàn isolated
- Không được share bất kỳ gì giữa các merchants
- Compliance: PCI-DSS Level 1
- Có 100 merchants

### Câu hỏi

**3.1.** Nếu dùng VM:
- Mỗi merchant = 1 VM riêng
- Tính cost (giả sử $200/VM/month)
- Đánh giá isolation level (1-10)
- Đánh giá compliance (Pass/Fail)

**3.2.** Nếu dùng Container:
- Mỗi merchant = 1 container riêng
- Tính cost (giả sử 10 containers/server, $200/server/month)
- Đánh giá isolation level (1-10)
- Đánh giá compliance (Pass/Fail)

**3.3.** PCI-DSS yêu cầu "hardware-level isolation" cho một số use cases. Giải thích:
- Hardware-level isolation là gì?
- VM có đáp ứng không? Tại sao?
- Container có đáp ứng không? Tại sao?

**3.4.** Nếu bạn là DevOps Engineer, bạn sẽ recommend approach nào? Giải thích quyết định của bạn.

---

## 📝 BÀI TẬP 4: PERFORMANCE ANALYSIS

### Scenario

Bạn đang benchmark một web application (Node.js) trên 3 environments:
- Native (chạy trực tiếp trên server)
- Virtual Machine
- Container

**Benchmark results:**

| Metric | Native | VM | Container |
|--------|--------|----|-----------|
| Requests/second | 10,000 | 9,000 | 9,900 |
| Latency (p50) | 10ms | 11ms | 10.1ms |
| Latency (p99) | 50ms | 55ms | 51ms |
| CPU usage | 80% | 85% | 81% |
| Memory usage | 2GB | 2.5GB | 2.1GB |

### Câu hỏi

**4.1.** Tính performance overhead:
- VM overhead: ?%
- Container overhead: ?%

**4.2.** Giải thích tại sao VM có overhead cao hơn Container?

**4.3.** Trong trường hợp nào thì overhead này quan trọng? Trong trường hợp nào thì không quan trọng?

**4.4.** Nếu application này là real-time trading system (latency cực kỳ quan trọng), bạn sẽ chọn approach nào? Tại sao?

---

## 📝 BÀI TẬP 5: USE CASE DECISION

### Scenario

Bạn là DevOps Engineer và cần quyết định dùng VM hay Container cho các use cases sau:

**Use Case 1: Microservices E-commerce Platform**
- 50 microservices
- Deploy multiple times per day
- Cần scale nhanh (auto-scaling)
- Traffic: 1M requests/day

**Use Case 2: Legacy Windows Application**
- Ứng dụng cũ viết bằng .NET Framework 4.0
- Chỉ chạy trên Windows Server 2012
- Deploy vài tháng một lần
- Không cần scale

**Use Case 3: Multi-OS Development Environment**
- Developers cần test trên Windows, Linux, macOS
- Cần snapshot/restore nhanh
- Chạy trên macOS host

**Use Case 4: High-Security Banking System**
- Core banking application
- Compliance: PCI-DSS, SOX
- Yêu cầu air-gapped (không internet)
- Mỗi tenant (bank) phải isolated hoàn toàn

**Use Case 5: CI/CD Pipeline**
- Build, test applications
- Cần consistency giữa CI và production
- Chạy hàng trăm builds mỗi ngày
- Cần fast startup

### Câu hỏi

Với mỗi use case, hãy:

**5.1.** Quyết định: **VM** hay **Container**? Giải thích.

**5.2.** Liệt kê 3 lý do tại sao chọn approach đó.

**5.3.** Liệt kê 2 limitations/rủi ro của approach đó.

**5.4.** Đề xuất alternative approach (nếu có) và so sánh trade-offs.

---

## 📝 BÀI TẬP 6: MIGRATION PLANNING

### Scenario

Công ty bạn hiện tại đang dùng VM cho tất cả applications:
- 30 VMs
- Cost: $15,000/month ($500/VM)
- Resource utilization: 25%
- Deployment time: 2 giờ
- Scaling time: 15 phút

Management muốn giảm cost và cải thiện deployment speed. Họ hỏi bạn: "Có nên migrate sang Container không?"

### Câu hỏi

**6.1.** Phân tích current state:
- Tính toán resource waste (bao nhiêu % resources không dùng)
- Identify pain points
- Estimate potential savings nếu migrate

**6.2.** Thiết kế migration plan:
- Chia thành phases
- Mỗi phase nên làm gì?
- Timeline estimate

**6.3.** Risk analysis:
- Liệt kê 5 risks khi migrate
- Cách mitigate mỗi risk

**6.4.** Cost-benefit analysis:
- Migration cost (developer time, testing, etc.)
- Monthly savings estimate
- ROI calculation
- Break-even point

**6.5.** Recommendation:
- Bạn có recommend migration không? Tại sao?
- Nếu có, timeline như thế nào?
- Nếu không, alternatives là gì?

---

## 📝 BÀI TẬP 7: HYBRID APPROACH

### Scenario

Công ty bạn có:
- 20 microservices (có thể containerize)
- 5 legacy Windows applications (cần VM)
- 3 high-security applications (compliance yêu cầu VM)

### Câu hỏi

**7.1.** Thiết kế hybrid architecture:
- VMs cho legacy và high-security apps
- Containers cho microservices
- Vẽ architecture diagram

**7.2.** Làm thế nào để manage cả VM và Container trong cùng infrastructure?

**7.3.** Liệt kê 3 challenges của hybrid approach và cách giải quyết.

**7.4.** So sánh hybrid approach với:
- Pure VM approach
- Pure Container approach
- Trade-offs của mỗi approach

---

## 📝 BÀI TẬP 8: TROUBLESHOOTING SCENARIO

### Scenario

Bạn đang vận hành một hệ thống với 50 containers chạy trên 5 VMs (Kubernetes cluster).

**Vấn đề:**
- Một container bị crash và không thể start lại
- Logs cho thấy: "Kernel panic" trong container
- Nhưng các containers khác trên cùng VM vẫn chạy bình thường

### Câu hỏi

**8.1.** Phân tích vấn đề:
- Kernel panic trong container có thể do nguyên nhân gì?
- Tại sao các containers khác không bị ảnh hưởng?

**8.2.** Nếu đây là VM thay vì container, vấn đề sẽ khác như thế nào?

**8.3.** Làm thế nào để debug vấn đề này?

**8.4.** Làm thế nào để prevent vấn đề này trong tương lai?

---

## ✅ CHECKLIST HOÀN THÀNH

Trước khi xem solutions, đảm bảo bạn đã:

- [ ] Đọc kỹ `theory.md`
- [ ] Hiểu được sự khác biệt giữa VM và Container
- [ ] Hiểu được isolation levels
- [ ] Làm tất cả các bài tập trên
- [ ] Viết câu trả lời chi tiết (không chỉ đáp án ngắn gọn)
- [ ] Vẽ diagrams nếu cần (architecture, resource usage)

---

## 💡 GỢI Ý

- **Think like architect**: Không chỉ trả lời "đúng", mà còn giải thích "tại sao"
- **Consider trade-offs**: Mỗi approach có ưu/nhược điểm, quan trọng là chọn đúng cho use case
- **Real-world context**: Nghĩ về production scenarios thực tế
- **Cost vs Benefits**: Đôi khi cost không phải là yếu tố quan trọng nhất

---

**Chúc bạn làm bài tốt! Sau khi hoàn thành, hãy xem `solutions.md` để so sánh với đáp án.**

