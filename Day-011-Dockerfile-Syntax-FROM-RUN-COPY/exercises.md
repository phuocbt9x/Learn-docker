# Day-011: Bài Tập Thực Hành

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi làm các bài tập này, bạn sẽ:

- Viết được Dockerfile với FROM, RUN, COPY
- Hiểu được layer caching và cách optimize
- Chọn base image đúng
- Combine RUN commands hiệu quả
- Optimize COPY instructions
- Viết Dockerfile production-ready

---

## 📝 BÀI TẬP 1: DOCKERFILE BASICS

### Scenario

Bạn cần viết Dockerfile đầu tiên cho một Python application.

### Câu hỏi

**1.1.** Viết Dockerfile cơ bản:
- Base image: Python 3.9
- Copy `app.py` vào `/app/`
- Copy `requirements.txt` vào `/app/`
- Install dependencies từ requirements.txt
- Set working directory `/app`
- Run `python app.py`

**1.2.** Build và test:
- Build image từ Dockerfile
- Run container từ image
- Verify application chạy
- Viết commands

**1.3.** Giải thích:
- Mỗi instruction tạo bao nhiêu layers?
- Layers nào được cache?
- Làm thế nào optimize?

---

## 📝 BÀI TẬP 2: FROM INSTRUCTION

### Scenario

Bạn cần chọn base image cho các applications sau.

### Câu hỏi

**2.1.** Python application:
- Application: Python 3.9, Flask
- Requirements: Minimal dependencies
- Options: `python:3.9`, `python:3.9-slim`, `python:3.9-alpine`
- Chọn base image nào? Tại sao?
- So sánh sizes?

**2.2.** Node.js application:
- Application: Node.js 16, Express
- Requirements: Build tools cần thiết
- Options: `node:16`, `node:16-slim`, `node:16-alpine`
- Chọn base image nào? Tại sao?

**2.3.** Nginx static site:
- Application: Static HTML/CSS/JS
- Requirements: Chỉ serve files
- Options: `nginx:latest`, `nginx:alpine`, `nginx:1.21`
- Chọn base image nào? Tại sao?

**2.4.** Best practices:
- Tại sao không nên dùng `latest` tag?
- Khi nào dùng alpine? Khi nào dùng slim?
- Trade-offs của mỗi option?

---

## 📝 BÀI TẬP 3: RUN INSTRUCTION

### Scenario

Bạn có Dockerfile với nhiều RUN commands cần optimize.

### Câu hỏi

**3.1.** Optimize Dockerfile:
```dockerfile
FROM ubuntu:20.04
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get install -y python3-pip
RUN apt-get install -y curl
RUN apt-get install -y wget
RUN apt-get clean
```
- Combine RUN commands
- Remove cache
- Optimize
- Viết Dockerfile tối ưu

**3.2.** Shell form vs Exec form:
- So sánh 2 forms
- Khi nào dùng shell form?
- Khi nào dùng exec form?
- Ví dụ cụ thể

**3.3.** Combine commands:
- Tại sao cần combine?
- Làm thế nào combine đúng cách?
- Best practices?

**3.4.** Remove cache:
- Tại sao cần remove cache?
- Làm thế nào remove?
- Commands nào cần?

---

## 📝 BÀI TẬP 4: COPY INSTRUCTION

### Scenario

Bạn cần copy files vào image một cách hiệu quả.

### Câu hỏi

**4.1.** Copy strategy:
```dockerfile
FROM python:3.9-slim
# Copy gì trước? Copy gì sau?
# Tại sao order quan trọng?
```

**4.2.** Optimize COPY:
```dockerfile
# Bad
FROM python:3.9-slim
COPY . /app/
RUN pip install -r requirements.txt

# Good?
FROM python:3.9-slim
COPY requirements.txt /app/
RUN pip install -r requirements.txt
COPY . /app/
```
- Giải thích tại sao order này tốt hơn?
- Cache behavior khác nhau như thế nào?

**4.3.** .dockerignore:
- Tạo .dockerignore file
- Exclude: node_modules, .git, *.log, .env
- Tại sao quan trọng?
- Best practices?

**4.4.** COPY vs ADD:
- So sánh COPY và ADD
- Khi nào dùng COPY?
- Khi nào dùng ADD?
- Best practices?

---

## 📝 BÀI TẬP 5: LAYER CACHING

### Scenario

Bạn có Dockerfile build chậm, cần optimize caching.

### Câu hỏi

