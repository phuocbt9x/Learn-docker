# Day-005: Bài Tập Thực Hành

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi làm các bài tập này, bạn sẽ:

- Phân biệt được Image và Container
- Hiểu được Image Layers và cách chúng hoạt động
- Hiểu được Container Filesystem (Copy-on-Write)
- Có thể tối ưu image size bằng cách optimize layers
- Debug được image/container filesystem issues

---

## 📝 BÀI TẬP 1: HIỂU IMAGE VS CONTAINER

### Scenario

Bạn là DevOps Engineer và cần giải thích cho team về Image và Container.

### Câu hỏi

**1.1.** Giải thích sự khác biệt giữa Image và Container:
- Image là gì? Container là gì?
- Mối quan hệ giữa chúng?
- Khi nào dùng Image? Khi nào dùng Container?

**1.2.** Vẽ diagram minh họa:
- Một Image với 3 layers
- 2 Containers được tạo từ Image đó
- Chỉ ra phần nào shared, phần nào unique

**1.3.** Giải thích lifecycle:
- Image lifecycle (build → push → pull → run)
- Container lifecycle (create → start → stop → delete)
- Điều gì xảy ra khi container delete? Image có bị ảnh hưởng không?

**1.4.** Nếu bạn có 10 containers chạy từ cùng một image:
- Storage usage: Bao nhiêu cho image? Bao nhiêu cho containers?
- Nếu một container modify file trong image, điều gì xảy ra?
- Containers khác có bị ảnh hưởng không?

---

## 📝 BÀI TẬP 2: IMAGE LAYERS

### Scenario

Bạn có Dockerfile sau:

```dockerfile
FROM ubuntu:20.04
RUN apt-get update
RUN apt-get install -y python3 python3-pip
COPY requirements.txt /app/
RUN pip3 install -r /app/requirements.txt
COPY . /app/
CMD ["python3", "/app/app.py"]
```

### Câu hỏi

**2.1.** Phân tích layers:
- Có bao nhiêu layers được tạo?
- Mỗi layer chứa gì?
- Kích thước ước tính của mỗi layer?

**2.2.** Layer caching:
- Build lần đầu: Layers nào được build?
- Build lần 2 (không thay đổi gì): Layers nào được cache?
- Build lần 3 (chỉ thay đổi app.py): Layers nào được rebuild?

**2.3.** Optimize Dockerfile:
- Làm thế nào để tối ưu layer caching?
- Sắp xếp lại instructions để maximize cache?
- Viết lại Dockerfile tối ưu.

**2.4.** Layer sharing:
- Nếu bạn có 2 images:
  - Image A: FROM ubuntu:20.04, install python3
  - Image B: FROM ubuntu:20.04, install nodejs
- Layers nào được share giữa 2 images?
- Storage saved là bao nhiêu?

---

## 📝 BÀI TẬP 3: COPY-ON-WRITE

### Scenario

Bạn có một image `my-app:1.0` và tạo 3 containers từ image đó.

### Câu hỏi

**3.1.** Giải thích Copy-on-Write:
- CoW hoạt động như thế nào?
- Khi nào file được copy?
- Tại sao CoW efficient?

**3.2.** Scenario: Container 1 modify file trong image:
- File `/app/config.json` trong image (read-only)
- Container 1 modify file này
- Điều gì xảy ra với file trong image?
- Điều gì xảy ra với file trong Container 1?
- Containers 2, 3 có bị ảnh hưởng không?

**3.3.** Scenario: Container 2 tạo file mới:
- Container 2 tạo file `/app/data.txt`
- File này được lưu ở đâu?
- Containers 1, 3 có thấy file này không?

**3.4.** Storage analysis:
- Image: 500MB (3 layers)
- Container 1: Modify 10 files (mỗi file 1MB)
- Container 2: Tạo 5 files mới (mỗi file 2MB)
- Container 3: Không modify gì
- Tính tổng storage usage?

---

## 📝 BÀI TẬP 4: TỐI ƯU IMAGE SIZE

### Scenario

Bạn có Dockerfile tạo image 2.5GB:

```dockerfile
FROM ubuntu:20.04
RUN apt-get update
RUN apt-get install -y \
    python3 \
    python3-pip \
    nodejs \
    npm \
    build-essential \
    git \
    vim \
    curl \
    wget \
    htop \
    net-tools
COPY . /app
RUN pip3 install -r requirements.txt
RUN npm install
RUN npm run build
CMD ["python3", "app.py"]
```

### Câu hỏi

**4.1.** Phân tích image size:
- Base image: ? MB
- Packages: ? MB
- Dependencies: ? MB
- Build artifacts: ? MB
- Application code: ? MB

**4.2.** Tối ưu Dockerfile:
- Dùng slim base image
- Chỉ install packages cần thiết
- Multi-stage build để remove build artifacts
- Optimize layer order
- Viết lại Dockerfile tối ưu

**4.3.** Estimate image size sau khi optimize:
- Base image: ? MB
- Packages: ? MB
- Dependencies: ? MB
- Application code: ? MB
- Total: ? MB (giảm bao nhiêu %?)

**4.4.** Trade-offs:
- Lợi ích của image nhỏ?
- Rủi ro của image quá nhỏ (thiếu packages)?
- Best practices?

