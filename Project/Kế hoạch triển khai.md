
Tuyệt vời! Việc chuyển từ "Queue-based scaling" (RabbitMQ) sang **"Traffic-based scaling" (HTTP Requests)** kết hợp với **Monitoring (Prometheus + Grafana)** sẽ nâng tầm đồ án của bạn lên mức "Production-grade". Đây là mô hình thực tế mà các công ty lớn (Grab, Shopee, Netflix) sử dụng.

# Luồng hoạt động ứng dụng
k6 (HTTP traffic)
   ↓
Podinfo (Web App + metrics)
   ↓
Prometheus (thu thập metrics)
   ↓
KEDA (query Prometheus)
   ↓
HPA (scale Podinfo)

### Nâng cấp kiến trúc: "Traffic-Driven Autoscaling"

Để hệ thống trực quan và "xịn" hơn, chúng ta sẽ thêm vào các thành phần sau:

1. **Web App:** Sử dụng **Podinfo** (một ứng dụng demo nổi tiếng cho K8s, có giao diện web đẹp và tích hợp sẵn metrics).
    
2. **Monitoring:** **Prometheus** (thu thập dữ liệu) & **Grafana** (vẽ biểu đồ Dashboard).
    
3. **Scaler:** KEDA sẽ không nhìn vào Queue nữa, mà sẽ "hỏi" Prometheus: _"Hiện tại có bao nhiêu request/giây?"_.
    
4. **Load Generator:** Sử dụng **k6** (công cụ test hiệu năng hiện đại) để giả lập hàng nghìn người dùng truy cập cùng lúc.
    

---

### Phần 1: Lộ trình triển khai nâng cao

1. **Cài đặt Monitoring Stack:** Triển khai Prometheus và Grafana bằng Helm (dùng gói `kube-prometheus-stack`).





2. **Deploy Web App (Podinfo):** Triển khai ứng dụng web mẫu.
    
3. **Tích hợp KEDA với Prometheus:** Cấu hình KEDA ScaledObject để query dữ liệu từ Prometheus.
    
4. **Giả lập Traffic (Attack):** Viết kịch bản `k6` để bắn request liên tục vào Web App.
    
5. **Quan sát (Demo):**
    
    - Trên terminal: Pods tăng từ 1 -> 10.
        
    - Trên Web (Grafana): Biểu đồ traffic đi lên cùng lúc với biểu đồ số lượng Pods.
        

---

### Phần 2: Prompt dành cho chuyên gia (Copy đoạn này)

Hãy dùng prompt này để AI tạo ra các file cấu hình chính xác cho bạn.

> "Tôi muốn nâng cấp hệ thống KEDA demo lên mức độ nâng cao. Hãy đóng vai trò là một Senior DevOps Engineer và giúp tôi thực hiện:
> 
> Mục tiêu: Autoscaling một Web App dựa trên lượng HTTP Requests (RPS - Requests Per Second).
> 
> Stack: K3s, KEDA, Prometheus, Grafana, Ứng dụng mẫu Podinfo.
> 
> Công cụ test: k6.
> 
> **Hãy cung cấp cho tôi:**
> 
> 1. **Lệnh Helm:** Để cài đặt `kube-prometheus-stack` và `podinfo`.
>     
> 2. **File `ScaledObject.yaml` (Quan trọng):**
>     
>     - Cấu hình trigger loại `prometheus`.
>         
>     - Query: Tính toán rate request trong 2 phút.
>         
>     - Threshold: 10 (Nếu RPS > 10 thì bắt đầu scale).
>         
>     - MinReplica: 1, MaxReplica: 10.
>         
> 3. **Script `load_test.js` (k6):**
>     
>     - Kịch bản: Tăng dần từ 0 lên 50 Virtual Users (VUs) trong 30 giây, giữ nguyên trong 1 phút để thấy Pods scale out, rồi giảm về 0.
>         
>     - Target: URL của Podinfo service.
>         
> 4. **Hướng dẫn Config Grafana:** Cách đơn giản để import dashboard có sẵn của Podinfo hoặc K8s cluster để demo."
>     

---

### Phần 3: Hướng dẫn triển khai các thành phần mới

Dưới đây là các bước thực hiện trên máy ảo của bạn để chuẩn bị cho prompt trên.

#### Bước 1: Cài đặt Prometheus & Grafana (Bộ não quan sát)

Chúng ta dùng `kube-prometheus-stack`, bộ này cài "tất cả trong một" (Prometheus, Grafana, Node Exporter...).

Bash

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

