
---

# Phân Tích Sâu Chương 14: Serverless Computing (FaaS)

### 1. Vấn đề của mô hình Client-Server truyền thống trên Cloud

Trước khi hiểu Serverless, ta cần nhìn lại các mô hình cũ để thấy tại sao nó ra đời:

- **Mô hình truyền thống:** Bạn phải bật một máy chủ (Server), máy chủ này chạy một vòng lặp vô tận chờ đợi Client kết nối. Dù không có ai truy cập, máy chủ vẫn chạy và tiêu tốn tài nguyên,.
- **Vấn đề Kinh tế:** Khi đưa lên Cloud (sử dụng VM hoặc Container), bạn phải dự đoán mức tải cao nhất (Peak load) để thuê máy chủ.
    - Nếu thuê thừa: Bạn lãng phí tiền cho các VM nhàn rỗi (Idle capacity).
    - Nếu thuê thiếu: Ứng dụng bị sập.
    - **Chi phí ẩn:** Bạn tốn tiền thuê chuyên gia (DevOps) để cấu hình Auto-scaling, Load Balancer, và vá lỗi hệ điều hành.

### 2. Giải pháp: Serverless Computing (FaaS)

**Serverless** (hay _Function as a Service - FaaS_) là mô hình mà nhà cung cấp đám mây chịu trách nhiệm hoàn toàn về việc quản lý máy chủ.

- **Tại sao gọi là Serverless?** Không phải là "không có máy chủ" (vẫn có máy chủ vật lý chạy bên dưới), mà là người lập trình **không cần quan tâm** đến máy chủ. Họ chỉ viết code (Function) và đẩy lên Cloud.
- **Cơ chế thanh toán đột phá:** Bạn chỉ trả tiền cho thời gian mà hàm (function) thực sự chạy. Nếu không có ai truy cập, bạn không trả xu nào.
- **Scale to Zero:** Đây là tính năng quan trọng nhất. Nếu không có tải, số lượng instance là 0. Khi có sự kiện, nó tự động bật lên.
- **Scale to Infinity:** Về lý thuyết, nó có thể mở rộng vô hạn để xử lý hàng triệu request cùng lúc mà không cần cấu hình thêm.

### 3. Hai đặc tính kỹ thuật cốt lõi (Bắt buộc nhớ)

Để Serverless hoạt động, ứng dụng phải tuân thủ 2 nguyên tắc nghiêm ngặt:

#### A. Stateless Servers (Máy chủ phi trạng thái)

- Do các container chạy function được sinh ra và hủy đi liên tục (ephemeral), chúng không thể lưu giữ dữ liệu trong bộ nhớ (RAM/biến toàn cục) để dùng cho lần chạy sau,.
- **Hệ quả:** Mọi dữ liệu cần lưu trữ (State) phải được ghi vào database bên ngoài (như S3, DynamoDB, Redis). Code của bạn chỉ làm nhiệm vụ xử lý logic.

#### B. Event-driven Paradigm (Mô hình hướng sự kiện)

- Serverless không chạy ngầm chờ đợi. Nó **chỉ thức dậy** khi có một sự kiện (Event) kích hoạt nó.
- **Các loại sự kiện:**
    - HTTP Request (REST API gọi tới).
    - Database change (Một dòng mới được thêm vào DB).
    - Storage event (Một file mới được upload lên S3).
    - Timer (Chạy định kỳ như Cron job).

### 4. Kiến trúc bên dưới (Under the hood)

Mặc dù "Serverless" với người dùng, nhưng bên dưới nhà cung cấp xây dựng nó dựa trên mô hình **Controller-based** (giống Kubernetes):

1. **Event Queue (Hàng đợi sự kiện):** Nơi tiếp nhận các yêu cầu (ví dụ: 1000 người dùng cùng upload ảnh).
2. **Dispatcher:** Lấy sự kiện từ hàng đợi và tìm Worker rảnh rỗi.
3. **Worker Nodes:** Các máy chủ vật lý thực thi code của bạn (thường chạy trong các Container rất nhẹ).

> **Ví dụ kinh điển (Netflix Video Transcoding):**
> 
> 1. Người dùng upload video $\rightarrow$ Tạo ra sự kiện "New Video" trên S3.
> 2. Sự kiện này kích hoạt hàm `Split` $\rightarrow$ Chia video thành các đoạn nhỏ 5 phút.
> 3. Mỗi đoạn nhỏ kích hoạt một hàm `Transcode` riêng biệt. Hàng ngàn hàm `Transcode` chạy song song.
> 4. Kết quả được gom lại. _Lợi ích:_ Xử lý xong video cực nhanh nhờ song song hóa mà không cần duy trì một cụm server khổng lồ tốn kém-.

### 5. Ưu và Nhược điểm (Dành cho so sánh/phản biện)

|Ưu điểm (Pros)|Nhược điểm (Cons)|
|:--|:--|
|**Chi phí:** Chỉ trả tiền khi chạy (Pay-as-you-go). Rẻ hơn rất nhiều cho các tác vụ không chạy liên tục.|**Độ trễ (Latency/Cold Start):** Khi function chưa chạy, mất một khoảng thời gian (vài giây) để khởi tạo container. Không phù hợp cho ứng dụng cần phản hồi tức thì (real-time gaming).|
|**Vận hành:** Không cần quản lý OS, vá lỗi, hay cấu hình Scaling. Developer chỉ lo code.|**Gỡ lỗi (Debugging):** Rất khó debug vì môi trường chạy trên mây, khó tái lập dưới local.|
|**Mở rộng:** Tự động scale từ 0 lên hàng nghìn instance ngay lập tức.|**Vendor Lock-in:** Code thường bị dính chặt vào API của nhà cung cấp (ví dụ viết cho AWS Lambda khó mang sang Azure Functions).|
||**Giới hạn tài nguyên:** Function thường bị giới hạn thời gian chạy (ví dụ tối đa 15 phút) và RAM. Không chạy được các tác vụ train AI lâu dài.|

### Tổng kết Chương 14

Chương này đánh dấu sự chuyển dịch sang mô hình **Cloud-native** thực thụ. Serverless là sự kết hợp của:

- Công nghệ **Container** (để khởi động nhanh).
- Mô hình **Orchestration** (để điều phối).
- Kiến trúc **Event-driven** (để kích hoạt). Nó là lựa chọn tối ưu cho các tác vụ xử lý sự kiện, microservices nhỏ, và các ứng dụng có lưu lượng truy cập không ổn định (bursty traffic).