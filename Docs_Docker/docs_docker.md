# 📦 Docker Notes

## 1. Các câu lệnh Docker cơ bản

| Lệnh                                                    | Mô tả                                       |
| ------------------------------------------------------- | ------------------------------------------- |
| `docker ps` / `docker ps -a`                            | Hiển thị các container (đang chạy / tất cả) |
| `docker images`                                         | Hiển thị các images                         |
| `docker rm -f <container_id\|name>`                     | Xóa container                               |
| `docker rm -f $(docker ps -a)`                          | Xóa toàn bộ container trên server           |
| `docker rmi -f <images_id\|name>`                       | Xóa image                                   |
| `docker start <container_id>`                           | Chạy container                              |
| `docker stop <container_id>`                            | Dừng container                              |
| `docker system df`                                      | Kiểm tra dung lượng trước khi dọn           |
| `docker system prune -af`                               | Xóa các container không dùng                |
| `docker builder prune -af`                              | Xóa cache build                             |
| `sudo truncate -s 0 /var/lib/docker/containers/*/*.log` | Dọn log container                           |

---

## 2. Cài đặt Docker

**Bước 1–2:** Tạo folder `tools` → tạo folder `docker`

**Bước 3:** Tạo file `install-docker.sh`:

```bash
#!/bin/bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install -y docker-ce
sudo systemctl start docker
sudo systemctl enable docker
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker --version
docker-compose --version
```

**Bước 4:** Cấp quyền file:

```bash
chmod +x install-docker.sh
```

**Bước 5:** Chạy file (3 cách):

```bash
./install-docker.sh
sh install-docker.sh
bash install-docker.sh
```

---

## 3. Pull & chạy Docker Ubuntu / Nginx

```bash
docker pull ubuntu:22.04

# Khởi động lần đầu (chỉ dùng 1 lần)
docker run --name ubuntu -it ubuntu:22.04

# Truy cập lại môi trường đã tồn tại
docker exec -it <container_name> bash
docker exec -it <container_name> sh

# Chạy webserver nginx
docker run --name nginx -dp 9999:80 nginx
# => Mở port 9999 trên AWS => http://18.143.40.118:9999/

# Chạy test
docker run --name car-serv -dp 8888:80 elroydevops/car-serv
```

---

## 4. Keyword trong Dockerfile

| Keyword        | Ý nghĩa                                                                                                 |
| -------------- | ------------------------------------------------------------------------------------------------------- |
| `FROM node:20` | Kéo version image (vd nodejs) về server                                                                 |
| `WORKDIR`      | Chỉ định thư mục làm việc trong container                                                               |
| `COPY . .`     | Copy source code vào container (vế 1: vị trí source hiện tại, vế 2: vị trí trong container = `WORKDIR`) |
| `RUN`          | Chạy câu lệnh                                                                                           |
| `ENV`          | Khai báo biến môi trường                                                                                |
| `EXPOSE`       | Định nghĩa port container sử dụng, vd `EXPOSE 80` (khi chạy container map `7777:80`)                    |
| `CMD`          | Xác định lệnh và giá trị mặc định                                                                       |
| `ENTRYPOINT`   | Giữ nguyên 1 lệnh cố định, cho phép truyền thêm tham số khi chạy container                              |

### Tư duy viết Dockerfile tối ưu

- Chọn image đúng version của dự án
- Chọn image từ nguồn **official / verified**
- Ưu tiên base image **alpine** để nhẹ hơn

---

## 5. Build dự án Backend

### Cách 1 — Cơ bản

`Dockerfile`:

```dockerfile
# build stage
FROM maven:3.9.9-eclipse-temurin-8 as build
WORKDIR /app
COPY . .
RUN mvn install -DskipTests=true

# run stage
FROM amazoncorretto:8u402-alpine-jre
WORKDIR /run
COPY --from=build /app/target/*.jar /run/shoe-ShoppingCart-0.0.1-SNAPSHOT.jar
EXPOSE 80
ENTRYPOINT java -jar shoe-ShoppingCart-0.0.1-SNAPSHOT.jar
```

Chạy container:

