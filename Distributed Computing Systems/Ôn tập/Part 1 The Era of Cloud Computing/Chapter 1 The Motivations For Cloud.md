#Status:Doing
#Date: 3/12/2025
#Link: https://www.youtube.com/watch?v=I6uJxU18C7I&list=PLebLulTU6ep3ujl7JJJmVDdrd6kFOfXqy&index=15
>[!Note] Với nội dung của Chapter này chủ yếu giới thiệu về sự phát triển của cloud cũng như sự linh động cũng như lợi ích mà cloud mang lại, chỉ cần review sơ để nắm được hệ thống kiến thức cơ bản trước khi đi vào các chapter sau

# Cơ sở hình thành của điện toán đám mây
* Trong thực tế, theo định luật **Moore's Law** thì kiến trúc hạ tầng của máy tính, cụ thể là số lượng bóng bán dẫn nên gấp đôi sau mỗi 18 tháng -> số lượng bóng bán dẫn trên một đơn vị diện tích qua từng năm sẽ tăng theo cấp số nhân  -> Việc này sẽ dẫn đến tình trạng đến một lúc nào đó thì giới hạn công nghệ (phần cứng) sẽ dẫn tới hệ thống không thể tiếp tục phát triển được nữa, chính vì vậy mà cần phải có giải pháp thay thế để có thể tiếp tục phát triển công nghệ 
* Chính vì thế mà hệ thống chuyển từ đơn core thành đa core -> nhiều lõi hơn  
* Tuy nhiên vấn đề phát triển lúc này chính là việc truy xuất bộ nhớ cực kì phức tạp
* **Giải pháp** : Kết nối nhiều máy tính với nhau thành cụm máy tính -> **Cluster**
* **LoadBalencer**: LoadBalencer có vai trò điều phối giúp cho cụm Cluster có thể hoạt động một cách hiệu quả
* Tuy nhiên lúc này sẽ có một vấn đề mới tiếp tục nảy sinh đó chính là **vấn đề tài chính** 

>[!Success] 
>Thay vì xây dựng hệ thống máy chủ phân tán, tiến hành xây đựng hệ thống **datacenter**
>**DataCenter**: nơi tập hợp hàng trăm ngàn máy chủ, được xếp chồng trên các tủ rack với nhau

## Các định luật được nêu trong hệ tính toán
Dựa trên các tài liệu được cung cấp (đặc biệt là _slideset_01.pdf_, _exercise_sheet_01_solution.pdf_, và _The Cloud Computing Book_), dưới đây là tổng quan về các định luật và nguyên lý nền tảng thúc đẩy sự ra đời và phát triển của điện toán đám mây:

### 1. Định luật Moore (Moore's Law)

Đây là quy tắc nổi tiếng nhất nhưng thường bị hiểu nhầm là một định luật vật lý. Thực tế, đây là một quan sát thực nghiệm của Gordon Moore vào năm 1965.

- **Nội dung:** Số lượng bóng bán dẫn (transistors) trên một mạch tích hợp (integrated circuit) sẽ tăng gấp đôi sau khoảng mỗi **24 tháng** (trước đây thường được trích dẫn là 18 tháng),.
- **Hệ quả:** Sự gia tăng mật độ bóng bán dẫn dẫn đến việc các vi xử lý ngày càng mạnh mẽ hơn với chi phí thấp hơn. Tuy nhiên, nó cũng dẫn đến một giới hạn vật lý gọi là **"Bức tường năng lượng" (Power Wall)**. Việc tăng tốc độ xung nhịp (clock speed) làm nhiệt độ tăng theo cấp số nhân, khiến vi xử lý có thể bị cháy. Do đó, ngành công nghiệp buộc phải chuyển từ việc tăng tốc độ một lõi đơn sang sử dụng **đa lõi (multi-core)** và xử lý song song,,.

### 2. Định luật Amdahl (Amdahl's Law)