---

## 📝 BÀI TẬP 5: CONTAINER DATA LOSS

### Scenario

Bạn có container chạy application lưu data vào `/app/data/`:

```bash
# Container chạy
$ docker run -d my-app:1.0
# Application process data và lưu vào /app/data/results.json
```

**Vấn đề:**
- Container restart → data mất
- Container delete → data mất
- **Root cause**: Không hiểu container filesystem là ephemeral

### Câu hỏi

**5.1.** Phân tích vấn đề:
- Tại sao data mất khi container restart?
- Data được lưu ở đâu trong container?
- Tại sao data không persistent?

**5.2.** Giải pháp:
- Làm thế nào để lưu data persistent?
- Dùng volumes như thế nào?
- Dùng bind mounts như thế nào?
- So sánh volumes vs bind mounts?

**5.3.** Implementation:
- Viết docker-compose.yml với volumes
- Viết docker run command với volumes
- Verify data persistent qua container restart

**5.4.** Best practices:
- Khi nào dùng volumes?
- Khi nào dùng bind mounts?
- Backup strategy?
- Data migration?

---

## 📝 BÀI TẬP 6: LAYER CACHING OPTIMIZATION

### Scenario

Bạn có Dockerfile build chậm (10 phút mỗi lần):

```dockerfile
FROM ubuntu:20.04
COPY . /app
RUN apt-get update
RUN apt-get install -y python3 python3-pip
RUN pip3 install -r /app/requirements.txt
RUN python3 /app/setup.py
CMD ["python3", "/app/app.py"]
```

**Vấn đề:**
- Mỗi lần thay đổi code → rebuild toàn bộ
- Không tận dụng layer cache
- Build time: 10 phút

### Câu hỏi

**6.1.** Phân tích vấn đề:
- Tại sao không cache được?
- Layers nào bị rebuild mỗi lần?
- Tại sao COPY . /app đặt đầu?

**6.2.** Optimize layer order:
- Sắp xếp lại instructions để maximize cache
- COPY dependencies trước code
- Viết lại Dockerfile tối ưu

**6.3.** Estimate build time sau khi optimize:
- Build lần đầu: ? phút
- Build lần 2 (không thay đổi): ? phút (cached)
- Build lần 3 (chỉ thay đổi code): ? phút (chỉ rebuild layers sau COPY)

**6.4.** Advanced optimization:
- Multi-stage build
- .dockerignore
- Layer combination
- Best practices?

---

## 📝 BÀI TẬP 7: DEBUG FILESYSTEM ISSUES

### Scenario

Bạn gặp vấn đề:

```bash
# Container không thấy file
$ docker exec container ls /app/config.json
ls: cannot access '/app/config.json': No such file or directory

# Nhưng file có trong image
$ docker run --rm my-app:1.0 ls /app/config.json
/app/config.json
```

### Câu hỏi

**7.1.** Phân tích vấn đề:
- Tại sao container không thấy file?
- File có trong image không?
- Container layer có override file không?

**7.2.** Debug steps:
- Check image layers
- Check container layer
- Check file permissions
- Check mount points

**7.3.** Common issues:
- File bị delete trong container layer
- File bị override bởi volume
- Permission issues
- Namespace issues

**7.4.** Solutions:
- Làm thế nào để fix?
- Prevent trong tương lai?
- Best practices?

---

## 📝 BÀI TẬP 8: IMAGE VS CONTAINER STORAGE

### Scenario

Bạn có:
- 1 Image: `my-app:1.0` (500MB, 5 layers)
- 10 Containers chạy từ image đó

### Câu hỏi

**8.1.** Tính toán storage:
- Image storage: ? MB
- Container storage (nếu không modify gì): ? MB
- Total storage: ? MB

**8.2.** Scenario: 5 containers modify files:
- Mỗi container modify 1 file (10MB mỗi file)
- Storage usage: ? MB
- So sánh với nếu mỗi container copy toàn bộ image?

**8.3.** Scenario: Container delete:
- Delete 1 container
- Storage freed: ? MB
- Image storage: ? MB (có thay đổi không?)

**8.4.** Storage optimization:
- Làm thế nào để giảm storage?
- Image optimization?
- Container cleanup?
- Best practices?

---

## ✅ CHECKLIST HOÀN THÀNH

Trước khi xem solutions, đảm bảo bạn đã:

- [ ] Đọc kỹ `theory.md`
- [ ] Hiểu được Image vs Container
- [ ] Hiểu được Layers và Copy-on-Write
- [ ] Hiểu được Container Filesystem
- [ ] Làm tất cả các bài tập trên
- [ ] Viết câu trả lời chi tiết (không chỉ đáp án ngắn gọn)

---

## 💡 GỢI Ý

- **Think about storage**: Hiểu cách Docker lưu images và containers
- **Optimize layers**: Layer order quan trọng cho caching
- **Understand CoW**: Copy-on-Write là key để hiểu container filesystem
- **Practice**: Thử build images, tạo containers để hiểu rõ hơn

---

**Chúc bạn làm bài tốt! Sau khi hoàn thành, hãy xem `solutions.md` để so sánh với đáp án.**

