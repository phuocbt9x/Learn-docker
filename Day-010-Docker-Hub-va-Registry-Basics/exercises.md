# Day-010: Bài Tập Thực Hành

## 🎯 MỤC TIÊU BÀI TẬP

Sau khi làm các bài tập này, bạn sẽ:

- Sử dụng Docker Hub để push/pull images
- Authenticate với Docker Hub và private registries
- Setup và sử dụng private registry
- Hiểu được image tags và versioning
- Apply best practices cho registry management

---

## 📝 BÀI TẬP 1: DOCKER HUB

### Scenario

Bạn cần sử dụng Docker Hub để share images.

### Câu hỏi

**1.1.** Docker Hub account:
- Tạo Docker Hub account (nếu chưa có)
- Login vào Docker Hub
- Verify login thành công
- Check credentials storage

**1.2.** Explore Docker Hub:
- Tìm 5 popular images
- So sánh official vs user images
- Check image tags
- Giải thích sự khác biệt

**1.3.** Rate limits:
- Docker Hub rate limits là gì?
- Anonymous vs Authenticated limits?
- Làm thế nào check current usage?
- Làm thế nào increase limits?

**1.4.** Public vs Private:
- Sự khác biệt giữa public và private repositories?
- Khi nào dùng public?
- Khi nào dùng private?
- Free tier limitations?

---

## 📝 BÀI TẬP 2: AUTHENTICATION

### Scenario

Bạn cần authenticate với Docker Hub và private registries.

### Câu hỏi

**2.1.** Docker login:
- Login vào Docker Hub
- Verify login
- Check credentials
- Logout

**2.2.** Login với token:
- Tạo access token trên Docker Hub
- Login với token
- So sánh với password login
- Security benefits?

**2.3.** Multiple registries:
- Login vào Docker Hub
- Login vào private registry
- Check credentials cho cả 2
- Logout từ specific registry

**2.4.** Credentials security:
- Credentials được lưu ở đâu?
- Làm thế nào secure credentials?
- Best practices?
- Không commit credentials vào Git?

---

## 📝 BÀI TẬP 3: PUSH IMAGES

### Scenario

Bạn cần push images lên Docker Hub.

### Câu hỏi

**3.1.** Tag image:
- Tag local image cho Docker Hub
- Tag format: `username/image:tag`
- Verify tag
- Giải thích tag format

**3.2.** Push image:
- Push image lên Docker Hub
- Verify image trên Docker Hub
- Check push process
- Giải thích push process

**3.3.** Push multiple tags:
- Tag image với nhiều tags (latest, v1.0.0, v1.0)
- Push tất cả tags
- Verify trên Docker Hub
- So sánh với push từng tag

**3.4.** Push to private registry:
- Setup local registry (hoặc dùng private registry)
- Tag image cho private registry
- Push image
- Verify push

---

## 📝 BÀI TẬP 4: PULL IMAGES

### Scenario

Bạn cần pull images từ Docker Hub và private registries.

### Câu hỏi

**4.1.** Pull từ Docker Hub:
- Pull public image
- Pull với specific tag
- Pull từ user repository
- So sánh pull times

**4.2.** Pull từ private registry:
- Login vào private registry
- Pull image từ private registry
- Verify image
- So sánh với Docker Hub

**4.3.** Pull by digest:
- Tìm digest của image
- Pull image bằng digest
- So sánh với pull by tag
- Khi nào nên dùng digest?

**4.4.** Pull process:
- Khi pull, Docker làm gì?
- Layers nào được download?
- Tại sao một số layers "Already exists"?
- Làm thế nào optimize pull time?

---

## 📝 BÀI TẬP 5: PRIVATE REGISTRY

### Scenario

Bạn cần setup private registry cho organization.

### Câu hỏi

**5.1.** Docker Registry (simple):
- Run Docker Registry container
- Push image to local registry
- Pull image from local registry
- Verify registry hoạt động

**5.2.** Registry với authentication:
- Setup registry với authentication
- Login vào registry
- Push/pull images
- Security considerations?

