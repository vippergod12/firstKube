# 🚀 Hướng Dẫn Triển Khai Hệ Thống (Deployment Guide)

**Tác giả:** Đặng Nhật Tiến  
**Vị trí:** Software Engineer / AI Engineer  
**Dự án:** DATN_AILMS

---

## 💻 1. Thiết Lập Môi Trường Phát Triển
Hệ thống được triển khai trên môi trường **WSL2 (Ubuntu)** thông qua PowerShell để đảm bảo tính đồng nhất.

1.  Mở **PowerShell** với quyền **Administrator**.
2.  Chuyển sang môi trường Linux:
    ```bash
    wsl -d Ubuntu
    ```
3.  Di chuyển vào thư mục dự án:
    ```bash
    cd /path/to/DATN_AILMS
    ```

---

## 🐳 2. Dockerization (Đóng gói ứng dụng)

Tôi thực hiện tạo các `Dockerfile` riêng biệt cho Backend và Frontend để tối ưu hóa việc quản lý tài nguyên.

### 🍃 Backend (Spring Boot)
Tạo `Dockerfile` tại thư mục gốc Backend bằng lệnh `nano Dockerfile`.

**Cấu trúc Dockerfile Backend:**
<div align="center">
    <img width="416" height="221" alt="Cấu trúc Dockerfile FE" src="https://github.com/user-attachments/assets/3599b3b4-26d1-4dc1-9c6c-ed0a9724316e" />

</div>

* **FROM:** Lựa chọn base image Java phù hợp.
* **WORKDIR:** Thiết lập môi trường làm việc trong container.
* **COPY & RUN:** Sao chép `pom.xml`, tải dependency và đóng gói ứng dụng bằng lệnh `mvn clean package -DskipTests`.

### 🌐 Frontend
Tương tự, tạo `Dockerfile` cho Frontend để đóng gói mã nguồn giao diện:
<div align="center">
  <img width="408" height="303" alt="Cấu trúc Dockerfile BE" src="https://github.com/user-attachments/assets/7cb51377-fb1d-4394-8e28-48cf5dcc4184" />
</div>

> **Lệnh Build:** `docker build -t <tên_image> .`

---

## 🏗️ 3. Điều Phối Dịch Vụ Với Docker Compose
Tại thư mục gốc, tôi sử dụng **Docker Compose** để quản lý đồng thời cả hai dịch vụ Backend và Frontend trong cùng một mạng nội bộ (`ailms_network`).

**Cấu hình `docker-compose.yml`:**
<div align="center">
  <img width="962" height="406" alt="Cấu hình Docker Compose" src="https://github.com/user-attachments/assets/cfad5fe8-c36b-401f-af05-d4323fe12aa2" />
</div>

* **Lệnh khởi chạy:**
    ```bash
    docker-compose up --build
    ```

---

## 📤 4. Lưu Trữ Image Trên Docker Hub
Sau khi kiểm tra ổn định, các Image được đẩy lên Docker Hub để phục vụ quá trình triển khai K8s:

1.  Đăng nhập: `docker login`
2.  Đẩy Image:
    * `docker push nhattien/ailms-fe:v1`
    * `docker push nhattien/ailms-be:v1`

---

## ☸️ 5. Triển Khai Kubernetes Với Kind
Do cấu hình máy tính, tôi lựa chọn **Kind** để giả lập cụm Cluster một cách nhẹ nhàng và hiệu quả.

### Thiết lập Kind:
```bash
# Tải và cấp quyền thực thi
curl -Lo ./kind [https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64](https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64)
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind

# Khởi tạo Cluster
kind create cluster --name kind
