# Cẩm nang triển khai CI/CD với GitLab Runner

---

# I. Cài đặt GitLab Runner

> Thực hiện trên **server sẽ thực hiện build/deploy dự án**.

## Bước 1. Cập nhật package

```bash
sudo apt-get update
```

---

## Bước 2. Thêm Repository GitLab Runner

```bash
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
```

---

## Bước 3. Cài GitLab Runner

```bash
sudo apt-get install gitlab-runner
```

---

## Bước 4. Kiểm tra phiên bản

```bash
gitlab-runner -v
```

---

# II. Cấu hình GitLab Runner

## Bước 1. Lấy thông tin Runner trên GitLab

Truy cập:

```
Project
└── Settings
    └── CI/CD
        └── Runners
            └── Expand
```

Tại đây sẽ thấy:

- Register URL
- Registration Token

Ví dụ:

```
Register URL:
http://gitlab.huynqdevops.vn/

Registration Token:
xxxxxxxxxxxxxxxx
```

---

## Bước 2. Register Runner

Trên server chạy:

```bash
gitlab-runner register
```

Nhập lần lượt:

- Register URL
- Registration Token
- Description
- Tags
- Executor

### Description

Đặt theo tên server.

Ví dụ:

```
ec2-server
```

### Tags

Đặt giống Description để sử dụng trong `.gitlab-ci.yml`.

Ví dụ:

```
ec2-server
```

### Executor

GitLab Runner hỗ trợ nhiều Executor:

- Shell ⭐ (Khuyến nghị cho người mới)
- Docker
- Kubernetes

---

## Bước 3. Cấu hình số lượng Pipeline chạy đồng thời

Mở file:

```bash
vi /etc/gitlab-runner/config.toml
```

Sửa:

```toml
concurrent = 1
```

Ý nghĩa:

> Runner chỉ được phép chạy **01 Pipeline** cùng thời điểm.

---

## Bước 4. Khởi động Runner

```bash
nohup gitlab-runner run \
--working-directory /home/gitlab-runner/ \
--config /etc/gitlab-runner/config.toml \
--service gitlab-runner \
--user gitlab-runner \
> runner.log 2>&1 &
```

Kiểm tra:

```bash
ps -ef | grep gitlab-runner
```

---

## Bước 5. Cho phép Runner dùng cho nhiều Project

GitLab:

```
Settings
└── CI/CD
    └── Available Specific Runners
```

Chọn:

```
Edit
```

Bỏ chọn:

```
When a runner is locked, it cannot be assigned to other projects
```

Hoàn thành cấu hình Runner.

---

# III. Viết Pipeline đầu tiên

Tạo file:

```
.gitlab-ci.yml
```

```yaml
stages:
  - build
  - deploy
  - checklog

build:
  stage: build

  script:
    - mvn install -DskipTests=true

  tags:
    - ec2-server
```

---

## Kiểm tra Pipeline

GitLab:

```
CI/CD
└── Pipelines
```

Kiểm tra Build đã chạy thành công hay chưa.

---

## Kiểm tra Source Code trên Runner

Source sẽ được clone về:

```bash
cd /home/gitlab-runner/builds/cp-eFsDZL/0/shoeshop/shoeshop
```

---

## Chuẩn bị thư mục Deploy

```bash
mkdir -p datas/shoeshop
```

> **Lưu ý**
>
> Ứng dụng khi chạy sẽ sử dụng **User của dự án**, không chạy trực tiếp bằng `gitlab-runner`.

---

# IV. Phân quyền cho GitLab Runner

Mở:

```bash
visudo
```

Thêm dưới dòng `root`:

```text
gitlab-runner ALL=(ALL) NOPASSWD: /bin/cp*
gitlab-runner ALL=(ALL) NOPASSWD: /bin/chown*
gitlab-runner ALL=(ALL) NOPASSWD: /bin/su shoeshop*
gitlab-runner ALL=(ALL) NOPASSWD: /bin/kill*
```

### Ý nghĩa

