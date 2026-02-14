# CHECKLIST ĐỒ ÁN: KEDA AUTO SCALING ON K3S

## PHẦN 1: CHUẨN BỊ MÔI TRƯỜNG (HOST & WSL2)

- [x] 1.1 Cài đặt WSL2 Ubuntu 22.04 trên Windows 11
    
- [x] 1.2 Cài đặt Docker Desktop và kích hoạt WSL2 Backend
![[Pasted image 20251218230820.png]]
    
- [x] 1.3 Cài đặt kubectl trên WSL2

    
- [x] 1.4 Cài đặt Helm trên WSL2
![[Pasted image 20251218230845.png]]
    
- [x] 1.5 Kiểm tra kết nối Docker giữa WSL2 và Windows (docker version)
    

## PHẦN 2: THIẾT LẬP CỤM K3S (VMWARE)

- [x] 2.1 Tạo VM Ubuntu Server (2 vCPU, 4GB RAM, Bridged Network)
    
- [x] 2.2 Cài đặt K3s bằng script (curl -sfL [https://get.k3s.io](https://get.k3s.io) | sh -)
    
- [x] 2.3 Lấy nội dung file k3s.yaml từ VM về máy Host
    
- [x] 2.4 Cấu hình file ~/.kube/config trên WSL2 (Sửa IP thành IP VM)
    
- [x] 2.5 Kiểm tra kết nối từ WSL2 tới Cluster (kubectl get nodes)
    

## PHẦN 3: TRIỂN KHAI NETWORKING (METALLB)

- [x] 3.1 Cài đặt MetalLB Manifest (Native mode) // Load balencer
- [ ] 
![[Pasted image 20251219000356.png]]

![[Pasted image 20251219000843.png]]
    
- [ ] 3.2 Xác định dải IP trống trong lớp mạng LAN của Router
    
- [ ] 3.3 Tạo file cấu hình IPAddressPool và L2Advertisement
    
- [ ] 3.4 Apply cấu hình và kiểm tra log của metallb-controller
    

## PHẦN 4: XÂY DỰNG ỨNG DỤNG DEMO (NODE.JS)

- [ ] 4.1 Viết source code Node.js (Express + Prometheus Client)
    
- [ ] 4.2 Viết Dockerfile tối ưu cho ứng dụng
    
- [ ] 4.3 Build image và Push lên Docker Hub
    
- [ ] 4.4 Viết file Deployment (Giới hạn CPU/RAM: resources.requests/limits)
    
- [ ] 4.5 Viết file Service (type: LoadBalancer)
    
- [ ] 4.6 Triển khai lên K3s và kiểm tra External-IP của Service
    

## PHẦN 5: MONITORING (PROMETHEUS STACK)

- [ ] 5.1 Thêm repo và cài đặt kube-prometheus-stack qua Helm
    
- [ ] 5.2 Kiểm tra các Pod của Prometheus và Grafana đã chạy Ready
    
- [ ] 5.3 Cấu hình ServiceMonitor để Prometheus nhận diện metric từ App Demo
    
- [ ] 5.4 Port-forward thử nghiệm giao diện Prometheus (9090)
    

## PHẦN 6: AUTO SCALING (KEDA)

- [ ] 6.1 Cài đặt KEDA thông qua Helm Chart
    
- [ ] 6.2 Kiểm tra namespace keda và các CRD (Custom Resource Definitions)
    
- [ ] 6.3 Viết file ScaledObject (Triggers: prometheus, Threshold: tùy chọn)
    
- [ ] 6.4 Triển khai ScaledObject và kiểm tra trạng thái HPA (kubectl get hpa)
    

## PHẦN 7: THỰC NGHIỆM & KIỂM CHỨNG (DATA COLLECT)

- [ ] 7.1 Chạy script Load Test (Gửi request liên tục bằng curl hoặc k6)
    
- [ ] 7.2 Quan sát quá trình scale-up của Pod (kubectl get pods -w)
    
- [ ] 7.3 Chụp ảnh màn hình biểu đồ Metric trong Prometheus/Grafana
    
- [ ] 7.4 Dừng tải và quan sát quá trình scale-down (Cooldown period)
    

## PHẦN 8: HOÀN THIỆN BÁO CÁO

- [ ] 8.1 Viết Chương 1: Giới thiệu đề tài và Công nghệ lựa chọn
    
- [ ] 8.2 Viết Chương 2: Cơ sở lý thuyết (HPA vs KEDA, Prometheus hoạt động ra sao)
    
- [ ] 8.3 Viết Chương 3: Thiết kế kiến trúc hệ thống (Sơ đồ kết nối)
    
- [ ] 8.4 Viết Chương 4: Triển khai chi tiết (Các bước cài đặt)
    
- [ ] 8.5 Viết Chương 5: Đánh giá kết quả thực nghiệm
    
- [ ] 8.6 Tổng kết và Tài liệu tham khảo
