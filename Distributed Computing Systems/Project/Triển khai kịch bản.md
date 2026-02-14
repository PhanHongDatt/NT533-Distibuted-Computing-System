Tuyệt 👍  
Ta tiến hành **BƯỚC 8 – LOAD TEST BẰNG k6 & DEMO AUTOSCALING HOÀN CHỈNH**  
Đây là **màn trình diễn “ăn điểm” nhất khi bảo vệ đồ án**.

Mình sẽ làm theo đúng trình tự:

1. Xác định **URL service Podinfo**
    
2. Viết **k6 script chuẩn RAMP-UP / HOLD / RAMP-DOWN**
    
3. Chạy test
    
4. Quan sát **Pods + Grafana**
    
5. Giải thích demo như một Senior DevOps
    

---

# 🟦 BƯỚC 8 – LOAD TEST & DEMO AUTOSCALING

## 🎯 Mục tiêu

- Tạo **traffic HTTP thật**
    
- Thấy **Pods scale 1 → N**
    
- Thấy **Pods scale về 1**
    
- Đồng bộ với **Grafana dashboard**
    

---

# PHẦN A – XÁC ĐỊNH URL PODINFO (RẤT QUAN TRỌNG)

## 8.1 Kiểm tra Service type

```bash
kubectl get svc my-web-podinfo
```

### Trường hợp thường gặp:

```
TYPE: NodePort
PORT: 9898:3xxxx/TCP
```

👉 Lấy:

- **NODE-IP**
    

```bash
ip a | grep inet
```

(hoặc IP VM)

- **NODEPORT** (3xxxx)
    

Ví dụ:

```
http://192.168.1.100:32000
```

📌 **Ghi lại URL này**

---

# PHẦN B – VIẾT KỊCH BẢN k6 (CHUẨN DEMO)

## 8.2 Tạo file `load_test.js`

```bash
vim load_test.js
```

Dán nội dung sau:

```javascript
import http from 'k6/http';
import { sleep } from 'k6';

export let options = {
  stages: [
    { duration: '30s', target: 50 }, // Tăng dần lên 50 users
    { duration: '1m', target: 50 },  // Giữ tải để scale out
    { duration: '30s', target: 0 },  // Giảm về 0 để scale in
  ],
};

export default function () {
  http.get('http://NODE-IP:NODEPORT');
  sleep(1);
}
```

🔴 **Bắt buộc sửa**

```js
http://NODE-IP:NODEPORT
```

→ URL Podinfo của bạn

---

## 🧠 Giải thích k6 script (để bảo vệ đồ án)

- **VUs (Virtual Users)** = người dùng giả lập
    
- `sleep(1)` → mỗi user ~1 request/giây
    
- 50 VUs ≈ **50 RPS**
    
- Vượt threshold 10 → **KEDA scale**
    

---

# PHẦN C – CHUẨN BỊ DEMO (MỞ 3 CỬA SỔ)

## Cửa sổ 1 – Theo dõi Pod

```bash
kubectl get pods -w
```

---

## Cửa sổ 2 – Grafana

- Dashboard:
    
    - Kubernetes / Pod
        
    - HTTP Requests
        
- Quan sát:
    
    - RPS
        
    - Replica count
        

---

## Cửa sổ 3 – Chạy k6

```bash
k6 run load_test.js
```

---

# PHẦN D – QUAN SÁT HIỆN TƯỢNG (ĐÚNG CHUẨN)

### ⏱ Timeline kỳ vọng

|Thời điểm|Hiện tượng|
|---|---|
|0–10s|RPS tăng trên Grafana|
|15–30s|Pod tăng từ 1 → 2|
|30–60s|Pod tăng dần → 5 → 8|
|Sau 2p|k6 giảm tải|
|+30s|Pod giảm dần → 1|

🎉 **Demo thành công**

---

# PHẦN E – CÁCH GIẢI THÍCH KHI BẢO VỆ (ĂN ĐIỂM)

Bạn nói:

> “Ở trạng thái bình thường, hệ thống chỉ chạy 1 Pod để tiết kiệm tài nguyên.  
> Khi lượng truy cập tăng đột biến, Prometheus ghi nhận số request/giây tăng cao.  
> KEDA liên tục truy vấn Prometheus, và khi vượt ngưỡng 10 RPS, nó kích hoạt HPA để scale Podinfo.  
> Khi tải giảm, hệ thống tự động thu hồi Pod, đảm bảo tối ưu chi phí.”

---

# ✅ CHECKLIST BƯỚC 8

|Hạng mục|Trạng thái|
|---|---|
|k6 chạy không lỗi|✅|
|RPS tăng trên Grafana|✅|
|Pod scale out|✅|
|Pod scale in|✅|

---

# 🏁 CHÚC MỪNG – BẠN ĐÃ HOÀN THÀNH HỆ THỐNG

🎯 **Autoscaling theo HTTP traffic thật**  
🎯 **Monitoring chuẩn production**  
🎯 **Demo rất mạnh khi bảo vệ đồ án**

---

## 👉 BẠN MUỐN MÌNH LÀM GÌ TIẾP?

1️⃣ Viết **phần báo cáo đồ án** (mô tả kiến trúc, sơ đồ, giải thích ScaledObject)  
2️⃣ Tối ưu **Grafana dashboard cho đẹp + rõ**  
3️⃣ Chuẩn bị **kịch bản Q&A** (thầy hỏi gì – trả lời sao)

👉 Chỉ cần nói: **“Làm tiếp mục X”**