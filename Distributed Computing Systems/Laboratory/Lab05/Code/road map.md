
Rất đúng trọng tâm 👍  
Mình sẽ **viết lại TOÀN BỘ LAB 5 – K3s** theo **đúng tài liệu bạn gửi**, nhưng **máy host là Windows**, hướng dẫn **từng bước nhỏ – không bỏ bước – đúng kiểu làm bài lab & chụp hình báo cáo**.

> 🎯 Kết quả cuối:
> 
> - Windows → điều khiển cluster
>     
> - 2 Ubuntu VM → thay Raspberry Pi
>     
> - Dùng **kubectl + k3sup + arkade**
>     
> - Có **Dashboard**
>     
> - Đủ để **nộp LAB 5**
>     

---

# 🧪 LAB 5 – TRIỂN KHAI K3s (HOST WINDOWS)

---

## 🧱 MÔ HÌNH TRIỂN KHAI

```
+---------------------+
| Windows 10 / 11     |
| (Host)              |
| kubectl, k3sup      |
| arkade              |
+----------+----------+
           |
           | SSH
           v
+------------------+        +------------------+
| Ubuntu VM 1      |        | Ubuntu VM 2      |
| k3s-master       |        | k3s-agent        |
| 192.168.56.101   |        | 192.168.56.102   |
+------------------+        +------------------+
```

---

# 🔹 PHẦN A – CHUẨN BỊ MÁY HOST WINDOWS

---

## 🔹 BƯỚC 1 – CÀI VIRTUALBOX + TẠO UBUNTU VM

### 1.1 Cài VirtualBox

- Download: [https://www.virtualbox.org/](https://www.virtualbox.org/)
    
- Cài **VirtualBox Extension Pack** (cùng version)
    

📸 _Chụp hình: VirtualBox version_

---

### 1.2 Download Ubuntu Server

- Link: [https://ubuntu.com/download/server](https://ubuntu.com/download/server)
    
- Chọn: **Ubuntu Server 22.04 LTS**
    

---

### 1.3 Tạo VM MASTER

|Thuộc tính|Giá trị|
|---|---|
|Name|k3s-master|
|Type|Linux|
|Version|Ubuntu (64-bit)|
|CPU|2|
|RAM|2048–4096 MB|
|Disk|20 GB|
|Network|**Bridged Adapter**|

📌 Network **bắt buộc Bridged** để SSH từ Windows

---

### 1.4 Cài Ubuntu Server

- Chọn:
    
    - OpenSSH Server ✅
        
    - User: `ubuntu`
        
    - Password: tự đặt
        

📸 _Chụp hình màn hình cài đặt_

---

### 1.5 Clone VM → k3s-agent

- Chuột phải → Clone
    
- Name: `k3s-agent`
    
- Chọn **Full Clone**
    

---

### 1.6 Lấy IP các VM

Trên từng VM:

```bash
ip a
```

Giả sử:

```text
k3s-master: 192.168.56.101
k3s-agent : 192.168.56.102
```

📸 _Chụp hình IP_

---

# 🔹 PHẦN B – CÀI TOOL TRÊN WINDOWS

---

## 🔹 BƯỚC 2 – CÀI WINDOWS TERMINAL + WSL

### 2.1 Cài Windows Terminal

- Microsoft Store → Windows Terminal
    

---

### 2.2 Cài WSL (Ubuntu)

Mở **PowerShell (Admin)**:

```powershell
wsl --install
```

Reboot → mở **Ubuntu (WSL)**

📸 _Chụp hình Ubuntu WSL_

---

# 🔹 PHẦN C – CÀI KUBECTL, K3SUP, ARKADE (TRONG WSL)

> ⚠️ **Toàn bộ lệnh từ đây gõ trong Ubuntu WSL**

---

## 🔹 BƯỚC 3 – CÀI KUBECTL

```bash
sudo apt update
sudo apt install -y kubectl
kubectl version --client
```

📸 _Chụp hình version kubectl_

---

## 🔹 BƯỚC 4 – CÀI K3SUP

```bash
curl -sLS https://get.k3sup.dev | sh
sudo install k3sup /usr/local/bin/
k3sup version
```

---

## 🔹 BƯỚC 5 – CÀI ARKADE

```bash
curl -sLS https://dl.get-arkade.dev | sudo sh
arkade version
```

---

# 🔹 PHẦN D – SSH TỪ WINDOWS → UBUNTU VM

---

## 🔹 BƯỚC 6 – TẠO SSH KEY (TRONG WSL)

```bash
ssh-keygen
```

Enter hết

---

## 🔹 BƯỚC 7 – COPY SSH KEY SANG VM

```bash
ssh-copy-id ubuntu@192.168.56.101
ssh-copy-id ubuntu@192.168.56.102
```

Test:

```bash
ssh ubuntu@192.168.56.101
```

📸 _Chụp hình SSH thành công_

---

# 🔹 PHẦN E – TRIỂN KHAI K3s (ĐÚNG SLIDE)

---

## 🔹 BƯỚC 8 – CÀI K3s MASTER

```bash
export SERVER=192.168.56.101
k3sup install --ip $SERVER --user ubuntu
```

Sau khi xong:

```bash
export KUBECONFIG=$HOME/kubeconfig
kubectl get node
```

📸 _Chụp node master Ready_

---

## 🔹 BƯỚC 9 – JOIN AGENT

```bash
export AGENT=192.168.56.102

k3sup join \
  --ip $AGENT \
  --server-ip $SERVER \
  --user ubuntu
```

Kiểm tra:

```bash
kubectl get node -o wide
```

📸 _Chụp 2 node Ready_

---

# 🔹 PHẦN F – KUBERNETES DASHBOARD

---

## 🔹 BƯỚC 10 – CÀI DASHBOARD

```bash
arkade install kubernetes-dashboard
```

---

## 🔹 BƯỚC 11 – TẠO ADMIN USER

```bash
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

---

## 🔹 BƯỚC 12 – TRUY CẬP DASHBOARD

### Chạy proxy

```bash
kubectl proxy
```

### Lấy token

```bash
kubectl -n kubernetes-dashboard describe secret \
$(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

### Truy cập trình duyệt (Windows)

```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

📸 _Chụp giao diện Dashboard_
![[Pasted image 20251217151225.png]]

---

# ✅ HOÀN THÀNH LAB 5

✔ Host Windows  
✔ Không cần Raspberry Pi  
✔ Đúng yêu cầu tài liệu  
✔ Có thể viết báo cáo & nộp bài

---

## 👉 TIẾP THEO MÌNH CÓ THỂ:

- 🔹 Hướng dẫn **OpenFaaS (2 điểm)**
    
- 🔹 Viết **mẫu báo cáo LAB 5 (Word/PDF)**
    
- 🔹 Debug lỗi hay gặp (WSL, SSH, Bridged)
    

👉 Bạn muốn **làm tiếp OpenFaaS hay viết báo cáo trước**?

faas-cli new \
  --lang python3-http \
  my-hello \
  --prefix hogdac