| Quyền               | Mục đích                           |
| ------------------- | ---------------------------------- |
| `/bin/cp*`          | Copy file deploy                   |
| `/bin/chown*`       | Đổi chủ sở hữu                     |
| `/bin/su shoeshop*` | Chạy ứng dụng bằng User `shoeshop` |
| `/bin/kill*`        | Dừng tiến trình cũ                 |

Sau đó phân quyền lại thư mục `/root`:

```bash
chmod -R 755 /root
```

---

# V. Pipeline hoàn chỉnh

```yaml
variables:
  projectname: shoe-ShoppingCart
  version: 0.0.1-SNAPSHOT
  projectuser: shoeshop
  projectpath: /root/datas/$projectuser/

stages:
  - build
  - deploy
  - showlog

build:
  stage: build

  variables:
    GIT_STRATEGY: clone

  script:
    - mvn install -DskipTests=true

  tags:
    - ec2-server

  only:
    - tags

deploy:
  stage: deploy

  variables:
    GIT_STRATEGY: none

  when: manual

  script:
    - |
      if [ "$GITLAB_USER_LOGIN" = "huynq1" ]; then

        sudo cp target/$projectname-$version.jar $projectpath

        sudo chown -R $projectuser. $projectpath

        sudo kill -9 $(ps -ef | grep $projectname-$version.jar | grep -v grep | awk '{print $2}') || true

        sudo su $projectuser -c "cd $projectpath && nohup java -jar $projectname-$version.jar > nohup.out 2>&1 &"

      else
        echo "Permission denied"
        exit 1
      fi

  tags:
    - ec2-server

  only:
    - tags

showlog:
  stage: showlog

  variables:
    GIT_STRATEGY: none

  when: manual

  script:
    - sleep 20
    - sudo su $projectuser -c "cd $projectpath && tail -n 1000 nohup.out"

  tags:
    - ec2-server

  only:
    - tags
```

---

# VI. Giải thích Pipeline

## Stage Build

- Clone source code
- Download dependencies
- Build project
- Sinh file `.jar`

---

## Stage Deploy

- Không clone source
- Chờ người dùng **Approve**
- Copy file `.jar`
- Đổi chủ sở hữu
- Kill tiến trình cũ
- Chạy phiên bản mới

---

## Stage Show Log

- Chờ 20 giây
- Hiển thị log của ứng dụng

---

# VII. Một số GitLab CI Variables thường dùng

| Variable                | Ý nghĩa                   |
| ----------------------- | ------------------------- |
| `CI_PROJECT_NAME`       | Tên Project               |
| `CI_PROJECT_DIR`        | Thư mục Project           |
| `CI_COMMIT_BRANCH`      | Nhánh hiện tại            |
| `CI_COMMIT_TAG`         | Tag hiện tại              |
| `CI_COMMIT_SHA`         | Commit SHA                |
| `CI_PIPELINE_ID`        | ID Pipeline               |
| `CI_JOB_ID`             | ID Job                    |
| `CI_RUNNER_ID`          | ID Runner                 |
| `CI_RUNNER_DESCRIPTION` | Tên Runner                |
| `CI_RUNNER_TAGS`        | Tags Runner               |
| `GITLAB_USER_LOGIN`     | User chạy Pipeline        |
| `GITLAB_USER_NAME`      | Tên người chạy Pipeline   |
| `GITLAB_USER_EMAIL`     | Email người chạy Pipeline |

Danh sách đầy đủ:

> https://docs.gitlab.com/ee/ci/variables/predefined_variables/

Hoặc tìm kiếm Google với từ khóa:

```
gitlab ci variables list
```

---

# VIII. Luồng hoạt động CI/CD

```text
Developer

      │
      ▼

Push Tag

      │
      ▼

GitLab

      │

Trigger Pipeline

      │
      ▼

GitLab Runner

      │

Clone Source

      │

Build

      │

Sinh file JAR

      │

Approve Deploy

      │

Copy JAR

      │

Kill Process cũ

      │

Start Application

      │

Show Log

      │

Deploy thành công
```