# Cài đặt vào namespace 'monitoring'
helm install my-monitoring prometheus-community/kube-prometheus-stack --namespace monitoring --create-namespace
```

_Sau khi cài xong, bạn có thể truy cập Grafana bằng cách port-forward (nếu chạy local) hoặc NodePort._

#### Bước 2: Deploy ứng dụng Web mẫu (Podinfo)

Podinfo là ứng dụng hoàn hảo cho demo vì nó phơi bày (export) metrics chuẩn Prometheus.

Bash

```
helm repo add stefanprodan https://stefanprodan.github.io/podinfo
helm install my-web stefanprodan/podinfo --set serviceMonitor.enabled=true --namespace default
```

_Lưu ý: tham số `serviceMonitor.enabled=true` rất quan trọng, nó tự động bảo Prometheus theo dõi ứng dụng này._

#### Bước 3: Cài đặt công cụ Load Test (k6)

Trên máy ảo Ubuntu của bạn:

Bash

```
sudo gpg -k
sudo gpg --no-default-keyring --keyring /usr/share/keyrings/k6-archive-keyring.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491B62E6A39
echo "deb [signed-by=/usr/share/keyrings/k6-archive-keyring.gpg] https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
sudo apt-get update
sudo apt-get install k6
```

### Kịch bản Demo "Triệu đô" cho đồ án

Khi bạn bảo vệ đồ án, hãy sắp xếp màn hình như sau:

1. **Góc trái trên:** Cửa sổ trình duyệt mở **Grafana**, hiển thị biểu đồ "Requests/second" đang đi ngang ở mức thấp.
    
2. **Góc trái dưới:** Terminal chạy lệnh `kubectl get pods -w`. Hiện tại chỉ có 1 pod `my-web`.
    
3. **Bên phải:** Terminal chạy lệnh **k6**.
    

**Hành động:**

- Bạn gõ lệnh: `k6 run load_test.js`
    
- **10 giây sau:** Biểu đồ Grafana vọt lên.
    
- **15 giây sau:** Terminal `kubectl` bắt đầu nhảy dòng liên tục: `ContainerCreating`, `Running`. Số pod tăng từ 1 -> 2 -> 5 -> 8.
    
- **Giải thích:** "Thầy/Cô có thể thấy, khi lượng truy cập tăng đột biến, KEDA lập tức phát hiện qua Prometheus và ra lệnh cho Kubernetes cấp thêm máy chủ để chịu tải. Khi script k6 dừng, hệ thống sẽ tự động thu hồi tài nguyên về 1 pod để tiết kiệm chi phí."
    

**Bạn có muốn tôi cung cấp luôn nội dung file `load_test.js` của k6 và file `ScaledObject.yaml` cấu hình query Prometheus để bạn chạy thử ngay không?**


# Step by Step
Oke 👍 Lộ trình này **rất chuẩn đồ án + rất đúng thực tế production**. Mình sẽ **hướng dẫn bạn step-by-step, theo đúng trình tự triển khai ngoài đời**, mỗi bước đều có **mục tiêu – lệnh – kiểm tra kết quả – lỗi hay gặp** để bạn làm tới đâu chắc tới đó.

> Giả định môi trường:
> 
> - **K3s đã cài**
>     
> - **kubectl + helm đã dùng được**
>     
> - **KEDA đã cài** (nếu chưa, mình sẽ nhắc lại đúng lúc)
>     
> - Namespace mặc định: `default`
>     
> - Bạn demo trên **1 VM**
>     

---

# 🧭 TỔNG QUAN FLOW (để bạn không bị “lạc”)

```
k6 (HTTP traffic)
   ↓
Podinfo (Web App + metrics)
   ↓
Prometheus (thu thập metrics)
   ↓
KEDA (query Prometheus)
   ↓
HPA (scale Podinfo)
```

---

# 🔹 STEP 0 – Kiểm tra nền tảng (BẮT BUỘC)

## 0.1 Kiểm tra cluster

```bash
kubectl get nodes
kubectl get pods -A
```

✔️ Thấy node `Ready`  
✔️ CoreDNS, traefik (k3s) đang chạy

---

## 0.2 Kiểm tra Helm

```bash
helm version
```

✔️ Có version là OK

---

## 0.3 Kiểm tra KEDA

```bash
kubectl get pods -n keda
```

✔️ Phải thấy:

- `keda-operator`
    
- `keda-metrics-apiserver`
    

❌ Nếu CHƯA có → nói mình, mình hướng dẫn cài lại ngay

---

# 🔹 STEP 1 – Cài PROMETHEUS + GRAFANA (Monitoring Stack)

## 🎯 Mục tiêu

- Có Prometheus để **thu thập RPS**
    
- Có Grafana để **vẽ biểu đồ demo**
    

---

## 1.1 Thêm Helm repo

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

---

## 1.2 Cài kube-prometheus-stack

```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```

⏳ Chờ 2–3 phút

---

## 1.3 Kiểm tra

```bash
kubectl get pods -n monitoring
```

✔️ Thấy các pod:

- `prometheus-*`
    
- `grafana-*`
    
- `alertmanager-*`
    
- `node-exporter-*`
    

---

## 1.4 Truy cập Grafana (demo local)

```bash
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
```

👉 Mở trình duyệt:  
`http://localhost:3000`

**Login:**

- User: `admin`
    
- Pass:
    

```bash
kubectl get secret -n monitoring monitoring-grafana \
  -o jsonpath="{.data.admin-password}" | base64 -d
```

---

