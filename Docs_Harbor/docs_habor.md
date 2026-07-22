# Hướng dẫn cài đặt Docker và Harbor

---

# I. Cài đặt Docker

## Bước 1. Tạo thư mục lưu công cụ

```bash
sudo mkdir -p /tools/docker
```

Di chuyển vào thư mục:

```bash
cd /tools/docker
```

---

## Bước 2. Tạo file cài đặt Docker

Tạo file:

```bash
vi docker-install.sh
```

Nội dung file:

```bash
#!/bin/bash

sudo apt update

sudo apt install -y \
  apt-transport-https \
  ca-certificates \
  curl \
  software-properties-common

curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
  | sudo gpg --dearmor \
  -o /usr/share/keyrings/docker-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update

sudo apt install -y docker-ce

sudo systemctl start docker

sudo systemctl enable docker

sudo curl -L \
  "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" \
  -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

docker --version

docker-compose --version
```

---

## Bước 3. Cấp quyền thực thi

```bash
chmod +x docker-install.sh
```

---

## Bước 4. Chạy file cài đặt

```bash
sudo sh docker-install.sh
```

Hoặc:

```bash
sudo ./docker-install.sh
```

---

## Bước 5. Kiểm tra Docker

Kiểm tra phiên bản Docker:

```bash
docker --version
```

Kiểm tra phiên bản Docker Compose:

```bash
docker-compose --version
```

Kiểm tra trạng thái Docker:

```bash
sudo systemctl status docker
```

---

# II. Cài đặt Harbor

## Bước 1. Cập nhật package

```bash
sudo apt update -y
```

---

## Bước 2. Cài đặt Certbot

```bash
sudo apt install certbot -y
```

Certbot được sử dụng để xin chứng chỉ SSL/TLS từ Let's Encrypt.

---

## Bước 3. Tạo thư mục Harbor

```bash
sudo mkdir -p /tools/harbor
```

Di chuyển vào thư mục:

```bash
cd /tools/harbor
```

---

## Bước 4. Tải Harbor Offline Installer

```bash
curl -s https://api.github.com/repos/goharbor/harbor/releases/latest \
  | grep browser_download_url \
  | cut -d '"' -f 4 \
  | grep '.tgz$' \
  | wget -i -
```

Lệnh trên thực hiện:

1. Lấy thông tin bản Harbor mới nhất từ GitHub.
2. Lọc đường dẫn tải xuống.
3. Lọc file có đuôi `.tgz`.
4. Tải file cài đặt về server.

---

## Bước 5. Giải nén Harbor

```bash
tar xvzf harbor-offline-installer*.tgz
```

Di chuyển vào thư mục Harbor:

```bash
cd harbor/
```

---

## Bước 6. Tạo file cấu hình Harbor

Copy file cấu hình mẫu:

```bash
cp harbor.yml.tmpl harbor.yml
```

File cấu hình chính:

```text
/tools/harbor/harbor/harbor.yml
```

---

# III. Cấp chứng chỉ SSL bằng Certbot

## Bước 1. Khai báo Domain

```bash
export DOMAIN="registry.huynqdevops.online"
```

---

## Bước 2. Khai báo Email

```bash
export EMAIL="quochuy2011nd@gmail.com"
```

---

## Bước 3. Xin chứng chỉ SSL

```bash
sudo certbot certonly \
  --standalone \
  -d "$DOMAIN" \
  --preferred-challenges http \
  --agree-tos \
  -m "$EMAIL" \
  --keep-until-expiring
```

---

## Giải thích các tham số Certbot

| Tham số                       | Ý nghĩa                                                 |
| ----------------------------- | ------------------------------------------------------- |
| `certonly`                    | Chỉ lấy chứng chỉ, không tự động cài đặt vào Web Server |
| `--standalone`                | Certbot sử dụng Web Server độc lập để xác thực Domain   |
| `-d`                          | Khai báo Domain cần cấp chứng chỉ                       |
| `--preferred-challenges http` | Xác thực quyền sở hữu Domain thông qua HTTP             |
| `--agree-tos`                 | Đồng ý với điều khoản sử dụng                           |
| `-m`                          | Khai báo Email quản trị                                 |
| `--keep-until-expiring`       | Chỉ cấp lại khi chứng chỉ sắp hết hạn                   |

Sau khi cấp thành công, chứng chỉ thường nằm tại:

```text
/etc/letsencrypt/live/registry.huynqdevops.online/
```

Trong đó:

```text
fullchain.pem
privkey.pem
```

---

# IV. Cấu hình Harbor

Mở file:

```bash
vi harbor.yml
```

Cấu hình Domain:

```yaml
hostname: registry.huynqdevops.online
```