```bash
docker run --name shoeshop -dp 8888:80 shoeshop:v1
```

### Cách 2 — Alpine + cài JDK thủ công

```bash
cp Dockerfile Dockerfile-v2
```

`Dockerfile-v2`:

```dockerfile
# build stage
FROM maven:3.9.9-eclipse-temurin-8 as build
WORKDIR /app
COPY . .
RUN mvn install -DskipTests=true

# run stage
FROM alpine:3.19
RUN apk update && apk add openjdk8
WORKDIR /run
COPY --from=build /app/target/*.jar /run/shoe-ShoppingCart-0.0.1-SNAPSHOT.jar
EXPOSE 80
ENTRYPOINT java -jar shoe-ShoppingCart-0.0.1-SNAPSHOT.jar
```

### Cách 3 — Chạy bằng user riêng (bảo mật hơn)

```bash
cp Dockerfile Dockerfile-v3
```

`Dockerfile-v3`:

```dockerfile
# build stage
FROM maven:3.9.9-eclipse-temurin-8 as build
WORKDIR /app
COPY . .
RUN mvn install -DskipTests=true

# run stage
FROM alpine:3.19
RUN adduser -D shoeshop
RUN apk update && apk add openjdk8
WORKDIR /run
COPY --from=build /app/target/*.jar /run/shoe-ShoppingCart-0.0.1-SNAPSHOT.jar
RUN chown -R shoeshop:shoeshop /run
USER shoeshop
EXPOSE 80
ENTRYPOINT java -jar shoe-ShoppingCart-0.0.1-SNAPSHOT.jar
```

---

## 6. Build dự án Frontend

```bash
# Copy project lên server
scp -i C:\Users\Admin\Downloads\ec2-server.pem F:\DevOps_Trainning\Docs_Devops\Project\todolist.zip ubuntu@18.141.224.15:/home/ubuntu

# Di chuyển vào thư mục gốc
mv /home/ubuntu/todolist.zip /data/

# Giải nén
unzip todolist.zip
```

`Dockerfile`:

```dockerfile
# build stage
FROM node:18.18-alpine as build
WORKDIR /app
COPY . .
RUN npm install
RUN npm run build

# run stage
FROM nginx:alpine
# Copy folder build vào thư mục mặc định của nginx
COPY --from=build /app/dist /usr/share/nginx/html
CMD ["nginx", "-g", "daemon off;"]
```

---

## 7. Docker Registry

### Các phương án triển khai

- **Docker Hub**
- **Self-certified** — tự tạo registry private
- **Harbor** — dùng thực tế, private

### Đẩy image lên Docker Hub

```bash
docker login
docker tag todolist:v1 huynq203/todolist:v1
docker push huynq203/todolist:v1
docker pull huynq203/todolist:v1
```

### Cài đặt Docker Private Registry

```bash
# B1. Update thư viện
sudo apt-get update

# B2. Cài OpenSSL (xác thực https)
apt-get install openssl

# B3. Tạo key bằng OpenSSL
openssl req -newkey rsa:4096 -nodes -sha256 \
  -keyout certs/domain.key \
  -subj "/CN=47.129.222.142" \
  -addext "subjectAltName = DNS:47.129.222.142,IP:47.129.222.142" \
  -x509 -days 365 -out certs/domain.crt
```

**B4.** Tại folder registry, tạo `docker-compose.yml`:

```yaml
version: "3"

services:
  registry:
    image: registry:2
    restart: always
    container_name: registry-server
    ports:
      - "5000:5000"
    volumes:
      - ./data:/var/lib/registry
      - ./certs:/certs
    environment:
      REGISTRY_HTTP_TLS_CERTIFICATE: /certs/domain.crt
      REGISTRY_HTTP_TLS_KEY: /certs/domain.key
```

