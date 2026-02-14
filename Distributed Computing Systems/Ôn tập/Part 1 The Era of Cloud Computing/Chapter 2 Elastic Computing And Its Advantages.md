#Status: Doing 
#Link: https://www.youtube.com/watch?v=FFr5eEQub7s
# Kiến thức
## 1. Khái niệm cốt lõi: Tính co giãn (Elasticity)
> [!INFO] Định nghĩa
> **Elasticity (Tính co giãn):** Khả năng hệ thống tự động mở rộng (scale out) hoặc thu hẹp (scale in) tài nguyên dựa trên nhu cầu thực tế tại thời điểm đó.

* **Phép ẩn dụ "Vòi nước" (Utility Computing):**
    * Điện toán đám mây giống như điện, nước.
    * Cắm vào là dùng, không cần quan tâm nhà máy điện vận hành ra sao.
    * **Pay-as-you-go:** Trả tiền cho những gì thực sự sử dụng (dùng bao nhiêu khối nước trả bấy nhiêu tiền).
* **Sự thay đổi mô hình:**
    * *Mô hình cũ (On-premise):* Mua server vật lý -> Cài đặt -> Tốn kém Capex & Thời gian (Time-to-market lâu).
    * *Mô hình Cloud:* Thuê server -> Có ngay trong vài phút -> Chuyển sang Opex.

---

## 2. Công nghệ nền tảng: Ảo hóa (Virtualization)
> [!QUESTION] Tại sao Ảo hóa là "Enabler" (kẻ kiến tạo) của Cloud?
> Nếu không có ảo hóa, Cloud chỉ là một Data Center cho thuê chỗ đặt máy (Colocation).

### Cơ chế hoạt động
* **Software-defined:** Máy chủ ảo (VM) được tạo/xóa/sao chép hoàn toàn bằng phần mềm (thông qua Hypervisor).
* **Decoupling:** Tách rời phần cứng vật lý khỏi hệ điều hành.

### Đặc tính kỹ thuật quan trọng
1.  **Consolidation (Hợp nhất):** Chạy nhiều máy chủ ảo trên một máy chủ vật lý.
2.  **Isolation (Sự cô lập):** Các VM chạy chung phần cứng nhưng lỗi ở VM này không ảnh hưởng VM kia (về mặt bảo mật và hiệu năng).
3.  **Rapid Provisioning:** Triển khai nhanh để phục vụ nhu cầu testing, scaling.

---

## 3. Các mô hình dịch vụ (Cloud Service Models)
*Mô hình SPI (SaaS - PaaS - IaaS)*

| Mô hình  | Tên đầy đủ                  | Định nghĩa & Trách nhiệm                                                                                                                              | Ví dụ thực tế                                    |
| :------- | :-------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------- |
| **IaaS** | Infrastructure as a Service | **Nhà cung cấp:** Lo phần cứng (Server, Storage, Net).<br>**Người dùng:** Quản lý OS, Middleware, App, Data.<br>*👉 Thuê "xác máy".*                  | AWS EC2, Google Compute Engine (GCE)             |
| **PaaS** | Platform as a Service       | **Nhà cung cấp:** Lo phần cứng + Môi trường chạy (Runtime, OS).<br>**Người dùng:** Chỉ lo code và deploy ứng dụng.<br>*👉 Thuê "nền tảng lập trình".* | Google App Engine, Heroku, AWS Elastic Beanstalk |
| **SaaS** | Software as a Service       | **Nhà cung cấp:** Quản lý tất cả (Full stack).<br>**Người dùng:** Chỉ việc đăng nhập và dùng.<br>*👉 Thuê "phần mềm thành phẩm".*                     | Google Docs, Spotify, Office 365, Zoom           |

---

## 4. Tương lai: Desktop as a Service (DaaS)
> [!ABSTRACT] 
> Ranh giới thiết bị mờ nhạt
> Khi mạng đủ nhanh, máy tính cá nhân không cần CPU/RAM mạnh nữa.

* **Khái niệm:** Toàn bộ hệ điều hành và ứng dụng chạy trên Cloud.
* **Thin Client:** Thiết bị người dùng chỉ đóng vai trò là cổng hiển thị (Input/Output). Mọi xử lý tính toán nằm ở Data Center.
* **Lợi ích:**
    * Dữ liệu an toàn (không nằm trên laptop dễ mất).
    * Truy cập Desktop của mình từ bất kỳ đâu.
* **Thách thức:** Cần đường truyền Internet cực tốt (Latency thấp) để trải nghiệm mượt mà.
---


# Câu hỏi trắc nghiệm

