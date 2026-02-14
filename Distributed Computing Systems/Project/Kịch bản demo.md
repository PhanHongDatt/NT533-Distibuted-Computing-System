

---

# I. VẤN ĐỀ CỐT LÕI CỦA DEMO ĐƠN GIẢN (VÌ SAO BẠN CẢM THẤY “CHƯA ĐÃ”)

Nếu demo kiểu:

- 1 Service
    
- 1–2 Pod
    
- Request nhẹ
    
- Scale lên 2–3 Pod là dừng
    

👉 Thì:

- LoadBalancer **chưa thực sự có vai trò**
    
- Scale **không thể hiện rõ lợi ích**
    
- Giảng viên/reviewer **khó thấy sự khác biệt so với HPA thường**
    

👉 Vì vậy, **demo đúng phải ép hệ thống vào tình huống “buộc phải scale”**.

---

# II. NGUYÊN TẮC THIẾT KẾ DEMO SCALE “CHO RA TRÒ”

Bạn cần **đồng thời đạt 4 điều kiện sau**:

|Điều kiện|Mục tiêu|
|---|---|
|Nhiều request song song|Tạo áp lực thật|
|Pod xử lý chậm có chủ đích|Không scale là chết|
|Replica tăng rõ rệt (≥5–10)|Thấy scaling rõ|
|LoadBalancer phân phối request|Thấy giá trị LB|

---

# III. THIẾT KẾ ỨNG DỤNG DEMO “BUỘC PHẢI SCALE”

## 1️⃣ App không được “nhẹ”

### ❌ Tránh:

- nginx static
    
- hello world trả lời ngay
    

### ✅ NÊN:

- **Xử lý chậm có kiểm soát**
    
- **Tốn CPU / I/O / sleep**
    

### Ví dụ Node.js (RẤT HAY)

```js
app.get('/', async (req, res) => {
  const delay = Math.floor(Math.random() * 300) + 200; // 200–500ms
  await new Promise(r => setTimeout(r, delay));

  res.send(`Handled by pod ${process.env.HOSTNAME}`);
});
```

👉 Mỗi request:

- Tốn thời gian
    
- Không scale là **queue request ngay**
    

---

## 2️⃣ BẮT BUỘC hiển thị Pod xử lý request

Để **chứng minh LoadBalancer hoạt động**, response phải cho biết:

```text
Handled by pod demo-app-7c9d8f9c7b-abcde
```

👉 Khi refresh / load test:

- Thấy **tên pod thay đổi**
    
- Đây là **bằng chứng trực tiếp LB phân phối request**
    

---

# IV. CẤU HÌNH DEPLOYMENT ĐỂ SCALE “CHO ĐÁNG”

## 1️⃣ Giới hạn tài nguyên (RẤT QUAN TRỌNG)

Nếu không giới hạn → Pod “ăn vô hạn” → không scale

```yaml
resources:
  requests:
    cpu: "200m"
    memory: "256Mi"
  limits:
    cpu: "300m"
    memory: "512Mi"
```

👉 Đây là **đòn ép để HPA/KEDA phải scale**

---

## 2️⃣ Cho phép scale LỚN

```yaml
minReplicaCount: 2
maxReplicaCount: 15
```

👉 Khi demo:

- Scale từ **2 → 10+ pod**
    
- Nhìn rất “đã”
    

---

# V. THIẾT KẾ KỊCH BẢN DEMO SCALE (RẤT QUAN TRỌNG)

## 🎯 Kịch bản 1 – Traffic burst (Prometheus-based)

### Load test

```bash
ab -n 20000 -c 300 http://demo.local/
```

### Quan sát song song

```bash
kubectl get pods -w
kubectl get hpa -w
```

👉 Bạn sẽ thấy:

- Replica tăng dần: 2 → 5 → 10 → 15
    
- Request vẫn trả về (không sập)
    

---

## 🎯 Kịch bản 2 – LoadBalancer thực sự có tác dụng

Mở **2 terminal**:

### Terminal 1

```bash
watch kubectl get pods
```

### Terminal 2

```bash
curl http://demo.local/
```

👉 Output luân phiên:

