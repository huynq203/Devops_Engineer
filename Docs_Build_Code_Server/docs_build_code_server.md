# Cẩm nang triển khai dự án Backend

## I. Tư duy triển khai dự án

### 1. Mỗi dự án sẽ có công cụ tương ứng

Ví dụ:

| Loại dự án       | Công cụ            |
| ---------------- | ------------------ |
| NodeJS           | NodeJS, npm        |
| ReactJS          | NodeJS, npm, Nginx |
| Java Spring Boot | JDK, Maven         |
| Python           | Python, pip        |

---

### 2. Chỉ cần lưu ý duy nhất là **file cấu hình**

Mỗi dự án đều có file cấu hình riêng để chỉnh sửa:

- Port
- Database
- Username
- Password
- API URL
- Environment

---

### 3. Triển khai dự án luôn gồm 2 bước chính

```
Bước 1: Build
Bước 2: Run
```

---

## Lưu ý chung

- Mỗi dự án có **thư mục riêng**
- Mỗi dự án có **user Linux riêng**
- Phân quyền đúng cho user chạy service

---

# II. Quy trình triển khai Backend

## Bước 1. Cài đặt công cụ cần thiết

Ví dụ:

- NodeJS
- Java
- Maven
- MariaDB
- Nginx

---

## Bước 2. Kiểm tra và sửa file cấu hình

Ví dụ:

- `.env`
- `application.properties`
- `application.yml`
- `config.json`

---

## Bước 3. Cài đặt Database

- Cài Database
- Tạo Database
- Tạo User
- Import dữ liệu

---

## Bước 4. Build dự án

Build theo từng công nghệ.

---

## Bước 5. Run dự án

Có thể dùng:

- systemd
- pm2
- nohup
- Docker

---

## Bước 6. Kiểm tra hoạt động

Kiểm tra:

- Service
- Log
- Port
- Database
- API

---

# III. Build dự án NodeJS

## 1. Cài NodeJS

```bash
sudo apt install nodejs npm -y
```

---

## 2. Nâng cấp NodeJS lên version 18

```bash
curl -s https://deb.nodesource.com/setup_18.x | sudo bash
sudo apt install nodejs -y
```

Kiểm tra:

```bash
node -v
npm -v
```

---

## 3. Cài thư viện

```bash
npm install
```

---

## 4. Build

```bash
npm run build
```

---

## 5. Tạo User

```bash
adduser todolist
```

---

## 6. Phân quyền

```bash
chmod -R 777 /projects/todolist/dist/
```

---

## 7. Cấu hình Nginx

Tạo file `todolist.conf`:

```
vi /etc/nginx/conf.d/todolist.conf
```

Nội dung:

```nginx
server {
    listen 8081;

    root /home/huynq/projects/todolist/dist/;

    index index.html;

    try_files $uri $uri/ /index.html;
}
```

---

## 8. Kiểm tra cấu hình

```bash
sudo nginx -t
```

---

## 9. Reload Nginx

Khuyến nghị:

```bash
sudo systemctl reload nginx
```

Hoặc

```bash
sudo systemctl restart nginx
```

---

## 10. Thêm user vào group

```bash
sudo usermod -aG todolist www-data
```

Sau đó restart nginx.

---

## 11. Kiểm tra log

```bash
sudo tail -f /var/log/nginx/error.log
```

---

## 12. Kiểm tra cấu hình đang chạy

```bash
sudo nginx -T | grep -n "listen 8081"
```

---

# IV. Triển khai ReactJS

## Bước 1

- Copy project
- Giải nén
- Tạo user
- Gán quyền

---

## Bước 2

Tạo service:

```
/lib/systemd/system/vision.service
```

```ini
[Service]
Type=simple
User=vision
Restart=on-failure
WorkingDirectory=/home/ec2-huynq1/projects/vision/
ExecStart=npm run start -- --port=3000
```

---

## Bước 3

Reload systemd

```bash
sudo systemctl daemon-reload
```

---

## Bước 4

Start service

```bash
sudo systemctl start vision
```

---

## Bước 5

Kiểm tra

```bash
sudo systemctl status vision
```

---

# V. Triển khai Java Spring Boot

## Bước 1

- Copy project
- Giải nén
- Tạo user
- Gán quyền

---

## Bước 2. Kiểm tra Java Version

Mở:

```
pom.xml
```

Xem version Java.

---

## Bước 3. Cài Java

```bash
sudo apt update
sudo apt upgrade

sudo apt install openjdk-17-jdk -y
```

Kiểm tra:

```bash
java -version
```

---

## Bước 4. Cài Maven

```bash
sudo apt update
sudo apt upgrade

sudo apt install maven -y
```

Kiểm tra:

```bash
mvn -version
```

---

# VI. Cài đặt MariaDB

## 1. Cài MariaDB

```bash
sudo apt install mariadb-server -y
```

---

## 2. Cho phép kết nối từ xa

```bash
sudo systemctl stop mariadb
```

Sửa:

```
/etc/mysql/mariadb.conf.d/50-server.cnf
```

```ini
bind-address = 0.0.0.0
```

Khởi động lại:

```bash
sudo systemctl restart mariadb
```

---

## 3. Đăng nhập

```bash
mysql -u root
```

---

## 4. Tạo Database

```sql
CREATE DATABASE shoeshop;
```

---

## 5. Tạo User

```sql
CREATE USER 'shoeshop'@'%' IDENTIFIED BY 'shoeshop';
```

---

## 6. Cấp quyền

```sql
GRANT ALL PRIVILEGES
ON shoeshop.*
TO 'shoeshop'@'%';
```

---

## 7. Áp dụng

```sql
FLUSH PRIVILEGES;
```

Restart:

```bash
sudo systemctl restart mariadb
```

---

## 8. Đăng nhập Database

```bash
mysql -h 192.168.158.99 \
-P 3306 \
-u admin \
-p
```

---

## 9. Chọn Database

```sql
USE shoeshop;
```

---

## 10. Kiểm tra bảng

```sql
SHOW TABLES;
```

---

## 11. Import dữ liệu

```sql
SOURCE /home/huynq/projects/shoeshop/shoe_shopdb.sql;
```

---

# VII. Cấu hình Spring Boot

Sửa file:

```
application.properties
```

Cập nhật:

- Database URL
- Username
- Password
- Port

---

# VIII. Build Spring Boot

```bash
mvn install -DskipTests=true
```

---

# IX. Chạy dự án

## Chạy trực tiếp

```bash
java -jar target/shoe-ShoppingCart-0.0.1-SNAPSHOT.jar
```

---

## Chạy nền

```bash
nohup java -jar target/shoe-ShoppingCart-0.0.1-SNAPSHOT.jar > app.log 2>&1 &
```

---

## Dừng ứng dụng

Tìm PID

```bash
ps -ef | grep shoe
```

Kill

```bash
kill -9 <PID>
```
