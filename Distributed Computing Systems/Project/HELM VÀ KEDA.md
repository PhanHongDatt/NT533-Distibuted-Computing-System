OK 👍 mình sẽ **gộp và hướng dẫn liên tục BƯỚC 3 → 4 → 5 → 6**, nhưng vẫn **giữ đúng thứ tự – dễ kiểm tra – không rối**.  
Bạn chỉ cần **copy–paste làm theo**, làm xong mỗi bước có checklist để tự xác nhận.

---

# 🟦 BƯỚC 3 – CÀI HELM (TRÌNH QUẢN LÝ ỨNG DỤNG K8S)

## 🎯 Mục tiêu

- Dùng **1 dòng lệnh** để cài Prometheus, Grafana, KEDA, Podinfo
    
- Chuẩn DevOps / Production
    

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

✔️ Thấy version là OK

---

## ✅ CHECKLIST BƯỚC 3

|Mục|Trạng thái|
|---|---|
|Helm cài thành công|✅|
|helm version chạy được|✅|

---

# 🟦 BƯỚC 4 – CÀI KEDA (AUTOSCALING ENGINE)

> KEDA = thứ giúp **scale theo traffic**, không chỉ CPU/RAM

---

## 4.1 Thêm Helm repo KEDA

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

⏳ ~30 giây

---

## 4.3 Kiểm tra

```bash
kubectl get pods -n keda
```

✔️ Phải thấy:

- `keda-operator`
    
- `keda-metrics-apiserver`  
    (trạng thái `Running`)
    

---

## 🧠 Giải thích ngắn (để thuyết trình)

- Kubernetes gốc **không hiểu HTTP request**
    
- KEDA:
    
    - Lấy metrics từ **Prometheus**
        
    - Ra lệnh cho **HPA**
        
    - Scale Pod
        

---

## ✅ CHECKLIST BƯỚC 4

|Mục|Trạng thái|
|---|---|
|Namespace keda tồn tại|✅|
|Pod keda Running|✅|

---

# 🟦 BƯỚC 5 – CÀI PROMETHEUS + GRAFANA (MONITORING)

## 🎯 Mục tiêu

- Thu thập metrics HTTP
    
- Có dashboard demo trực quan
    

---

## 5.1 Thêm Helm repo Prometheus

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

---

## 5.2 Cài kube-prometheus-stack

```bash
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring \
  --create-namespace
```

⏳ 2–3 phút (kiên nhẫn)

---

## 5.3 Kiểm tra

```bash
kubectl get pods -n monitoring
```

✔️ Các pod sau phải `Running`:

- prometheus
    
- grafana
    
- alertmanager
    
- node-exporter
    

---

## 5.4 Truy cập Grafana (tạm thời)

```bash
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
```

👉 Mở trình duyệt:  
`http://localhost:3000`

**User:** `admin`  
**Password:**

```bash
kubectl get secret -n monitoring monitoring-grafana \
-o jsonpath="{.data.admin-password}" | base64 -d
```

---

## ✅ CHECKLIST BƯỚC 5

|Mục|Trạng thái|
|---|---|
|Prometheus Running|✅|
|Grafana truy cập được|✅|

---

# 🟦 BƯỚC 6 – DEPLOY WEB APP (PODINFO)

> Podinfo = app demo **chuẩn K8s**, có sẵn HTTP metrics

---

## 6.1 Thêm Helm repo Podinfo

```bash
helm repo add stefanprodan https://stefanprodan.github.io/podinfo
helm repo update
```

---

## 6.2 Deploy Podinfo (CỰC KỲ QUAN TRỌNG)

```bash
helm install my-web stefanprodan/podinfo \
  --namespace default \
  --set serviceMonitor.enabled=true
```

📌 `serviceMonitor.enabled=true`  
→ Prometheus **tự động scrape metrics**

---

## 6.3 Kiểm tra

```bash
kubectl get pods
kubectl get svc
kubectl get servicemonitor
```

✔️ Thấy:

- Pod `my-web-podinfo-*`
    
- Service `my-web-podinfo`
    
- ServiceMonitor `my-web-podinfo`
    

---

## 6.4 Test web

```bash
kubectl port-forward svc/my-web-podinfo 9898:9898
```

Mở:

```
http://localhost:9898
```

✔️ Thấy giao diện Podinfo

---

## 🧠 Giải thích để bảo vệ đồ án

- Podinfo:
    
    - Nhận HTTP request
        
    - Export `http_requests_total`
        
- Đây chính là **nguồn dữ liệu** cho autoscaling
    

---

## ✅ CHECKLIST BƯỚC 6

|Mục|Trạng thái|
|---|---|
|Podinfo chạy|✅|
|ServiceMonitor tồn tại|✅|
|Truy cập web OK|✅|

---

# 🎉 SAU BƯỚC 3–6, BẠN ĐÃ CÓ

✅ Kubernetes  
✅ Helm  
✅ KEDA  
✅ Prometheus + Grafana  
✅ Web App có metrics

👉 **HỆ THỐNG ĐÃ SẴN SÀNG SCALE THEO TRAFFIC**

---

## 👉 BƯỚC TIẾP THEO (QUAN TRỌNG NHẤT)

Mình đề xuất làm tiếp **BƯỚC 7**:

> 🔥 **Cấu hình KEDA ScaledObject dùng Prometheus (RPS-based scaling)**

Hoặc nếu bạn muốn:

- Giải thích **Prometheus query** trước
    
- Kiểm tra metrics đã vào Prometheus chưa
    
- Hoặc viết luôn **k6 load test**
    

👉 Bạn trả lời:  
**“Bước 7”** hoặc **“Giải thích Prometheus trước”**