# 🔹 STEP 2 – DEPLOY WEB APP (PODINFO)

## 🎯 Mục tiêu

- Có **HTTP service**
    
- Có **metrics Prometheus**
    
- Có **ServiceMonitor** để Prometheus tự scrape
    

---

## 2.1 Thêm Helm repo Podinfo

```bash
helm repo add stefanprodan https://stefanprodan.github.io/podinfo
helm repo update
```

---

## 2.2 Deploy Podinfo (CỰC QUAN TRỌNG)

```bash
helm install my-web stefanprodan/podinfo \
  --namespace default \
  --set serviceMonitor.enabled=true
```

---

## 2.3 Kiểm tra

```bash
kubectl get pods
kubectl get svc
kubectl get servicemonitor
```

✔️ Có:

- Pod `my-web-podinfo-*`
    
- Service `my-web-podinfo`
    
- ServiceMonitor `my-web-podinfo`
    

---

## 2.4 Test web

```bash
kubectl port-forward svc/my-web-podinfo 9898:9898
```

Mở:

```
http://localhost:9898
```

✔️ Thấy giao diện Podinfo  
✔️ `/metrics` hoạt động

---

# 🔹 STEP 3 – KIỂM TRA METRICS TRONG PROMETHEUS (RẤT QUAN TRỌNG)

## 🎯 Mục tiêu

Xác nhận **Prometheus đã thu thập HTTP request metrics** → nếu không có, KEDA không scale được

---

## 3.1 Mở Prometheus UI

```bash
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-prometheus 9090
```

👉 `http://localhost:9090`

---

## 3.2 Query test (DÁN VÀO PROMETHEUS)

```promql
rate(http_requests_total[2m])
```

✔️ Nếu có graph → OK  
✔️ Không có → báo mình ngay (80% lỗi nằm ở đây)

---

# 🔹 STEP 4 – CẤU HÌNH KEDA SCALEDOBJECT (CORE CỦA ĐỒ ÁN)

## 🎯 Mục tiêu

- KEDA hỏi Prometheus:
    
    > “RPS hiện tại là bao nhiêu?”
    
- Nếu **RPS > 10 → scale**
    

---

## 4.1 Tạo file `scaledobject-podinfo.yaml`

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: podinfo-rps-scaler
  namespace: default
spec:
  scaleTargetRef:
    name: my-web-podinfo
  minReplicaCount: 1
  maxReplicaCount: 10
  pollingInterval: 15
  cooldownPeriod: 30
  triggers:
  - type: prometheus
    metadata:
      serverAddress: http://monitoring-kube-prometheus-prometheus.monitoring:9090
      metricName: http_requests_per_second
      threshold: "10"
      query: |
        sum(rate(http_requests_total[2m]))
```

---

## 4.2 Apply

```bash
kubectl apply -f scaledobject-podinfo.yaml
```

---

## 4.3 Kiểm tra

```bash
kubectl get scaledobject
kubectl describe scaledobject podinfo-rps-scaler
```

✔️ Không error  
✔️ Không báo `unable to fetch metrics`

---

# 🔹 STEP 5 – CÀI LOAD TEST (k6)

## 🎯 Mục tiêu

Tạo **traffic giả** giống người dùng thật

---

## 5.1 Cài k6

```bash
sudo apt update
sudo apt install -y k6
```

---

## 5.2 Tạo file `load_test.js`

```js
import http from 'k6/http';
import { sleep } from 'k6';

export let options = {
  stages: [
    { duration: '30s', target: 50 }, // ramp-up
    { duration: '1m', target: 50 },  // sustain
    { duration: '30s', target: 0 },  // ramp-down
  ],
};