## Chủ đề: Tính toán Đàn hồi và Các Mô hình Dịch vụ

### Câu 1
**Khái niệm "Elastic Computing" (Tính toán đàn hồi) trong điện toán đám mây chủ yếu đề cập đến khả năng gì?**
A. Khả năng máy chủ tự động sửa chữa phần cứng khi bị hỏng.
B. Khả năng khách hàng có thể tăng hoặc giảm số lượng máy chủ thuê bất cứ lúc nào và chỉ trả tiền cho những gì họ sử dụng.
C. Khả năng kéo dài tuổi thọ của các thiết bị phần cứng.
D. Khả năng sử dụng năng lượng mặt trời để chạy trung tâm dữ liệu.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Một khía cạnh cơ bản của điện toán đám mây là khả năng khách hàng thuê máy chủ và chỉ trả tiền cho số lượng họ cần. Quan trọng hơn, khách hàng có thể thay đổi sự phân bổ một cách linh hoạt (dynamically), thêm máy chủ vào giờ cao điểm và giảm bớt khi không cần thiết,.

---

### Câu 2
**Thuật ngữ "Multi-tenant" (Đa thuê bao) trong bối cảnh Data Center có nghĩa là gì?**
A. Một khách hàng thuê nhiều trung tâm dữ liệu khác nhau.
B. Một trung tâm dữ liệu phục vụ khách hàng từ nhiều tổ chức khác nhau trên cùng một hạ tầng chia sẻ.
C. Nhiều nhân viên IT cùng quản lý một máy chủ vật lý.
D. Một ứng dụng chạy trên nhiều hệ điều hành khác nhau.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Thay vì xử lý tính toán cho một tổ chức duy nhất, nhà cung cấp đám mây xây dựng trung tâm dữ liệu để xử lý tính toán cho nhiều khách hàng. Thuật ngữ "multi-tenant" dùng để chỉ một trung tâm dữ liệu phục vụ khách hàng từ nhiều tổ chức,.

---

### Câu 3
**Công nghệ nào là chìa khóa cho phép nhà cung cấp đám mây tạo và hủy máy chủ nhanh chóng (trong vài mili-giây) để đáp ứng nhu cầu đàn hồi?**
A. Lắp đặt robot vật lý để cắm dây máy chủ.
B. Sử dụng máy chủ ảo hóa (Virtualized Servers).
C. Sử dụng ổ cứng SSD tốc độ cao.
D. Tăng băng thông mạng Internet.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Thay vì cấp phát máy chủ vật lý, nhà cung cấp chạy phần mềm cho phép tạo ra các máy chủ ảo hóa. Việc tạo hoặc loại bỏ một máy chủ ảo hóa được thực hiện hoàn toàn bằng phần mềm, diễn ra cực nhanh mà không cần thay đổi phần cứng vật lý,.

---

### Câu 4
**Trong ảo hóa, thuật ngữ "Logical Isolation" (Cô lập logic) mang ý nghĩa gì?**
A. Các máy ảo được đặt trong các phòng cách ly vật lý khác nhau.
B. Mặc dù nhiều máy ảo chạy trên cùng một máy chủ vật lý, dữ liệu và tính toán của máy ảo này không thể bị nhìn thấy hoặc ảnh hưởng bởi máy ảo khác.
C. Máy ảo không được kết nối với Internet.
D. Mỗi máy ảo phải chạy một hệ điều hành giống hệt nhau.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Mặc dù nhiều máy chủ ảo hóa có thể chạy trên cùng một máy chủ vật lý, mỗi máy chủ ảo hóa hoàn toàn bị cô lập với các máy khác. Dữ liệu và tính toán của một khách hàng không thể bị quan sát hoặc ảnh hưởng bởi người khác.

---

### Câu 5
**Mô hình dịch vụ nào cung cấp các tài nguyên vật lý như tòa nhà, điện, điều hòa, máy chủ và thiết bị mạng, nhưng khách hàng phải tự quản lý hệ điều hành và ứng dụng?**
A. Software as a Service (SaaS).
B. Platform as a Service (PaaS).
C. Infrastructure as a Service (IaaS).
D. Desktop as a Service (DaaS).

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Trong trường hợp đơn giản nhất, công ty IaaS cung cấp các tài nguyên vật lý và máy chủ, thiết bị mạng. Khách hàng IaaS không cần quản lý hạ tầng đám mây nhưng có thể chọn hệ điều hành và ứng dụng để chạy.

---