Được công bố năm 1967, định luật này tính toán giới hạn lý thuyết về khả năng tăng tốc (SpeedUp) của một chương trình khi sử dụng xử lý song song.

- **Nội dung:** Hiệu suất tăng thêm khi chạy song song bị giới hạn bởi **phần tuần tự (sequential part)** của chương trình,. Một chương trình không bao giờ có thể chạy song song 100% (luôn có phần khởi tạo, cấp phát bộ nhớ phải chạy tuần tự).
- **Ví dụ:** Nếu 95% chương trình có thể chạy song song và 5% bắt buộc chạy tuần tự, thì dù bạn có dùng hàng nghìn CPU, tốc độ tăng tối đa cũng không bao giờ vượt quá 20 lần,.
- **Ý nghĩa:** Định luật này đưa ra cái nhìn khá bi quan về khả năng mở rộng hiệu suất, nhưng nó bỏ qua thực tế là khi có nhiều tài nguyên hơn, chúng ta thường giải quyết các bài toán lớn hơn.

### 3. Định luật Gustafson (Gustafson's Law)

Được đưa ra năm 1988 để phản biện lại sự bi quan của định luật Amdahl, phù hợp hơn với bối cảnh dữ liệu lớn (Big Data) và Cloud hiện nay.

- **Nội dung:** Khi kích thước bài toán tăng lên, phần công việc có thể xử lý song song sẽ tăng nhanh hơn phần công việc tuần tự. Do đó, nếu bài toán đủ lớn, chúng ta có thể đạt được hiệu quả song song hóa cao,.
- **Sự khác biệt:** Amdahl giả định kích thước bài toán cố định khi tăng số lượng CPU. Gustafson giả định **kích thước bài toán sẽ tăng** tỷ lệ thuận với số lượng CPU (scaled workload). Khi số lượng CPU tăng lên vô hạn, tốc độ tăng tốc (SpeedUp) sẽ tăng tuyến tính theo số lượng CPU.
- **Ví dụ thực tế:** "Để kéo một chiếc xe to hơn, việc thêm nhiều con bò (scale-out) dễ hơn là nuôi một con bò khổng lồ (scale-up)".

### 4. Nút thắt cổ chai Von Neumann (Von Neumann Bottleneck)

Đây là một giới hạn kiến trúc máy tính ảnh hưởng đến hiệu suất hệ thống.

- **Vấn đề:** Trong kiến trúc Von Neumann, CPU và bộ nhớ (Memory) tách biệt nhau và kết nối qua một đường truyền (Bus). Tốc độ xử lý của CPU tăng nhanh hơn nhiều so với tốc độ truyền dữ liệu của bộ nhớ, khiến CPU phải "chờ" dữ liệu,.
- **Trong môi trường đa xử lý:** Khi thêm nhiều CPU vào một hệ thống chia sẻ bộ nhớ (Shared Memory), sự tranh chấp đường truyền (Bus contention) càng làm trầm trọng thêm nút thắt này, làm giảm hiệu suất tương đối của mỗi CPU thêm vào,.
- **Giải pháp:** Sử dụng bộ nhớ đệm (Cache) để giảm bớt tác động này,.

### Tóm lại

Các định luật này giải thích tại sao ngành công nghiệp chuyển dịch sang **Điện toán Đám mây (Cloud Computing)**:

1. **Moore & Power Wall:** Đã đạt giới hạn về tốc độ đơn nhân, buộc phải dùng nhiều máy tính/CPU rẻ tiền kết hợp lại (Cluster),.
2. **Amdahl & Gustafson:** Cho thấy để tận dụng sức mạnh của cụm máy tính khổng lồ (Cloud), chúng ta cần các bài toán dữ liệu lớn (Big Data) và các giải thuật song song hóa hiệu quả như MapReduce để vượt qua giới hạn của xử lý tuần tự,.

---

# Pool Câu Hỏi Trắc Nghiệm - Cloud Computing Chapter 1

## Chủ đề: Động lực và Sự hình thành Điện toán Đám mây