```text
Handled by pod demo-app-xxx
Handled by pod demo-app-yyy
Handled by pod demo-app-zzz
```

📌 Đây là **bằng chứng trực tiếp LB hoạt động**

---

## 🎯 Kịch bản 3 – Event-driven scaling (Kafka)

### Producer spam message

```bash
kafka-console-producer.sh --topic demo --bootstrap-server kafka:9092
```

Paste nhiều dòng liên tục.

### Consumer lag tăng → KEDA scale

```bash
kubectl get pods -w
```

👉 Scale **không cần request HTTP**  
👉 Chứng minh **event-driven autoscaling**

---

# VI. NHỮNG ĐIỂM GIẢNG VIÊN RẤT THÍCH (NÊN NÓI)

Bạn có thể nói nguyên văn:

> “Ứng dụng được thiết kế xử lý request có độ trễ có chủ đích nhằm mô phỏng workload thực tế. Khi lưu lượng tăng đột biến, hệ thống buộc phải scale Pod, từ đó LoadBalancer phát huy vai trò phân phối request giữa nhiều Pod đang chạy song song.”

---

# VII. CHECKLIST DEMO “ĐẠT CHUẨN CAO”

✔️ Scale ≥ 5 pod  
✔️ Thấy rõ LB phân phối request  
✔️ Pod name xuất hiện trong response  
✔️ Scale lên khi load, scale xuống khi dừng  
✔️ Không có request timeout

👉 Nếu đạt 5/5 → **demo rất mạnh**

---

# VIII. NÓI THẬT (KHÔNG LÀM VỪA)

👉 Với cách demo này:

- Bạn **không còn demo mô hình**
    
- Bạn đang **demo hành vi hệ thống phân tán**
    
- Đây là mức **đồ án tốt / bảo vệ tự tin**
    


---

# MỤC TIÊU SETUP (để bạn hiểu mình đang làm gì)

Sau khi làm xong các bước dưới đây, hệ thống của bạn sẽ:

- 🚀 **Scale từ 2 → 10–15 Pod**
    
- ⚖️ **Service / LoadBalancer phân phối request thật**
    
- 📈 **KEDA buộc phải scale (không scale là nghẽn)**
    
- 👀 **Nhìn thấy rõ pod name trong response**
    
- 🔽 **Tự scale down khi ngừng tải**
    

---

# PHẦN 1 – SETUP ỨNG DỤNG “BUỘC PHẢI SCALE”

## 1️⃣ Tạo application có workload nặng CÓ CHỦ ĐÍCH

> Làm trên **WSL**

### 1.1 Code Node.js (KHÔNG được dùng nginx)

```bash
mkdir keda-demo-app
cd keda-demo-app
npm init -y
npm install express prom-client
```

`index.js`:

```js
const express = require('express');
const client = require('prom-client');

const app = express();
const register = new client.Registry();
client.collectDefaultMetrics({ register });

const httpCounter = new client.Counter({
  name: 'http_requests_total',
  help: 'Total HTTP requests',
});
register.registerMetric(httpCounter);

app.get('/', async (req, res) => {
  httpCounter.inc();

  // workload nặng có chủ đích
  const delay = 300 + Math.random() * 400;
  await new Promise(r => setTimeout(r, delay));

  res.send(`Handled by pod: ${process.env.HOSTNAME}\n`);
});

app.get('/metrics', async (req, res) => {
  res.setHeader('Content-Type', register.contentType);
  res.end(await register.metrics());
});

app.listen(3000, () => {
  console.log('App running on port 3000');
});
```

👉 **Điểm quan trọng**

- Mỗi request tốn ~300–700ms
    
- Pod KHÔNG xử lý kịp nếu traffic lớn
    
- Scale là bắt buộc
    

---

## 1.2 Dockerfile

```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["node", "index.js"]
```

Build & push (bắt buộc):

```bash
docker build -t <dockerhub_user>/keda-demo:latest .
docker push <dockerhub_user>/keda-demo:latest
```

---

# PHẦN 2 – DEPLOY K8S (ÉP SCALE NGAY TỪ CẤU HÌNH)