**5.1.** Analyze Dockerfile:
```dockerfile
FROM ubuntu:20.04
COPY . /app/
RUN apt-get update
RUN apt-get install -y python3 python3-pip
RUN pip3 install -r /app/requirements.txt
CMD ["python3", "/app/app.py"]
```
- Vấn đề gì với Dockerfile này?
- Cache behavior như thế nào?
- Làm thế nào optimize?

**5.2.** Optimize order:
- Sắp xếp lại instructions
- Maximize cache hits
- Viết Dockerfile tối ưu

**5.3.** Test caching:
- Build lần đầu: Time?
- Build lần 2 (không thay đổi): Time? Cache hits?
- Build lần 3 (chỉ thay đổi code): Time? Cache hits?
- So sánh với Dockerfile chưa optimize

**5.4.** Cache invalidation:
- Khi nào cache bị invalidate?
- Làm thế nào prevent unnecessary invalidation?
- Best practices?

---

## 📝 BÀI TẬP 6: PRACTICAL DOCKERFILE

### Scenario 1: Python Web Application

Bạn có Python Flask application:
- `app.py`: Main application
- `requirements.txt`: Dependencies
- `config/`: Configuration files
- `static/`: Static files
- `templates/`: HTML templates

### Câu hỏi

**6.1.** Viết Dockerfile:
- Chọn base image
- Install dependencies
- Copy files
- Set working directory
- Expose port
- Run application
- Viết Dockerfile production-ready

**6.2.** Optimize:
- Optimize layer caching
- Combine RUN commands
- Use .dockerignore
- Best practices

### Scenario 2: Node.js Application

Bạn có Node.js Express application:
- `package.json`: Dependencies
- `src/`: Source code
- `public/`: Public files
- Build process: `npm run build`

### Câu hỏi

**6.3.** Viết Dockerfile:
- Chọn base image
- Install dependencies
- Build application
- Copy files
- Run application
- Viết Dockerfile

**6.4.** Optimize:
- Separate build và runtime?
- Optimize layers?
- Best practices?

---

## 📝 BÀI TẬP 7: TROUBLESHOOTING

### Scenario

Bạn gặp các vấn đề khi build Dockerfile.

### Câu hỏi

**7.1.** Build fails:
```bash
$ docker build -t my-app .
Step 3/5 : RUN apt-get install -y package
 ---> Running in abc123...
E: Unable to locate package package
```
- Root cause là gì?
- Làm thế nào fix?
- Best practices?

**7.2.** Build chậm:
- Build mất 20 phút
- Mỗi lần build lại từ đầu
- Root cause?
- Làm thế nào optimize?

**7.3.** Image quá lớn:
- Image size: 2GB
- Chỉ cần 200MB
- Root cause?
- Làm thế nào giảm size?

**7.4.** Cache không work:
- Build lần 2 vẫn rebuild toàn bộ
- Cache không được dùng
- Root cause?
- Làm thế nào fix?

---

## 📝 BÀI TẬP 8: REFACTOR DOCKERFILE

### Scenario

Bạn có Dockerfile xấu cần refactor.

### Câu hỏi

**8.1.** Refactor Dockerfile:
```dockerfile
FROM ubuntu:latest
RUN apt-get update
RUN apt-get install -y python3
RUN apt-get install -y python3-pip
RUN apt-get install -y git
RUN apt-get install -y curl
RUN apt-get install -y wget
RUN apt-get install -y vim
RUN apt-get install -y htop
COPY . /app/
RUN pip3 install -r requirements.txt
CMD python3 app.py
```
- Identify issues
- Refactor
- Viết Dockerfile tối ưu

**8.2.** So sánh:
- Before: Size? Layers? Build time?
- After: Size? Layers? Build time?
- Improvements?

**8.3.** Best practices applied:
- Liệt kê best practices đã apply
- Giải thích mỗi practice
- Trade-offs?

---

## ✅ CHECKLIST HOÀN THÀNH

Trước khi xem solutions, đảm bảo bạn đã:

- [ ] Viết Dockerfile với FROM, RUN, COPY
- [ ] Build và test images
- [ ] Hiểu layer caching
- [ ] Optimize Dockerfile
- [ ] Làm tất cả các bài tập trên
- [ ] Thực hành build images trên terminal

---

## 💡 GỢI Ý

- **Practice**: Viết nhiều Dockerfiles khác nhau
- **Experiment**: Thử các base images khác nhau
- **Measure**: Measure build time, image size
- **Compare**: So sánh before/after optimization

---

**Chúc bạn làm bài tốt! Sau khi hoàn thành, hãy xem `solutions.md` để so sánh với đáp án.**