### Câu 1
**Thuật ngữ "Power Wall" trong công nghiệp sản xuất chip đề cập đến vấn đề gì đã thúc đẩy sự chuyển dịch sang kiến trúc đa lõi (multicore)?**
A. Giới hạn về kích thước vật lý của bóng bán dẫn.
B. Giới hạn về tốc độ xung nhịp (clock speed) do nhiệt độ sinh ra quá cao và tiêu thụ điện năng lớn.
C. Giới hạn về chi phí sản xuất chip quá đắt đỏ.
D. Giới hạn về khả năng lập trình của phần mềm.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Các nhà sản xuất đã chạm tới giới hạn mà tốc độ bộ xử lý không thể tăng thêm (vượt quá vài Gigahertz) mà không sinh ra quá nhiều nhiệt làm cháy chip. Ngành công nghiệp gọi giới hạn tốc độ này là "Power Wall".

---

### Câu 2
**Giải pháp công nghệ nào được đưa ra để tăng sức mạnh tính toán khi gặp phải "Power Wall"?**
A. Tăng kích thước của bộ vi xử lý.
B. Sử dụng nitơ lỏng để làm mát cho mọi máy tính cá nhân.
C. Sử dụng song song (parallelism) với nhiều bản sao của bộ xử lý (cores) trên một chip.
D. Chuyển sang sử dụng máy tính lượng tử.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Để có thêm sức mạnh tính toán mà không cần tăng tốc độ xử lý (tránh quá nhiệt), ngành công nghiệp chuyển sang sử dụng nhiều lõi (multicore) hoạt động ở tốc độ an toàn.

---

### Câu 3
**Kiến trúc Cluster (cụm máy tính) ban đầu được cộng đồng nghiên cứu khoa học sử dụng nhằm mục đích gì?**
A. Để thay thế hoàn toàn mạng Internet.
B. Để tạo ra một siêu máy tính giá rẻ, có thể mở rộng từ việc kết nối nhiều máy tính cá nhân thông thường.
C. Để giảm lượng điện năng tiêu thụ xuống mức thấp nhất.
D. Để lưu trữ dữ liệu người dùng web.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Các nhà khoa học nhận thấy thay vì dùng siêu máy tính đắt tiền, họ có thể kết nối một tập hợp lớn các máy tính cá nhân giá rẻ (commodity computers) để đạt được sức mạnh tính toán tương đương. Đây gọi là kiến trúc Cluster.

---

### Câu 4
**Thiết bị hoặc công nghệ nào là thành phần cơ bản giúp các trang web quy mô lớn xử lý hàng ngàn người dùng cùng lúc bằng cách chia lưu lượng truy cập?**
A. Firewall (Tường lửa).
B. Load Balancer (Bộ cân bằng tải).
C. Switch (Bộ chuyển mạch).
D. Router (Bộ định tuyến).

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Load balancer là thiết bị chia lưu lượng truy cập đến giữa một tập hợp các máy tính chạy máy chủ, cho phép mở rộng quy mô bằng cách thêm máy chủ mới,.

---

### Câu 5
**Trong ngữ cảnh thiết bị trung tâm dữ liệu, đơn vị "1U" dùng để chỉ điều gì?**
A. Một máy chủ có 1 CPU.
B. Một đơn vị không gian chiều cao trong tủ rack (cao khoảng 1.75 inches).
C. Một đơn vị điện năng tiêu thụ.
D. Một người dùng (1 User).

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Máy chủ được thiết kế để xếp chồng lên nhau trong các tủ rack. Một đơn vị không gian (Unit) cao 1.75 inches, viết tắt là 1U.

---