**5.3.** Registry options:
- So sánh Docker Registry vs Harbor
- Khi nào dùng Docker Registry?
- Khi nào dùng Harbor?
- Trade-offs?

**5.4.** Production registry:
- Thiết kế registry architecture cho production
- High availability?
- Backup strategy?
- Monitoring?

---

## 📝 BÀI TẬP 6: IMAGE TAGS & VERSIONING

### Scenario

Bạn cần quản lý image tags và versioning.

### Câu hỏi

**6.1.** Tag strategy:
- Thiết kế tag strategy cho project
- Semantic versioning?
- Environment tags?
- Date tags?
- Best practices?

**6.2.** Tag management:
- Tag image với version
- Tag image với multiple tags
- Update tag
- Remove tag
- Viết commands

**6.3.** Latest tag:
- Latest tag là gì?
- Tại sao không nên dùng latest trong production?
- Khi nào có thể dùng latest?
- Best practices?

**6.4.** Image digests:
- Digest là gì?
- Tại sao digest quan trọng?
- Pull image bằng digest
- Use digests trong production?
- Best practices?

---

## 📝 BÀI TẬP 7: PRACTICAL SCENARIOS

### Scenario 1: Development Workflow

Bạn là developer và cần push images cho team.

**Requirements:**
- Push images lên Docker Hub (public hoặc private)
- Tag với versions
- Share với team

### Câu hỏi

**7.1.** Development workflow:
- Build image
- Tag với version
- Push lên Docker Hub
- Share với team
- Viết workflow

**7.2.** CI/CD integration:
- Push images trong CI/CD pipeline
- Tag với build number
- Tag với git commit
- Automate tagging
- Viết script

### Scenario 2: Production Deployment

Bạn là DevOps và cần manage images cho production.

**Requirements:**
- Private registry
- Version pinning
- Security
- Monitoring

### Câu hỏi

**7.3.** Production setup:
- Setup private registry
- Configure authentication
- Setup versioning strategy
- Security hardening
- Viết plan

**7.4.** Deployment process:
- Pull images từ private registry
- Pin specific versions
- Verify images
- Rollback strategy
- Viết process

---

## 📝 BÀI TẬP 8: TROUBLESHOOTING

### Scenario

Bạn gặp các vấn đề với registry.

### Câu hỏi

**8.1.** Push failures:
```bash
$ docker push my-app:latest
denied: requested access to the resource is denied
```
- Root cause là gì?
- Làm thế nào fix?
- Authentication issues?

**8.2.** Pull failures:
```bash
$ docker pull my-app:latest
Error response from daemon: toomanyrequests
```
- Root cause là gì?
- Rate limiting?
- Làm thế nào fix?

**8.3.** Tag not found:
```bash
$ docker pull my-app:v999.999
Error: manifest for my-app:v999.999 not found
```
- Root cause là gì?
- Làm thế nào check available tags?
- Fix như thế nào?

**8.4.** Registry connectivity:
```bash
$ docker pull registry.example.com/my-app:latest
Error: Get "https://registry.example.com/v2/": dial tcp: lookup registry.example.com
```
- Root cause là gì?
- Network issues?
- DNS issues?
- Làm thế nào debug?

---

## ✅ CHECKLIST HOÀN THÀNH

Trước khi xem solutions, đảm bảo bạn đã:

- [ ] Tạo Docker Hub account
- [ ] Login vào Docker Hub
- [ ] Push image lên Docker Hub
- [ ] Pull image từ Docker Hub
- [ ] Setup private registry (nếu có thể)
- [ ] Hiểu được tags và versioning
- [ ] Làm tất cả các bài tập trên
- [ ] Thực hành các commands trên terminal

---

## 💡 GỢI Ý

- **Practice**: Thực hành push/pull nhiều images
- **Experiment**: Thử các registry options
- **Security**: Luôn secure credentials
- **Document**: Document tag strategy cho team

---

**Chúc bạn làm bài tốt! Sau khi hoàn thành, hãy xem `solutions.md` để so sánh với đáp án.**

**🎉 Chúc mừng! Bạn đã hoàn thành Phase 2: Core Docker Usage!**

