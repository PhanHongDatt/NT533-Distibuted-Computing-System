*Thành phần hệ thống bao gồm*
Hệ thống máy ảo triển khai: 3 Máy ảo
+ Hệ điều hành: Ubuntu 24.04 LTS
+ Network: Bridge 
+ Bao gồm: 1 Master và 2 Worker



+ Sau khi tiến hành cài đặt các máy ảo xong, tiến hành setup như sau
	+ Set up chung cho cả 3 máy ảo
```
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl net-tools ufw
sudo ufw allow 22/tcp  # SSH
sudo ufw allow 6443/tcp  # K3s API
sudo ufw allow 10250/tcp  # Kubelet
sudo ufw allow 80/tcp 443/tcp  # Ingress traffic
sudo ufw enable
sudo swapoff -a
sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo modprobe overlay && sudo modprobe br_netfilter
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
sudo reboot
```

**Kiểm tra sau reboot**: sudo ufw status (ports allow), free -h (no swap), ip a (IP bridge OK).
#### Bước 3: Install K3s trên VM1 (Master, disable defaults)
SSH VM1, chạy:
```
curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable traefik --disable servicelb --write-kubeconfig-mode 644" sh -
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
echo 'export KUBECONFIG=/etc/rancher/k3s/k3s.yaml' >> ~/.bashrc && source ~/.bashrc
```
![[Pasted image 20260101152715.png]]
Lấy token: 
```
sudo cat /var/lib/rancher/k3s/server/node-token
```

```bash
hongdat@hongdat-master:~$ sudo cat /var/lib/rancher/k3s/server/node-token
K10028184a171748ed4d2c0f6c854f9ded05ee5b0a8a6d7fb8249cfe312ac03e70a::server:4e2f720346752a88aedec94037829923
```
#### Bước 4: Join Workers (VM2/VM3)

SSH VM2 và VM3, chạy:
```
curl -sfL https://get.k3s.io | K3S_URL=https://<IP_VM1>:6443 K3S_TOKEN=<token> sh -
```
Phiên bản đã được fill IP và Token
```
curl -sfL https://get.k3s.io | K3S_URL=https://192.168.1.248:6443 K3S_TOKEN=4e2f720346752a88aedec94037829923 sh -
```

Worker 1
![[Pasted image 20260101153031.png]]

Worker 2
![[Pasted image 20260101153047.png]]

**Kiểm tra trên VM1**: kubectl get nodes (3 Ready).
![[Pasted image 20260101153108.png]]
#### Bước 5: Install Helm trên VM1

```
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

![[Pasted image 20260101153142.png]]
**Kiểm tra**: helm version (v3.19+).
![[Pasted image 20260101153201.png]]
#### Bước 6: Deploy NGINX Ingress (cho anti-DoS demo)

Add repo: 
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx && helm repo update. Install: helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace --set controller.service.type=NodePort.
```



```
hongdat@hongdat-master:~$ helm install ingress-nginx ingress-nginx/ingress-nginx --namespace ingress-nginx --create-namespace --set controller.service.type=NodePort
NAME: ingress-nginx
LAST DEPLOYED: Thu Jan  1 15:43:51 2026
NAMESPACE: ingress-nginx
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The ingress-nginx controller has been installed.
Get the application URL by running these commands:
  export HTTP_NODE_PORT=$(kubectl get service --namespace ingress-nginx ingress-nginx-controller --output jsonpath="{.spec.ports[0].nodePort}")
  export HTTPS_NODE_PORT=$(kubectl get service --namespace ingress-nginx ingress-nginx-controller --output jsonpath="{.spec.ports[1].nodePort}")
  export NODE_IP="$(kubectl get nodes --output jsonpath="{.items[0].status.addresses[1].address}")"

  echo "Visit http://${NODE_IP}:${HTTP_NODE_PORT} to access your application via HTTP."
  echo "Visit https://${NODE_IP}:${HTTPS_NODE_PORT} to access your application via HTTPS."

An example Ingress that makes use of the controller:
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: example
    namespace: foo
  spec:
    ingressClassName: nginx
    rules:
      - host: www.example.com
        http:
          paths:
            - pathType: Prefix
              backend:
                service:
                  name: exampleService
                  port:
                    number: 80
              path: /
    # This section is only required if TLS is to be enabled for the Ingress
    tls:
      - hosts:
        - www.example.com
        secretName: example-tls

If TLS is enabled for the Ingress, a Secret containing the certificate and key must also be provided:

  apiVersion: v1
  kind: Secret
  metadata:
    name: example-tls
    namespace: foo
  data:
    tls.crt: <base64 encoded cert>
    tls.key: <base64 encoded key>
  type: kubernetes.io/tls
```
- Demo anti-DoS: Tạo Ingress rule với rate limit (sau khi deploy RabbitMQ, route đến UI): 

  nano ingress-rabbitmq.yaml

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rabbitmq-ingress
  namespace: messaging  # Tạo namespace sau
  annotations:
    nginx.ingress.kubernetes.io/limit-rps: "10"  # Giới hạn 10 requests/sec per IP (anti-flood)
    nginx.ingress.kubernetes.io/limit-rpm: "600"  # 600 req/min
    nginx.ingress.kubernetes.io/limit-connections: "5"  # Max 5 concurrent connections/IP
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: rabbitmq-management
            port:
              number: 15672