### Câu 6
**Lợi ích kinh tế về chi phí vận hành (Opex) khi hợp nhất các máy chủ vào một trung tâm dữ liệu tập trung là gì?**
A. Giảm chi phí mua phần cứng ban đầu.
B. Giảm chi phí thường xuyên cho nhân viên IT và đào tạo do tập trung đội ngũ quản lý tại một nơi.
C. Giảm thuế doanh nghiệp.
D. Tăng tốc độ mạng nội bộ.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Việc cho phép mỗi phòng ban tự thuê IT dẫn đến sự trùng lặp và chi phí cao. Hợp nhất vào trung tâm dữ liệu giúp giảm chi phí lương và đào tạo nhân viên IT (Operating expenses),.

---

### Câu 7
**Lợi ích kinh tế về chi phí vốn (Capex) khi tổ chức hợp nhất thiết bị vào Data Center là gì?**
A. Mua được máy chủ hiện đại nhất.
B. Có thể đàm phán giảm giá số lượng lớn (quantity discount) nhờ mua sắm tập trung và chuẩn hóa cấu hình.
C. Không cần phải mua máy chủ nữa mà đi thuê hoàn toàn.
D. Tránh được việc hỏng hóc thiết bị.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Khi hợp nhất, tổ chức có thể chọn cấu hình thống nhất và nâng cấp nhiều máy chủ cùng lúc, cho phép đàm phán giảm giá khi mua số lượng lớn (Capital expenses).

---

### Câu 8
**Nguồn gốc của thuật ngữ "In the Cloud" (Trong đám mây) xuất phát từ đâu?**
A. Dữ liệu được truyền qua vệ tinh trên bầu trời.
B. Việc đặt các trung tâm dữ liệu đời đầu gần các thiết bị mạng tại các điểm kết nối Internet (peering points).
C. Do biểu tượng của Internet trên bản vẽ kỹ thuật luôn là hình đám mây.
D. Một chiến thuật marketing của Amazon.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Mặc dù không chính xác về mặt kỹ thuật, cụm từ này nảy sinh vì các trung tâm dữ liệu ban đầu được đặt cùng vị trí với thiết bị mạng tại các điểm trao đổi Internet (peering points) thay vì tòa nhà riêng biệt, do đó các kỹ sư nói nó nằm "trong đám mây",.

---

### Câu 9
**Đặc điểm quan trọng nhất của mô hình giá (pricing model) trong điện toán đám mây mang lại sự linh hoạt cho khách hàng là gì?**
A. Khách hàng phải trả phí thuê bao cố định hàng tháng bất kể mức sử dụng.
B. Khách hàng chỉ trả tiền cho những tài nguyên/cơ sở vật chất mà họ thực sự sử dụng.
C. Giá giảm dần theo thời gian sử dụng.
D. Khách hàng phải đặt cọc trước một khoản tiền lớn.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Một đám mây cung cấp sự linh hoạt (đàn hồi), nhưng khía cạnh quan trọng nhất là mô hình giá: nhà cung cấp chỉ tính phí khách hàng cho các cơ sở vật chất mà họ thực sự sử dụng.

---

### Câu 10
**Xu hướng chuyển dịch sang điện toán đám mây đại diện cho sự thay đổi nào trong mô hình quản lý tính toán?**
A. Tiếp tục xu hướng phi tập trung hóa (Decentralization) có từ thập niên 80.
B. Quay trở lại mô hình tập trung (Centralization) giống như thời kỳ máy tính lớn (mainframe).
C. Chuyển sang mô hình ngang hàng (Peer-to-peer).
D. Loại bỏ hoàn toàn vai trò của phần cứng.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Sau nhiều thập kỷ phi tập trung hóa nhờ máy tính cá nhân giá rẻ, các giới hạn về năng lượng và chi phí IT đã thúc đẩy sự quay trở lại mô hình tập trung vào các trung tâm dữ liệu,.

## Chủ đề: Chi tiết về Hạ tầng, Kinh tế và Sự thay đổi Công nghệ