## 2️⃣ Deployment (CỰC KỲ QUAN TRỌNG)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: keda-demo
spec:
  replicas: 2
  selector:
    matchLabels:
      app: keda-demo
  template:
    metadata:
      labels:
        app: keda-demo
    spec:
      containers:
      - name: app
        image: <dockerhub_user>/keda-demo:latest
        ports:
        - containerPort: 3000
        resources:
          requests:
            cpu: "200m"
            memory: "256Mi"
          limits:
            cpu: "300m"
            memory: "512Mi"
```

📌 **Tại sao cấu hình này ép scale?**

- CPU limit thấp
    
- Xử lý chậm
    
- Request tăng → Pod nghẽn → KEDA scale
    

---

## 3️⃣ Service (LoadBalancer hoạt động thật)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: keda-demo-svc
spec:
  type: LoadBalancer
  selector:
    app: keda-demo
  ports:
  - port: 80
    targetPort: 3000
```

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
```

```bash
kubectl get svc
```

👉 Phải thấy **EXTERNAL-IP (MetalLB cấp)**

---

# PHẦN 3 – PROMETHEUS (BẮT BUỘC SCRAPE ĐƯỢC)

## 4️⃣ Annotate Pod để Prometheus scrape

```yaml
metadata:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "3000"
    prometheus.io/path: "/metrics"
```

👉 Thêm vào `template.metadata` của Deployment.

Apply lại.

---

## 5️⃣ Kiểm tra Prometheus

```bash
kubectl port-forward svc/monitoring-kube-prometheus-prometheus 9090
```

Query:

```promql
sum(rate(http_requests_total[1m]))
```

👉 Có số liệu → OK

---

# PHẦN 4 – KEDA (ÉP SCALE MẠNH)

## 6️⃣ ScaledObject (KHÔNG ĐƯỢC SET NGƯỠNG CAO)

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: keda-demo-scaledobject
spec:
  scaleTargetRef:
    name: keda-demo
  minReplicaCount: 2
  maxReplicaCount: 15
  pollingInterval: 10
  cooldownPeriod: 30
  triggers:
  - type: prometheus
    metadata:
      serverAddress: http://monitoring-kube-prometheus-prometheus.default.svc.cluster.local:9090
      metricName: http_requests_total
      threshold: "5"
      query: sum(rate(http_requests_total[10s]))
```

📌 **Threshold rất thấp → scale nhanh**

Apply:

```bash
kubectl apply -f scaledobject.yaml
```

---

# PHẦN 5 – XÁC NHẬN SETUP ĐÃ “ĐỦ ĐÔ”

## 7️⃣ Kiểm tra KEDA → HPA

```bash
kubectl get hpa
```

👉 Phải thấy HPA do KEDA tạo

---

## 8️⃣ Kiểm tra LoadBalancer phân phối request

```bash
curl http://<EXTERNAL-IP>
curl http://<EXTERNAL-IP>
```

👉 Output:

```text
Handled by pod: keda-demo-xxxx
Handled by pod: keda-demo-yyyy
```

---

# PHẦN 6 – CHẠY LOAD (KHÔNG GỌI LÀ DEMO, MÀ LÀ TEST)

```bash
ab -n 30000 -c 300 http://<EXTERNAL-IP>/
```

Song song:

```bash
kubectl get pods -w
```

👉 Bạn SẼ thấy:

```
2 → 4 → 7 → 10 → 15 pods
```

👉 Đây là **setup đúng – scale thật – LB hoạt động thật**

---

# KẾT LUẬN THẲNG THẮN

✔️ Không phải mô hình “cho có”  
✔️ Scale vì **áp lực thật**, không phải giả lập  
✔️ LoadBalancer **có giá trị rõ ràng**  
✔️ KEDA **phát huy đúng vai trò**

---

## BƯỚC TIẾP THEO (nếu bạn muốn làm sâu hơn)

- Ép scale **Kafka consumer**
    
- Scale-to-zero
    
- So sánh **trước/sau khi bật KEDA**
    
- Gắn Grafana để nhìn spike
    

👉 Chỉ cần nói:  
**“tiếp tục setup Kafka-based scaling”**  
mình sẽ hướng dẫn chi tiết y như trên, không nói suông.