export default function () {
  http.get('http://<NODE-IP>:<NODEPORT>');
  sleep(1);
}
```

👉 Mình sẽ hướng dẫn bạn **lấy đúng URL Podinfo** ở bước kế tiếp

---

# 🔹 STEP 6 – CHẠY DEMO “TRIỆU ĐÔ”

## 6.1 Lấy URL Podinfo

```bash
kubectl get svc my-web-podinfo
```

Nếu là `NodePort`:

```
http://NODE_IP:NODEPORT
```

---

## 6.2 Mở 3 cửa sổ

### Terminal 1 – Theo dõi pod

```bash
kubectl get pods -w
```

---

### Terminal 2 – Chạy k6

```bash
k6 run load_test.js
```

---

### Trình duyệt – Grafana

- Import dashboard Podinfo / Kubernetes
    
- Xem:
    
    - Requests/sec
        
    - Pod count
        

---

## 🎬 Kết quả mong đợi

|Hiện tượng|Ý nghĩa|
|---|---|
|RPS tăng|Traffic thật|
|Pod tăng 1 → 10|KEDA scale|
|k6 dừng → Pod giảm|Scale in|

---

# 🔚 SAU BƯỚC NÀY, BẠN ĐẠT ĐƯỢC

✅ Autoscaling theo **HTTP traffic thật**  
✅ Monitoring chuẩn production  
✅ Demo cực thuyết phục  
✅ Đồ án **ăn đứt Queue-based scaling**

---



Dưới đây là hướng dẫn cụ thể và chi tiết nhất để triển khai mô hình kiến trúc Kubernetes của bạn (dựa trên sơ đồ: Users -> NGINX Edge Proxy/MetalLB -> Ingress Controller -> Load Balancer/Service -> Pods, với RabbitMQ làm message broker, KEDA + HPA cho autoscaling dựa trên queue metrics, Prometheus exporter cho monitoring). Tôi sẽ sử dụng **K3s** (như khuyến nghị trước) trên **Ubuntu 24.04.3 LTS** (phiên bản point release mới nhất của Noble Numbat vào tháng 8/2025, với các bản vá bảo mật và hỗ trợ tốt cho Kubernetes v1.35+). Hướng dẫn dựa trên các best practices từ tài liệu chính thức K3s và các tutorial cập nhật đến cuối 2025, tập trung vào môi trường virtualized/local (sử dụng hypervisor như VirtualBox hoặc VMware).

### 1. **Kế hoạch tổng thể và số lượng máy ảo**
Để triển khai production-like, hỗ trợ scaling và high availability (HA) cơ bản (ví dụ: RabbitMQ cluster, pods phân tán), tôi đề xuất **multi-node cluster với 3 máy ảo (VM)**:
- **VM1: Master Node** (control plane chính, chạy K3s server; nơi bạn sẽ quản lý cluster qua kubectl và Helm).
- **VM2 và VM3: Worker Nodes** (chạy K3s agent; nơi deploy pods, RabbitMQ, và các services để test scaling).

Lý do: Single-node (chỉ 1 VM) đơn giản nhưng không test được phân tán; 3 VM là minimum cho HA cơ bản mà không quá phức tạp. Nếu resources host hạn chế, bắt đầu với single-node (chỉ VM1), rồi mở rộng sau.

**Specs khuyến nghị cho mỗi VM** (dựa trên requirements K3s: min 2 cores/2GB RAM, nhưng tăng để chạy mượt với mô hình):
- CPU: 4 cores (để handle scaling và Prometheus scraping).
- RAM: 8GB (RabbitMQ và K3s ăn RAM; min 4GB nhưng sẽ chậm khi scale).
- Disk: 50GB SSD (20GB min, nhưng cần space cho container images và logs).
- Network: Bridge adapter (không NAT) để MetalLB và external access hoạt động (IP range ví dụ: 192.168.1.100-192.168.1.150).

**Host machine requirements**: Ít nhất 12-16 cores CPU, 32GB RAM, 200GB SSD để chạy 3 VM song song mà không lag. Enable nested virtualization trong BIOS để VM chạy container tốt.

**Cấu hình chung cho tất cả VM**:
- OS: Ubuntu 24.04.3 LTS Server (headless, tải ISO từ ubuntu.com/releases/24.04/ubuntu-24.04.3-live-server-amd64.iso).
- Hostname: Đặt khác nhau (ví dụ: k3s-master, k3s-worker1, k3s-worker2) trong cài đặt OS.
- Network: Static IP trong cùng subnet (ví dụ: master 192.168.1.10, worker1 192.168.1.11, worker2 192.168.1.12). DNS: 8.8.8.8.
- User: Tạo user non-root (ví dụ: ubuntu) với sudo, enable SSH.

### 2. **Quy trình triển khai chi tiết**
Theo thứ tự: Tạo VM -> Cài OS -> Config cơ bản -> Install K3s -> Deploy components. Thời gian ước tính: 1-2 giờ cho setup ban đầu, cộng test.

#### Bước 1: Tạo và cài đặt OS trên các VM
- Sử dụng VirtualBox/VMware: Tạo 3 VM với specs trên.
- Boot từ ISO Ubuntu 24.04.3 LTS Server.
- Trong installer:
  - Chọn "Minimal installation".
  - Partition: Default (full disk với LVM).
  - Enable SSH server.
  - Không install third-party drivers trừ khi cần.
- Sau install, reboot và SSH vào từng VM từ host (ssh ubuntu@IP).

#### Bước 2: Config cơ bản trên tất cả 3 VM
Chạy các lệnh sau trên mỗi VM (làm song song để tiết kiệm thời gian):
```
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl net-tools ufw
sudo ufw allow 22/tcp  # SSH
sudo ufw allow 6443/tcp  # Kubernetes API
sudo ufw allow 10250/tcp  # Kubelet
sudo ufw enable
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab  # Disable swap permanently
sudo modprobe overlay && sudo modprobe br_netfilter
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
sudo reboot
```

#### Bước 3: Install K3s trên VM1 (Master Node)
Chỉ chạy trên VM1:
```
curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644  # Install K3s server (default Traefik Ingress, nhưng ta sẽ dùng NGINX)
```
- Check status: `sudo k3s kubectl get nodes` (nên thấy master ready).
- Lấy token join cho workers: `sudo cat /var/lib/rancher/k3s/server/node-token` (copy token này, ví dụ: K10abc::server:secret).
- Export kubeconfig để dùng kubectl: `export KUBECONFIG=/etc/rancher/k3s/k3s.yaml`. Thêm vào ~/.bashrc để persistent: `echo 'export KUBECONFIG=/etc/rancher/k3s/k3s.yaml' >> ~/.bashrc && source ~/.bashrc`.

#### Bước 4: Install K3s trên VM2 và VM3 (Worker Nodes)
Chạy trên từng worker (thay K3S_URL bằng IP của master, ví dụ https://192.168.1.10:6443, và K3S_TOKEN bằng token từ bước 3):
```
curl -sfL https://get.k3s.io | K3S_URL=https://<master-ip>:6443 K3S_TOKEN=<token> sh -
```
* Token lấy được từ Server về : 
```
K10226de37466f6daba77a7a33107c2ebafbbd790d15c75972b3d8e4a029ec3d181::server:17ef95d9d93e3a4d417e0d1462add1dc
```

curl -sfL https://get.k3s.io | K3S_URL=https://192.168.1.153:6443 K3S_TOKEN=17ef95d9d93e3a4d417e0d1462add1dc sh -

K10226de37466f6daba77a7a33107c2ebafbbd790d15c75972b3d8e4a029ec3d181::server:17ef95d9d93e3a4d417e0d1462add1dc


17ef95d9d93e3a4d417e0d1462add1dc
- Trở về master (VM1), check: `kubectl get nodes` (nên thấy 3 nodes ready).

#### Bước 5: Install Helm trên VM1 (Master)
Helm dùng để deploy các charts (RabbitMQ, KEDA, etc.). Chạy trên VM1:
```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version  # Check v3.16+ (mới nhất 2025)
```

#### Bước 6: Deploy các thành phần mô hình trên cluster (từ VM1)
Tất cả deploy qua Helm/kubectl trên master; components sẽ phân tán tự động trên workers.
- **MetalLB (cho Load Balancer trong local)**:
  ```
  helm repo add metallb https://metallb.github.io/metallb
  helm install metallb metallb/metallb --create-namespace --namespace metallb-system
  ```
  - Tạo ConfigMap cho IP pool (edit YAML và apply: `kubectl apply -f metallb-config.yaml`):
    ```
    apiVersion: v1
    kind: ConfigMap
    metadata:
      namespace: metallb-system
      name: config
    data:
      config: |
        address-pools:
        - name: default
          protocol: layer2
          addresses:
          - 192.168.1.100-192.168.1.150  # Thay bằng range IP của VM network
    ```

- **NGINX Ingress Controller (cho Ingress và Edge Proxy)**:
  ```
  helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
  helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace --set controller.service.type=LoadBalancer
  ```
  - Nếu cần NGINX Edge Proxy riêng (additional layer), deploy như Deployment custom với NGINX image (YAML đơn giản với reverse proxy config).

- **RabbitMQ (với AMQP port 5672, Management API, và Prometheus exporter)**:
  ```
  helm repo add bitnami https://charts.bitnami.com/bitnami
  helm install rabbitmq bitnami/rabbitmq --namespace default --set metrics.enabled=true --set plugins="rabbitmq_prometheus" --set service.type=ClusterIP
  ```
  - Access management UI để test: `kubectl port-forward svc/rabbitmq 15672:15672` (truy cập localhost:15672 từ host, default user/pass: guest/guest).

- **Prometheus (cho monitoring và exporter integration)**:
  ```
  helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
  helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace
  ```
  - Config scrape RabbitMQ exporter: Edit Prometheus config để add job scrape port 15692 của RabbitMQ service.

- **KEDA (cho autoscaling với HPA)**:
  ```
  helm repo add kedacore https://kedacore.github.io/charts
  helm install keda kedacore/keda --namespace keda --create-namespace
  ```

- **Tạo ScaledObject cho KEDA (autoscaling pods dựa trên RabbitMQ queue)**:
  - Đầu tiên, deploy một sample Deployment cho pods (ví dụ: consumer app process messages từ RabbitMQ). Tạo YAML (your-deployment.yaml) và apply:
    ```
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: your-consumer
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: consumer
      template:
        metadata:
          labels:
            app: consumer
        spec:
          containers:
          - name: consumer
            image: your-image  # Thay bằng image app của bạn (ví dụ: rabbitmq-consumer)
    ```
  - Sau đó, tạo ScaledObject (scaledobject.yaml) và apply:
    ```
    apiVersion: keda.sh/v1alpha1
    kind: ScaledObject
    metadata:
      name: rabbitmq-scaler
      namespace: default
    spec:
      scaleTargetRef:
        name: your-consumer
      minReplicaCount: 1
      maxReplicaCount: 10
      triggers:
      - type: rabbitmq
        metadata:
          protocol: http
          queueName: your-queue  # Tạo queue trong RabbitMQ
          queueLength: '10'  # Scale nếu queue > 10 messages/pod
          host: rabbitmq.default.svc.cluster.local:15672
          username: guest
          password: guest
    ```
  - HPA sẽ tự động tạo bởi KEDA.

#### Bước 7: Test và verify mô hình
- Từ VM1: `kubectl get all --all-namespaces` (check pods/services running).
- Tạo queue và gửi messages vào RabbitMQ (dùng producer test), observe scaling: `kubectl get hpa` và `kubectl get pods -w`.
- Test traffic: Curl IP từ MetalLB (expose Ingress), kiểm tra routing đến pods.
- Monitoring: `kubectl port-forward svc/prometheus-server 9090:9090 -n monitoring` (access localhost:9090 để xem metrics từ RabbitMQ exporter).

#### Bước 8: Troubleshooting và best practices
- Logs: `journalctl -u k3s` trên nodes, hoặc `kubectl logs <pod>`.
- Common issues: Network (kiểm tra ufw/firewall), resources (tăng RAM nếu pod evicted).
- Backup: Sao lưu /etc/rancher/k3s trên master.
- Uninstall nếu cần: `/usr/local/bin/k3s-uninstall.sh` trên master, `/usr/local/bin/k3s-agent-uninstall.sh` trên workers.
- Mở rộng: Add worker mới tương tự bước 4.

Nếu gặp lỗi cụ thể (ví dụ: "no IP allocated" từ MetalLB), chia sẻ logs để hỗ trợ. Bắt đầu với single-node nếu phức tạp, rồi join workers sau!



#### Bước 6: Deploy các thành phần mô hình trên cluster (từ VM1)
Tất cả các lệnh deploy đều thực hiện trên **VM1 (Master Node)** vì đây là nơi bạn quản lý cluster qua kubectl và Helm. Các components sẽ được phân tán tự động lên các worker nodes (VM2 và VM3) dựa trên scheduler của Kubernetes. Trước khi bắt đầu, đảm bảo bạn đã export kubeconfig trên VM1: `export KUBECONFIG=/etc/rancher/k3s/k3s.yaml` (nếu chưa có, thêm vào ~/.bashrc và source lại).

Tôi sẽ làm chi tiết từng component, bao gồm:
- Lệnh cài đặt.
- Các tùy chỉnh cần thiết (dựa trên sơ đồ của bạn).
- Bước kiểm tra ngay sau để đảm bảo hoạt động (sử dụng `kubectl` để verify pods, services, logs).

Lưu ý: Nếu gặp lỗi "repository not found", kiểm tra internet connection trên VM1 (ping google.com). Các Helm repo dựa trên dữ liệu cập nhật đến tháng 12/2025. Nếu cần version cụ thể (ví dụ: MetalLB 0.15.3), thêm `--version=0.15.3` vào `helm install`.

- **MetalLB (cho Load Balancer trong local)**:
  - Thêm repo: `helm repo add metallb https://metallb.github.io/metallb` (chạy trên VM1).
  - Update repo để lấy latest: `helm repo update` (chạy trên VM1).
  - Install: `helm install metallb metallb/metallb --create-namespace --namespace metallb-system` (chạy trên VM1). Điều này install MetalLB latest (khoảng 0.15.3 vào 12/2025).
  - Tạo ConfigMap cho IP pool: Tạo file `metallb-config.yaml` trên VM1 (sử dụng nano hoặc vi), paste nội dung sau và save:
    ```
    apiVersion: v1
    kind: ConfigMap
    metadata:
      namespace: metallb-system
      name: config
    data:
      config: |
        address-pools:
        - name: default
          protocol: layer2
          addresses:
          - 192.168.1.100-192.168.1.200-250  # Thay bằng range IP thực tế của network VM (check với `ip addr` trên VM1, chọn range free trong subnet, ví dụ nếu subnet 192.168.1.0/24 thì dùng 192.168.1.200-192.168.1.250 để tránh conflict).
    ```
    Apply: `kubectl apply -f metallb-config.yaml` (chạy trên VM1).
  - **Kiểm tra**:
    - `kubectl get pods -n metallb-system` (chạy trên VM1): Nên thấy pods như controller và speaker running/ready (1-2 phút chờ). Nếu pending, check logs: `kubectl logs -n metallb-system <pod-name>`.
    - `kubectl get configmap config -n metallb-system -o yaml` (chạy trên VM1): Xác nhận IP pool đúng.
    - Nếu lỗi "no IP allocated" sau này, kiểm tra firewall trên tất cả VM: `sudo ufw status` (đảm bảo allow ports 7946 cho BGP nếu dùng mode BGP thay layer2).