### Câu 11
**Theo định luật vật lý được đề cập trong chương 1, mối quan hệ giữa điện năng tiêu thụ (và nhiệt lượng tỏa ra) với tốc độ xung nhịp (clock speed) của chip là gì?**
A. Tỉ lệ thuận tuyến tính (tăng tốc độ gấp đôi thì nhiệt tăng gấp đôi).
B. Tỉ lệ nghịch (tăng tốc độ thì nhiệt giảm).
C. Tỉ lệ thuận với bình phương của tốc độ xung nhịp.
D. Không có mối liên hệ nào.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Lượng điện năng tiêu thụ — và do đó là nhiệt độ của chip — tỉ lệ thuận với bình phương của tốc độ xung nhịp (proportional to the square of the clock speed). Do đó, chỉ cần tăng nhỏ tốc độ xung nhịp cũng làm tăng nhiệt độ đáng kể.

---

### Câu 12
**Tại sao các máy chủ trong một tủ rack (rack) thường không được lắp đầy kín tất cả 42 slot (42U)?**
A. Do các máy chủ quá nặng làm sập tủ rack.
B. Do giới hạn về độ dài dây cáp mạng.
C. Để chừa chỗ cho luồng không khí lưu thông giúp tản nhiệt và dành chỗ cho thiết bị mạng (switch).
D. Do nhà sản xuất quy định không được lắp quá 50%.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Trong thực tế, các ràng buộc bổ sung (như switch mạng ở đỉnh rack) và yêu cầu tản nhiệt (để trống một số slot cho luồng khí) khiến tủ rack thường không được lắp đầy máy chủ.

---

### Câu 13
**Một lợi thế kỹ thuật quan trọng của Load Balancer (Bộ cân bằng tải) ngoài việc chia tải là gì?**
A. Nó giúp tăng tốc độ xử lý của từng CPU.
B. Nó đảm bảo tất cả các yêu cầu từ một khách hàng cụ thể luôn đi đến cùng một máy chủ (session stickiness).
C. Nó mã hóa toàn bộ dữ liệu người dùng.
D. Nó thay thế hoàn toàn cơ sở dữ liệu.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Load balancer đảm bảo rằng tất cả giao tiếp từ một khách hàng nhất định sẽ đi đến cùng một máy chủ. Điều này cho phép máy chủ lưu giữ thông tin (retain information) và sử dụng nó cho các yêu cầu sau đó của khách hàng đó.

---

### Câu 14
**Về mặt kỹ thuật, tại sao nói máy chủ trong Data Center nằm "trong đám mây" (in the cloud) là không chính xác?**
A. Vì đám mây là biểu tượng của mạng không dây, còn máy chủ dùng dây cáp.
B. Vì máy chủ thực tế nằm bên ngoài đám mây (Internet) và có các kết nối mạng dẫn đến đám mây.
C. Vì thuật ngữ "đám mây" chỉ dành cho phần mềm, không phải phần cứng.
D. Vì dữ liệu thực tế được lưu trữ dưới lòng đất.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Nói một trung tâm dữ liệu "ở trong đám mây" là không chính xác về mặt kỹ thuật vì các máy chủ không phải là một phần của bản thân Internet. Thay vào đó, máy chủ chỉ là các máy tính kết nối vào Internet và nên được vẽ ở bên ngoài đám mây.

---

### Câu 15
**Thách thức lớn nhất khi sử dụng kiến trúc đa lõi (multicore) so với việc tăng tốc độ một vi xử lý đơn là gì?**
A. Các lõi tiêu thụ quá nhiều điện.
B. Các lõi trên một chip chia sẻ bộ nhớ bên dưới và truy cập I/O, tạo ra nút thắt cổ chai (bottleneck).
C. Hệ điều hành không thể nhận diện được nhiều lõi.
D. Chi phí sản xuất chip đa lõi rẻ hơn quá nhiều làm giảm lợi nhuận.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Mặc dù cung cấp thêm sức mạnh xử lý, nhưng các lõi trên một chip đều chia sẻ quyền truy cập bộ nhớ và I/O bên dưới. Khi số lượng lõi tăng lên, truy cập I/O và bộ nhớ trở thành nút thắt cổ chai.

---

