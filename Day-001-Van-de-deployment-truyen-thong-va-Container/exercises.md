# Day-001: Bài Tập Thực Hành

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi làm các bài tập này, bạn sẽ:

- Phân tích được vấn đề của deployment truyền thống
- Hiểu được khi nào nên dùng container
- Đánh giá được trade-offs của các approaches

---

## 📝 BÀI TẬP 1: PHÂN TÍCH VẤN ĐỀ DEPLOYMENT

### Scenario

Bạn là DevOps Engineer tại một công ty startup. Hiện tại công ty đang deploy applications theo cách truyền thống:

**Production Server:**
```
Server: ubuntu-prod-01
├── Web App (Python 3.8, Django 2.2)
├── API Service (Python 3.10, FastAPI)
├── Background Worker (Python 3.9, Celery)
├── Admin Panel (Node.js 14, Express)
└── Analytics Service (Node.js 18, NestJS)
```

**Vấn đề đang gặp:**
1. Hôm qua, team deploy API Service mới (cần Python 3.10)
2. Sau khi deploy, Web App bắt đầu crash với lỗi: `ImportError: cannot import name 'X'`
3. Team phát hiện Python 3.10 đã overwrite một số system libraries mà Web App đang dùng

### Câu hỏi

**1.1.** Vấn đề root cause là gì? Giải thích tại sao.

**1.2.** Nếu bạn là DevOps Engineer, bạn sẽ giải quyết vấn đề này như thế nào trong ngắn hạn (không dùng container)?

**1.3.** Giải pháp dài hạn là gì? Tại sao container giải quyết được vấn đề này?

**1.4.** Liệt kê 3 hậu quả có thể xảy ra nếu không fix vấn đề này.

---

## 📝 BÀI TẬP 2: "IT WORKS ON MY MACHINE"

### Scenario

Team có 5 developers:

- **Dev A**: macOS Monterey, Python 3.11
- **Dev B**: Windows 11, Python 3.10
- **Dev C**: Ubuntu 22.04, Python 3.9
- **Dev D**: macOS Ventura, Python 3.12
- **Dev E**: Ubuntu 20.04, Python 3.8

**Production Server:**
- Ubuntu 18.04, Python 3.8

**Vấn đề:**
- Code của Dev A chạy perfect trên máy của anh ấy
- Code của Dev B cũng chạy tốt
- Nhưng khi deploy lên Production, application crash với lỗi: `AttributeError: 'X' object has no attribute 'Y'`

Sau khi debug, team phát hiện:
- Python 3.11 có feature mới mà Dev A đang dùng
- Python 3.8 (Production) không có feature này

### Câu hỏi

**2.1.** Tại sao code chạy trên máy Dev A nhưng fail trên Production?

**2.2.** Làm thế nào để đảm bảo tất cả developers dùng cùng Python version?

**2.3.** Giải thích tại sao container giải quyết được vấn đề "it works on my machine"?

**2.4.** Nếu không dùng container, bạn sẽ làm gì để prevent vấn đề này trong tương lai?

---

## 📝 BÀI TẬP 3: QUYẾT ĐỊNH CÓ NÊN DÙNG CONTAINER?

### Scenario

Bạn đang làm việc tại một công ty và cần quyết định có nên containerize các applications sau không:

**Application 1: Simple Python Script**
- Một script Python đơn giản chạy mỗi ngày một lần
- Không có dependencies phức tạp
- Chạy trên một server duy nhất
- Không cần scale

**Application 2: Microservices Architecture**
- 10 microservices
- Mỗi service có dependencies khác nhau
- Cần scale độc lập
- Deploy thường xuyên (multiple times per day)

**Application 3: Legacy Monolithic Application**
- Ứng dụng cũ, viết bằng Java 8
- Chạy trên dedicated server
- Rất ít khi deploy (vài tháng một lần)
- Team không có kinh nghiệm với container

**Application 4: Real-time Trading System**
- Latency cực kỳ quan trọng (microseconds matter)
- Cần direct kernel access
- Chạy trên bare metal servers
- Không thể chấp nhận overhead

### Câu hỏi

Với mỗi application, hãy:

**3.1.** Quyết định: **NÊN** hay **KHÔNG NÊN** containerize? Giải thích.