### Câu 6
**Mô hình dịch vụ nào cung cấp môi trường để khách hàng xây dựng và triển khai phần mềm (bao gồm trình biên dịch, thư viện, runtime) mà không cần quan tâm đến việc cấu hình máy chủ hay hệ điều hành bên dưới?**
A. IaaS.
B. PaaS.
C. SaaS.
D. DaaS.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Mục tiêu chính của PaaS là cơ sở cho phép khách hàng xây dựng và triển khai phần mềm trong đám mây mà không tốn công sức cấu hình hay quản lý hạ tầng bên dưới (như server, OS).

---

### Câu 7
**Office 365, Google Docs, hay các ứng dụng chạy trên trình duyệt mà người dùng trả phí thuê bao hàng tháng thuộc mô hình dịch vụ nào?**
A. IaaS.
B. PaaS.
C. SaaS.
D. FaaS.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Software as a Service (SaaS) đề cập đến mô hình thuê bao trong đó khách hàng trả phí hàng tháng để sử dụng phần mềm thay vì mua một lần. Ứng dụng chạy trên máy chủ đám mây (ví dụ: Office 365).

---

### Câu 8
**Lợi ích "Guaranteed synchronization" (Đồng bộ hóa được đảm bảo) của mô hình SaaS nghĩa là gì?**
A. Tất cả người dùng đều truy cập cùng một lúc.
B. Dữ liệu được sao lưu ra băng từ hàng ngày.
C. Chỉ có một bản sao duy nhất của mỗi tệp tin nằm trên đám mây, nên người dùng truy cập từ bất kỳ thiết bị nào cũng thấy dữ liệu mới nhất.
D. Tốc độ mạng luôn được đảm bảo ổn định.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Với SaaS, chỉ có một bản sao của mỗi tệp tồn tại trên đám mây. Mọi thay đổi đều được áp dụng cho bản sao này, do đó tệp tin sẽ luôn đồng bộ khi người dùng truy cập từ thiết bị khác, không bao giờ bị tình trạng "out of sync",.

---

### Câu 9
**Desktop as a Service (DaaS) khác biệt so với SaaS thông thường ở điểm nào?**
A. DaaS chỉ dành cho lập trình viên.
B. DaaS cung cấp toàn bộ trải nghiệm máy tính để bàn (desktop) từ xa, bao gồm hệ điều hành, tệp tin và ứng dụng, thay vì chỉ một ứng dụng đơn lẻ.
C. DaaS chạy chậm hơn SaaS.
D. DaaS yêu cầu người dùng phải có máy tính cấu hình rất mạnh.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Thay vì chỉ cung cấp quyền truy cập vào một ứng dụng, DaaS "vẽ" một màn hình desktop lên thiết bị của người dùng, cho phép họ nhấp vào biểu tượng, chạy ứng dụng và duyệt tệp tin y như đang dùng máy tính cục bộ, nhưng mọi thứ đều chạy trên đám mây.

---

### Câu 10
**Thiết bị truy cập (access device) được sử dụng cho SaaS hoặc DaaS thường được gọi là "Thin Client" vì lý do gì?**
A. Vì thiết bị này rất mỏng và nhẹ.
B. Vì thiết bị này không cần bộ vi xử lý mạnh, bộ nhớ lớn hay lưu trữ nhiều, nó chỉ cần giao diện người dùng và kết nối mạng.
C. Vì thiết bị này rẻ tiền.
D. Vì thiết bị này chỉ chạy được hệ điều hành Linux.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Vì việc xử lý và lưu trữ diễn ra trong trung tâm dữ liệu đám mây, thiết bị dùng để truy cập SaaS không cần bộ vi xử lý mạnh hay bộ nhớ lớn. Ngành công nghiệp sử dụng thuật ngữ "thin client" để mô tả thiết bị như vậy.


---


# Pool Câu Hỏi Thực Tế & Vận Dụng - Cloud Computing Chapter 2

## Chủ đề: Lựa chọn Mô hình Dịch vụ và Kịch bản Kinh doanh

### Câu 1
**Tình huống:** Một công ty khởi nghiệp (Startup) vừa phát triển một ứng dụng web mới. Họ dự đoán lưu lượng truy cập sẽ tăng đột biến vào ngày ra mắt nhưng ngân sách rất hạn chế. Họ không muốn đầu tư mua máy chủ vật lý vì rủi ro ứng dụng có thể thất bại. Đặc tính kinh tế nào của điện toán đám mây giải quyết trực tiếp nỗi lo này của họ?**
A. Khả năng truy cập toàn cầu (Global Reach).
B. Chuyển đổi từ chi phí vốn (Capex) sang chi phí vận hành (Opex) với mô hình "Pay-as-you-go".
C. Khả năng ảo hóa phần cứng (Hardware Virtualization).
D. Tính bảo mật cao của các trung tâm dữ liệu.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Đặc điểm quan trọng nhất về mặt kinh tế là mô hình giá: nhà cung cấp chỉ tính phí cho những gì khách hàng thực sự sử dụng. Điều này cho phép Startup tránh được chi phí đầu tư ban đầu lớn (Capex) và chỉ trả tiền theo mức sử dụng thực tế (Opex).