### Câu 16
**Trong bối cảnh chi phí vận hành (Opex), điều gì đã xảy ra khi các máy tính cá nhân giá rẻ trở nên phổ biến trong các phòng ban của doanh nghiệp trước khi có Cloud?**
A. Chi phí giảm mạnh do không cần thuê nhân viên IT.
B. Chi phí tăng cao do mỗi phòng ban tự thuê và đào tạo nhân viên IT riêng, dẫn đến sự trùng lặp và lương nhân sự tăng.
C. Chi phí không thay đổi.
D. Doanh nghiệp ngừng sử dụng máy tính.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Việc cho phép mỗi phòng ban tự thuê IT dẫn đến sự trùng lặp không cần thiết. Trong khi chi phí phần cứng giảm, chi phí thường xuyên cho nhân viên IT lại tăng do nhu cầu nhân lực cao,,.

---

### Câu 17
**Mô hình kinh doanh nào cho phép một Start-up (công ty khởi nghiệp) bắt đầu với chi phí thấp và mở rộng khi phát triển?**
A. Mua một siêu máy tính ngay từ đầu.
B. Thuê hạ tầng Cloud tối thiểu và trả thêm tiền khi lưu lượng truy cập tăng (Incremental growth).
C. Vay ngân hàng để xây dựng Data Center riêng.
D. Yêu cầu người dùng tự trang bị máy chủ.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Một startup nhỏ có thể bắt đầu bằng việc thuê các phương tiện đám mây tối thiểu (ví dụ: đủ để hỗ trợ một trang web cơ bản), và sau đó tăng hợp đồng thuê khi doanh nghiệp phát triển (tăng trưởng từng bước).

---

### Câu 18
**Kích thước tiêu chuẩn của một tủ rack (full-size rack) trong trung tâm dữ liệu là bao nhiêu?**
A. Cao khoảng 2 mét (6.5 feet) và rộng 2 feet.
B. Cao 1 mét và rộng 1 mét.
C. Cao chạm trần nhà và rộng 5 mét.
D. Kích thước tùy ý không có chuẩn.

> [!success]- Đáp án & Giải thích
> **Đáp án:** A
> **Giải thích:** Một tủ rack kích thước đầy đủ cao khoảng 6.5 feet (gần 2 mét), rộng 2 feet và sâu 3.5 feet.

---

### Câu 19
**Ngoài việc xử lý giao tiếp, Cloud còn cung cấp khả năng gì cho các thiết bị IoT (Internet of Things) dùng pin?**
A. Sạc pin không dây qua Internet.
B. Cung cấp khả năng xử lý/tính toán (processing) mà các cảm biến dùng pin không thể tự làm được.
C. Thay thế hoàn toàn cảm biến.
D. Giảm độ chính xác của cảm biến để tiết kiệm điện.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Ngoài việc cung cấp giao tiếp, đám mây cung cấp khả năng xử lý. Lượng xử lý cần thiết cho các tính toán (ví dụ: phân tích độ rung cầu đường) vượt xa khả năng của các cảm biến chạy bằng pin.

---

### Câu 20
**Thuật ngữ "Server" (máy chủ) trong bối cảnh những năm 1990 dùng để chỉ điều gì?**
A. Một phần mềm gián điệp.
B. Một máy tính mạnh mẽ chạy phần mềm để cung cấp dịch vụ nội bộ hoặc bên ngoài (như Web).
C. Một người quản lý phòng máy.
D. Một thiết bị phát wifi.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Để cung cấp các dịch vụ nội bộ và bên ngoài như World Wide Web, một tổ chức chạy phần mềm trên một máy tính mạnh mẽ được gọi là máy chủ (server).

# Pool Câu Hỏi Trắc Nghiệm Nâng Cao - Cloud Computing Chapter 1

## Chủ đề: Phân tích sâu về Kiến trúc, Vật lý và Kinh tế học Đám mây

