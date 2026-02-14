# Kiến thức
## 1. Cơ sở hạ tầng vật lý (Physical Infrastructure)

### Đơn vị cơ bản
> [!NOTE] Rack & Unit (U)
> * **Rack:** Tủ chứa thiết bị tiêu chuẩn (cao ~6.5 ft, rộng ~2 ft).
> * **Unit (U):** Đơn vị đo chiều cao thiết bị trong rack. `1U = 1.75 inches`. Một rack tiêu chuẩn thường chứa được **42U**.

### Pod (Point of Delivery)
> [!INFO] Định nghĩa
> Một đơn vị thiết kế mô-đun (modular design unit) của Data Center. Thay vì lắp từng server lẻ tẻ, người ta thiết kế và lắp đặt nguyên một **Pod**.

* **Thành phần:** Gồm các tủ rack chứa Servers, Storage, Networking và nguồn điện (PDU).
* **Lợi ích:** Dễ dàng mở rộng (Incremental growth) và quản lý độc lập. Kích thước trung bình hiện nay khoảng **12-16 racks/pod** (giảm so với trước kia để tối ưu nguồn/nhiệt).

---

## 2. Năng lượng & Làm mát (Power & Cooling)

Đây là vấn đề sống còn của Data Center (tiêu thụ ~2% điện năng toàn cầu).

| Thuật ngữ           | Định nghĩa & Ý nghĩa                                                                                                                                                                                                               |
| :------------------ | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Power Density**   | Mật độ năng lượng tiêu thụ trên mỗi rack. Server càng mạnh -> Power density càng cao -> Tỏa nhiệt càng nhiều.                                                                                                                      |
| **Raised Floor**    | **Sàn nâng.** Thiết kế sàn giả, bên dưới là khoảng trống để đi dây điện và **thổi luồng khí lạnh** từ dưới lên làm mát server.                                                                                                     |
| **Hot/Cold Aisles** | **Lối đi nóng/lạnh.** Cách sắp xếp rack thông minh: Mặt trước rack đối nhau (hút khí lạnh) tạo thành *Cold Aisle*. Mặt sau rack đối nhau (xả khí nóng) tạo thành *Hot Aisle*. Mục tiêu là **Thermal Containment** (cách ly nhiệt). |
| **Chimney**         | **Ống khói.** Ống dẫn đặt trên "hot spot" để hút khí nóng bay thẳng lên trần, tránh lan tỏa ra xung quanh.                                                                                                                         |
| **Lights-out DC**   | **Data Center tắt đèn.** Vận hành hoàn toàn tự động, không cần con người (trừ khi sửa chữa). Giúp tiết kiệm điện chiếu sáng và giảm nhiệt độ.                                                                                      |

---

## 3. Kiến trúc mạng (Data Center Networking)

### Phân loại lưu lượng (Traffic Patterns)
> [!IMPORTANT] Phân biệt North-South & East-West
> * **North-South Traffic:** Lưu lượng đi **vào/ra** Data Center (giữa Internet và Server). Ví dụ: Người dùng truy cập Web.
> * **East-West Traffic:** Lưu lượng **nội bộ** Data Center (giữa Server và Server). Ví dụ: Web server gọi Database server, hoặc sao chép dữ liệu backup. Đây là loại lưu lượng chiếm ưu thế trong Cloud hiện đại.

### Kiến trúc mạng (Topologies)
1.  **Hierarchical (Fat Tree):** Kiến trúc cũ, hình cây. Nhược điểm là nghẽn cổ chai (bottleneck) ở các tầng trên cùng khi xử lý lưu lượng East-West.
2.  **Leaf-Spine Architecture:** Kiến trúc chuẩn cho Cloud hiện đại.
    * **Leaf Switch (ToR):** Nằm ở đầu mỗi rack (Top-of-Rack), kết nối trực tiếp với server.
    * **Spine Switch:** Kết nối với tất cả Leaf switches.
    * *Ưu điểm:* Tăng băng thông cho East-West traffic, độ trễ thấp và đồng đều (mọi server cách nhau đúng 2 hops).
3.  **Super Spine:** Mở rộng Leaf-Spine cho quy mô lớn. Dùng để kết nối các **Pods** với nhau.

### Thiết bị mạng
* **ToR Switch (Top-of-Rack):** Switch đặt ở đỉnh rack, gom kết nối của toàn bộ server trong rack đó.
* **Smart NIC & Offload:** Card mạng thông minh có khả năng xử lý hộ CPU các tác vụ mạng (checksum, encryption). Giúp CPU rảnh tay để xử lý ứng dụng (**Offloading**).

---

