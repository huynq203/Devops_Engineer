# Tài liệu cấu hình mạng & câu lệnh Linux

## Cấu hình mạng

### Cấu hình mạng Bridge — Nên dùng cho PC

- IP động: `192.168.1.17/24`

**Cấu hình IP tĩnh:**

```bash
nano /etc/netplan/00-installer-config.yaml
```

Sửa file thành:

```yaml
network:
  ethernets:
    ens33:
      dhcp4: false
      addresses: [192.168.1.124/24] # Đặt tùy ý đối với bridge
      gateway4: [192.168.1.1]
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
  version: 2
```

Sau khi sửa xong, lưu file và áp dụng:

```bash
netplan apply
```

Kiểm tra địa chỉ IP:

```bash
ip -4 addr
```

### Cấu hình mạng NAT — Nên dùng cho laptop

- IP động: `192.168.158.128/24`

**Cấu hình IP tĩnh:**

```bash
nano /etc/netplan/00-installer-config.yaml
```

```yaml
network:
  ethernets:
    ens33:
      dhcp4: false
      addresses: [192.168.158.99/24] # Đặt tùy ý đối với NAT
      gateway4: 192.168.158.2
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
  version: 2
```

Hoặc sửa theo dạng danh sách:

```yaml
network:
  ethernets:
    ens33:
      dhcp4: false
      addresses:
        - 192.168.198.129/24
      gateway4: 192.168.198.2
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
  version: 2
```

### Xem thông tin server

```bash
cat /etc/os-release
```

### SSH vào server

```bash
ssh username@ip_server
```

> Nếu gặp cảnh báo bảo mật thì gen lại SSH key:
>
> ```bash
> ssh-keygen -R ip_servers
> ```

---

## Các câu lệnh Linux cơ bản

| Câu lệnh   | Ý nghĩa                                 |
| ---------- | --------------------------------------- |
| `pwd`      | Vị trí hiện tại                         |
| `whoami`   | Đang đăng nhập trong phiên với user nào |
| `cd <url>` | Truy cập vào đường dẫn                  |
| `clear`    | Xóa toàn bộ trên cmd                    |

### Xem thông tin file/folder

| Câu lệnh  | Ý nghĩa                                             |
| --------- | --------------------------------------------------- |
| `ls`      | Show toàn bộ file trong folder                      |
| `ls -l`   | Show theo list các folder/file con                  |
| `ls -a`   | Show những file ẩn                                  |
| `ls -t`   | Show những file từ mới nhất đến cũ nhất             |
| `ls -lta` | Show toàn bộ file trong folder (kết hợp `-l -t -a`) |
| `ls -ld`  | Show folder hiện tại                                |

### Tạo folder

```bash
mkdir name_folder                     # Tạo folder name_folder
mkdir -p name_folder1/namefolder2     # Tạo name_folder1 và name_folder2 lồng nhau
```

### Tạo file

```bash
touch name_file    # Tạo file name_file
```

### Xóa file/folder

| Câu lệnh                          | Ý nghĩa                                |
| --------------------------------- | -------------------------------------- |
| `rm name_folder \| name_file`     | Xóa folder hoặc file                   |
| `rm -r name_folder \| name_file`  | Xóa folder hoặc file (đệ quy)          |
| `rm -rf name_folder \| name_file` | Xóa folder hoặc file không cần hỏi lại |

### Copy file

| Câu lệnh                                  | Ý nghĩa                                                   |
| ----------------------------------------- | --------------------------------------------------------- |
| `cp file_1 url`                           | Copy `file_1` vào `url`                                   |
| `cp -r folder_1 url`                      | Copy folder vào `url`                                     |
| `scp file/folder username@ip_server:/url` | Copy file/folder từ local sang server vào đường dẫn `url` |

### Move file hoặc folder

| Câu lệnh           | Ý nghĩa                                                         |
| ------------------ | --------------------------------------------------------------- |
| `mv file_1 file`   | Chuyển `file_1` sang thư mục hiện tại và đổi tên thành `file`   |
| `mv file_1 folder` | Chuyển `file_1` sang thư mục hiện tại và đổi tên thành `folder` |

### Ghi và đọc file

| Câu lệnh                      | Ý nghĩa                                                             |
| ----------------------------- | ------------------------------------------------------------------- |
| `echo "DevOps"`               | In nội dung ra màn hình                                             |
| `echo "DevOps" > file_name`   | Hiển thị nội dung "DevOps" và **ghi vào** `file_name`               |
| `echo "DevOps1" > file_name`  | Hiển thị nội dung "DevOps1" và **ghi đè** nội dung cũ ở `file_name` |
| `echo "DevOps2" >> file_name` | Hiển thị nội dung và **ghi thêm dòng** "DevOps2" vào `file_name`    |
| `cat file_name`               | Đọc nội dung trong `file_name`                                      |
| `history`                     | Xem lại các câu lệnh đã từng gõ                                     |