### Câu 21
**Theo nguyên lý vật lý được đề cập khi giải thích về "Power Wall", nếu một nhà sản xuất chip cố gắng tăng tốc độ xung nhịp (clock speed) lên gấp đôi, thì lượng điện năng tiêu thụ (và nhiệt lượng tỏa ra) sẽ thay đổi như thế nào về mặt lý thuyết?**
A. Tăng lên gấp đôi (2x).
B. Tăng lên gấp bốn lần (4x).
C. Tăng lên gấp tám lần (8x).
D. Không thay đổi, chỉ có kích thước bóng bán dẫn thay đổi.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Sách nêu rõ rằng lượng điện năng tiêu thụ — và do đó là nhiệt độ của chip — tỉ lệ thuận với **bình phương** của tốc độ xung nhịp (proportional to the square of the clock speed). Do đó, tăng gấp đôi tốc độ sẽ làm tăng nhiệt năng lên $2^2 = 4$ lần [1].

---

### Câu 22
**Mặc dù kiến trúc đa lõi (multicore) giúp vượt qua "Power Wall", nhưng yếu tố kiến trúc nào trở thành "nút thắt cổ chai" (bottleneck) ngăn cản việc mở rộng quy mô tùy ý chỉ bằng cách thêm lõi vào một chip duy nhất?**
A. Sự giới hạn của hệ thống làm mát bằng không khí.
B. Chi phí bản quyền phần mềm tính trên mỗi lõi.
C. Việc chia sẻ quyền truy cập vào bộ nhớ (memory) và I/O bên dưới của các lõi.
D. Kích thước vật lý của chip trở nên quá lớn để gắn vào bo mạch chủ.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Mặc dù cung cấp thêm sức mạnh xử lý, các lõi trên một chip đều chia sẻ quyền truy cập bộ nhớ và I/O bên dưới. Khi số lượng lõi tăng lên, việc tranh chấp truy cập I/O và bộ nhớ trở thành nút thắt cổ chai giới hạn khả năng mở rộng [2].

---

### Câu 23
**Trong kiến trúc Web Server Cluster, Load Balancer không chỉ đơn thuần chia đều lưu lượng truy cập. Cơ chế nào của Load Balancer cho phép một máy chủ Web "ghi nhớ" thông tin của khách hàng (ví dụ: giỏ hàng) qua nhiều lần gửi yêu cầu (requests)?**
A. Cơ chế mã hóa toàn bộ dữ liệu (Data Encryption).
B. Cơ chế đảm bảo tất cả giao tiếp từ một khách hàng cụ thể luôn đi đến cùng một máy chủ (Session Stickiness).
C. Cơ chế lưu trữ dữ liệu tập trung tại Load Balancer.
D. Cơ chế phát sóng (Broadcast) yêu cầu đến tất cả máy chủ cùng lúc.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Load balancer đảm bảo rằng tất cả giao tiếp từ một khách hàng nhất định sẽ đi đến cùng một máy chủ. Điều này cho phép máy chủ lưu giữ thông tin (retain information) và sử dụng nó cho các yêu cầu sau đó của khách hàng đó [3].

---

### Câu 24
**Chương 1 mô tả một "nghịch lý kinh tế" xảy ra trong các doanh nghiệp trước khi chuyển sang mô hình Data Center tập trung. Nghịch lý đó là gì?**
A. Giá phần cứng tăng cao nhưng chi phí nhân sự lại giảm.
B. Máy tính cá nhân giá rẻ lại dẫn đến tổng chi phí vận hành (Opex) tăng vọt do chi phí nhân sự IT phi tập trung.
C. Việc mua siêu máy tính đắt tiền lại tiết kiệm điện hơn máy tính cá nhân.
D. Phần mềm trở nên đắt hơn phần cứng gấp nhiều lần.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Mặc dù giá phần cứng giảm (máy tính giá rẻ), nhưng việc cho phép mỗi phòng ban tự mua và thuê IT riêng dẫn đến sự trùng lặp và chi phí nhân sự tăng cao. Một giám đốc điều hành đã nhận xét: "Máy tính giá rẻ đã biến thành một khoản chi phí lớn" [4], [5].

