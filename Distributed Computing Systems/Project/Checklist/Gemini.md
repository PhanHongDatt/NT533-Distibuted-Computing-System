Chào con đã tổng hợp, biên tập và "chuốt" lại toàn bộ file hướng dẫn của má. Con đã bổ sung những đoạn "xương máu" mà má con mình vừa fix xong chiều nay: từ chuyện **NodePort 32485**, lỗi **503/429**, cho đến cái **Script Python** dội bom và **Script Bash** đốt CPU.

Má chỉ cần Copy đoạn này vào file tài liệu (Markdown) của má là bao đẹp và chuyên nghiệp luôn!

---

# HƯỚNG DẪN TRIỂN KHAI HỆ THỐNG K8S HA, AUTO-SCALING (KEDA) & CHỐNG DOS

## I. THIẾT LẬP HẠ TẦNG (VIRTUAL MACHINES)

- **Hệ điều hành:** Ubuntu 24.04 LTS (3 VMs: 1 Master, 2 Workers).
    
- **Network:** Bridge Mode (Cùng dải IP: `192.168.1.x`).
    
- **Cấu hình chung:** Disable Swap, mở Ports UFW.
    

### Bước 1: Setup chung (Thực hiện trên cả 3 VM)

Bash

```
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl net-tools ufw
# Mở các cổng cần thiết
sudo ufw allow 22/tcp && sudo ufw allow 6443/tcp && sudo ufw allow 10250/tcp
sudo ufw allow 80/tcp && sudo ufw allow 443/tcp
sudo ufw allow 30000:32767/tcp # Cho NodePort
sudo ufw --force enable
# Tắt Swap hoàn toàn
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
# Cấu hình Kernel Modules
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
sudo reboot
```

### Bước 2: Cài đặt K3s Cluster

**Tại Master (VM1):**

Bash

```
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable traefik --disable servicelb --write-kubeconfig-mode 644" sh -
# Lấy Token để Worker Join
sudo cat /var/lib/rancher/k3s/server/node-token
```

**Tại Workers (VM2 & VM3):**

Bash

```
# Thay IP_MASTER và TOKEN của má vào đây
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.1.248:6443 K3S_TOKEN=<TOKEN_CỦA_MÁ> sh -
```

---

## II. CÀI ĐẶT CÁC THÀNH PHẦN BỔ TRỢ (HELM & STORAGE)

### Bước 3: Helm & Local Storage

Để RabbitMQ chạy được (Stateful), ta cần trình quản lý ổ đĩa Local.

Bash

```
# Cài Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Cài Local Path Provisioner (Storage Class)
kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.30/deploy/local-path-storage.yaml
# Set làm mặc định
kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
```

---

## III. TRIỂN KHAI RABBITMQ HA & INGRESS (ANTI-DOS)

### Bước 4: RabbitMQ Cluster Operator

Do cơ chế bảo mật mới của Bitnami, cần force registry từ AWS ECR.

Bash

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install rabbitmq-operator bitnami/rabbitmq-cluster-operator \
  --namespace messaging --create-namespace \
  --set global.imageRegistry=public.ecr.aws \
  --set global.security.allowInsecureImages=true
```

### Bước 5: Deploy RabbitMQ Cluster & Ingress

**File: `rabbitmq-cluster.yaml`**

YAML

```
apiVersion: rabbitmq.com/v1beta1
kind: RabbitmqCluster
metadata:
  name: rabbit
  namespace: messaging
spec:
  replicas: 3
  image: "rabbitmq:4.0-management"
  persistence:
    storageClassName: local-path
    storage: 2Gi
```

**File: `rabbit-ingress.yaml` (Cấu hình chặn DoS)**

YAML

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rabbit-ingress
  namespace: messaging
  annotations:
    nginx.ingress.kubernetes.io/limit-rps: "20"
    nginx.ingress.kubernetes.io/limit-connections: "20"
    nginx.ingress.kubernetes.io/limit-req-status-code: "429" # Trả về 429 khi bị chặn
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: rabbit
            port:
              number: 15672
```

Lưu ý: Xóa Validation Webhook nếu bị lỗi khi apply Ingress:

kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission

---

## IV. GIÁM SÁT (PROMETHEUS) & AUTO-SCALING (KEDA)

### Bước 6: Deploy Prometheus & KEDA

Bash

```
# Prometheus
helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace --set server.service.type=NodePort

# KEDA
helm install keda kedacore/keda --namespace keda --create-namespace
```

### Bước 7: Cấu hình Multi-Trigger ScaledObject

Để demo "nhạy", ta đặt ngưỡng CPU và RAM thấp.

File: keda-scaler.yaml

YAML

```
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: multi-trigger-scaler
  namespace: default
spec:
  scaleTargetRef:
    name: worker-app
  minReplicaCount: 1
  maxReplicaCount: 10
  triggers:
  - type: rabbitmq
    metadata:
      queueName: ha-demo
      value: "20"
      host: amqp://user:pass@rabbit.messaging.svc.cluster.local:5672/%2f
  - type: prometheus
    metadata:
      serverAddress: http://prometheus-server.monitoring.svc.cluster.local:9090
      threshold: '0.1' # 100m CPU
      query: sum(rate(container_cpu_usage_seconds_total{namespace="default", container="worker"}[1m]))
  - type: prometheus
    metadata:
      serverAddress: http://prometheus-server.monitoring.svc.cluster.local:9090
      threshold: '52428800' # 50MB RAM
      query: sum(container_memory_usage_bytes{namespace="default", container="worker"})
```

---

## V. KỊCH BẢN DEMO (SCRIPTS)

### 1. Script Stress CPU/RAM (Chạy trên Master)

**File: `stress-test.sh`**

Bash

```
#!/bin/bash
POD_NAME=$(kubectl get pods -l app=worker-app -o jsonpath="{.items[0].metadata.name}")
echo "1. Đốt CPU | 2. Bơm RAM | 3. Dọn dẹp"
read -p "Chọn: " choice
case $choice in
  1) kubectl exec -it $POD_NAME -- /bin/sh -c "for i in 1 2 3 4; do while :; do :; done & done; wait" ;;
  2) kubectl exec -it $POD_NAME -- /bin/sh -c "dd if=/dev/zero of=/dev/shm/hulk_ram bs=1M count=500" ;;
  3) kubectl exec -it $POD_NAME -- killall sh; kubectl exec -it $POD_NAME -- rm -f /dev/shm/hulk_ram ;;
esac
```

### 2. Script Tấn công DoS (Chạy trên Windows - Python)

**File: `dos_test.py`**

Python

```
import threading, requests
url = "http://192.168.1.248:32485" # Cổng NodePort thực tế
def attack():
    while True:
        try:
            r = requests.get(url)
            print(f"Status: {r.status_code}") # Hy vọng thấy 429 hoặc 503 (Limiting)
        except: pass

for i in range(20): # 20 luồng dội bom
    threading.Thread(target=attack, daemon=True).start()
import time; time.sleep(60)
```

---

**Má thấy sao?** Con đã tích hợp cả cái cổng **32485** vào script Python luôn rồi đó. Má cứ theo file này mà triển khai, chỗ nào vướng lại "hú" con nhé!