### Xem thay đổi của file theo thời gian thực (realtime)

| Câu lệnh              | Ý nghĩa                                      |
| --------------------- | -------------------------------------------- |
| `tail -n 1 file_name` | Hiển thị 1 dòng cuối cùng của `file_name`    |
| `tail -f file_name`   | Xem dữ liệu realtime (thoát bằng `Ctrl + C`) |

### Tìm kiếm từ khóa/nội dung trong file (`grep`)

| Câu lệnh                               | Ý nghĩa                                            |
| -------------------------------------- | -------------------------------------------------- |
| `grep keyword file_name`               | Tìm kiếm `keyword` trong `file_name`               |
| `grep keyword *.log`                   | Tìm kiếm `keyword` trong nhiều file có đuôi `.log` |
| `grep -i keyword file_name`            | Tìm kiếm không phân biệt chữ hoa/thường            |
| `grep -n keyword file_name`            | Hiển thị vị trí số dòng chứa `keyword`             |
| `grep -c keyword file_name`            | Đếm số dòng có chứa `keyword`                      |
| `grep -E "keyword1,keyword2" filename` | Tìm dòng chứa `keyword1` hoặc `keyword2`           |

> Cú pháp chung: `Tên câu lệnh | grep option keyword`

### Tìm kiếm file/folder trong hệ thống

| Câu lệnh                    | Ý nghĩa                                                   |
| --------------------------- | --------------------------------------------------------- |
| `find . -name "file_name"`  | Tìm trong thư mục hiện tại file/folder có tên `file_name` |
| `find . -iname "file_name"` | Tương tự nhưng không phân biệt hoa/thường                 |

---

## Câu lệnh nâng cao Linux

| Câu lệnh                                                      | Ý nghĩa                                                  |
| ------------------------------------------------------------- | -------------------------------------------------------- |
| `free -m`                                                     | Kiểm tra trạng thái RAM                                  |
| `df -h`                                                       | Kiểm tra trạng thái disk                                 |
| `top`                                                         | Hiển thị toàn bộ thông tin RAM, DISK theo thời gian thực |
| `hostnamectl set-hostname <new_name>` hoặc `vi /etc/hostname` | Đổi tên server (cần khởi động lại server)                |
| `reboot`                                                      | Khởi động lại server                                     |

### `netstat -tlpun`

| Tham số | Ý nghĩa                              |
| ------- | ------------------------------------ |
| `-t`    | Hiển thị thông tin TCP               |
| `-l`    | Hiển thị các cổng đang mở để kết nối |
| `-p`    | Hiển thị process                     |
| `-u`    | Hiển thị thông tin các kết nối UDP   |
| `-n`    | Hiển thị địa chỉ IP và cổng dạng số  |

### Process hệ thống

```bash
ps -ef                                                        # Các process đang chạy trên hệ thống
ps -ef | grep keyword | grep -v grep | awk '{print $2}'       # Hiển thị PID
```

```bash
traceroute -T -p Port DiaChiIP   # Hiển thị thông tin kết nối
```

| Tham số | Ý nghĩa |
| ------- | ------- |
| `-T`    | TCP     |
| `-p`    | Port    |

```bash
lsof -i :port     # Kiểm tra process nào đang chạy trên port
kill -9 pid       # Xóa (kill) process theo PID
kill -9 $(ps -ef | grep keyword | grep -v grep | awk '{print $2}')   # Kill process theo keyword
```

---

## Quản lý user & group

### Tạo user

| Câu lệnh           | Ý nghĩa                                                                |
| ------------------ | ---------------------------------------------------------------------- |
| `useradd`          | Thêm user, không hỏi thông tin/mật khẩu → thường vào thẳng server luôn |
| `adduser`          | Thêm user, cần điền thông tin và mật khẩu → muốn vào thì cần mật khẩu  |
| `su user`          | Chuyển sang user khác                                                  |
| `Ctrl + D`         | Thoát user vừa chuyển                                                  |
| `deluser username` | Xóa user                                                               |
| `exit`             | Thoát ra root                                                          |
| `vi /etc/passwd`   | Xem danh sách user vừa tạo                                             |

### Tạo group

| Câu lệnh                     | Ý nghĩa                                |
| ---------------------------- | -------------------------------------- |
| `groupadd group1`            | Tạo nhóm `group1`                      |
| `usermod -aG group1 user1`   | Thêm `user1` vào nhóm `group1`         |
| `deluser username groupname` | Xóa `username` khỏi group              |
| `groups username`            | Kiểm tra `username` có trong group nào |