## 4. Kiến trúc lưu trữ (Storage Infrastructure)

Chương 4 nhấn mạnh sự dịch chuyển từ ổ cứng gắn trong (Local Disk) sang lưu trữ qua mạng.

> [!Warning] Vấn đề của Local Disk
> Trong môi trường Multi-tenant, nếu ổ cứng gắn chết vào server vật lý, khi server hỏng hoặc cần di chuyển VM thì dữ liệu sẽ bị kẹt lại.

| Công nghệ                                | Đặc điểm                                                                                                                                                                                          | Xu hướng hiện tại                                                 |
| :--------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | :---------------------------------------------------------------- |
| **NAS** (Network Attached Storage)       | Lưu trữ **File** (File-level). Chia sẻ file qua mạng (như NFS). Tốt cho chia sẻ dữ liệu chung.                                                                                                    | Vẫn dùng nhưng độ trễ cao hơn Block storage.                      |
| **SAN** (Storage Area Network)           | Lưu trữ **Block** (Block-level). Server nhìn thấy như một ổ cứng vật lý gắn rời. Hiệu năng cao.                                                                                                   | Dùng mạng riêng đắt tiền -> Chuyển sang dùng chung mạng Ethernet. |
| **HCI** (Hyper-Converged Infrastructure) | **Hạ tầng siêu hội tụ.** Sử dụng chính mạng Ethernet của Data Center để truyền tải dữ liệu lưu trữ (Storage traffic) thay vì xây mạng SAN riêng biệt. Nhờ có kiến trúc Leaf-Spine băng thông lớn. | **Xu hướng chủ đạo** của Cloud hiện đại.                          |

---

# Pool Câu Hỏi Trắc Nghiệm - Cloud Computing Chapter 4

## Chủ đề: Hạ tầng Vật lý, Làm mát và Kiến trúc Mạng Data Center

### Câu 1
**Trong thiết kế trung tâm dữ liệu, thuật ngữ "Pod" (Point of Delivery) dùng để chỉ điều gì?**
A. Một loại container chứa hàng hóa.
B. Một đơn vị thiết bị cơ bản có thể lặp lại (bao gồm server, storage, network, nguồn điện) được thiết kế để lắp đặt theo mô-đun.
C. Một ứng dụng nghe nhạc.
D. Một thiết bị làm mát di động.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Thay vì chỉ là các hàng tủ rack dài, trung tâm dữ liệu được xây dựng bằng cách nhân bản một tập hợp thiết bị cơ bản gọi là Pod (Point of Delivery). Một Pod bao gồm server, lưu trữ, thiết bị mạng và đơn vị phân phối nguồn (PDU).

---

### Câu 2
**Mục đích chính của thiết kế "Raised Floor" (Sàn nâng) trong các trung tâm dữ liệu truyền thống là gì?**
A. Để tránh ngập lụt.
B. Để tạo không gian giấu dây cáp bừa bộn.
C. Để bơm không khí lạnh từ hệ thống điều hòa xuống dưới sàn và đẩy lên qua các tấm gạch đục lỗ (perforated tiles) làm mát thiết bị.
D. Để cách âm cho các máy chủ.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Thiết kế sàn nâng cho phép không khí lạnh được đẩy xuống không gian dưới sàn. Các tấm gạch dưới tủ rack được đục lỗ, cho phép không khí lạnh thổi ngược lên để làm mát thiết bị,.

---

### Câu 3
**Kỹ thuật "Thermal Containment" sử dụng mô hình "Hot aisle / Cold aisle" (Lối đi nóng / Lối đi lạnh) nhằm giải quyết vấn đề gì?**
A. Giúp nhân viên kỹ thuật không bị lạnh khi làm việc.
B. Ngăn chặn khí nóng thải ra từ thiết bị này bị hút vào đầu vào (khe lấy khí) của thiết bị khác, gây giảm hiệu quả làm mát.
C. Tăng tính thẩm mỹ cho trung tâm dữ liệu.
D. Giảm tiếng ồn của quạt.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Kỹ thuật này sắp xếp các tủ rack sao cho mặt trước đối diện mặt trước (lối lạnh) và mặt sau đối diện mặt sau (lối nóng). Điều này giúp hướng khí nóng vào các khu vực được kiểm soát và ngăn nó bị hút ngược lại vào thiết bị khác.

---

