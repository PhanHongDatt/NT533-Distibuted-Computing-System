
Rất tốt 👍 ta **tiếp tục BƯỚC 2 – CÀI KUBERNETES (K3s)**  
Đây là **trái tim của toàn bộ đồ án**, nên mình sẽ làm **chậm – chắc – dễ kiểm tra**.

---

# 🟦 BƯỚC 2 – CÀI ĐẶT KUBERNETES BẰNG K3s

## 🎯 Mục tiêu của bước này

Sau bước 2, bạn sẽ:

- Có **1 Kubernetes cluster chạy thật**
    
- Dùng được `kubectl`
    
- Sẵn sàng deploy app + autoscaling
    

---

## 🧠 Vì sao chọn K3s?

|Lý do|Giải thích|
|---|---|
|Nhẹ|Phù hợp VM|
|Dễ cài|1 lệnh|
|Chuẩn K8s|Không phải “đồ chơi”|
|Được dùng thật|Edge / Lab / Demo|

---

# 2.1 Chuẩn bị bắt buộc (KHÔNG BỎ QUA)

## 2.1.1 Tắt swap (Kubernetes không cho phép)

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

✔️ Kiểm tra:

```bash
free -h
```

Swap = `0`

---

## 2.1.2 Bật ip_forward

```bash
sudo sysctl -w net.ipv4.ip_forward=1
```

---

# 2.2 Cài K3s (1 lệnh duy nhất)

```bash
curl -sfL https://get.k3s.io | sh -
```

⏳ Chờ ~30–60 giây

---

## 2.3 Kiểm tra K3s đã chạy chưa

```bash
sudo systemctl status k3s
```

✔️ `active (running)`

---

# 2.4 Kiểm tra Kubernetes node

```bash
sudo kubectl get nodes
```

### Kết quả đúng:

```
NAME           STATUS   ROLES                  AGE   VERSION
your-hostname  Ready    control-plane,master   ...
```

✔️ STATUS = `Ready`

---

# 2.5 Cho user thường dùng kubectl (RẤT QUAN TRỌNG)

## 2.5.1 Copy kubeconfig

```bash
mkdir -p ~/.kube
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER ~/.kube/config
```

---

## 2.5.2 Test kubectl (KHÔNG sudo)

```bash
kubectl get nodes
```

✔️ Chạy được → OK  
❌ Nếu lỗi permission → báo mình

---

# 2.6 Kiểm tra các thành phần hệ thống

```bash
kubectl get pods -A
```

### Bạn phải thấy:

- `coredns`
    
- `traefik`
    
- `local-path-provisioner`
    
- `metrics-server`
    

Tất cả đều `Running`

---

# 🧠 Giải thích để bảo vệ đồ án

- **K3s** = Kubernetes rút gọn
    
- Bao gồm:
    
    - API Server
        
    - Scheduler
        
    - Controller Manager
        
- Mọi workload sau này (**Podinfo, Prometheus, KEDA**) đều chạy trên đây
    

👉 _Nếu Kubernetes chết → tất cả chết_

---

# ✅ CHECKLIST BƯỚC 2

|Mục|Trạng thái|
|---|---|
|K3s cài thành công|✅|
|Node Ready|✅|
|kubectl dùng không sudo|✅|
|Core pods Running|✅|

---

👉 **XÁC NHẬN GIÚP MÌNH**  
Trả lời đúng 1 dòng:

> `K3s OK`

Sau đó mình sẽ chuyển sang **BƯỚC 3 – CÀI HELM (quản lý toàn bộ stack bằng 1 dòng lệnh)** 🚀