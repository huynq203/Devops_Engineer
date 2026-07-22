# Tài liệu cài Jenkins Server

## 1. Cài đặt Jenkins và các chức năng chính

📺 **Bài giảng:** [Bài 28 - Cài đặt Jenkins và các chức năng chính](https://devopsedu.vn/courses/devops-for-freshers/lesson/bai-28-cai-dat-jenkins-va-cac-chuc-nang-chinh/)

### Cài Jenkins

**Bước 1:** Tạo folder tools:

```bash
mkdir -p tools/jenkins
```

**Bước 2:** Tạo file batch `install-jenkins.sh`:

```sh
#!/bin/bash
set -e

apt-get update

apt-get install -y wget fontconfig openjdk-21-jre

mkdir -p /etc/apt/keyrings

wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" > \
  /etc/apt/sources.list.d/jenkins.list

apt-get update
apt-get install -y jenkins

systemctl enable --now jenkins

ufw allow 8080 || true

echo "Jenkins initial password:"

cat /var/lib/jenkins/secrets/initialAdminPassword
```

**Bước 3:** Cấp quyền thực thi:

```bash
chmod +x install-jenkins.sh
```

**Bước 4:** Cài đặt Jenkins:

```bash
sh install-jenkins.sh
```

**Bước 5:** Cấu hình domain Jenkins:

```bash
vi /etc/hosts
```

```
# thêm dòng
192.168.198.131 jenkins.elroydevops.tech
```

> Cấu hình bên laptop ở file hosts để có thể truy cập bằng domain.

Sau khi cấu hình xong, truy cập: **`jenkins.elroydevops.tech:8080`**

### Cấu hình reverse proxy từ port 8080 sang port 80

**Bước 1:** Cài nginx:

```bash
apt-get update
apt-get install nginx -y
```

**Bước 2:** Vào folder `conf.d`, tạo file `jenkins.elroydevops.tech.conf`:

```bash
cd /etc/nginx/conf.d
vi jenkins.elroydevops.tech.conf
```

```nginx
server {
    listen 80;
    server_name jenkins.elroydevops.tech;
    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection keep_alive;
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**Bước 3:** Restart nginx:

```bash
systemctl restart nginx
```

---

## 2. Triển khai Jenkins CI/CD — Continuous Deployment

> Tự động deployment nếu pipeline pass.

📺 **Bài giảng:** [Bài 29 - Triển khai Jenkins CI/CD Continuous Deployment](https://devopsedu.vn/courses/devops-for-freshers/lesson/bai-29-trien-khai-jenkins-ci-cd-continuous-deployment/)

### Bước 1 — Đồng bộ version Java

Trước tiên kiểm tra 2 version Java trên `ec2-server` và `jenkins-server` xem có giống nhau không.

- Nếu không giống, phải cài Java cùng 1 version trên cả 2 server:

  ```bash
  apt install openjdk-11-jdk -y
  ```

- Chọn Java version 11:

  ```bash
  update-alternatives --config java
  # => Chọn số tương ứng
  ```

### Bước 2 — Tạo user jenkins trên ec2-server

```bash
adduser jenkins
```

### Bước 3 — Tạo Node trên UI Jenkins

Lên UI Jenkins → **Manage Jenkins** → **Nodes** → **New Node** → điền các thông tin cần thiết:

- **Name**
- **Label**
- **Number of executors**: số lượng dự án

Nếu có **Custom workdir path** thì điền `/var/lib/jenkins`, và phải tạo thư mục này trên `ec2-server`:

```bash
mkdir -p /var/lib/jenkins
```

> Thư mục này sẽ giống thư mục `/var/lib/jenkins` trên `jenkins-server`, nhưng trên `ec2-server` chỉ chứa dự án.

Vào **Manage Jenkins** → **Security** → chọn **Agents** → chọn **Fixed** → điền port ít sử dụng trên `jenkins-server` (ví dụ `8999`). Kiểm tra port đã mở chưa:

```bash
netstat -tlpun
```

### Bước 4 — Kết nối Jenkins agent với ec2-server

1. Ấn vào node vừa tạo sẽ ra hướng dẫn kết nối.
2. Phân người sở hữu / nhóm sở hữu `/var/lib/jenkins` cho user `jenkins`:

   ```bash
   chown jenkins. /var/lib/jenkins
   ```

3. Vào folder `/var/lib/jenkins`, chuyển sang user `jenkins`:

   ```bash
   su jenkins
   ```

4. Trên UI Jenkins, chọn hướng dẫn _"Or run from agent command line, with the secret stored in a file: (Unix)"_ (vì bảo mật, secret được lưu ở file). Chạy lần lượt trên `ec2-server`, ở folder `/var/lib/jenkins`, với user `jenkins`:

   ```bash
   echo 36cc6373ed9b172692ebbc95837ba291b74a33252d18fbf09289ff14782ef5ce > secret-file
   curl -sO http://jenkins.elroydevops.tech:8080/jnlpJars/agent.jar
   java -jar agent.jar -url http://jenkins.elroydevops.tech:8080/ -secret @secret-file -name "ec2-server" -webSocket -workDir "/var/lib/jenkins" > nohup.out 2>&1 &
   ```

> ✅ Hoàn tất kết nối giữa server dự án và server Jenkins.

### Bước 5 — Tạo folder chứa dự án trên UI Jenkins

- Ra dashboard Jenkins.
- Chọn **New Item** → chọn **Folder** → đặt tên và save.

### Bước 6 — Kết nối Jenkins server tới GitLab server

1. Vào **Manage Jenkins** → **Plugins** → chọn **Available plugins** → tìm kiếm `gitlab` và `blue ocean`, tích chọn → **Install** → tick restart Jenkins server khi cài thành công.
2. Vào **Manage Jenkins** → chọn **System** → kéo xuống mục GitLab, điền các thông tin:
   - **Tên:** `gitlab-server`
   - **URL:** `http://gitlab.elroydevops.tech/`
   - **Token API:**
     - Vào GitLab tạo user `jenkins` có quyền admin.
     - Vào edit profile → **Access Token** → điền Token name, Select scopes chọn `api` → Token: `kqrZXCvez1cwY_HX3UHF`
   - **Credentials:** chọn **Add gitlab api token**, điền các thông tin vào.
3. Test connection — nếu thành công hiển thị **Success**.

### Bước 7 — Tạo project

1. Vào agent vừa tạo → chọn **New Item** → chọn **Pipeline** → đặt tên project `shoeshop`.
2. Cấu hình chọn các thông tin như URL GitLab, nhánh, và file `Jenkinsfile`.

### Bước 8 — Cấu hình trên GitLab

1. Lên GitLab, vào project → **Settings** → chọn **Webhooks**.

   ```
   URL: http://<user jenkins>:<token jenkins>@<địa chỉ jenkins>/project/<đường dẫn dự án trên jenkins>
   ```

   > Lưu ý: không có dấu `()`.

2. Vào **Admin** → **Settings** → **Network** → tìm **Outbound requests** → chọn _"Allow requests to the local network from web hooks and services"_ để mở cổng.

3. Lên Jenkins → **Profile** → **Security** → chọn API token, tạo token: `111a425aa822d39671f142fb020de16cf0`

   ```
   URL: http://admin:111a425aa822d39671f142fb020de16cf0@jenkins.elroydevops.tech/project/Action_in_lab/shoeshop
   ```

4. **Triggers** chọn: **Push events**, **Tag push events**, **Merge request events**, và bỏ tích **Enable SSL verification**.

5. Test ra HTTP **200** là thành công.

---

### Tạo file Jenkinsfile

#### Cách triển khai thứ 1

```groovy
pipeline {
    agent {
        label 'ec2-server'
    }

    environment {
        appUser = "shoeshop"
        appName = "shoe-ShoppingCart"
        appVersion = "0.0.1-SNAPSHOT"
        appType = "jar"
        pathParent = "/home/shoeshop"
        processName = "${appName}-${appVersion}.${appType}"
        folderDeploy = "${pathParent}/datas/${appUser}"
        projectFolder = "${pathParent}/projects/shoeshop"
    }

    stages {
        stage('build') {
            steps {
                sh(
                    label: "Build with Maven",
                    script: '''
                        sudo -n -u ${appUser} -H bash -lc "
                            cd ${projectFolder}
                            mvn clean install -DskipTests=true
                        "
                    '''
                )
            }
        }

        stage('deploy') {
            steps {
                sh(
                    label: "Prepare deploy folder",
                    script: '''
                        sudo -n chown -R ${appUser}. ${folderDeploy}
                    '''
                )

                sh(
                    label: "Copy jar file",
                    script: '''
                        sudo -n -u ${appUser} -H bash -lc "
                            cp ${projectFolder}/target/${processName} ${folderDeploy}/
                        "
                    '''
                )

                sh(
                    label: "Kill old process",
                    script: '''
                        PID=$(pgrep -f "${processName}" || true)

                        if [ -n "$PID" ]; then
                            echo "Killing old process: $PID"
                            sudo -n kill -9 $PID
                        else
                            echo "No old process found"
                        fi
                    '''
                )

                sh(
                    label: "Run new app",
                    script: '''
                        sudo -n -u ${appUser} -H bash -lc "
                            cd ${folderDeploy}
                            nohup java -jar ${processName} > nohup.out 2>&1 &
                        "
                    '''
                )

                sh(
                    label: "Check app process",
                    script: '''
                        sleep 3
                        ps -ef | grep ${processName} | grep -v grep || true
                        echo "Log:"
                        tail -n 50 ${folderDeploy}/nohup.out || true
                    '''
                )
            }
        }
    }
}
```

#### Cách triển khai thứ 2

```groovy
pipeline {
    agent {
        label 'ec2-server'
    }

    environment {
        appUser = "shoeshop"
        appName = "shoe-ShoppingCart"
        appVersion = "0.0.1-SNAPSHOT"
        appType = "jar"
        pathParent = "/home/shoeshop"
        processName = "${appName}-${appVersion}.${appType}"
        folderDeploy = "${pathParent}/datas/${appUser}"
        projectFolder = "${pathParent}/projects/shoeshop"
    }

    stages {
        stage('checkout scm') {
            steps {
                checkout scm
            }
        }

        stage('build') {
            steps {
                sh(
                    label: "Build with Maven",
                    script: '''
                        mvn clean install -DskipTests=true
                    '''
                )
            }
        }

        stage('deploy') {
            steps {
                sh(
                    label: "Prepare deploy folder",
                    script: '''
                        sudo -n chown -R ${appUser}. ${folderDeploy}
                    '''
                )

                sh(
                    label: "Copy jar file",
                    script: '''
                        sudo -n -u ${appUser} -H bash -lc "
                            cp ${WORKSPACE}/target/${processName} ${folderDeploy}/
                        "
                    '''
                )

                sh(
                    label: "Kill old process",
                    script: '''
                        PID=$(pgrep -f "${processName}" || true)

                        if [ -n "$PID" ]; then
                            echo "Killing old process: $PID"
                            sudo -n kill -9 $PID
                        else
                            echo "No old process found"
                        fi
                    '''
                )

                sh(
                    label: "Run new app",
                    script: '''
                        sudo -n -u ${appUser} -H bash -lc "
                            cd ${folderDeploy}
                            nohup java -jar ${processName} > nohup.out 2>&1 &
                        "
                    '''
                )

                sh(
                    label: "Check app process",
                    script: '''
                        sleep 3
                        ps -ef | grep ${processName} | grep -v grep || true
                        echo "Log:"
                        tail -n 50 ${folderDeploy}/nohup.out || true
                    '''
                )
            }
        }
    }
}
```

### Kết luận

1. Khởi động 3 server: `ec2-server`, `jenkins-server`, `gitlab-server`.
2. Chạy agent để kết nối Jenkins với GitLab.
3. Kiểm tra Jenkins đã kết nối tới GitLab chưa ở **Manage system**.
4. Kiểm tra webhook project ở GitLab.

---

## 3. Triển khai Jenkins CI/CD — Continuous Delivery

> Luôn sẵn sàng deploy, nhưng deploy production cần người bấm duyệt.

Mỗi lần server tắt đi sẽ mất kết nối `ec2-server` tới Jenkins, nên sẽ tạo 1 service để khi server khởi động thì service sẽ tự chạy cùng.

**Bước 1:** Ra ngoài root, tạo file service (thông tin giống file `Docs_Jenkins/jenkins-agent.service`):

```bash
vi /etc/systemd/system/jenkins-agent.service
```

```ini
[Unit]
# Mô tả service
Description=Jenkins Agent Service
# Chỉ khởi động khi mạng đã được bật
After=network.target

[Service]
Type=simple

# Thư mục chính của jenkins trên service
WorkingDirectory=/var/lib/jenkins

# Thực thi câu lệnh kết nối tới jenkins
ExecStart=/bin/bash -c 'java -jar agent.jar -url http://jenkins.elroydevops.tech:8080/ -secret @secret-file -name "ec2-server" -webSocket -workDir "/var/lib/jenkins"'
User=jenkins

# Nếu server hoặc jenkins lỗi thì sẽ tự khởi động lại
Restart=always

[Install]
# Cho phép service được bật để tự động chạy khi máy khởi động vào chế độ multi-user
WantedBy=multi-user.target
```

**Bước 2:** Reload lại systemd:

```bash
systemctl daemon-reload
```

**Bước 3:** Khởi động service:

```bash
systemctl start jenkins-agent.service
```

> ✅ Chạy service thành công.

**Bước 4:** Kiểm tra trạng thái:

```bash
systemctl status jenkins-agent.service
```
