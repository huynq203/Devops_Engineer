# Tài liệu và lộ trình Kubernetes

## I. Mở đầu Kubernetes

**Cấu trúc Kubernetes:**

```
User / CLI  --->  Control Plane / Master Node  --->  Worker Nodes
```

### 1. Control Plane / Master Node

Chịu trách nhiệm quản lý toàn bộ cluster Kubernetes.

Gồm các thành phần chính: **API Server**, **etcd**, **Controller Manager (CM)**, **Scheduler**.

#### API Server

Thành phần trung tâm nhất của Kubernetes Control Plane, có nhiệm vụ:

- Nhận request từ `kubectl`, UI hoặc hệ thống bên ngoài
- Kiểm tra request có hợp lệ không
- Kiểm tra quyền truy cập
- Ghi hoặc đọc trạng thái cluster từ etcd
- Làm cổng giao tiếp giữa các thành phần Kubernetes

#### etcd

Là nơi lưu trữ dữ liệu trạng thái Kubernetes, gồm các thông tin:

- Danh sách Node
- Danh sách Pod
- Deployment
- Service
- ConfigMap
- Secret
- Namespace
- Trạng thái mong muốn của ứng dụng
- Trạng thái thực tế của cluster

#### Controller Manager

Là thành phần chạy nhiều controller khác nhau trong Kubernetes. Controller có nhiệm vụ liên tục quan sát cluster và đảm bảo:

> **Trạng thái thực tế = Trạng thái mong muốn**

**Ví dụ:** Bạn khai báo muốn có 3 Pod, nhưng thực tế chỉ còn 2 Pod vì 1 Pod bị lỗi. Controller Manager sẽ phát hiện sự chênh lệch này và yêu cầu tạo thêm Pod mới.

#### Scheduler

Chịu trách nhiệm chọn Worker Node phù hợp để chạy Pod mới. Khi một Pod mới được tạo ra, ban đầu Pod đó chưa biết sẽ chạy ở Node nào. Scheduler sẽ xem xét nhiều yếu tố để quyết định.

Scheduler kiểm tra:

- Node nào còn CPU
- Node nào còn RAM
- Pod yêu cầu tài nguyên bao nhiêu
- Node có label phù hợp không
- Có taint/toleration không
- Có affinity/anti-affinity không
- Có giới hạn hoặc ràng buộc nào không

Sau đó Scheduler sẽ gán Pod vào một Node cụ thể.

### 2. Worker Node

#### a. Kubelet

Kubelet là một agent chạy trên mỗi Worker Node. Nhiệm vụ chính của Kubelet là đảm bảo các container trong Pod đang chạy đúng theo yêu cầu từ Control Plane. Kubelet trên Worker Node sẽ nhận thông tin đó, rồi gọi Container Runtime để tạo container.

Kubelet làm các việc như:

- Nhận yêu cầu chạy Pod từ API Server
- Gọi Container Runtime để tạo container
- Theo dõi trạng thái container
- Restart container nếu bị lỗi, tùy theo restart policy
- Báo trạng thái Pod và Node về API Server

> **Lưu ý:** Kubelet không tự quyết định chạy Pod nào. Việc Pod chạy ở Node nào là do Scheduler quyết định. Sau đó Kubelet thực thi việc chạy Pod đó.

#### b. Kube-Proxy

Kube-Proxy là thành phần xử lý network trên mỗi Worker Node. Nó giúp các Pod và Service có thể giao tiếp với nhau.

**Ví dụ:** Bạn có một Service tên là `payment-service`, service này trỏ tới 3 Pod: `payment-pod-1`, `payment-pod-2`, `payment-pod-3`. Khi một request đi tới `payment-service`, Kube-Proxy sẽ giúp điều hướng request đó tới một trong các Pod phù hợp.

Kube-Proxy quản lý các rule mạng trên node, ví dụ: `iptables`, `IPVS`.

Nhiệm vụ của Kube-Proxy:

- Quản lý network rules
- Cho phép Pod giao tiếp với Pod khác
- Cho phép Service route traffic tới Pod
- Hỗ trợ load balancing nội bộ giữa các Pod

#### c. Container Runtime

Container Runtime là thành phần thực sự chạy container. Kubernetes không tự mình chạy container trực tiếp — nó cần một runtime để làm việc này.