### Câu 4
**"North-South traffic" (Lưu lượng Bắc-Nam) trong trung tâm dữ liệu đề cập đến loại lưu lượng nào?**
A. Lưu lượng giữa các máy chủ (server-to-server) bên trong data center.
B. Lưu lượng điều khiển từ vệ tinh.
C. Lưu lượng trao đổi giữa các máy chủ bên trong data center và các máy khách (clients) bên ngoài Internet.
D. Lưu lượng di chuyển dữ liệu sao lưu sang cực Bắc.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Thuật ngữ North-South traffic mô tả lưu lượng được gửi giữa các máy tính bất kỳ trên Internet và các máy chủ trong trung tâm dữ liệu (ví dụ: người dùng truy cập web).

---

### Câu 5
**Sự bùng nổ của Cloud Computing đã làm gia tăng loại lưu lượng nào, dẫn đến sự thay đổi kiến trúc mạng Data Center?**
A. North-South traffic.
B. East-West traffic.
C. Voice traffic.
D. Broadcast traffic.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Khi chuyển sang điện toán đám mây, các ứng dụng cần truy cập cơ sở dữ liệu, dịch vụ nội bộ, và giao tiếp giữa các máy chủ thuê bao. Điều này tạo ra lưu lượng di chuyển "trái-phải" giữa các máy chủ trong data center, gọi là East-West traffic.

---

### Câu 6
**Kiến trúc mạng "Leaf-Spine" được sử dụng trong các Data Center hiện đại để thay thế kiến trúc phân cấp (Hierarchical) nhằm mục đích gì?**
A. Giảm chi phí dây cáp.
B. Tăng dung lượng cho lưu lượng East-West và cho phép mở rộng băng thông bằng cách thêm spine switch (scale-out) thay vì phải thay thế thiết bị (scale-up).
C. Hỗ trợ tốt hơn cho kết nối Wifi.
D. Giảm số lượng thiết bị chuyển mạch cần sử dụng.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Kiến trúc Leaf-Spine cung cấp dung lượng cao hơn cho lưu lượng East-West nhờ nhiều đường dẫn song song và cho phép tăng dung lượng mạng từng bước chỉ bằng cách thêm các Spine switch mới,.

---

### Câu 7
**Trong kiến trúc Leaf-Spine, giao thức hoặc công nghệ nào cho phép chia tải đều qua nhiều đường dẫn song song giữa Leaf và Spine?**
A. Spanning Tree Protocol (STP).
B. Equal Cost Multipath Routing (ECMP).
C. Border Gateway Protocol (BGP).
D. DHCP.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Một Leaf switch được trang bị công nghệ ECMP có thể chia lưu lượng đều giữa các đường dẫn (paths) đi qua các Spine switch khác nhau, giúp tận dụng tối đa băng thông.

---

### Câu 8
**Lợi ích chính của việc sử dụng "Smart NIC" (Card mạng thông minh) trong máy chủ Data Center là gì?**
A. Tự động kết nối Wifi.
B. Offload (giảm tải) các tác vụ xử lý gói tin (như tính checksum, mã hóa/giải mã) từ CPU chính sang phần cứng của card mạng.
C. Tăng dung lượng lưu trữ của máy chủ.
D. Giảm nhiệt độ máy chủ.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Smart NIC chứa phần cứng đặc biệt để xử lý các tác vụ mạng (như mã hóa). Việc offload này giúp CPU chính rảnh rang để tập trung vào việc tính toán cho người dùng thay vì xử lý gói tin.

---

### Câu 9
**Tại sao các Data Center hiện đại chuyển từ mô hình lưu trữ cục bộ (mỗi server có ổ cứng riêng) sang mô hình lưu trữ tập trung (SAN/NAS) hoặc Block Storage qua mạng?**
A. Vì ổ cứng cục bộ quá đắt.
B. Để hỗ trợ mô hình đa thuê bao (multi-tenant) tốt hơn, cho phép di chuyển VM dễ dàng và tránh việc nhân viên phải đi tìm từng rack để thay ổ cứng hỏng.
C. Để giảm băng thông mạng.
D. Vì các hệ điều hành hiện đại không hỗ trợ ổ cứng cục bộ.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Trong hệ thống multi-tenant, việc dùng ổ cứng cục bộ gây khó khăn cho việc giới hạn dung lượng và thay thế thiết bị hỏng. Data center dùng storage ảo hóa tập trung để giải quyết các vấn đề này.

---

### Câu 10
**"Unified Data Center Network" (Mạng trung tâm dữ liệu thống nhất) đề cập đến xu hướng gì?**
A. Sử dụng một loại cáp duy nhất cho toàn bộ tòa nhà.
B. Hợp nhất mạng lưu trữ (Storage network) và mạng dữ liệu thông thường vào chung một hạ tầng Ethernet duy nhất.
C. Chỉ sử dụng Wifi thay vì dây cáp.
D. Bắt buộc tất cả khách hàng dùng chung một địa chỉ IP.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Sự sẵn có của phần cứng Ethernet tốc độ cao và kiến trúc Leaf-Spine đã cho phép các data center loại bỏ các mạng lưu trữ chuyên dụng tốn kém, chuyển sang dùng một mạng thống nhất cho cả truy cập lưu trữ và các giao tiếp khác,.


