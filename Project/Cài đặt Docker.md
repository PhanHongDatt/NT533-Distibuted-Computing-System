

---

# 🟦 BƯỚC 1 – CÀI ĐẶT DOCKER (CHUẨN, SẠCH, ỔN ĐỊNH)

## 🎯 Mục tiêu của bước này

Sau bước này, bạn sẽ:

- Chạy được container
    
- Không cần `sudo` khi dùng Docker
    
- Sẵn sàng cho Kubernetes
    

---

## 1.1 Kiểm tra hệ điều hành

```bash
lsb_release -a
```

✔️ Phải là **Ubuntu 20.04 hoặc 22.04**  
❌ Nếu khác → báo mình trước khi tiếp tục

---

## 1.2 Gỡ Docker cũ (nếu có – cho sạch)

```bash
sudo apt remove -y docker docker-engine docker.io containerd runc
```

> Nếu báo “package not found” → **bình thường**, cứ tiếp tục

---

## 1.3 Cài Docker bằng script chính thức (khuyến nghị)

```bash
curl -fsSL https://get.docker.com | sudo sh
```

⏳ Chờ khoảng 1 phút

---

## 1.4 Kiểm tra Docker đã chạy chưa

```bash
sudo systemctl status docker
```

✔️ Thấy `active (running)`  
❌ Nếu không → gửi lỗi mình xử lý ngay

---

## 1.5 Cho phép user thường dùng Docker (RẤT QUAN TRỌNG)

```bash
sudo usermod -aG docker $USER
newgrp docker
```

> ❗ Nếu bỏ bước này → sau này rất **khó chịu** khi làm K8s

---

## 1.6 Kiểm tra Docker version

```bash
docker version
```

✔️ Có cả:

- Client
    
- Server
    

---

## 1.7 Test container đầu tiên (bắt buộc)

```bash
docker run hello-world
```

### Kết quả đúng:

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
```

🎉 **CHÚC MỪNG – DOCKER ĐÃ HOẠT ĐỘNG**

---

# 🧠 GIẢI THÍCH NGẮN GỌN (để bạn nhớ khi thuyết trình)

- **Docker** = nền tảng chạy container
    
- **Container** = ứng dụng + môi trường + thư viện
    
- Kubernetes **không chạy app trực tiếp**, mà chạy **container**
    
- Vì vậy:  
    👉 _Docker ổn → K8s mới ổn_
    

---

# ✅ CHECKLIST BƯỚC 1

|Hạng mục|Trạng thái|
|---|---|
|Docker cài thành công|✅|
|docker run không cần sudo|✅|
|hello-world chạy được|✅|

---

👉 **XÁC NHẬN GIÚP MÌNH**  
Bạn hãy trả lời **đúng 1 dòng** theo mẫu:

> `Docker OK`

Sau đó mình sẽ **chuyển sang BƯỚC 2 – CÀI KUBERNETES (K3s)**  
(bước này bắt đầu “vào lõi” đồ án 🚀)