```bash
# B5. Chạy server
docker-compose up -d

# B6. Tạo thư mục chứng thực tự ký (do lỗi không chứng thực)
mkdir -p /etc/docker/certs.d/47.129.222.142:5000

# B7. Copy key domain sang folder vừa tạo
cp certs/domain.crt /etc/docker/certs.d/47.129.222.142:5000/ca.crt

# B8. Restart docker
systemctl restart docker

# B9. Login (username/password tùy ý do chưa cài xác thực)
docker login 47.129.222.142:5000

# B10. Tại server muốn pull image cũng phải xác thực https
mkdir -p /etc/docker/certs.d/47.129.222.142:5000
scp certs/domain.crt ubuntu@47.129.222.142:/home/ubuntu
cp /home/ubuntu/domain.crt /etc/docker/certs.d/47.129.222.142:5000/ca.crt
systemctl restart docker
```

---

## 8. Các thành phần chính của Docker

### Docker Volumes

Nơi Docker dùng để lưu dữ liệu bền vững cho container.

Ví dụ — mount volume cho MySQL container:

```bash
mkdir -p /db/mariadb-1
docker run -v /db/mariadb-1:/var/lib/mysql \
  -e MYSQL_ROOT_PASSWORD=root \
  -p 3307:3306 \
  --name mariadb-1 \
  -d mariadb:10.6
```

### Docker Compose

Tiếp tục dự án Backend — tạo `docker-compose.yml`:

```yaml
version: "3.8" # theo docker engine release
services:
  db1:
    image: mariadb:10.6
    volumes:
      - /db/mariadb-1:/var/lib/mysql # mount dữ liệu từ container ra server
    environment:
      MYSQL_ROOT_PASSWORD: root
    ports:
      - "3307:3306"
    container_name: mariadb-1
    restart: always # tự động restart service khi server restart
```

### Docker Network

_(chưa có nội dung chi tiết)_

---

## 9. Triển khai GitLab CI/CD cho dự án Docker

File `.gitlab-ci.yml`:

```yaml
# Tạo biến
variables:
  DOCKER_IMAGE: ${REGISTRY_URL}/${REGISTRY_PROJECT}/${CI_PROJECT_NAME}:${CI_COMMIT_TAG}_${CI_COMMIT_SHORT_SHA}
  # REGISTRY_URL, REGISTRY_PROJECT: tạo ở Settings -> CI/CD -> Variables
  # CI_PROJECT_NAME, CI_COMMIT_TAG, CI_COMMIT_SHORT_SHA: biến có sẵn của GitLab
  #   (tra cứu: "gitlab runner predefined variables")
  DOCKER_CONTAINER: ${CI_PROJECT_NAME}

# Luồng stages
stages:
  - buildandpush
  - deploy
  - showlog

buildandpush:
  stage: buildandpush
  variables:
    GIT_STRATEGY: clone # bước build sẽ clone code về server
  before_script:
    - docker login ${REGISTRY_URL} -u ${REGISTRY_USER} -p ${REGISTRY_PASSWORD}
    # REGISTRY_URL, REGISTRY_USER, REGISTRY_PASSWORD: tạo ở Settings -> CI/CD -> Variables
  script:
    - docker build -t $DOCKER_IMAGE .
    - docker push ${DOCKER_IMAGE}
  tags:
    - ec2-server # gắn tag thì mới chạy được
  only:
    - tags # chỉ chạy khi tạo tag trên GitLab

deploy:
  stage: deploy
  variables:
    GIT_STRATEGY: none
  before_script:
    - docker login ${REGISTRY_URL} -u ${REGISTRY_USER} -p ${REGISTRY_PASSWORD}
  script:
    - docker pull $DOCKER_IMAGE
    - docker rm -f $DOCKER_CONTAINER
    - docker run --name $DOCKER_CONTAINER -dp 8080:8080 $DOCKER_IMAGE
  tags:
    - ec2-server
  only:
    - tags

showlog:
  stage: showlog
  variables:
    GIT_STRATEGY: none
  script:
    - sleep 20
    - docker logs $DOCKER_CONTAINER
  tags:
    - ec2-server
  only:
    - tags
```

> **Lưu ý:** Trong bản gốc, `script` của stage `buildandpush` có `docker build` còn `before_script` có `docker push` — thứ tự này đã được sửa lại cho đúng logic (build trước, push sau) trong bản format này. Stage `deploy` trong bản gốc ghi nhầm là `de[loy]`, đã sửa thành `deploy`.