- **NGINX Ingress Controller (cho Ingress và Edge Proxy)**:
  - Thêm repo: `helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx` (chạy trên VM1).
  - Update repo: `helm repo update` (chạy trên VM1).
  - Install: `helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace --set controller.service.type=LoadBalancer` (chạy trên VM1). Sử dụng LoadBalancer để MetalLB assign external IP. Latest version khoảng 4.14.1.
  - Nếu cần NGINX Edge Proxy riêng (additional layer cho edge routing, như trong sơ đồ): Tạo file `nginx-edge-proxy.yaml` trên VM1:

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-edge-proxy
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-edge
  template:
    metadata:
      labels:
        app: nginx-edge
    spec:
      containers:
      - name: nginx
        image: nginx:latest
        ports:
        - containerPort: 80
        volumeMounts:
        - name: config
          mountPath: /etc/nginx/conf.d
      volumes:
      - name: config
        configMap:
          name: nginx-edge-config
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-edge-config
  namespace: default
data:
  default.conf: |
    server {
      listen 80;
      location / {
        proxy_pass http://ingress-nginx.ingress-nginx.svc.cluster.local; # Proxy to Ingress
      }
    }
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-edge-service
spec:
  type: LoadBalancer # For MetalLB to assign IP
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx-edge
    
    Apply: `kubectl apply -f nginx-edge-proxy.yaml` (chạy trên VM1).
  - **Kiểm tra**:
    - `kubectl get pods -n ingress-nginx` (chạy trên VM1): Pods ready (ingress-nginx-controller).
    - `kubectl get svc -n ingress-nginx` (chạy trên VM1): Service ingress-nginx-controller có External-IP từ MetalLB (chờ 1-2 phút). Nếu pending, check MetalLB logs.
    - Test basic: `curl <external-ip>` từ host machine (nên thấy NGINX default page nếu chưa có Ingress rules).
    - Nếu Edge Proxy: `kubectl get pods | grep nginx-edge` (chạy trên VM1): Pod running. Check nào VM chạy pod: `kubectl describe pod <pod-name>` (xem Node).