```
![[Pasted image 20260101154557.png]]
Kiểm tra:
```
 kubectl get svc -n ingress-nginx
 ```
![[Pasted image 20260101154617.png]]
(NodePort OK), **kubectl get pods -n ingress-nginx** (Running).
![[Pasted image 20260101154649.png]]


#### Bước 7: Deploy RabbitMQ HA (replicas=3, clustering)
**Siêu lưu ý là cái đoạn này bug lòi le nên cân nhắc đọc kỹ hướng dẫn tại fix muốn ỉa trong quần luôn á má**
Cài đặt Bitnami
```
# Thêm repo Bitnami
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

Sau đó tiến hành cài rabbit OP, tạo namespace luôn
```
$ helm install rabbitmq-operator bitnami/rabbitmq-cluster-operator \
  --namespace messaging \
  --create-namespace
```
**vì cái gì đó nó không còn miễn phí nữa nên cần tiến hành chuyển từ hub -> aws và bỏ qua cái gì á quên rồi**
```
helm upgrade rabbitmq-operator bitnami/rabbitmq-cluster-operator \
  --namespace messaging \
  --set global.imageRegistry=public.ecr.aws \
  --set global.security.allowInsecureImages=true
```

>[!Solution]
>Vấn đề bạn đang gặp phải là một cơ chế bảo mật mới của Bitnami (áp dụng từ cuối năm 2025). Họ yêu cầu xác thực nếu bạn thay đổi Registry mặc định. Để giải quyết lệnh `ErrImagePull` và tiến hành các bước cải tiến "khủng" cho đồ án, bạn cần thêm một flag xác nhận "bỏ qua kiểm tra bảo mật" cho các image từ AWS ECR.



**Kiểm tra:** Đợi khoảng 1-2 phút, chạy `kubectl get pods -n messaging`. Khi thấy 2 Pod Operator chuyển sang **Running**, bạn mới thực hiện các bước tiếp theo.


Sau đó tiến hành thiết lập phân vùng ổ đĩa để tiến hành lưu trữ
## 1. Tại sao phải thiết lập Storage?

- **Tránh mất dữ liệu:** RabbitMQ là một "Stateful Application" (ứng dụng có trạng thái). Nếu Pod bị khởi động lại hoặc máy ảo gặp sự cố mà không có ổ đĩa gắn ngoài (Persistent Volume), toàn bộ tin nhắn (messages), hàng đợi (queues) và tài khoản người dùng sẽ **biến mất hoàn toàn**.
    
- **Điều kiện để Pod có thể chạy (Running):** Trong file cấu hình `rabbitmq-cluster.yaml`, bạn đã khai báo phần `persistence`. Khi bạn `apply`, RabbitMQ Operator sẽ yêu cầu cụm K8s cấp cho nó 3 "ổ đĩa ảo" (PVC). Nếu bạn không cấu hình StorageClass, K8s sẽ không biết lấy dung lượng ở đâu để cấp, dẫn đến các Pod RabbitMQ sẽ mãi ở trạng thái **`Pending`**.
    
- **Cơ chế của Operator:** RabbitMQ Cluster Operator được thiết kế để tự động hóa mọi thứ, nhưng nó cần một "nguồn cấp ổ đĩa" (Storage Provisioner) để hoạt động.
    

---

## 2. Giải pháp "Local Path" cho 3 máy ảo Ubuntu

Vì bạn đang chạy Lab trên máy ảo (On-premise), bạn không có các ổ đĩa đám mây như AWS EBS hay Google Persistent Disk. Giải pháp tốt nhất là dùng **Local Path Provisioner**.

**Cơ chế hoạt động:** Nó sẽ lấy một thư mục ngay trên ổ cứng của chính máy ảo Ubuntu (ví dụ: `/opt/local-path-provisioner`) và biến nó thành một "ổ đĩa" cho Kubernetes Pod.

---