# Pool Câu Hỏi Trắc Nghiệm Nâng Cao (Hard Mode) - Cloud Computing Chapter 4

## Chủ đề: Phân tích sâu về Kiến trúc Mạng, Vật lý và Tối ưu hóa Hạ tầng

### Câu 11
**Trong kiến trúc mạng Leaf-Spine, giả sử bạn cần tăng gấp đôi băng thông (bandwidth) khả dụng cho giao tiếp East-West giữa các máy chủ mà không thay đổi số lượng tủ rack. Bạn sẽ thực hiện thao tác nào sau đây?**
A. Thay thế toàn bộ dây cáp quang bằng loại cáp mới có tiết diện lớn hơn.
B. Thêm số lượng Spine switches lên gấp đôi và kết nối lại các Leaf switches vào các Spine mới này.
C. Nâng cấp phần mềm trên các Leaf switches để hỗ trợ giao thức nén dữ liệu.
D. Mua thêm Leaf switches và cắm chồng lên các Leaf switches cũ.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Đây là tính năng "Scale-out" đặc trưng của kiến trúc Leaf-Spine. Băng thông giữa các rack được quyết định bởi số lượng đường dẫn song song qua các Spine. Thêm Spine switch tạo ra nhiều đường dẫn hơn, tăng dung lượng mạng mà không cần thay thế thiết bị cũ (Scale-up).

---

### Câu 12
**Tại sao thiết kế "Fat Tree" (Cây béo) trong kiến trúc mạng phân cấp truyền thống lại trở nên không kinh tế và khó mở rộng khi áp dụng cho các Data Center hiện đại xử lý Big Data?**
A. Vì nó yêu cầu trồng cây thật trong Data Center để giảm nhiệt.
B. Vì các liên kết ở tầng trên cùng (Root) đòi hỏi băng thông cực lớn (100% lưu lượng đi qua), buộc phải dùng các thiết bị mạng chuyên dụng cực đắt tiền và khó mua được phần cứng có tốc độ đáp ứng nổi.
C. Vì cấu trúc cây khiến dữ liệu dễ bị lạc đường.
D. Vì Fat Tree chỉ hỗ trợ lưu lượng East-West mà không hỗ trợ North-South.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Trong kiến trúc phân cấp, càng lên cao băng thông càng phải lớn (links near the top carry more traffic). Việc mua các thiết bị mạng có dung lượng khổng lồ ở tầng trên cùng tốn kém hơn nhiều so với việc dùng nhiều thiết bị nhỏ, và đôi khi phần cứng thương mại không đáp ứng được tốc độ đó.

---

### Câu 13
**Một kỹ sư thiết kế Data Center quyết định giảm kích thước của mỗi Pod (số lượng rack trong một Pod) từ 48 xuống còn 12 rack. Ngoài lý do dễ vận chuyển, động lực kỹ thuật chính liên quan đến "Power Density" (Mật độ năng lượng) cho quyết định này là gì?**
A. Để giảm tiền thuê bảo vệ trông coi Pod.
B. Vì mật độ năng lượng của các máy chủ hiện đại tăng cao, một Pod quá lớn sẽ vượt quá khả năng cung cấp của một đường dây điện (electrical feed) duy nhất hoặc vượt quá khả năng làm mát cục bộ.
C. Để tăng số lượng Pod nhằm làm đẹp bản vẽ thiết kế.
D. Vì các rack nhỏ hơn thì tiêu thụ ít điện hơn.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Mỗi Pod thường có một nguồn cấp điện riêng. Khi các máy chủ ngày càng mạnh (nhiều core hơn), chúng tiêu thụ nhiều điện hơn (tăng power density). Nếu giữ Pod quá lớn, tổng lượng điện tiêu thụ sẽ vượt quá giới hạn thiết kế của nguồn cấp hoặc hệ thống làm mát.

---

