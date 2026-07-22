# Hướng dẫn cài đặt và cấu hình GitLab Server

---

# I. Chuẩn bị Server

## 1. Tạo Server GitLab riêng

Có thể tạo server mới bằng cách:

- Tạo một server riêng.
- Tạo snapshot từ server ban đầu.
- Khởi tạo server mới từ snapshot.

---

# II. Cài đặt GitLab Server

Package GitLab sử dụng:

```text
gitlab-ee_14.4.1-ee.0_amd64.deb
```

Tham khảo package:

```text
https://packages.gitlab.com/gitlab/gitlab-ee/packages/ubuntu/focal/gitlab-ee_14.4.1-ee.0_amd64.deb
```

## Bước 1. Thêm Repository GitLab EE

```bash
curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash
```

## Bước 2. Cài đúng phiên bản GitLab

```bash
sudo apt-get install gitlab-ee=14.4.1-ee.0
```

---

# III. Cấu hình GitLab Server

## 1. Thêm Domain trên Server

Mở file:

```bash
sudo vi /etc/hosts
```

Thêm dòng:

```text
192.168.198.130 gitlab.elroydevops.tech
```

Ý nghĩa:

| Thành phần | Giá trị |
|---|---|
| IP GitLab Server | `192.168.198.130` |
| Domain GitLab | `gitlab.elroydevops.tech` |

---

## 2. Cấu hình `external_url`

Mở file cấu hình GitLab:

```bash
sudo vi /etc/gitlab/gitlab.rb
```

Tìm cấu hình mặc định:

```ruby
external_url 'http://gitlab.example.com'
```

Sửa thành:

```ruby
external_url 'http://gitlab.elroydevops.tech'
```

---

## 3. Áp dụng cấu hình GitLab

Sau khi sửa `gitlab.rb`, chạy:

```bash
sudo gitlab-ctl reconfigure
```

Lệnh này sẽ:

- Đọc lại file `/etc/gitlab/gitlab.rb`.
- Sinh lại các file cấu hình thành phần.
- Áp dụng cấu hình mới.
- Khởi động hoặc cập nhật các service liên quan.

---

# IV. Cấu hình Domain trên Máy Cá nhân

Trên máy Windows, mở file:

```text
C:\Windows\System32\drivers\etc\hosts
```

Thêm dòng:

```text
192.168.198.130 gitlab.elroydevops.tech
```

> Cần mở Notepad hoặc VS Code bằng quyền **Run as administrator** mới có thể lưu file `hosts`.

---

# V. Truy cập GitLab

Mở trình duyệt và truy cập:

```text
http://gitlab.elroydevops.tech
```

---

# VI. Lấy tài khoản đăng nhập ban đầu

GitLab tạo sẵn tài khoản quản trị:

```text
Username: root
```

Lấy mật khẩu ban đầu bằng lệnh:

```bash
sudo cat /etc/gitlab/initial_root_password
```

Trong file sẽ có dòng dạng:

```text
Password: xxxxxxxxxxxxxxxxxxxx
```

Sau khi đăng nhập lần đầu, nên đổi mật khẩu ngay.

> Không nên ghi mật khẩu thật vào tài liệu, source code hoặc Git repository.

Ví dụ tài khoản sử dụng:

```text
Username: root
Password: <mật khẩu quản trị>
```

Hoặc tài khoản cá nhân:

```text
Username: huynq1
Password: <mật khẩu cá nhân>
```

---

# VII. Một số câu lệnh GitLab thường dùng

## 1. Kiểm tra trạng thái GitLab

```bash
sudo gitlab-ctl status
```

Lệnh này hiển thị trạng thái các service như:

- Nginx
- PostgreSQL
- Redis
- Puma
- Sidekiq
- Gitaly

---

## 2. Restart toàn bộ GitLab

```bash
sudo gitlab-ctl restart
```

---

## 3. Dừng GitLab