## 3. Cách thực hiện (Chỉ cần gõ trên máy Master)

Bạn không cần phải chia lại phân vùng ổ cứng (re-partition) bằng tay trên Ubuntu. Bạn chỉ cần chạy 2 lệnh sau trên **Master** để Kubernetes tự động hóa việc này:

1. **Cài đặt trình quản lý ổ đĩa:**
    
    Bash
    
    ```
    kubectl apply -f https://raw.githubusercontent.com/rancher/local-path-provisioner/v0.0.30/deploy/local-path-storage.yaml
    ```
    
2. **Thiết lập nó làm mặc định cho hệ thống:**
    
    Bash
    
    ```
    kubectl patch storageclass local-path -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'
    ```
    

---

## 4. Kiểm tra xem ổ đĩa đã sẵn sàng chưa

Sau khi chạy các lệnh trên và `apply` file `rabbitmq-cluster.yaml`, bạn hãy kiểm tra bằng lệnh:

Bash

```
kubectl get pvc -n messaging
```

- **Nếu cột STATUS hiện `Bound`:** Chúc mừng, 3 Pod RabbitMQ đã nhận được ổ đĩa và sẽ sớm chuyển sang màu xanh (`Running`).
    
- **Nếu hiện `Pending`:** Nghĩa là hệ thống vẫn chưa cấp được đĩa, lúc này cần xem lại tên `storageClassName` trong file YAML có khớp với tên `local-path` hay không.
**Tuy nhiên chỗ này sẽ check ra là no resources gì đó** thì để fix cần tiến hành gắn cụm cluster dô cho nó
Tiến hành nano rabbitmq-cluster.yaml, sau đó paste cái đống dưới dô má:
```
apiVersion: rabbitmq.com/v1beta1
kind: RabbitmqCluster
metadata:
  name: rabbit
  namespace: messaging
spec:
  replicas: 3 # Chạy 3 Pod để đảm bảo HA
  image: "rabbitmq:4.0-management"
  persistence:
    storageClassName: local-path
    storage: 2Gi
  rabbitmq:
    additionalConfig: |
      cluster_formation.peer_discovery_backend = k8s
      cluster_formation.k8s.host = kubernetes.default
      cluster_formation.k8s.address_type = hostname
```
Sau khi tạo xong nhớ áp dụng cấu hình nha má nội:
```
kubectl apply -f rabbitmq-cluster.yaml
```


---

```
nano rabbitmq-cluster.yaml
```
	

```
apiVersion: rabbitmq.com/v1beta1
kind: RabbitmqCluster
metadata:
  name: rabbitmq
  namespace: messaging
spec:
  replicas: 3  # HA
  rabbitmq:
    additionalPlugins:
    - rabbitmq_management
    - rabbitmq_peer_discovery_k8s
    - rabbitmq_prometheus
  service:
    type: NodePort  # Expose UI nếu cần, hoặc ClusterIP + Ingress
```

![[Pasted image 20260101155410.png]]
Apply: kubectl apply -f rabbitmq-cluster.yaml.
![[Pasted image 20260101155436.png]]
- Service cho management: nano rabbitmq-service.yaml (cho Ingress route):
    
```
    nano rabbitmq-service.yaml
```

    
```
    apiVersion: v1
    kind: Service
    metadata:
      name: rabbitmq-management
      namespace: messaging
    spec:
      selector:
        app.kubernetes.io/name: rabbitmq
      ports:
      - name: http
        port: 15672
        targetPort: 15672
      - name: metrics
        port: 15692
        targetPort: 15692
      type: ClusterIP
```

![[Pasted image 20260101155623.png]]
    Apply: kubectl apply -f rabbitmq-service.yaml.
![[Pasted image 20260101155655.png]]

- Apply Ingress cho demo anti-DoS: kubectl apply -f ingress-rabbitmq.yaml.
_Lưu ý_: Chỗ này cần phải xóa bỏ cấu hình xóa Validation Webhook (Khuyên dùng cho đồ án/Lab)
Trong môi trường Lab, việc kiểm tra cú pháp này không quá quan trọng. Xóa nó đi sẽ giúp bạn `apply` file YAML mượt mà mà không bị chặn đường.
```
kubectl delete -A ValidatingWebhookConfiguration ingress-nginx-admission
```

![[Pasted image 20260101155840.png]]
**Kiểm tra**: kubectl get rabbitmqcluster -n messaging (Ready), kubectl get pods -n messaging (3 Running), port-forward 15672 để UI (guest/guest, tạo queues).
![[Pasted image 20260101180319.png]]

