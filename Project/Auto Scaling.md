Rất chuẩn 👍  
Bây giờ ta làm **BƯỚC 7 – KIỂM TRA METRICS & TRIỂN KHAI AUTOSCALING THEO TRAFFIC (RPS)**  
Đây là **bước quan trọng nhất của toàn bộ đồ án**, mình sẽ làm **cực kỳ kỹ – đúng chuẩn học thuật**.

---

# 🟦 BƯỚC 7 – KIỂM TRA METRICS & TRIỂN KHAI KEDA SCALEDOBJECT

## 🎯 Mục tiêu bước 7

Sau bước này, hệ thống của bạn sẽ:

- ✅ Thu thập được **HTTP Requests metrics**
    
- ✅ KEDA đọc metrics từ **Prometheus**
    
- ✅ **Tự động scale Podinfo theo RPS**
    
- ✅ Sẵn sàng demo với k6
    

---

# PHẦN A – KIỂM TRA PROMETHEUS ĐÃ THU METRICS CHƯA (BẮT BUỘC)

> ⚠️ **Nếu bước này không OK → autoscaling chắc chắn không hoạt động**

---

## 7.1 Mở Prometheus UI

```bash
kubectl port-forward -n monitoring \
  svc/monitoring-kube-prometheus-prometheus 9090
```

👉 Truy cập:

```
http://localhost:9090
```

---

## 7.2 Kiểm tra target Podinfo

Vào tab **Status → Targets**

✔️ Bạn phải thấy target dạng:

```
serviceMonitor/default/my-web-podinfo/0
State: UP
```

❌ Nếu DOWN → báo mình (thường do ServiceMonitor hoặc label)

---

## 7.3 Query metric HTTP request

Trong ô query Prometheus, nhập:

```promql
http_requests_total
```

✔️ Có dữ liệu trả về  
❌ Không có → autoscaling **chưa thể làm**

---

## 7.4 Kiểm tra RPS (Requests Per Second)

```promql
rate(http_requests_total[2m])
```

✔️ Thấy đường biểu diễn (dù thấp)  
→ **Metric đã sẵn sàng**

---

## 🧠 Giải thích (để bảo vệ đồ án)

- `http_requests_total`  
    → tổng số request từ khi pod chạy
    
- `rate(...[2m])`  
    → tốc độ request / giây trong 2 phút
    
- Đây chính là **RPS – thước đo traffic thật**
    

---

# PHẦN B – TRIỂN KHAI KEDA SCALEDOBJECT (CORE LOGIC)

## 🎯 Mục tiêu

- Nếu **RPS > 10** → scale out
    
- Nếu **RPS thấp** → scale in
    

---

## 7.5 Tạo file `scaledobject-podinfo.yaml`

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

## 7.6 Apply ScaledObject

```bash
kubectl apply -f scaledobject-podinfo.yaml
```

---

## 7.7 Kiểm tra ScaledObject

```bash
kubectl get scaledobject
kubectl describe scaledobject podinfo-rps-scaler
```

✔️ Không có error  
✔️ Không có dòng `unable to get metrics`

---

## 7.8 Kiểm tra HPA được tạo tự động

```bash
kubectl get hpa
```

✔️ Thấy HPA dạng:

```
keda-hpa-podinfo-rps-scaler
```

👉 Điều này chứng tỏ:

> **KEDA đã chuyển metrics → Kubernetes HPA**

---

## 🧠 Giải thích học thuật (rất hay khi bảo vệ)

- KEDA **không scale trực tiếp**
    
- KEDA:
    
    1. Query Prometheus
        
    2. Chuyển metrics → HPA
        
    3. Kubernetes scale Pod
        
- Kiến trúc này **tách biệt rõ ràng**  
    → Dễ mở rộng, đúng chuẩn production
    

---

# PHẦN C – TEST NHANH SCALE (CHƯA CẦN k6)

## 7.9 Tạo traffic thủ công

Mở 1 terminal:

```bash
kubectl port-forward svc/my-web-podinfo 9898:9898
```

Mở terminal khác:

```bash
while true; do curl -s http://localhost:9898 > /dev/null; done
```

---

## 7.10 Quan sát pod

```bash
kubectl get pods -w
```

⏳ Sau ~30–60 giây:

✔️ Pod tăng từ:

```
1 → 2 → 3 → ...
```

🎉 **AUTOSCALING HOẠT ĐỘNG**

---

# ✅ CHECKLIST BƯỚC 7

|Hạng mục|Trạng thái|
|---|---|
|Prometheus có http_requests_total|✅|
|rate(...) trả dữ liệu|✅|
|ScaledObject active|✅|
|HPA được tạo|✅|
|Pod scale khi có traffic|✅|

---

# 🔥 TỚI ĐÂY, BẠN ĐÃ ĐẠT MỨC

✅ Autoscaling theo **HTTP traffic thật**  
✅ Monitoring chuẩn production  
✅ Demo cực kỳ thuyết phục hội đồng

---

## 👉 BƯỚC TIẾP THEO (MÌNH ĐỀ XUẤT)

1️⃣ **BƯỚC 8 – Viết k6 load test + demo hoàn chỉnh**  
2️⃣ Chuẩn hoá **Grafana Dashboard** để trình chiếu  
3️⃣ Viết **phần thuyết minh đồ án (mô tả kiến trúc + flow)**

👉 Bạn chọn:

- **“Bước 8”**
    
- hoặc **“Chuẩn bị thuyết minh đồ án”**