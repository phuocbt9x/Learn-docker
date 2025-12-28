# Day-007: Bài Tập Thực Hành

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi làm các bài tập này, bạn sẽ:

- Pull được images từ Docker Hub và private registries
- Hiểu được image tags và cách quản lý
- Tag và retag images đúng cách
- Inspect images để xem thông tin chi tiết
- Quản lý images hiệu quả

---

## 📝 BÀI TẬP 1: PULL IMAGES

### Scenario

Bạn cần pull các images để sử dụng.

### Câu hỏi

**1.1.** Pull images từ Docker Hub:
- Pull `nginx` với tag `latest`
- Pull `nginx` với tag `1.21`
- Pull `nginx` với tag `alpine`
- So sánh sizes của 3 images này

**1.2.** Pull từ user/organization:
- Pull `ubuntu:20.04`
- Pull `python:3.9-slim`
- Giải thích image name format

**1.3.** Pull từ private registry:
- Giả sử bạn có private registry tại `registry.example.com`
- Pull image `registry.example.com/my-app:v1.0`
- Cần authentication không? Làm thế nào?

**1.4.** Pull process:
- Khi pull image, Docker làm gì?
- Layers nào được download?
- Tại sao một số layers "Already exists"?

---

## 📝 BÀI TẬP 2: IMAGE TAGS

### Scenario

Bạn cần hiểu và quản lý image tags.

### Câu hỏi

**2.1.** Tag naming:
- Giải thích các tags sau:
  - `nginx:latest`
  - `nginx:1.21`
  - `nginx:1.21-alpine`
  - `my-app:v1.0.0`
  - `my-app:production`
- Tag nào nên dùng trong production? Tại sao?

**2.2.** Latest tag:
- `latest` tag là gì?
- Tại sao không nên dùng `latest` trong production?
- Khi nào có thể dùng `latest`?

**2.3.** Tag conventions:
- Semantic versioning là gì?
- Environment tags (dev, staging, production) - khi nào dùng?
- Date tags - khi nào dùng?

**2.4.** Tag management:
- Làm thế nào để list tất cả tags của một image?
- Một image có thể có nhiều tags không?
- Tag có tốn storage không?

---

## 📝 BÀI TẬP 3: TAG IMAGES

### Scenario

Bạn cần tag images cho deployment.

### Câu hỏi

**3.1.** Tag local images:
- Tag `nginx:latest` thành `my-nginx:v1.0`
- Tag với registry: `registry.example.com/my-nginx:v1.0`
- Verify tags đã được tạo

**3.2.** Tag after build:
- Build image `my-app:latest`
- Tag thành `my-app:v1.0.0`
- Tag thành `my-app:production`
- Tag với registry
- Viết commands

**3.3.** Multiple tags:
- Một image có thể có nhiều tags không?
- Tạo 3 tags cho cùng một image
- Verify tất cả tags point đến cùng image ID
- Remove một tag - image có bị xóa không?

**3.4.** Retag images:
- Retag `nginx:1.21` thành `nginx:stable`
- Retag `my-app:v1.0` thành `my-app:latest`
- Giải thích retag process

---

## 📝 BÀI TẬP 4: INSPECT IMAGES

### Scenario

Bạn cần inspect images để hiểu cấu trúc và thông tin.

### Câu hỏi

**4.1.** Basic inspect:
- Inspect `nginx:latest`
- Giải thích các fields quan trọng trong output
- Image ID là gì? Digest là gì?

**4.2.** Inspect specific fields:
- Lấy Image ID
- Lấy Created date
- Lấy Size
- Lấy Architecture
- Lấy Environment variables
- Lấy Exposed ports
- Lấy CMD
- Viết commands với format

**4.3.** Image history:
- Xem history của `nginx:latest`
- Giải thích output
- Layers nào lớn nhất? Tại sao?
- Có thể xem command tạo mỗi layer không?

**4.4.** Compare images:
- Inspect `nginx:latest` và `nginx:alpine`
- So sánh sizes
- So sánh layers
- Tại sao alpine nhỏ hơn?

---

## 📝 BÀI TẬP 5: IMAGE MANAGEMENT

### Scenario

Bạn cần quản lý images trên local system.

### Câu hỏi

**5.1.** List images:
- List tất cả images
- List images với filter (name, tag)
- List dangling images
- List với custom format