Cấu hình HTTPS:

```yaml
https:
  port: 443
  certificate: /etc/letsencrypt/live/registry.huynqdevops.online/fullchain.pem
  private_key: /etc/letsencrypt/live/registry.huynqdevops.online/privkey.pem
```

Cấu hình mật khẩu tài khoản quản trị:

```yaml
harbor_admin_password: <mat-khau-harbor>
```

Ví dụ cấu hình chính:

```yaml
hostname: registry.huynqdevops.online

http:
  port: 80

https:
  port: 443
  certificate: /etc/letsencrypt/live/registry.huynqdevops.online/fullchain.pem
  private_key: /etc/letsencrypt/live/registry.huynqdevops.online/privkey.pem

harbor_admin_password: <mat-khau-harbor>

data_volume: /data
```

> Không nên lưu mật khẩu Harbor thật trong tài liệu hoặc commit lên Git.

---

# V. Cài đặt Harbor

## Bước 1. Chuẩn bị cấu hình

```bash
sudo ./prepare
```

Lệnh `prepare` sẽ:

- Kiểm tra file `harbor.yml`.
- Sinh các file cấu hình cần thiết.
- Chuẩn bị cấu hình cho các container Harbor.

---

## Bước 2. Chạy cài đặt Harbor

```bash
sudo ./install.sh
```

Quá trình này sẽ:

- Load Harbor Docker Images.
- Tạo Docker Network.
- Tạo Volume.
- Khởi động các container Harbor.

---

## Bước 3. Kiểm tra trạng thái Harbor

```bash
sudo docker-compose ps
```

Có thể kiểm tra thêm:

```bash
sudo docker ps
```

Các container Harbor thường bao gồm:

- `harbor-core`
- `harbor-db`
- `harbor-portal`
- `harbor-registry`
- `harbor-jobservice`
- `redis`
- `nginx`
- `registryctl`

---

# VI. Truy cập Harbor

Mở trình duyệt:

```text
https://registry.huynqdevops.online
```

Tài khoản mặc định:

```text
Username: admin
```

Mật khẩu:

```text
Giá trị được cấu hình tại harbor_admin_password
```

---

# VII. Một số câu lệnh Harbor thường dùng

## Kiểm tra trạng thái

```bash
docker-compose ps
```

## Xem log

```bash
docker-compose logs -f
```

Xem log một service:

```bash
docker-compose logs -f harbor-core
```

## Dừng Harbor

```bash
docker-compose down
```

## Khởi động Harbor

```bash
docker-compose up -d
```

## Restart Harbor

```bash
docker-compose restart
```

---

# VIII. Đăng nhập Harbor bằng Docker

```bash
docker login registry.huynqdevops.online
```

Nhập:

```text
Username: admin
Password: <mật khẩu Harbor>
```

---

# IX. Push Image lên Harbor

Giả sử có image:

```text
t24app:1.0
```

## Bước 1. Tag Image

```bash
docker tag t24app:1.0 registry.huynqdevops.online/t24/t24app:1.0
```

Trong đó:

| Thành phần                    | Ý nghĩa             |
| ----------------------------- | ------------------- |
| `registry.huynqdevops.online` | Domain Harbor       |
| `t24`                         | Project trên Harbor |
| `t24app`                      | Tên Image           |
| `1.0`                         | Tag Image           |

## Bước 2. Push Image

```bash
docker push registry.huynqdevops.online/t24/t24app:1.0
```

---

# X. Luồng cài đặt tổng thể

```text
Tạo thư mục /tools/docker
        │
        ▼
Tạo script cài Docker
        │
        ▼
Cài Docker và Docker Compose
        │
        ▼
Tạo thư mục /tools/harbor
        │
        ▼
Tải Harbor Offline Installer
        │
        ▼
Giải nén Harbor
        │
        ▼
Xin chứng chỉ SSL bằng Certbot
        │
        ▼
Cấu hình harbor.yml
        │
        ▼
Chạy ./prepare
        │
        ▼
Chạy ./install.sh
        │
        ▼
Kiểm tra bằng docker-compose ps
        │
        ▼
Truy cập Harbor bằng HTTPS
```

---

# XI. Lưu ý

- Domain phải trỏ đúng về IP Public của Harbor Server.
- Port `80` phải được mở để Certbot xác thực HTTP.
- Port `443` phải được mở để truy cập Harbor bằng HTTPS.
- Không được có service khác chiếm port `80` khi dùng Certbot với `--standalone`.
- Mật khẩu Harbor cần đủ mạnh và không lưu trực tiếp trong source code.
- Nên sao lưu thư mục dữ liệu Harbor, mặc định thường là:

```text
/data
```