**3.2.** Liệt kê 2 lợi ích nếu containerize (nếu nên).

**3.3.** Liệt kê 2 rủi ro nếu containerize (nếu không nên).

**3.4.** Đề xuất alternative approach nếu không containerize.

---

## 📝 BÀI TẬP 4: PHÂN TÍCH PRODUCTION STORY

### Scenario

Đọc lại Production Story #1 trong `theory.md` (Dependency Conflict tại Startup Fintech).

### Câu hỏi

**4.1.** Nếu bạn là DevOps Engineer trong story đó, bạn sẽ làm gì khác đi?

**4.2.** Tại sao virtualenv không đủ để giải quyết vấn đề?

**4.3.** Tính toán cost savings:
- Trước: 3 servers × $200/server = $600/month
- Sau: 2 servers × $200/server = $400/month
- Tiết kiệm: $200/month

Nếu migration cost là $5,000 (developer time, testing, etc.), sau bao lâu thì ROI (Return on Investment)?

**4.4.** Liệt kê 3 risks khi migrate từ traditional deployment sang container. Làm thế nào để mitigate các risks này?

---

## 📝 BÀI TẬP 5: THIẾT KẾ GIẢI PHÁP

### Scenario

Bạn là DevOps Engineer mới join một công ty. Công ty đang có:

**Current Setup:**
- 20 applications chạy trên 10 servers
- Mỗi server chạy 2-3 applications
- Deploy bằng cách SSH vào server và chạy scripts
- Không có CI/CD
- Developers deploy trực tiếp lên production
- Thường xuyên có issues: "it works on my machine", dependency conflicts

**Requirements:**
- Giảm deployment time từ 2 giờ xuống < 30 phút
- Giảm deployment failures từ 40% xuống < 5%
- Đảm bảo consistency giữa dev, staging, production
- Không làm gián đoạn service hiện tại (zero downtime migration)

### Câu hỏi

**5.1.** Thiết kế migration plan từ traditional deployment sang container-based deployment.

**5.2.** Chia plan thành các phases. Mỗi phase nên làm gì?

**5.3.** Liệt kê 5 risks của migration plan và cách mitigate.

**5.4.** Làm thế nào để đảm bảo zero downtime trong quá trình migration?

**5.5.** Metrics nào bạn sẽ track để đo lường success của migration?

---

## 📝 BÀI TẬP 6: TRADE-OFFS ANALYSIS

### Scenario

Bạn cần quyết định giữa 2 approaches:

**Approach A: Traditional Deployment với Virtualenv**
- Dùng virtualenv để cô lập Python dependencies
- Mỗi application có virtualenv riêng
- Chạy trên shared server

**Approach B: Container-based Deployment**
- Mỗi application là một container
- Container có Python và dependencies riêng
- Chạy trên cùng server

### Câu hỏi

**6.1.** So sánh 2 approaches theo các tiêu chí:
- Isolation level
- Resource usage
- Deployment complexity
- Security
- Portability
- Performance overhead

**6.2.** Khi nào nên dùng Approach A? Khi nào nên dùng Approach B?

**6.3.** Liệt kê 3 use cases mà Approach A tốt hơn Approach B.

**6.4.** Liệt kê 3 use cases mà Approach B tốt hơn Approach A.

---

## ✅ CHECKLIST HOÀN THÀNH

Trước khi xem solutions, đảm bảo bạn đã:

- [ ] Đọc kỹ `theory.md`
- [ ] Hiểu được vấn đề của deployment truyền thống
- [ ] Hiểu được container giải quyết vấn đề gì
- [ ] Làm tất cả các bài tập trên
- [ ] Viết câu trả lời chi tiết (không chỉ đáp án ngắn gọn)

---

## 💡 GỢI Ý

- **Đừng vội xem solutions**: Cố gắng tự suy nghĩ trước
- **Viết ra giấy**: Viết câu trả lời giúp bạn nhớ lâu hơn
- **Nghiên cứu thêm**: Nếu không chắc, search thêm về topic đó
- **Think like Senior Engineer**: Không chỉ trả lời "đúng", mà còn giải thích "tại sao"

---

**Chúc bạn làm bài tốt! Sau khi hoàn thành, hãy xem `solutions.md` để so sánh với đáp án.**