```
hongdat@hongdat-master:~$ nano rabbit-ingress.yaml
hongdat@hongdat-master:~$ kubectl apply -f rabbit-ingress.yaml

# Lấy Password để đăng nhập
kubectl get secret rabbit-default-user -n messaging -o jsonpath="{.data.password}" | base64 --decode ; echo
Warning: annotation "kubernetes.io/ingress.class" is deprecated, please use 'spec.ingressClassName' instead
ingress.networking.k8s.io/rabbit-ingress created
LijwWo1AzVA9AQCP1SB2Y0AGChfuW3J9
```

Mật khẩu của Grafana: admin
```
sQoBi5CYgnqMGf4KlDGEu59NFBZ7bXQEYyXIEbzZ
```

Mật khẩu Rabbit MQ
```
default_user_G3rhE-gfYmoi2U1959R
LijwWo1AzVA9AQCP1SB2Y0AGChfuW3J9
```
#### Bước 8: Deploy Prometheus (cho multi-trigger CPU/RAM)

Add repo: helm repo add prometheus-community https://prometheus-community.github.io/helm-charts && helm repo update. Install: helm install prometheus prometheus-community/prometheus --namespace monitoring --create-namespace --set server.service.type=NodePort.

- ServiceMonitor cho RabbitMQ: Nano servicemonitor.yaml (như doc):
    
    text
    
    ```
    apiVersion: monitoring.coreos.com/v1
    kind: ServiceMonitor
    metadata:
      name: rabbitmq
      namespace: messaging
    spec:
      selector:
        matchLabels:
          app.kubernetes.io/name: rabbitmq
      endpoints:
      - port: metrics
        interval: 15s
    ```
    
    Apply: kubectl apply -f servicemonitor.yaml.
- Để scale CPU/RAM, Prometheus scrape kubelet metrics (default OK).

**Kiểm tra**: kubectl get pods -n monitoring (Running), port-forward 9090 UI, query "container_cpu_usage_seconds_total" cho CPU.

#### Bước 9: Deploy KEDA với multi-trigger

Add repo: helm repo add kedacore https://kedacore.github.io/charts && helm repo update. Install: helm install keda kedacore/keda --namespace keda --create-namespace.

- ScaledObject multi-trigger: Nano scaledobject.yaml (cho mỗi consumer-q1/q2/q3, lặp với queueName khác):
    
    text
    
    ```
    apiVersion: keda.sh/v1alpha1
    kind: ScaledObject
    metadata:
      name: consumer-q1-scaler
      namespace: messaging
    spec:
      scaleTargetRef:
        name: consumer-q1  # Deployment consumer
      minReplicaCount: 0
      maxReplicaCount: 20
      cooldownPeriod: 150
      triggers:
      - type: rabbitmq
        metadata:
          protocol: http
          queueName: queue1
          value: "20"  # Messages per pod
          host: http://admin:admin@rabbitmq-management.messaging.svc.cluster.local:15672
      - type: prometheus
        metadata:
          serverAddress: http://prometheus-server.monitoring.svc.cluster.local:9090
          metricName: cpu_usage
          threshold: "50"  # CPU % averageValue
          query: avg(rate(container_cpu_usage_seconds_total{namespace="messaging",pod=~"consumer-q1-.*"}[5m])) * 100
      - type: prometheus
        metadata:
          serverAddress: http://prometheus-server.monitoring.svc.cluster.local:9090
          metricName: memory_usage
          threshold: "200"  # MiB averageValue
          query: avg(container_memory_working_set_bytes{namespace="messaging",pod=~"consumer-q1-.*"} / 1024 / 1024)
    ```
    
    Apply: kubectl apply -f scaledobject.yaml (lặp cho q2/q3).

**Kiểm tra**: kubectl get scaledobject -n messaging (Ready), kubectl get hpa -n messaging (HPA multi).

#### Bước 10: Deploy Consumers và Producer

- Dockerfile/consumer.py như doc, build/push image dinhtri69/rabbitmq-consumer-test:latest.
- Deployment consumers: Nano consumer-deploy.yaml (như doc, replicas=0), apply.
- Producer: Chạy local script send_messages.py với port-forward đến RabbitMQ 5672.

**Kiểm tra**: kubectl get pods -n messaging -w (scale khi queue/CPU/RAM vượt threshold).

#### Bước 11: Demo anti-DoS/DDoS

- Access RabbitMQ UI qua Ingress NodePort (e.g., http://<IP_VM1>:<node-port>).
- Demo flood: Từ host, dùng ab: ab -n 1000 -c 100 http://<IP_VM1>:<node-port>/ (thấy 429 Too Many Requests nếu vượt rate).
- Adjust limit in annotations nếu cần, rollout restart Ingress.