```bash
sudo gitlab-ctl stop
```

---

## 4. Khởi động GitLab

```bash
sudo gitlab-ctl start
```

---

## 5. Áp dụng lại cấu hình

```bash
sudo gitlab-ctl reconfigure
```

---

## 6. Xem log GitLab

```bash
sudo gitlab-ctl tail
```

Xem log một service cụ thể:

```bash
sudo gitlab-ctl tail nginx
```

Ví dụ:

```bash
sudo gitlab-ctl tail puma
```

---

# VIII. Git Commit Conventional

## 1. Cấu trúc Commit chuẩn

```text
<type>(<scope>): <description>
```

Ví dụ:

```text
feat(auth): add login with Google
```

Trong đó:

| Thành phần | Ý nghĩa |
|---|---|
| `type` | Loại thay đổi |
| `scope` | Phạm vi thay đổi |
| `description` | Nội dung thay đổi ngắn gọn |

---

## 2. Các `type` phổ biến

| Type | Ý nghĩa | Khi sử dụng |
|---|---|---|
| `feat` | Thêm chức năng mới | Thêm API, function, module hoặc tính năng mới |
| `fix` | Sửa lỗi | Khắc phục bug hoặc lỗi xử lý |
| `docs` | Thay đổi tài liệu | Sửa README, tài liệu hướng dẫn |
| `config` | Thay đổi cấu hình | Sửa file cấu hình, biến môi trường |
| `style` | Định dạng code | Format code, khoảng trắng, dấu chấm phẩy; không đổi logic |
| `refactor` | Tái cấu trúc code | Thay đổi cấu trúc nhưng không thêm tính năng hoặc sửa lỗi |
| `test` | Thay đổi kiểm thử | Thêm hoặc sửa unit test, integration test |
| `chore` | Công việc phụ trợ | Build, dependency, cấu hình công cụ |
| `perf` | Cải thiện hiệu năng | Tối ưu tốc độ, bộ nhớ hoặc truy vấn |
| `ci` | Thay đổi CI/CD | Sửa `.gitlab-ci.yml`, pipeline hoặc runner |

---

# IX. Ví dụ Commit

## Thêm API đăng nhập

```bash
git commit -m "feat(auth): add login API"
```

## Sửa lỗi tạo hợp đồng

```bash
git commit -m "fix(contract): fix contract generation error"
```

## Sửa tài liệu

```bash
git commit -m "docs(readme): update deployment guide"
```

## Sửa cấu hình Database

```bash
git commit -m "config(database): update production database connection"
```

## Cải thiện hiệu năng

```bash
git commit -m "perf(report): optimize report query"
```

## Sửa Pipeline GitLab

```bash
git commit -m "ci(deploy): update production deployment pipeline"
```

---

# X. Luồng cài đặt GitLab Server

```text
Tạo Server
    │
    ▼
Cài GitLab EE
    │
    ▼
Cấu hình /etc/hosts
    │
    ▼
Cấu hình external_url
    │
    ▼
Chạy gitlab-ctl reconfigure
    │
    ▼
Cấu hình hosts trên máy cá nhân
    │
    ▼
Truy cập GitLab bằng Domain
    │
    ▼
Lấy mật khẩu root ban đầu
    │
    ▼
Đăng nhập và đổi mật khẩu
```

---

# XI. Lưu ý bảo mật

- Không lưu mật khẩu GitLab thật trong tài liệu.
- Không commit mật khẩu, token hoặc SSH private key lên Git.
- Nên sử dụng HTTPS thay cho HTTP khi triển khai thực tế.
- Nên thay đổi mật khẩu `root` ngay sau lần đăng nhập đầu tiên.
- Nên giới hạn truy cập GitLab bằng firewall hoặc Security Group.
- File `/etc/gitlab/initial_root_password` chỉ tồn tại trong thời gian giới hạn sau khi cài đặt và có thể bị GitLab tự động xóa.