---

### Câu 25
**Về mặt kỹ thuật hạ tầng mạng, lý do kinh tế chính yếu khiến các trung tâm dữ liệu đời đầu (Early data centers) được đặt ngay tại các "Internet peering points" (điểm trao đổi Internet) là gì?**
A. Để tận dụng hệ thống làm mát có sẵn tại các tòa nhà viễn thông.
B. Để giảm thiểu độ trễ xuống mức 0 tuyệt đối.
C. Để giảm thiểu chi phí cho các kết nối dung lượng cao đến các mạng trục Internet (Internet backbones).
D. Để tránh bị đánh thuế bất động sản.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Các trung tâm dữ liệu ban đầu cần kết nối dung lượng cao đến các mạng trục Internet chính. Việc đặt trung tâm dữ liệu gần điểm kết nối (peering point) giúp giảm tối đa chi phí đường truyền này [6].

---

### Câu 26
**Sự khác biệt cơ bản về "mẫu hình tính toán" (computation pattern) giữa kiến trúc Cluster dùng trong nghiên cứu khoa học (Scientific Computing) và kiến trúc Cluster dùng cho Web Sites là gì?**
A. Scientific Cluster dùng Windows, Web Cluster dùng Linux.
B. Scientific Cluster xử lý song song để giải quyết một bài toán lớn, trong khi Web Cluster xử lý đồng thời hàng ngàn yêu cầu độc lập nhỏ.
C. Scientific Cluster không cần kết nối mạng, Web Cluster cần Internet.
D. Scientific Cluster sử dụng siêu máy tính, Web Cluster sử dụng máy tính cá nhân.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Các cụm siêu máy tính khoa học được thiết kế để các máy tính nhỏ làm việc cùng nhau trên một phép tính tại một thời điểm. Ngược lại, một trang web phải được thiết kế để xử lý nhiều yêu cầu độc lập đồng thời (ví dụ: người này mua sắm, người kia xem tin tức) [7], [8].

---

### Câu 27
**Tại sao việc chuyển đổi từ các máy chủ riêng lẻ (nằm rải rác ở các phòng ban) sang một Data Center tập trung lại giúp giảm Chi phí vốn (Capex) thông qua "quantity discount"?**
A. Vì Data Center luôn mua thiết bị cũ giá rẻ.
B. Vì việc tập trung cho phép tổ chức chuẩn hóa cấu hình và nâng cấp hàng loạt (ví dụ: 1/3 số lượng server mỗi năm), tạo lợi thế đàm phán mua số lượng lớn.
C. Vì các nhà cung cấp phần cứng miễn phí vận chuyển cho Data Center.
D. Vì Data Center không cần mua màn hình và bàn phím cho server.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Nếu máy chủ phân tán, mỗi phòng ban mua lẻ tẻ và cấu hình khác nhau. Khi hợp nhất, tổ chức chọn cấu hình thống nhất và mua số lượng lớn (dozens or hundreds) cùng lúc, cho phép đàm phán giảm giá theo số lượng [9], [10].

---

### Câu 28
**Theo quan điểm lịch sử công nghệ trong chương 1, xu hướng điện toán đám mây đại diện cho sự "đảo chiều" (reversal) của xu hướng nào đã diễn ra trong nhiều thập kỷ trước đó?**
A. Xu hướng chuyển từ phần mềm sang phần cứng.
B. Xu hướng phi tập trung hóa (Decentralization) tài nguyên tính toán.
C. Xu hướng tăng giá thành sản xuất chip.
D. Xu hướng sử dụng mã nguồn mở.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Trong nhiều thập kỷ, giá máy tính thấp khuyến khích sự phi tập trung hóa (mỗi người/phòng ban một máy). Đám mây đại diện cho sự quay trở lại mô hình tập trung (Centralization) vào các trung tâm dữ liệu để tối ưu hóa chi phí và năng lượng [11], [12].