- **RabbitMQ (với AMQP port 5672, Management API, và Prometheus exporter)**:
  - Thêm repo: `helm repo add bitnami https://charts.bitnami.com/bitnami` (chạy trên VM1).
  - Update repo: `helm repo update` (chạy trên VM1).
  - Install: `helm install rabbitmq bitnami/rabbitmq --namespace default --set metrics.enabled=true --set plugins="rabbitmq_prometheus" --set service.type=ClusterIP` (chạy trên VM1). Latest khoảng 16.0.14. Để HA, thêm `--set replicaCount=3` nếu muốn cluster 3 nodes.
  - **Kiểm tra**:
    - `kubectl get pods | grep rabbitmq` (chạy trên VM1): Pods ready (rabbitmq-0, etc.). Nếu HA, chúng phân tán trên VM2/VM3.
    - `kubectl get svc rabbitmq` (chạy trên VM1): Service ClusterIP với ports 5672 (AMQP), 15672 (Management), 15692 (Exporter).
    - Test access: Trên host machine (không trên VM), chạy `kubectl port-forward svc/rabbitmq 15672:15672 --address 0.0.0.0` (trên VM1 để forward), rồi mở browser host: http://localhost:15672, login guest/guest. Tạo test queue.
    - Check exporter: `kubectl port-forward svc/rabbitmq 15692:15692` (trên VM1), curl localhost:15692/metrics từ host (nên thấy Prometheus metrics).