### Câu 14
**Trong mô hình "Hot aisle / Cold aisle", nếu một kỹ thuật viên vô tình lắp đặt một máy chủ bị ngược hướng (mặt trước quay về phía lối nóng) trong một tủ rack, hậu quả vật lý nào sẽ xảy ra?**
A. Máy chủ đó sẽ chạy nhanh hơn do hút được khí nóng năng lượng cao.
B. Không có gì xảy ra, máy chủ hoạt động bình thường.
C. Máy chủ đó sẽ hút khí nóng thải ra từ các máy chủ khác vào làm khí đầu vào (intake), dẫn đến quá nhiệt và hỏng hóc thiết bị.
D. Toàn bộ Data Center sẽ bị mất điện ngay lập tức.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Nguyên lý là máy chủ hút khí lạnh từ mặt trước (lối lạnh) và thải khí nóng ra mặt sau (lối nóng). Nếu lắp ngược, nó sẽ hút khí từ lối nóng (nơi chứa khí thải 60-80 độ C của các máy khác) để làm mát, khiến CPU không thể tản nhiệt và sẽ bị quá nhiệt.

---

### Câu 15
**Giao thức ECMP (Equal Cost Multipath Routing) đóng vai trò sống còn như thế nào trong kiến trúc Leaf-Spine khi một Spine switch bị hỏng?**
A. Nó tự động sửa chữa phần cứng của Spine switch bị hỏng.
B. Nó là một giao thức báo động reo chuông để gọi nhân viên.
C. Nó cho phép các Leaf switch tự động nhận biết mất một đường dẫn và phân chia lại (re-hash) lưu lượng sang các đường dẫn còn lại (các Spine switch đang hoạt động) mà không gây gián đoạn dịch vụ.
D. Nó chặn toàn bộ lưu lượng mạng để bảo vệ dữ liệu.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** ECMP cho phép chia tải đều qua nhiều đường dẫn. Khi một Spine hỏng, phần cứng và phần mềm quản lý mạng sẽ phát hiện và ECMP sẽ định tuyến lại các gói tin qua các Spine còn lại. Mặc dù băng thông tổng giảm, nhưng kết nối vẫn duy trì.

---

### Câu 16
**Sự chuyển dịch từ mạng lưu trữ chuyên dụng (như Fibre Channel trong SAN truyền thống) sang "Unified Data Center Network" (Mạng thống nhất dùng Ethernet) được thúc đẩy bởi yếu tố công nghệ nào?**
A. Sự ra đời của Wifi 6.
B. Sự sẵn có của phần cứng Ethernet tốc độ cao giá rẻ và kiến trúc Leaf-Spine cung cấp đủ băng thông cho lưu lượng East-West.
C. Do các nhà sản xuất ngừng sản xuất cáp quang.
D. Do nhu cầu lưu trữ dữ liệu giảm xuống.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Trước đây SAN cần mạng riêng vì mạng LAN quá chậm hoặc tắc nghẽn. Ngày nay, Ethernet tốc độ cao (10/40/100 Gbps) và kiến trúc Leaf-Spine (tối ưu cho East-West) đã đủ mạnh để gánh cả lưu lượng truy cập đĩa, cho phép hợp nhất vào một mạng duy nhất để tiết kiệm chi phí.

---

### Câu 17
**Tại sao việc sử dụng Smart NIC (Card mạng thông minh) lại được coi là một quyết định kinh tế quan trọng đối với nhà cung cấp Cloud, mặc dù giá thành của Smart NIC đắt hơn NIC thường?**
A. Vì Smart NIC có đèn LED đẹp mắt hơn.
B. Vì Smart NIC offload (giảm tải) việc xử lý gói tin (vốn rất nặng trong môi trường ảo hóa) khỏi CPU chính, giải phóng CPU để chạy các máy ảo của khách hàng - thứ trực tiếp sinh ra tiền (revenue-generating).
C. Vì Smart NIC tiêu thụ nhiều điện hơn giúp sưởi ấm Data Center.
D. Vì Smart NIC có thể thay thế ổ cứng.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Xử lý mạng (đóng gói, mã hóa, checksum) tốn nhiều tài nguyên CPU. Nếu dùng CPU server để làm việc này thì lãng phí khả năng tính toán có thể bán cho khách hàng. Smart NIC xử lý việc này bằng phần cứng chuyên dụng, giúp "bán" được nhiều CPU core hơn.

---

### Câu 18
**Thuật ngữ "Lights-out Data Center" thực chất ám chỉ điều gì về phương thức vận hành, thay vì chỉ đơn giản là tắt đèn?**
A. Data Center không có cửa sổ.
B. Vận hành dựa trên nguyên tắc loại bỏ sự hiện diện của con người tại hiện trường: quản lý từ xa hoàn toàn, tự động hóa việc phục hồi lỗi (như tự chuyển mạch khi link chết), và con người chỉ vào khi cần thay thế phần cứng vật lý.
C. Data Center sử dụng năng lượng mặt trời.
D. Data Center chỉ hoạt động vào ban đêm.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** "Lights-out" nghĩa là vận hành trong bóng tối (không cần người). Điều này đòi hỏi hệ thống quản lý tự động cao, tự xử lý các lỗi phần mềm/mạng để giảm thiểu sự can thiệp thủ công, giảm chi phí nhân sự và tránh lỗi do con người.