**5.2.** Remove images:
- Remove một image
- Remove image by ID
- Force remove image (đang dùng bởi container)
- Remove multiple images
- Remove all unused images

**5.3.** Image size:
- Check size của tất cả images
- Check detailed size breakdown
- Tại sao VirtualSize khác Size?
- Làm thế nào để giảm image sizes?

**5.4.** Image search:
- Search "nginx" trên Docker Hub
- Filter official images
- Filter automated images
- Limit results

---

## 📝 BÀI TẬP 6: PRACTICAL SCENARIOS

### Scenario 1: Development Workflow

Bạn là developer và cần quản lý images cho development.

**Requirements:**
- Pull base images (python, node, etc.)
- Tag images với version
- Inspect images để hiểu cấu trúc
- Cleanup unused images

### Câu hỏi

**6.1.** Setup development images:
- Pull `python:3.9-slim`
- Pull `node:16-alpine`
- Tag với tên dễ nhớ
- Verify images

**6.2.** Image inspection:
- Inspect `python:3.9-slim`
- Xem environment variables
- Xem exposed ports
- Xem CMD/ENTRYPOINT

**6.3.** Cleanup:
- Remove images không dùng
- Check disk usage
- Best practices?

### Scenario 2: Production Deployment

Bạn là DevOps và cần quản lý images cho production.

**Requirements:**
- Tag strategy cho production
- Version pinning
- Image verification
- Registry management

### Câu hỏi

**6.4.** Tag strategy:
- Thiết kế tag strategy cho production
- Semantic versioning
- Environment tags
- Best practices?

**6.5.** Version pinning:
- Tại sao cần pin versions?
- Làm thế nào pin versions trong deployment?
- Làm thế nào track versions?

---

## 📝 BÀI TẬP 7: IMAGE LAYERS

### Scenario

Bạn muốn hiểu sâu về image layers.

### Câu hỏi

**7.1.** Layer structure:
- Inspect `nginx:latest`
- Xem history để hiểu layers
- Layers nào là base image?
- Layers nào là application-specific?

**7.2.** Layer sharing:
- Pull `nginx:latest` và `nginx:alpine`
- So sánh layers
- Layers nào được share?
- Storage saved là bao nhiêu?

**7.3.** Layer optimization:
- Làm thế nào để giảm số layers?
- Làm thế nào để giảm layer size?
- Best practices?

**7.4.** Layer inspection:
- Inspect từng layer
- Xem command tạo layer
- Xem size của mỗi layer
- Identify optimization opportunities

---

## 📝 BÀI TẬP 8: TROUBLESHOOTING

### Scenario

Bạn gặp các vấn đề với images.

### Câu hỏi

**8.1.** Pull failures:
```bash
$ docker pull nginx
Error response from daemon: Get "https://registry-1.docker.io/v2/": net/http: request canceled
```
- Root cause là gì?
- Làm thế nào debug?
- Làm thế nào fix?

**8.2.** Tag not found:
```bash
$ docker pull nginx:999.999
Error response from daemon: manifest for nginx:999.999 not found
```
- Root cause là gì?
- Làm thế nào check available tags?
- Làm thế nào fix?

**8.3.** Image too large:
- Image quá lớn → pull chậm
- Làm thế nào giảm image size?
- Làm thế nào optimize layers?

**8.4.** Tag confusion:
- Deploy `my-app:latest` → unexpected behavior
- Root cause: latest tag changed
- Làm thế nào prevent?
- Best practices?

---

## ✅ CHECKLIST HOÀN THÀNH

Trước khi xem solutions, đảm bảo bạn đã:

- [ ] Pull được images từ Docker Hub
- [ ] Hiểu được image tags
- [ ] Tag và retag images
- [ ] Inspect images để xem thông tin
- [ ] Quản lý images (list, remove, cleanup)
- [ ] Làm tất cả các bài tập trên
- [ ] Thực hành các commands trên terminal

---

## 💡 GỢI Ý

- **Practice**: Thực hành pull, tag, inspect nhiều images
- **Experiment**: Thử các options khác nhau
- **Compare**: So sánh sizes, layers của different images
- **Document**: Document tag strategy cho projects

---

**Chúc bạn làm bài tốt! Sau khi hoàn thành, hãy xem `solutions.md` để so sánh với đáp án.**