- **Prometheus (cho monitoring và exporter integration)**:
  - Thêm repo: `helm repo add prometheus-community https://prometheus-community.github.io/helm-charts` (chạy trên VM1).
  - Update repo: `helm repo update` (chạy trên VM1).
  - Install: `helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace` (chạy trên VM1). Latest khoảng 27.52.0.
  - Config scrape RabbitMQ exporter: Edit Prometheus config (sử dụng ConfigMap hoặc values.yaml nếu reinstall). Tạo file `prometheus-additional.yaml` trên VM1:
    ```
    - job_name: rabbitmq
      static_configs:
      - targets: ['rabbitmq.default.svc.cluster.local:15692']
    ```
    Apply như additionalScrapeConfigs: Reinstall Helm với `--set server.additionalScrapeConfigsSecret.name=additional-scrape-configs --set server.additionalScrapeConfigsSecret.key=prometheus-additional.yaml`, hoặc edit ConfigMap `prometheus-server` sau install: `kubectl edit cm prometheus-server -n monitoring` (thêm scrape job dưới scrape_configs).
  - **Kiểm tra**:
    - `kubectl get pods -n monitoring` (chạy trên VM1): Pods ready (prometheus-server, node-exporter, etc.).
    - `kubectl port-forward svc/prometheus-server 9090:9090 -n monitoring` (chạy trên VM1), mở browser host: http://localhost:9090/targets – thấy job rabbitmq up.
    - Query metrics: Trong Prometheus UI, search "rabbitmq_queue_messages" – nên thấy data từ exporter nếu queue có messages.

- **KEDA (cho autoscaling với HPA)**:
  - Thêm repo: `helm repo add kedacore https://kedacore.github.io/charts` (chạy trên VM1).
  - Update repo: `helm repo update` (chạy trên VM1).
  - Install: `helm install keda kedacore/keda --namespace keda --create-namespace` (chạy trên VM1). Latest khoảng 2.18.2.
  - **Kiểm tra**:
    - `kubectl get pods -n keda` (chạy trên VM1): Pods ready (keda-operator, metrics-server).
    - `kubectl get crd | grep keda` (chạy trên VM1): Thấy ScaledObject CRD installed.

