
### 1. Tổng quan về Apache Kafka

Apache Kafka là một nền tảng **phân tán (distributed streaming platform)** dựa trên mô hình **Publish-Subscribe**.

- **Nguồn gốc:** Được phát triển ban đầu bởi **LinkedIn** để xử lý lượng dữ liệu khổng lồ của họ, sau đó trở thành dự án mã nguồn mở của Apache và hiện được bảo trì bởi Confluent.
- **Vấn đề giải quyết:** Kafka giải quyết bài toán giao tiếp dữ liệu giữa các hệ thống nguồn (Producers) và hệ thống đích (Consumers) một cách lỏng lẻo (decoupled), thay thế cho các kết nối điểm-điểm (point-to-point) phức tạp và chậm chạp.
- **Hiệu năng:** Kafka cực kỳ nhanh với độ trễ thấp (**< 10ms**) và thông lượng (throughput) cao, có khả năng xử lý hàng triệu tin nhắn mỗi giây,.

### 2. Các thành phần cốt lõi (Core Components)

Để hiểu Kafka, bạn cần nắm vững các thuật ngữ sau trong kiến trúc của nó:

#### A. Topic & Partition (Chủ đề & Phân vùng)

- **Topic:** Là một danh mục (category) hoặc tên feed mà dữ liệu được gửi vào. Nó tương tự như một "bảng" trong cơ sở dữ liệu nhưng không có các ràng buộc chặt chẽ. Bạn có thể tạo bao nhiêu topic tùy ý.
- **Partition:** Mỗi Topic được chia nhỏ thành nhiều phần gọi là **Partition**.
    - Dữ liệu trong Partition được sắp xếp theo thứ tự và được gán một ID tăng dần gọi là **Offset**.
    - **Lưu ý quan trọng:** Thứ tự của tin nhắn chỉ được đảm bảo **trong cùng một Partition**, không đảm bảo trên toàn bộ Topic.

#### B. Broker (Người môi giới)

- Một Kafka cluster bao gồm một hoặc nhiều máy chủ, mỗi máy chủ được gọi là một **Broker**.
- Broker chứa các Topic và Partition. Tuy nhiên, một Broker không chứa toàn bộ dữ liệu của hệ thống mà chỉ chứa một phần. Các Broker phối hợp với nhau và chỉ cần kết nối tới một Broker (Bootstrap broker) là có thể kết nối tới toàn bộ cluster.

#### C. Producer & Consumer (Người gửi & Người nhận)

- **Producer:** Ứng dụng gửi (publish) dữ liệu vào các Topic. Producer tự động biết nên gửi dữ liệu vào Partition nào và Broker nào.
- **Consumer:** Ứng dụng đăng ký (subscribe) vào các Topic để đọc và xử lý dữ liệu.

#### D. Zookeeper (Thành phần quản lý bắt buộc)

Trong kiến trúc Kafka truyền thống (được mô tả trong tài liệu):

- **Vai trò sống còn:** Kafka **không thể chạy** nếu thiếu Zookeeper.
- **Chức năng:**
    - Lưu trữ thông tin cấu hình cluster và trạng thái của các Broker.
    - Quản lý danh sách các Broker đang hoạt động (Heartbeat).
    - **Bầu chọn Leader:** Quyết định Broker nào làm "Leader" cho một Partition khi có sự cố xảy ra.

### 3. Cơ chế hoạt động chi tiết (Under the hood)

#### A. Replication (Sao chép & Chịu lỗi)

Để đảm bảo an toàn dữ liệu khi một Broker bị hỏng, Kafka sử dụng cơ chế **Replication**.

- Mỗi Partition có một hệ số sao chép (Replication Factor, thường là 2 hoặc 3).
- **Mô hình Leader/Follower:**
    - Với mỗi Partition, một Broker được bầu làm **Leader**, các bản sao ở Broker khác gọi là **Followers** (ISR - In-Sync Replicas),.
    - **Quy tắc vàng:** Mọi thao tác Đọc/Ghi (Read/Write) của Client đều **chỉ thực hiện với Leader**. Follower chỉ có nhiệm vụ đồng bộ dữ liệu từ Leader để dự phòng.

#### B. Message Keys (Khóa tin nhắn)

Khi Producer gửi tin nhắn, nó có 2 lựa chọn:

1. **Không gửi Key (Key = Null):** Kafka sẽ phân phối dữ liệu theo cơ chế **Round-robin** (chia đều) vào các Partition để cân bằng tải.
2. **Có gửi Key (Key != Null):** Tất cả các tin nhắn có cùng Key (ví dụ: `ID_KhachHang_123`) sẽ luôn được đưa vào **cùng một Partition**. Điều này rất quan trọng nếu bạn cần xử lý dữ liệu của một khách hàng theo đúng thứ tự thời gian.

#### C. Acknowledgement (Cơ chế xác nhận)

Producer có thể chọn mức độ an toàn khi gửi tin nhắn thông qua `acks`:

- `acks=0`: Gửi và quên (Fire & forget). Nhanh nhất nhưng rủi ro mất dữ liệu cao nhất.
- `acks=1`: Chờ Leader xác nhận đã nhận. An toàn trung bình.
- `acks=all`: Chờ Leader và tất cả Followers xác nhận. An toàn tuyệt đối nhưng chậm nhất.

### 4. Ưu và Nhược điểm

|Ưu điểm (Pros)|Nhược điểm (Cons)|
|:--|:--|
|**Độ trễ thấp & Thông lượng cao:** Xử lý hàng triệu tin nhắn/giây với độ trễ <10ms.|**Thiếu công cụ giám sát:** Kafka thiếu bộ công cụ quản lý và giám sát (monitoring) toàn diện có sẵn, khiến người mới e ngại.|
|**Bền bỉ (Durability):** Dữ liệu được ghi xuống đĩa (disk) và sao chép ra nhiều nơi, an toàn hơn việc chỉ lưu trên RAM.|**Vấn đề Wildcard:** Không hỗ trợ chọn Topic theo ký tự đại diện (wildcard selection), phải khớp chính xác tên Topic.|
|**Khả năng mở rộng (Scalability):** Dễ dàng mở rộng phân tán (Scale-out) mà không gián đoạn hệ thống.|**Hiệu năng giảm khi message thay đổi:** Nếu cần sửa đổi (tweaking) nội dung tin nhắn trên đường truyền, hiệu năng Kafka giảm đáng kể.|

### 5. Ứng dụng thực tế (Use Cases)

Dựa trên tài liệu, Kafka được dùng trong các kịch bản lớn:

- **LinkedIn:** Theo dõi hoạt động người dùng, newsfeed.
- **Uber:** Kết nối hệ sinh thái tài xế và hành khách, xử lý luồng dữ liệu thời gian thực.
- **Netflix:** Đường ống dữ liệu (Keystone Pipeline) để xử lý và định tuyến luồng sự kiện.
- **Log Aggregation:** Gom log từ nhiều server về một chỗ trung tâm.
- **IoT & Sensor Data:** Xử lý dữ liệu cảm biến thời gian thực.

### Tổng kết

Kafka là "xương sống" cho các hệ thống xử lý dữ liệu thời gian thực hiện đại. Nếu **HDFS** là kho chứa hàng khổng lồ (Lưu trữ), **MapReduce/Spark** là nhà máy chế biến (Xử lý), thì **Kafka** chính là hệ thống băng chuyền siêu tốc vận chuyển nguyên liệu giữa các bộ phận đó một cách liên tục và tin cậy.