---

### Câu 2
**Tình huống:** Một nhóm lập trình viên muốn triển khai một ứng dụng Python nhanh chóng. Họ không muốn tốn thời gian cài đặt Linux, cấu hình mạng, hay lo lắng về việc vá lỗ hổng bảo mật cho hệ điều hành (OS patching). Họ chỉ muốn "tải code lên và chạy". Mô hình dịch vụ nào phù hợp nhất với nhu cầu này?
A. Infrastructure as a Service (IaaS).
B. Platform as a Service (PaaS).
C. Software as a Service (SaaS).
D. On-premises Data Center.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** PaaS cung cấp môi trường để khách hàng xây dựng và triển khai phần mềm (bao gồm trình biên dịch, thư viện, runtime) mà không cần quan tâm đến việc cấu hình hay quản lý hạ tầng bên dưới như server hay hệ điều hành.

---

### Câu 3
**Tình huống:** Một ngân hàng lớn cần chuyển đổi hệ thống lõi (Core Banking) cũ lên đám mây. Hệ thống này yêu cầu một cấu hình hệ điều hành rất đặc thù và các thiết lập mạng riêng biệt để tuân thủ quy định bảo mật. Họ cần quyền kiểm soát tối đa đối với hệ điều hành và các ứng dụng cài đặt trên đó. Mô hình nào là lựa chọn bắt buộc?
A. SaaS.
B. PaaS.
C. IaaS.
D. DaaS.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Khách hàng IaaS không cần quản lý phần cứng vật lý nhưng có quyền chọn và quản lý hệ điều hành, ứng dụng, và có thể kiểm soát các kết nối mạng (ví dụ: tường lửa). Đây là mô hình trao quyền kiểm soát cao nhất cho khách hàng.

---

### Câu 4
**Tình huống:** Một công ty tài chính lo ngại về việc dữ liệu nhạy cảm của họ bị "lộ" sang đối thủ cạnh tranh khi cả hai cùng thuê máy chủ của một nhà cung cấp Cloud (mô hình Multi-tenant). Công nghệ nào trong Cloud đảm bảo rằng điều này không xảy ra về mặt kỹ thuật?
A. Tường lửa vật lý (Physical Firewalls).
B. Cô lập logic (Logical Isolation) trong ảo hóa.
C. Sử dụng các tòa nhà data center khác nhau.
D. Cam kết pháp lý của nhà cung cấp.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Mặc dù nhiều máy chủ ảo hóa chạy trên cùng một máy chủ vật lý, công nghệ ảo hóa đảm bảo mỗi máy chủ ảo hoàn toàn bị cô lập. Dữ liệu và tính toán của khách hàng này không thể bị quan sát hoặc ảnh hưởng bởi khách hàng khác.

---

### Câu 5
**Tình huống:** Trong đại dịch, một công ty cần cho phép 500 nhân viên làm việc tại nhà bằng máy tính cá nhân (laptop cá nhân cấu hình thấp). Công ty muốn đảm bảo mọi dữ liệu công việc không được lưu trên laptop cá nhân của nhân viên để tránh rò rỉ, nhưng nhân viên vẫn phải có trải nghiệm như đang ngồi tại văn phòng với đầy đủ ứng dụng công ty. Giải pháp nào giải quyết vấn đề này?**
A. Cung cấp laptop mới cho từng nhân viên.
B. Sử dụng Desktop as a Service (DaaS).
C. Yêu cầu nhân viên cài VPN.
D. Sử dụng IaaS để tạo máy chủ file.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** DaaS cung cấp trải nghiệm máy tính để bàn từ xa. Hệ điều hành, ứng dụng và tệp tin đều nằm trên đám mây, không phải trên thiết bị cục bộ. Thiết bị của người dùng chỉ đóng vai trò là "thin client" (thiết bị hiển thị), đảm bảo an toàn dữ liệu,.

---

