Rất tốt 👍 Cách bạn yêu cầu **“coi như chưa có gì cả”** là đúng chuẩn tư duy kỹ sư hệ thống.  
Mình sẽ hướng dẫn bạn **từ con số 0**, theo **đúng thứ tự logic – ít rủi ro – dễ debug**, và **mỗi bước đều có mục tiêu + kiểm tra**.

> 📌 **Cam kết**:
> 
> - Không nhảy cóc
>     
> - Không giả định bạn “đã biết”
>     
> - Làm tới đâu **chạy được tới đó**
>     
> - Phù hợp **đồ án + demo bảo vệ**
>     

---

# 🧱 KIẾN TRÚC CUỐI CÙNG (để bạn hình dung ngay từ đầu)

```
Ubuntu VM
 ├── Docker (container runtime)
 ├── K3s (Kubernetes)
 │    ├── Podinfo (Web App)
 │    ├── KEDA (Autoscaler)
 │    ├── Prometheus (Metrics)
 │    └── Grafana (Dashboard)
 └── k6 (Load Test)
```

---

# 🧩 PHẦN 0 – CHUẨN BỊ MÁY (NỀN TẢNG)

## 🎯 Yêu cầu tối thiểu

| Thành phần | Khuyến nghị          |
| ---------- | -------------------- |
| OS         | Ubuntu 20.04 / 22.04 |
| RAM        | ≥ 4 GB (8 GB là đẹp) |
| CPU        | ≥ 2 cores            |
| Disk       | ≥ 30 GB              |
| User       | Có quyền `sudo`      |

---

## 0.1 Cập nhật hệ thống

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 0.2 Cài các gói cơ bản

```bash
sudo apt install -y \
  curl wget git vim \
  ca-certificates \
  gnupg lsb-release
```

---

## 0.3 Tắt swap (⚠️ bắt buộc cho Kubernetes)

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

✔️ Kiểm tra:

```bash
free -h
```

→ Swap phải là `0`

---

# 🧩 PHẦN 1 – CÀI DOCKER (CONTAINER RUNTIME)

> K3s **có thể không cần Docker**, nhưng cài Docker giúp bạn:
> 
> - Debug dễ
>     
> - Hiểu container từ gốc
>     
> - Thuyết trình đồ án tốt hơn
>     

---

## 1.1 Cài Docker

```bash
curl -fsSL https://get.docker.com | sudo sh
```

---

## 1.2 Cho user dùng Docker không cần sudo

```bash
sudo usermod -aG docker $USER
newgrp docker
```

---

## 1.3 Kiểm tra

```bash
docker version
docker run hello-world
```

✔️ Thấy “Hello from Docker!”

---

# 🧩 PHẦN 2 – CÀI KUBERNETES (K3s)

## 🎯 Mục tiêu

- Có **1-node Kubernetes**
    
- Nhẹ – ổn định – dùng cho đồ án
    

---

## 2.1 Cài K3s

```bash
curl -sfL https://get.k3s.io | sh -
```

⏳ ~30–60 giây

---

## 2.2 Kiểm tra node

```bash
sudo kubectl get nodes
```

✔️ Node `Ready`

---

## 2.3 Copy kubeconfig cho user thường

```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
```

---

## 2.4 Test kubectl

```bash
kubectl get pods -A
```

✔️ Thấy:

- coredns
    
- traefik
    
- local-path-provisioner
    

---

# 🧩 PHẦN 3 – CÀI HELM (QUẢN LÝ CHART)

## 🎯 Mục tiêu

- Cài Prometheus, Grafana, Podinfo **bằng 1 lệnh**
    

---

## 3.1 Cài Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

---

## 3.2 Kiểm tra

```bash
helm version
```

---

# 🧩 PHẦN 4 – CÀI KEDA (AUTOSCALING ENGINE)

## 🎯 Mục tiêu

- Cho phép scale dựa trên **metrics ngoài Kubernetes**
    

---

## 4.1 Thêm Helm repo

```bash
helm repo add kedacore https://kedacore.github.io/charts
helm repo update
```

---

## 4.2 Cài KEDA

```bash
helm install keda kedacore/keda \
  --namespace keda \
  --create-namespace
```

---

## 4.3 Kiểm tra

```bash
kubectl get pods -n keda
```

✔️ Có:

- keda-operator
    
- keda-metrics-apiserver
    

---

# 🧩 PHẦN 5 – CÀI PROMETHEUS + GRAFANA

## 🎯 Mục tiêu

- Thu thập metrics
    
- Trực quan hóa để demo
    

---

## 5.1 Thêm repo

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

---

## 5.2 Cài monitoring stack

```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```

⏳ 2–3 phút

---

## 5.3 Kiểm tra

```bash
kubectl get pods -n monitoring
```

✔️ Tất cả `Running`

---

# 🧩 PHẦN 6 – DEPLOY WEB APP (PODINFO)

## 🎯 Mục tiêu

- Có Web App
    
- Có HTTP metrics
    

---

## 6.1 Cài Podinfo

```bash
helm repo add stefanprodan https://stefanprodan.github.io/podinfo
helm repo update

helm install my-web stefanprodan/podinfo \
  --set serviceMonitor.enabled=true
```

---

## 6.2 Kiểm tra

```bash
kubectl get pods
kubectl get svc
kubectl get servicemonitor
```

---

# 🧩 PHẦN 7 – CÀI LOAD TEST (k6)

```bash
sudo apt install -y k6
```

```bash
k6 version
```

---

# 🧩 PHẦN 8 – KIỂM TRA END-TO-END (TRƯỚC KHI SCALE)

|Thành phần|Check|
|---|---|
|Web|Podinfo UI|
|Metrics|Prometheus query OK|
|Autoscale|KEDA ready|
|Monitoring|Grafana mở được|

---

# 🔚 TỚI ĐÂY BẠN ĐÃ CÓ

✅ Kubernetes từ số 0  
✅ Monitoring chuẩn production  
✅ Autoscaling engine  
✅ Sẵn sàng bước vào **Traffic-based Scaling**

---

👉 **Bước tiếp theo bạn muốn làm gì?**

1. **Cấu hình Prometheus query & ScaledObject** (phần khó nhất – mình làm rất kỹ)
    
2. **Viết k6 script + demo scaling**
    
3. **Giải thích kiến trúc + sơ đồ để đưa vào báo cáo**
    

👉 Bạn chỉ cần trả lời: **“Tiếp theo làm bước X”**