Một số container runtime phổ biến: **containerd**, **CRI-O**, **Docker**.

> Trong các phiên bản Kubernetes hiện đại, containerd thường được dùng phổ biến hơn Docker.

Container Runtime làm các việc như:

- Pull image từ registry
- Tạo container
- Start container
- Stop container
- Quản lý vòng đời container

### Luồng đơn giản

```
API Server thông báo cho Kubelet
        ↓
Kubelet yêu cầu Container Runtime
        ↓
Container Runtime pull image và chạy container
        ↓
Container nằm trong Pod bắt đầu chạy
```

---

## II. Cài đặt cụm Kubernetes On-premise

📺 **Link bài giảng:** [Bài 5 - Triển khai Kubernetes Cluster trên On-premise](https://devopsedu.vn/courses/khoa-hoc-kubenetes-thuc-te/lesson/bai-5-trien-khai-kubernetes-cluster-tren-on-premise-2/?page_tab=overview)

### 1. Cấu hình tài nguyên

| Hostname     | OS           | IP              | RAM (tối thiểu) | CPU (tối thiểu) |
| ------------ | ------------ | --------------- | --------------- | --------------- |
| k8s-master-1 | Ubuntu 22.04 | 192.168.109.131 | 3G              | 2               |
| k8s-master-2 | Ubuntu 22.04 | 192.168.109.132 | 3G              | 2               |
| k8s-master-3 | Ubuntu 22.04 | 192.168.109.133 | 3G              | 2               |

### 2. Thực hiện trên cả 3 servers

Cấu hình `/etc/hosts`:

```bash
vi /etc/hosts
```

```
192.168.109.131 k8s-master-1
192.168.109.132 k8s-master-2
192.168.109.133 k8s-master-3
```

### 3. Tạo user devops và chuyển sang user devops trên 3 server

```bash
adduser devops
```

### 4. Phân quyền user devops vào group sudo để có quyền thực thi

```bash
chmod -aG sudo devops
```

### 5. Tắt swap để có thể call API của cụm

```bash
sudo swapoff -a
```

> Chỉ tắt tạm thời, khi server khởi động lại sẽ tự bật.

Để tắt vĩnh viễn, mở `/etc/fstab` và comment dòng `/swap.img none swap sw 0 0`, hoặc dùng câu lệnh:

```bash
sudo sed -i '/swap.img/s/^/#/' /etc/fstab
```

### 6. Cấu hình module kernel

```bash
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
```

### 7. Tải module kernel

```bash
sudo modprobe overlay
sudo modprobe br_netfilter
```

### 8. Cấu hình hệ thống mạng

```bash
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables=1
net.bridge.bridge-nf-call-iptables=1
net.ipv4.ip_forward=1
EOF
```

### 9. Áp dụng cấu hình sysctl

```bash
sudo sysctl --system
```

### 10. Cài các gói phụ trợ và thêm kho Docker

```bash
sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates
```

```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg
```

```bash
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

### 11. Cài đặt containerd

```bash
sudo apt update -y
sudo apt install -y containerd.io
```

### 12. Cấu hình containerd

```bash
containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
```

### 13. Khởi động containerd

```bash
sudo systemctl restart containerd
sudo systemctl enable containerd
```

### 14. Thêm kho lưu trữ Kubernetes

```bash
sudo mkdir -p /etc/apt/keyrings
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

### 15. Cài đặt các gói Kubernetes

```bash
sudo apt update -y
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm
```

---

### 🔹 Mô hình đầu tiên: 1 master – 2 worker

**Thực hiện trên server `k8s-master-1` (làm master):**

```bash
sudo kubeadm init
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl get nodes
```

Thêm network Calico để cụm hoạt động:

```bash
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
```

**Thực hiện trên server `k8s-master-2` và `k8s-master-3` (làm worker):**

```bash
sudo kubeadm join 192.168.1.111:6443 --token your_token --discovery-token-ca-cert-hash your_sha
```

Ví dụ:

```bash
sudo kubeadm join 192.168.109.131:6443 \
  --token 8frm5a.ku9amr3mclw623p9 \
  --discovery-token-ca-cert-hash sha256:8bb7f14eada01ee1af0adb3ad424e3e3528e3ac59b9e1fab58277ae73933fc6c
```

### 🔹 Mô hình thứ hai: 3 server vừa master vừa worker

> Xem video bài giảng để hiểu rõ hơn.

**Thực hiện trên server `k8s-master-1`:**

```bash
sudo kubeadm init --control-plane-endpoint "192.168.109.131:6443" --upload-certs
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml
```

**Thực hiện trên server `k8s-master-2` và `k8s-master-3`:**

```bash
sudo kubeadm join 192.168.1.111:6443 --token your_token \
  --discovery-token-ca-cert-hash your_sha \
  --control-plane --certificate-key your_cert

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

**Chỉ định node đang làm master làm cả worker:**

```bash
kubectl taint nodes k8s-master-1 node-role.kubernetes.io/control-plane:NoSchedule-
```

> ⚠️ **Chú ý:** Để reset trắng cụm và cài lại, dùng các câu lệnh sau trên **tất cả** server:
>
> ```bash
> sudo kubeadm reset -f
> sudo rm -rf /var/lib/etcd
> sudo rm -rf /etc/kubernetes/manifests/*
> ```

---

## Cài đặt cụm Kubernetes Cloud bằng GCP (GKE)

📺 **Link bài giảng:** [Bài 6 - Triển khai Kubernetes Cluster trên Cloud (GKE)](https://devopsedu.vn/courses/khoa-hoc-kubenetes-thuc-te/lesson/bai-6-trien-khai-kubernetes-cluster-tren-cloud-gke-2/)

1. Lên Google Cloud Platform → Tìm kiếm **Kubernetes Cluster** → Chọn cluster → Enable **Kubernetes API Server**
2. Vào **Kubernetes Engine** → Chọn cluster → Chọn **Create**
   - Chọn **Standard**: You manage your cluster

---

## File YAML cấu hình trong Kubernetes

Trong Kubernetes, file cấu hình thường viết bằng YAML. File YAML dùng để khai báo tài nguyên như Namespace, Pod, Deployment, Service, Ingress, ConfigMap, Secret,...

### Cấu trúc cơ bản của một file YAML Kubernetes

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shoeshop-app
  namespace: shoeshop
spec:
  replicas: 2
  selector:
    matchLabels:
      app: shoeshop
  template:
    metadata:
      labels:
        app: shoeshop
    spec:
      containers:
        - name: shoeshop
          image: nginx:latest
          ports:
            - containerPort: 80
```

### Các phần chính

#### `apiVersion` — Phiên bản API của Kubernetes resource

| Resource      | apiVersion             |
| ------------- | ---------------------- |
| Pod           | `v1`                   |
| Service       | `v1`                   |
| Namespace     | `v1`                   |
| ConfigMap     | `v1`                   |
| Secret        | `v1`                   |
| ResourceQuota | `v1`                   |
| Deployment    | `apps/v1`              |
| StatefulSet   | `apps/v1`              |
| DaemonSet     | `apps/v1`              |
| Job           | `batch/v1`             |
| CronJob       | `batch/v1`             |
| Ingress       | `networking.k8s.io/v1` |

> **Ví dụ:** Dùng cho Deployment → `apiVersion: apps/v1`

#### `kind` — Loại tài nguyên Kubernetes bạn muốn tạo

| Kind                  | Mô tả                                              |
| --------------------- | -------------------------------------------------- |
| Pod                   | Đơn vị nhỏ nhất, chứa container chạy app           |
| Deployment            | Deploy app, quản lý Pod, replica, rolling update   |
| Service               | Mở đường truy cập vào Pod                          |
| Namespace             | Chia môi trường/project trong cluster              |
| ConfigMap             | Lưu cấu hình thường, ví dụ biến môi trường         |
| Secret                | Lưu thông tin nhạy cảm như password, token         |
| Ingress               | Routing HTTP/HTTPS từ bên ngoài vào service        |
| PersistentVolume      | Khai báo volume lưu trữ vật lý                     |
| PersistentVolumeClaim | App yêu cầu dung lượng lưu trữ                     |
| StatefulSet           | Chạy app có trạng thái như database                |
| DaemonSet             | Chạy một Pod trên mỗi Node                         |
| Job                   | Chạy task một lần rồi kết thúc                     |
| CronJob               | Chạy task theo lịch                                |
| ReplicaSet            | Đảm bảo số lượng Pod, thường do Deployment quản lý |
| ResourceQuota         | Giới hạn tài nguyên                                |

> **Ví dụ:** Dùng cho Deployment → `kind: Deployment`

#### `metadata` — Thông tin nhận diện của resource

Resource này tên gì. Các field hay dùng:

| Field         | Mô tả                                                     |
| ------------- | --------------------------------------------------------- |
| `name`        | Tên resource                                              |
| `namespace`   | Namespace chứa resource (không gian để triển khai FE, BE) |
| `labels`      | Nhãn để phân loại, match selector                         |
| `annotations` | Ghi chú/cấu hình phụ cho tool khác                        |

**Ví dụ:**

```yaml
metadata:
  name: shoeshop-app
  namespace: shoeshop
  labels:
    app: shoeshop
  annotations:
    description: "Shoeshop Java application"
```

#### `spec` — Khai báo cách resource chạy/cấu hình

Resource này phải làm gì, chạy ra sao.

**Ví dụ:**

```yaml
spec:
  type: NodePort # expose service ra ngoài node
  selector: # service trỏ tới Pod có label app=shoeshop
    app: shoeshop
  ports:
    - port: 80 # port của service
      targetPort: 80 # port container bên trong Pod
      nodePort: 30080 # port mở trên node
```

---

## Namespace trong Kubernetes

📺 **Link bài giảng:** [Bài 9 - Namespace trong Kubernetes](https://devopsedu.vn/courses/khoa-hoc-kubenetes-thuc-te/lesson/bai-9-namespace-trong-kubernetes-2/)

Namespace là một không gian riêng để có thể triển khai FE, BE.

### Các câu lệnh thường dùng

| Câu lệnh                              | Ý nghĩa                                                              |
| ------------------------------------- | -------------------------------------------------------------------- |
| `kubectl get pod`                     | Hiển thị các pod                                                     |
| `kubectl get pod --namespace default` | Hiển thị pod trong namespace `default`                               |
| `kubectl get ns` (hoặc `namespace`)   | Hiển thị các namespace trên k8s                                      |
| `kubectl create ns project-1`         | Tạo namespace với tên `project-1` (vd: `kubectl create ns shoeshop`) |
| `kubectl delete ns project-1`         | Xóa namespace với tên `project-1` (vd: `kubectl delete ns shoeshop`) |
| `kubectl apply -f ns.yaml`            | Chạy chỉ định file `ns.yaml` để tạo namespace                        |
| `kubectl delete -f ns.yaml`           | Xóa namespace được khai báo trong `ns.yaml` (không phải xóa file)    |

> Tuy nhiên, thông thường sẽ tạo 1 file YAML thay vì chạy trực tiếp bằng câu lệnh.

### Tạo file YAML cho Namespace

**Bước 1:** Tạo project

```bash
mkdir -p projects/project-1
```

**Bước 2:** Tạo file YAML

```bash
vi ns.yaml
```

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: project-1
```

**Bước 3:** Áp dụng file

```bash
kubectl apply -f ns.yaml
```

**Bước 4:** Tạo file `resourcequota.yaml` để giới hạn tài nguyên

```bash
vi resourcequota.yaml
```

```yaml
apiVersion: v1
kind: ResourceQuota # sử dụng tài nguyên ResourceQuota
metadata:
  name: mem-cpu-quota # đặt tên tùy ý
  namespace: project-1 # trỏ đến namespace
spec:
  hard:
    requests.cpu: "2" # giới hạn CPU là 2
    requests.memory: 4Gi # giới hạn RAM là 4Gi
```

---

## III. Phương pháp triển khai dự án trên Kubernetes

📺 **Link bài giảng:** [Bài 10 - Phương pháp triển khai dự án trên Kubernetes hiệu quả](https://devopsedu.vn/courses/khoa-hoc-kubenetes-thuc-te/lesson/bai-10-phuong-phap-trien-khai-du-an-tren-kubernetes-hieu-qua-2/)

## IV. Triển khai dự án Kubernetes

## V. Triển khai công cụ trên Kubernetes

## VI. Giám sát và quản trị cụm Kubernetes

## VII. Kubernetes thực tế doanh nghiệp
