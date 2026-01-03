bật Powershell với adminstrator sau đó chạy lệnh wsl -d Ubuntu để chuyển sang linux command line
cd vào folder chứa dự án
tạo Dockerfile trong các folder chứa dự án gốc
- BE spring thì tạo Dockerfile cùng cấp với dự án bằng câu lệnh: nano Dockerfile
Dockerfile có cấu trúc:
FROM chọn image phù hợp với dự án
WORKDIR tạo thư mục bên trong container
COPY pom.xml do BE thiết kế bằng springboot nên chép file pom vào thư mục mới tạo ở trên
RUN  mvn dependency -> tải tất cả các thư viện
COPY src ./src copy toàn bộ code vào container
RUN mvn clean package -DskipTests -> đóng gói bỏ qua test
EXPOSE chọn cổng
ENTYPOINT các lệnh sẽ chạy khi container start
- FE spring tạo tương tự với câu lệnh nano Dockerfile
=> Build 2 file với lệnh docker build -t 

Sau khi tạo xong Dockerfile mình sẽ chuyển về thư mục góc chứa 2 folder dự án

nano docker-compose.yml.
Mình sẽ tạo services ở đây gồm frontend và backend:
services:
  backend:
    build:
      context: ./DATN_AILMS_BE/DATN_AILMS_BE
      dockerfile: Dockerfile
    container_name: ailms_be_container
    ports:
      - "8080:8080"
    environment:
      - SPRING_DATASOURCE_URL=jdbc:postgresql://ep-shy-union-a1ob30sd-pooler.ap-southeast-1.aws.neon.tech:5432/neondb?s>      - SPRING_DATASOURCE_USERNAME=neondb_owner
      - SRPING_DATASOURCE_PASSWORD=npg_mfYR0sAwu6LN
    networks:
      - ailms_network
  frontend:
    build:
      context: ./DATN_AILMS/DATN_AILMS_FE
      dockerfile: Dockerfile
    container_name: ailms_fe_container
    ports:
      - "80:80"
    depends_on:
      - backend
    networks:
      - ailms_network
networks:
  ailms_network:
    driver: bridge

sau đó build docker-compose bằng câu lệnh docker-compose up --build

docker login sau đó

push lên dockerhub: 
- docker push nhattien/ailms-fe:v1
- docker push nhattien/ailms-be:v1

từ thư mục gốc sẽ tạo ra thư mục k8s bằng lệnh mkdir k8s
sau đó tạo nano fe-deployment.yaml và be-deployment.yaml
sau khi thiết lập config cho 2 file, sẽ dùng lệnh kubectl apply -f tên các file deployment và chạy kubectl get pods để check reuslt
* Có thể tạo kind deploy và Service trong cùng 1 file deployment.yaml
* có thể dùng câu lệnh kubectl apply -f k8s/ để chạy toàn bộ file trong folder k8s
Do máy yếu nên sẽ dùng kind, 
chạy lệnh: curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
và chmod +x ./kind để cấp quyền
dùng lệnh sudo mv ./kind /usr/local/bin/kind để gọi lệnh
sau đó sẽ tạo cluster bằng câu lệnh kind create cluster --name kind <- có thể đổi tên.
dùng các câu lệnh kubectl port-forward svc/<tên service được tạo ở file deployment.yaml> <port>:<port lắng nghe>