### Câu 6
**Tình huống:** Một trang thương mại điện tử gặp sự cố: vào giờ cao điểm, server bị quá tải khiến web sập. Vào đêm khuya, server lại rảnh rỗi lãng phí tiền thuê. Tính năng kỹ thuật nào của Cloud giải quyết triệt để vấn đề "thừa thiếu" tài nguyên này một cách tự động?**
A. High Availability (Tính sẵn sàng cao).
B. Elastic Computing (Tính toán đàn hồi).
C. Data Locality (Định vị dữ liệu).
D. Grid Computing (Tính toán lưới).

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Khía cạnh cơ bản của điện toán đám mây là khả năng thay đổi sự phân bổ tài nguyên một cách linh hoạt (dynamically), thêm máy chủ vào giờ cao điểm và giảm bớt khi không cần thiết. Đây chính là khái niệm Elastic Computing.

---

### Câu 7
**Tình huống:** Bạn đang soạn thảo một tài liệu trên Google Docs (SaaS) bằng laptop tại cơ quan, sau đó bạn đóng laptop và mở tài liệu đó trên điện thoại khi đang đi xe bus. Bạn thấy nội dung vừa gõ đã xuất hiện đầy đủ. Cơ chế nào của SaaS mang lại trải nghiệm này?
A. Cơ chế sao chép dữ liệu liên tục giữa các thiết bị (Continuous Replication).
B. Cơ chế đồng bộ hóa được đảm bảo (Guaranteed synchronization) do chỉ có một bản sao dữ liệu duy nhất trên đám mây.
C. Do tốc độ mạng 5G rất nhanh.
D. Do điện thoại có bộ nhớ cache lớn.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Với SaaS, chỉ có một bản sao của tệp tồn tại trên đám mây. Mọi thay đổi từ bất kỳ thiết bị nào đều áp dụng lên bản sao này, nên người dùng không bao giờ gặp tình trạng dữ liệu không đồng bộ ("out of sync").

---

### Câu 8
**Tình huống:** Một nhà cung cấp Cloud nhận thấy một máy chủ vật lý đang chạy 10 máy ảo (VM) nhưng CPU chỉ hoạt động 20%, trong khi một máy chủ vật lý khác đang quá tải. Nhà cung cấp thực hiện hành động gì để tối ưu hóa mà không làm gián đoạn dịch vụ của khách hàng?
A. Yêu cầu khách hàng tắt bớt VM.
B. Sử dụng ảo hóa để di chuyển (migrate) các VM sang máy chủ khác nhằm cân bằng tải (Load Balancing).
C. Mua thêm máy chủ mới ngay lập tức.
D. Giảm tốc độ CPU của máy chủ quá tải.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Ảo hóa cho phép nhà cung cấp đặt các VM lên các máy chủ vật lý sao cho tối ưu nhất. Họ có thể cân bằng tải trên các máy chủ vật lý để tránh tình trạng một máy quá tải trong khi máy khác nhàn rỗi.

---

### Câu 9
**Câu hỏi vận dụng:** Tại sao các thiết bị dùng để truy cập dịch vụ DaaS hoặc SaaS thường được gọi là "Thin Client"?
A. Vì chúng được thiết kế siêu mỏng và nhẹ để dễ mang theo.
B. Vì chúng có giá thành rất rẻ.
C. Vì chúng không cần bộ vi xử lý mạnh hay bộ nhớ lớn, gánh nặng tính toán đã được chuyển lên đám mây.
D. Vì chúng sử dụng hệ điều hành phiên bản rút gọn.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Vì việc xử lý và lưu trữ diễn ra trong trung tâm dữ liệu đám mây, thiết bị truy cập không cần cấu hình mạnh. Ngành công nghiệp dùng thuật ngữ "thin client" để chỉ các thiết bị này vì chúng phụ thuộc vào server để thực hiện các tác vụ nặng.

---

### Câu 10
**Tình huống:** Doanh nghiệp X muốn sử dụng phần mềm quản lý quan hệ khách hàng (CRM). Họ không muốn tuyển nhân viên IT để cài đặt, bảo trì máy chủ database, hay lo việc backup dữ liệu hàng ngày. Họ chỉ muốn trả phí hàng tháng và nhân viên kinh doanh có thể đăng nhập vào dùng ngay. Họ nên chọn nhà cung cấp dịch vụ loại nào?
A. IaaS (thuê máy chủ ảo Amazon EC2).
B. PaaS (thuê nền tảng Google App Engine).
C. SaaS (thuê phần mềm Salesforce hoặc tương tự).
D. Private Cloud (tự xây dựng server).

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** SaaS là mô hình thuê bao phần mềm. Khách hàng trả phí để sử dụng ứng dụng chạy trên đám mây mà không cần quan tâm đến hạ tầng, cài đặt hay bảo trì. Đây là lựa chọn phù hợp nhất cho nhu cầu "dùng ngay" và giảm gánh nặng IT.