---

### Câu 19
**Trong kiến trúc Super Spine (dùng cho các Data Center quy mô cực lớn), luồng dữ liệu đi từ một server ở Pod A sang một server ở Pod B sẽ đi qua các thiết bị theo trình tự nào?**
A. Leaf A -> Spine A -> Super Spine -> Spine B -> Leaf B.
B. Leaf A -> Leaf B -> Spine B.
C. Leaf A -> Super Spine -> Leaf B.
D. Server A -> Server B trực tiếp qua wifi.

> [!success]- Đáp án & Giải thích
> **Đáp án:** A
> **Giải thích:** Để mở rộng quy mô, các Pod được kết nối với nhau qua lớp Super Spine. Dữ liệu đi từ Server -> Leaf (trong Pod A) -> Spine (trong Pod A) -> Super Spine -> Spine (trong Pod B) -> Leaf (trong Pod B) -> Server.

---

### Câu 20
**Về mặt vật lý, tại sao việc chuyển đổi từ làm mát bằng khí (Air cooling) sang làm mát bằng chất lỏng (Liquid cooling) lại là một thách thức hạ tầng cực lớn, dù hiệu quả làm mát tốt hơn?**
A. Vì nước làm hỏng máy tính ngay lập tức.
B. Vì nó đòi hỏi thay thế toàn bộ hệ thống đường ống, lắp đặt hệ thống thủy lực bơm chất lỏng đến từng rack, và quan trọng nhất là phải thay thế toàn bộ server/switch bằng loại có đầu nối ống dẫn chất lỏng (hydraulic fittings).
C. Vì chất lỏng quá đắt đỏ.
D. Vì nhân viên kỹ thuật không biết bơi.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Liquid cooling yêu cầu thay đổi hạ tầng cơ bản (đường ống, bơm) và thay đổi cả thiết bị IT (phải có tấm tản nhiệt nước, đầu nối ống). Không thể chỉ đơn giản là "đổ nước" vào hệ thống cũ.


# Pool Câu Hỏi Thực Tế & Vận Dụng - Cloud Computing Chapter 4

## Chủ đề: Giải quyết vấn đề Hạ tầng, Tối ưu hóa Mạng và Vận hành

### Câu 1
**Tình huống:** Một trung tâm dữ liệu cũ đang gặp vấn đề về hiệu suất làm mát. Mặc dù máy lạnh hoạt động hết công suất, một số máy chủ vẫn bị quá nhiệt. Khi kiểm tra, kỹ sư phát hiện các tủ rack được xếp thành hàng dài, tất cả đều quay mặt về một hướng (mặt trước rack này nhìn vào mặt sau rack kia). Theo nguyên lý "Thermal Containment", bạn cần đề xuất thay đổi gì để khắc phục vấn đề này ngay lập tức?**
A. Lắp thêm quạt vào từng máy chủ.
B. Sắp xếp lại các tủ rack theo mô hình "Hot aisle / Cold aisle" (Lối đi nóng / Lối đi lạnh), tức là mặt trước đối diện mặt trước và mặt sau đối diện mặt sau.
C. Thay thế toàn bộ vỏ máy chủ bằng vật liệu dẫn nhiệt tốt hơn.
D. Hạ thấp trần nhà xuống để giảm thể tích không khí cần làm mát.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Việc xếp các rack cùng hướng khiến khí nóng thải ra từ rack trước bị hút vào rack sau, làm giảm hiệu quả làm mát. Giải pháp chuẩn là sắp xếp sao cho mặt trước đối diện mặt trước (tạo lối lạnh) và mặt sau đối diện mặt sau (tạo lối nóng) để quản lý luồng khí hiệu quả.

---

### Câu 2
**Tình huống:** Công ty của bạn đang chuyển đổi từ ứng dụng Web truyền thống sang kiến trúc Microservices. Đội ngũ giám sát mạng báo cáo rằng băng thông nội bộ bị nghẽn nghiêm trọng, mặc dù lưu lượng từ Internet vào (North-South) không tăng. Nguyên nhân kỹ thuật là do kiến trúc mạng phân cấp (Hierarchical) hiện tại không xử lý tốt loại lưu lượng nào mới phát sinh?**
A. Lưu lượng DNS.
B. Lưu lượng East-West (giữa các máy chủ với nhau trong trung tâm dữ liệu).
C. Lưu lượng VPN.
D. Lưu lượng Email.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Các ứng dụng hiện đại (như microservices) tạo ra lượng lớn giao tiếp giữa các máy chủ nội bộ (East-West traffic). Kiến trúc phân cấp cũ tối ưu cho North-South, không đủ băng thông cho East-West, dẫn đến nghẽn,.

