# 🚀 Hướng Dẫn Triển Khai Hệ Thống (Deployment Guide)

**Tác giả:** Đặng Nhật Tiến  
**Vị trí:** Software Engineer  
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
```

### 📂 Chi tiết Manifest Files

<details>
<summary><b>📄 be-deployment.yaml (Click để xem code)</b></summary>
<div>
    <img width="327" height="501" alt="image" src="https://github.com/user-attachments/assets/bcfbbdd1-7463-4870-89a7-cb3b6d424ab9" />

</div>
</details>



<details>
    ## 📑 Giải thích chi tiết Manifest: fe-deployment.yaml

Dưới đây là phân tích kỹ thuật về cách cấu hình hạ tầng cho phần Frontend, giúp Mentor hiểu rõ cách các tài nguyên được liên kết trong cụm Cluster.

<details>
<summary><b>🔍 Xem giải thích chi tiết cấu hình (Deployment & Service)</b></summary>

### 1. Thành phần Deployment (Quản lý Pod)
Phần này định nghĩa cách Kubernetes tạo và duy trì các bản sao chạy ứng dụng Frontend.

| Thuộc tính | Ý nghĩa kỹ thuật |
| :--- | :--- |
| **`kind: Deployment`** | Khai báo đối tượng quản lý vòng đời của các Pod (bản sao ứng dụng). |
| **`metadata.name`** | Định danh duy nhất cho bản triển khai: `ailms-fe-deploy`. |
| **`replicas: 1`** | Duy trì 1 Pod hoạt động liên tục. Có thể tăng số này để Scale-up hệ thống. |
| **`selector`** | Cơ chế tìm kiếm: Quản lý tất cả các Pod có gắn nhãn (label) `app: ailms-fe`. |
| **`template`** | "Bản thiết kế" của Pod: Sử dụng Image `nhattien1101/ailms-fe:v1` từ Docker Hub. |
| **`imagePullPolicy: Always`** | Đảm bảo hệ thống luôn tải bản Image mới nhất mỗi khi Pod được khởi tạo lại. |
| **`containerPort: 80`** | Cổng mà ứng dụng Frontend bên trong Container đang lắng nghe. |



---

### 2. Thành phần Service (Quản lý Mạng)
Service đóng vai trò là một địa chỉ IP tĩnh và bộ cân bằng tải, cho phép truy cập vào các Pod vốn có IP thay đổi liên tục.

| Thuộc tính | Ý nghĩa kỹ thuật |
| :--- | :--- |
| **`kind: Service`** | Khai báo đối tượng quản lý giao tiếp mạng cho Pod. |
| **`type: NodePort`** | Cho phép truy cập từ bên ngoài Cluster bằng cách mở một cổng trên IP của máy chủ (Node). |
| **`selector`** | Rất quan trọng: Kết nối Service này tới đúng các Pod có nhãn `app: ailms-fe`. |
| **`port: 80`** | Cổng ảo mà Service đại diện trong mạng nội bộ. |
| **`targetPort: 80`** | Luồng dữ liệu sẽ được chuyển tiếp chính xác vào cổng 80 của Container. |



---

### 💡 Cơ chế liên kết "Label & Selector"
Điểm mấu chốt trong file này chính là nhãn **`app: ailms-fe`**. 
- **Deployment** gắn nhãn này lên các Pod nó tạo ra.
- **Service** dùng nhãn này để biết phải gửi dữ liệu cho Pod nào.
Đây là cơ chế giúp hạ tầng Kubernetes vận hành linh hoạt và tự động hóa cao.

</details>
    
</details>

<details> <summary><b>📄 fe-deployment.yaml (Click để xem code)</b></summary>
    <div>
        <img width="321" height="518" alt="image" src="https://github.com/user-attachments/assets/6855ffef-ebfb-4cd8-ba0e-543ffebe2ae0" />

    </div>
</details>
## 📑 Giải thích chi tiết Manifest: be-deployment.yaml

Tài liệu này phân tích cách cấu hình phần Backend (Spring Boot), tập trung vào khả năng quản lý container và giao tiếp nội bộ giữa các dịch vụ.

<details>
<summary><b>🔍 Xem giải thích chi tiết cấu hình (Backend Deployment & Service)</b></summary>

### 1. Thành phần Deployment (Quản lý ứng dụng Backend)
Deployment này đảm bảo ứng dụng Spring Boot luôn sẵn sàng xử lý các yêu cầu từ Frontend hoặc các dịch vụ khác.

| Thuộc tính | Ý nghĩa kỹ thuật |
| :--- | :--- |
| **`kind: Deployment`** | Khai báo đối tượng điều phối các Pod chạy mã nguồn Backend. |
| **`metadata.name`** | Tên định danh: `ailms-be-deploy`. |
| **`replicas: 1`** | Số lượng bản sao ứng dụng chạy đồng thời. |
| **`selector`** | Gắn kết Deployment với các Pod thông qua nhãn (label) `app: ailms-be`. |
| **`image`** | Sử dụng bản build Backend: `nhattien1101/ailms-be:v1`. |
| **`imagePullPolicy: Always`** | Kubernetes sẽ luôn kiểm tra và tải bản Image mới nhất từ Docker Hub mỗi khi Pod restart. |
| **`containerPort: 8080`** | Cổng mặc định của ứng dụng Spring Boot bên trong Container. |



---

### 2. Thành phần Service (Giao tiếp nội bộ)
Khác với Frontend dùng NodePort để ra ngoài, Backend thường sử dụng **ClusterIP** để bảo mật và tối ưu hóa kết nối trong mạng bộ.

| Thuộc tính | Ý nghĩa kỹ thuật |
| :--- | :--- |
| **`kind: Service`** | Khai báo điểm truy cập cố định cho các Pod Backend. |
| **`type: ClusterIP`** | (Mặc định) Chỉ cho phép các dịch vụ **bên trong** Cluster (như Frontend) truy cập vào Backend. Điều này giúp bảo mật mã nguồn và dữ liệu khỏi truy cập trực tiếp từ Internet. |
| **`selector`** | Định hướng lưu lượng đến đúng các Pod có nhãn `app: ailms-be`. |
| **`port: 8080`** | Cổng dịch vụ lắng nghe trong mạng nội bộ của Cluster. |
| **`targetPort: 8080`** | Cổng đích mà dữ liệu sẽ được chuyển vào bên trong Pod Backend. |



---

### 💡 Tại sao Backend dùng ClusterIP thay vì NodePort?
Trong kiến trúc microservices chuyên nghiệp:
1. **Bảo mật:** Backend không nên lộ diện trực tiếp ra Internet. Chỉ Frontend (đóng vai trò Proxy/Gateway) mới cần cổng mở (`NodePort` hoặc `LoadBalancer`) để người dùng truy cập.
2. **Hiệu năng:** `ClusterIP` giúp các dịch vụ giao tiếp với nhau qua mạng nội bộ của Kubernetes với độ trễ thấp nhất.
3. **Quản lý:** Giúp tránh việc lãng phí các cổng (port) trên máy chủ vật lý.

</details>