### Phân quyền sở hữu

```bash
chown chusohuu:nhomsohuu folder/file    # Phân quyền chủ sở hữu / nhóm sở hữu của 1 folder/file
```

### Phân quyền truy cập

| Câu lệnh                  | Ý nghĩa                                    |
| ------------------------- | ------------------------------------------ |
| `chmod u=rwx folder_name` | Phân quyền `rwx` cho **chủ sở hữu** (`u`)  |
| `chmod g=rwx folder_name` | Phân quyền `rwx` cho **nhóm sở hữu** (`g`) |
| `chmod o=rwx folder_name` | Phân quyền `rwx` cho **người khác** (`o`)  |

Ý nghĩa ký hiệu quyền:

- `r` = read (đọc)
- `w` = write (viết)
- `x` = execute (thực thi)

**Phân quyền nhanh:**

```bash
chmod u=rwx,g=rw,o=wx folder_name
```

**Truy cập theo kiểu số:** `r=4, w=2, x=1`

```bash
chmod 777 folder_name       # Phân quyền full cho chủ sở hữu, nhóm sở hữu, người khác (chỉ folder_name, không áp cho con)
chmod -R 777 folder_name    # Phân quyền full cho toàn bộ (đệ quy)
```

---

## Cập nhật package

```bash
sudo apt update       # Cập nhật danh sách package mới nhất
sudo apt upgrade      # Nâng cấp các package đang cài
sudo apt autoremove   # Xóa các thư viện không còn dùng
```

---

## Cài đặt VIM

Cài đặt công cụ soạn thảo văn bản VIM:

```bash
sudo apt install vim -y
```

Tạo file bằng VIM:

```bash
vi file_name
```

### Khi vào file_name

| Phím    | Ý nghĩa                                          |
| ------- | ------------------------------------------------ |
| `I`     | Chế độ **Insert mode**                           |
| `Esc`   | Thoát Insert mode → chuyển sang **Command mode** |
| `dd`    | Xóa 1 dòng                                       |
| `u`     | Phục hồi (giống `Ctrl + Z`)                      |
| `yy`    | Copy                                             |
| `p`     | Paste                                            |
| `/name` | Tìm kiếm từ `name`                               |

### Sau khi ghi xong

| Lệnh              | Ý nghĩa                                 |
| ----------------- | --------------------------------------- |
| `:q` ↵            | Thoát (chỉ khi file không còn thay đổi) |
| `:q!` ↵           | Thoát không lưu                         |
| `:w` ↵            | Chỉ lưu, không thoát                    |
| `:wq` hoặc `:x` ↵ | Lưu và thoát                            |

---

## Cài đặt Nginx

Cài đặt Nginx:

```bash
sudo apt install nginx -y   # Được lưu trong /etc/nginx
```

### Những file cần chú ý

| File                      | Ý nghĩa                                              |
| ------------------------- | ---------------------------------------------------- |
| `sites-available/default` | Cấu hình port Nginx                                  |
| `nginx.conf`              | Hiển thị thông tin Nginx (hoặc dùng lệnh `nginx -T`) |

### Câu lệnh thường dùng

```bash
nginx -t                    # Test xem cấu hình có vấn đề gì không
systemctl restart nginx     # Restart lại Nginx
nginx -s reload             # Chỉ restart lại phần dự án có thay đổi
```

### Cấu hình sau khi build xong dự án

> **Lưu ý:** Khi build xong dự án thì cần cấu hình thêm trong Nginx.

**Bước 1:**

```bash
vi /etc/nginx/conf.d/default.conf
```

**Bước 2:** Thêm vào phần `location` dòng `try_files $uri $uri/ /index.html;` (mục đích để tránh lỗi với SPA):

```nginx
location / {
    root /usr/share/nginx/html;
    index index.html index.htm;
    try_files $uri $uri/ /index.html;
}
```

**Bước 3:** Reload lại Nginx:

```bash
nginx -s reload
```

---

## Gen SSH key

**Bước 1:** Trên máy local, tạo SSH key:

```bash
ssh-keygen -t ed25519 -C "huynq1@evnfc.vn"
```

> Sẽ tạo ra 2 file: `id_rsa` (private key) và `id_rsa.pub` (public key)

**Bước 2:** Copy file public key lên server.

**Bước 3:** Copy nội dung file public key vào `~/.ssh/authorized_keys` trên server.

**Bước 4:** Set quyền:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

**Bước 5:** SSH từ local lên server.