---

### Câu 3
**Tình huống:** Để giải quyết vấn đề nghẽn mạng ở tình huống trên, bạn quyết định chuyển sang kiến trúc mạng "Leaf-Spine". Giả sử sau 6 tháng, nhu cầu băng thông East-West lại tăng gấp đôi. Với kiến trúc Leaf-Spine, bạn có thể mở rộng băng thông (Scale-out) bằng cách nào đơn giản nhất mà không làm gián đoạn hệ thống?**
A. Thay thế toàn bộ dây cáp bằng cáp quang mới.
B. Thêm các Spine switch mới và đấu nối chúng vào tất cả các Leaf switch hiện có.
C. Thay thế các Leaf switch bằng thiết bị có nhiều cổng hơn.
D. Yêu cầu lập trình viên viết lại code để giảm giao tiếp mạng.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Ưu điểm lớn nhất của Leaf-Spine là khả năng mở rộng băng thông ngang (incrementally). Bằng cách thêm Spine switch, bạn tạo thêm các đường dẫn song song cho dữ liệu, tăng tổng dung lượng băng thông mà không cần thay thế thiết bị cũ.

---

### Câu 4
**Tình huống:** Một nhà cung cấp Cloud muốn tối đa hóa doanh thu bằng cách bán (cho thuê) càng nhiều vCPU trên một máy chủ vật lý càng tốt. Tuy nhiên, họ nhận thấy CPU của máy chủ đang phải dành tới 20% tài nguyên chỉ để xử lý việc đóng gói tin, mã hóa và kiểm tra checksum cho mạng ảo hóa. Giải pháp phần cứng nào giúp họ giải phóng 20% CPU này để bán cho khách hàng?**
A. Sử dụng ổ cứng SSD NVMe.
B. Trang bị Smart NIC (Card mạng thông minh) để offload các tác vụ xử lý mạng.
C. Nâng cấp RAM.
D. Sử dụng hệ thống làm mát bằng chất lỏng.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Smart NIC chứa phần cứng chuyên dụng để xử lý các tác vụ mạng (checksum, encryption). Việc "offload" này giải phóng CPU chính, cho phép CPU tập trung chạy các máy ảo của khách hàng, tối ưu hóa doanh thu.

---

### Câu 5
**Tình huống:** Bạn đang thiết kế hệ thống điện cho một "Pod" mới gồm 20 tủ rack chứa các máy chủ mật độ cao (high-density servers). Bạn lo ngại rằng nếu kích thước Pod quá lớn, một đường dây cấp điện (electrical feed) duy nhất sẽ không đủ tải. Quyết định thiết kế nào dưới đây phù hợp nhất với khuyến nghị về quản lý năng lượng trong Data Center hiện đại?**
A. Giữ nguyên kích thước Pod nhưng yêu cầu các máy chủ chạy chậm lại.
B. Giảm kích thước Pod (số lượng rack trong Pod) để phù hợp với giới hạn công suất của nguồn cấp điện và khả năng làm mát cục bộ.
C. Nối hai đường dây điện vào cùng một ổ cắm.
D. Loại bỏ quạt làm mát để tiết kiệm điện.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Mật độ năng lượng (Power density) tăng cao buộc các nhà thiết kế phải giảm kích thước Pod (số lượng rack) để đảm bảo một Pod không vượt quá khả năng cung cấp của một đường dây điện hoặc hệ thống làm mát.

---

### Câu 6
**Tình huống:** Một sự cố nghiêm trọng xảy ra: Một kỹ thuật viên vô tình làm đứt dây cáp nối đến một Spine switch trong kiến trúc mạng Leaf-Spine. Tuy nhiên, các ứng dụng của khách hàng hầu như không bị gián đoạn, chỉ có băng thông giảm nhẹ. Giao thức/Công nghệ nào trên các Leaf switch đã tự động xử lý tình huống này?**
A. DHCP (Dynamic Host Configuration Protocol).
B. ECMP (Equal Cost Multipath Routing).
C. FTP (File Transfer Protocol).
D. DNS (Domain Name System).

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Trong kiến trúc Leaf-Spine, ECMP chia tải qua nhiều đường dẫn song song. Khi một đường (hoặc một Spine) mất, ECMP tự động nhận biết và phân phối lại lưu lượng sang các đường còn lại (các Spine khác), đảm bảo tính sẵn sàng,.