- **Tạo ScaledObject cho KEDA (autoscaling pods dựa trên RabbitMQ queue)**:
  - Đầu tiên, deploy sample Deployment cho pods (consumer app): Tạo file `your-deployment.yaml` trên VM1, paste nội dung (thay image bằng một consumer thực tế, ví dụ: rmqio/rabbitmq-perf-test nếu test):
    ```
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: your-consumer
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: consumer
      template:
        metadata:
          labels:
            app: consumer
        spec:
          containers:
          - name: consumer
            image: rmqio/rabbitmq-perf-test:latest  # Ví dụ image test, thay bằng app thật của bạn
            env:
            - name: RABBITMQ_HOST
              value: rabbitmq.default.svc.cluster.local
    ```
    Apply: `kubectl apply -f your-deployment.yaml` (chạy trên VM1).
  - Sau đó, tạo ScaledObject: Tạo file `scaledobject.yaml` trên VM1, paste:
    ```
    apiVersion: keda.sh/v1alpha1
    kind: ScaledObject
    metadata:
      name: rabbitmq-scaler
      namespace: default
    spec:
      scaleTargetRef:
        name: your-consumer
      minReplicaCount: 1
      maxReplicaCount: 10
      triggers:
      - type: rabbitmq
        metadata:
          protocol: http
          queueName: your-queue  # Thay bằng queue name bạn tạo ở RabbitMQ UI
          queueLength: '10'
          host: rabbitmq.default.svc.cluster.local:15672
          username: guest
          password: guest
    ```
    Apply: `kubectl apply -f scaledobject.yaml` (chạy trên VM1).
  - **Kiểm tra**:
    - `kubectl get deployment your-consumer` (chạy trên VM1): Replicas 1.
    - `kubectl get scaledobject rabbitmq-scaler` (chạy trên VM1): Ready.
    - `kubectl get hpa` (chạy trên VM1): HPA tạo bởi KEDA.

#### Bước 7: Test và verify mô hình
Thực hiện trên **VM1** trừ khi chỉ định, nhưng test traffic có thể từ host machine.

- Check tổng quát: `kubectl get all --all-namespaces` (chạy trên VM1): Liệt kê tất cả pods/services, đảm bảo hầu hết ready (không có CrashLoop).
- Test RabbitMQ và scaling:
  - Tạo queue: Sử dụng port-forward như trên, vào UI http://localhost:15672, tạo queue "your-queue".
  - Gửi messages: Sử dụng tool như rabbitmqadmin (install trên VM1: `sudo apt install python3-pip && pip install rabbitmqadmin`), hoặc producer sample. Ví dụ, deploy pod producer tạm: Tạo YAML tương tự deployment với image producer, apply, gửi 100 messages vào queue.
  - Observe scaling: `kubectl get hpa -w` và `kubectl get pods -w` (chạy trên VM1): Khi queue >10 messages/pod, replicas tăng (chờ 1-2 phút vì polling interval).
- Test traffic: 
  - Lấy External-IP từ `kubectl get svc -n ingress-nginx` (chạy trên VM1).
  - Curl từ host: `curl http://<external-ip>` (kiểm tra response từ pods nếu có Ingress rule; nếu chưa, tạo Ingress YAML point đến service của your-consumer).
- Monitoring: `kubectl port-forward svc/prometheus-server 9090:9090 -n monitoring --address 0.0.0.0` (chạy trên VM1), truy cập http://localhost:9090 từ host, query metrics như rabbitmq_up để confirm exporter hoạt động.

#### Bước 8: Troubleshooting và best practices
- Logs: Trên bất kỳ VM nào chạy pod lỗi: `journalctl -u k3s` (system logs), hoặc `kubectl logs <pod-name> -n <namespace>` (pod logs, chạy trên VM1).
- Common issues:
  - Network: Kiểm tra ufw trên tất cả VM: `sudo ufw status` (allow thêm ports nếu cần, ví dụ 7946 cho MetalLB).
  - Resources: Nếu pod evicted, tăng RAM/CPU VM (check `kubectl describe pod <pod>` trên VM1). Trên worker (VM2/VM3): `top` để xem usage.
  - Scaling không trigger: Check KEDA logs `kubectl logs -n keda <keda-pod>`, đảm bảo RabbitMQ credentials đúng.
- Backup: Sao lưu trên VM1: `cp -r /etc/rancher/k3s ~/backup-k3s`.
- Uninstall: Trên VM1: `/usr/local/bin/k3s-uninstall.sh`; trên VM2/VM3: `/usr/local/bin/k3s-agent-uninstall.sh`.
- Mở rộng: Để add VM4 như worker, lặp bước 4 trên VM mới.

Nếu lỗi cụ thể, chạy `kubectl describe <resource>` trên VM1 và chia sẻ output!