---

### Câu 7
**Tình huống:** Doanh nghiệp X muốn xây dựng Data Center mới. Giám đốc tài chính phàn nàn rằng việc xây dựng mạng riêng biệt cho lưu trữ (SAN network) bằng cáp quang quá đắt đỏ và phức tạp. Bạn đề xuất xu hướng "Unified Data Center Network". Điều này có nghĩa là gì về mặt triển khai vật lý?**
A. Sử dụng Wifi cho toàn bộ kết nối để tiết kiệm dây.
B. Loại bỏ hoàn toàn mạng lưu trữ và lưu dữ liệu trên USB.
C. Hợp nhất mạng lưu trữ và mạng dữ liệu thông thường chạy chung trên hạ tầng Ethernet tốc độ cao, tận dụng kiến trúc Leaf-Spine.
D. Sử dụng mạng điện thoại để truyền dữ liệu.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Xu hướng hiện đại là sử dụng "Unified Data Center Network". Nhờ Ethernet tốc độ cao và kiến trúc Leaf-Spine (tối ưu cho East-West), ta có thể loại bỏ mạng lưu trữ chuyên dụng tốn kém và chạy tất cả trên một hạ tầng mạng duy nhất,.

---

### Câu 8
**Tình huống:** Một Data Center áp dụng mô hình "Lights-out". Một đêm nọ, hệ thống giám sát báo cáo một máy chủ bị lỗi phần cứng nghiêm trọng. Theo quy trình vận hành của mô hình này, điều gì sẽ xảy ra?**
A. Một robot sẽ tự động thay thế máy chủ ngay lập tức.
B. Hệ thống tự động di chuyển các máy ảo sang máy chủ khác, tắt máy chủ lỗi, và con người chỉ đến thay thế thiết bị vào thời điểm bảo trì định kỳ (hoặc sáng hôm sau), không cần can thiệp khẩn cấp vào ban đêm.
C. Toàn bộ Data Center tự động tắt nguồn để bảo vệ an toàn.
D. Đèn sẽ tự động bật sáng để báo hiệu.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** "Lights-out" nghĩa là vận hành không cần con người hiện diện thường xuyên. Hệ thống tự động hóa sẽ xử lý lỗi (như di chuyển VM) để duy trì dịch vụ. Con người chỉ vào Data Center để thay thế vật lý khi cần thiết, giảm thiểu lỗi do con người và tăng an ninh,.

---

### Câu 9
**Tình huống:** Bạn cần triển khai thêm 50 tủ rack máy chủ trong vòng 2 tuần để đáp ứng một dự án gấp. Việc lắp đặt từng server, đi dây và cấu hình từng switch thủ công là không khả thi. Giải pháp công nghiệp nào được đề cập trong sách giúp giải quyết bài toán "tốc độ triển khai" này?**
A. Thuê thêm 100 nhân viên thời vụ.
B. Mua các "Pre-built Pods" (Pod được lắp ráp sẵn) từ nhà cung cấp, trong đó thiết bị đã được gắn vào rack, đi dây sẵn, chỉ cần vận chuyển đến và cắm điện/mạng.
C. Bỏ qua bước kiểm tra kỹ thuật để tiết kiệm thời gian.
D. Sử dụng máy chủ cũ không cần cài đặt.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Thay vì lắp đặt từng thành phần, Data Center có thể mua các "Pre-built Pods". Các Pod này đã được lắp ráp, đi dây hoàn chỉnh, cho phép triển khai cực nhanh (chỉ cần cắm điện, lạnh, mạng) và mở rộng theo mô-đun,.

---

### Câu 10
**Tình huống:** Một máy chủ trong Data Center cần truy cập dữ liệu từ một ổ đĩa. Trong kiến trúc đám mây hiện đại, thay vì truy cập ổ đĩa gắn trực tiếp (Local Disk), hệ điều hành gửi yêu cầu đọc/ghi qua mạng đến một hệ thống lưu trữ tập trung. Mặc dù đi qua mạng, hệ điều hành vẫn "nhìn thấy" đây là một ổ đĩa vật lý thông thường. Công nghệ này gọi là gì?**
A. FTP Storage.
B. Block Storage (Lưu trữ khối).
C. Object Storage (Lưu trữ đối tượng).
D. Tape Storage (Lưu trữ băng từ).

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Block storage (thường qua SAN) cung cấp giao diện ổ đĩa ảo. Từ góc độ phần mềm trên máy chủ, nó hoạt động y hệt ổ đĩa cục bộ (đọc/ghi theo block), dù thực tế dữ liệu được truyền qua mạng đến thiết bị lưu trữ tập trung,.

