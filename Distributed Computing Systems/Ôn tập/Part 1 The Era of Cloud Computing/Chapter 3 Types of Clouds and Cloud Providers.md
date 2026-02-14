
# Kiến thức
## 1. Phân loại mô hình triển khai (Deployment Models)

###  Private Cloud (Đám mây riêng)
> [!INFO] Định nghĩa
> Hệ thống trung tâm dữ liệu được vận hành và sở hữu **độc quyền** bởi một tổ chức duy nhất. Không chia sẻ tài nguyên với bên ngoài (Single-tenant).

* **Đặc điểm:**
    * Tổ chức có **toàn quyền kiểm soát** (Full Control) về phần cứng, bảo mật, và lịch trình bảo trì.
    * Dữ liệu được lưu trữ cục bộ (On-premise) hoặc tại Colocation center dành riêng.
* **Nhược điểm:**
    * **Gánh nặng vận hành:** Tự lo từ A-Z (Điện, lạnh, phần cứng, nhân sự IT).
    * **Chi phí:** Capex cao (phải mua phần cứng trước), khó scale linh hoạt như Public.

###  Public Cloud (Đám mây công cộng)
> [!INFO] Định nghĩa
> Hạ tầng được sở hữu bởi nhà cung cấp dịch vụ (CSP), bán tài nguyên cho công chúng dưới dạng dịch vụ (Multi-tenant).

* **Đặc điểm:**
    * **No Maintenance:** Khách hàng không cần bảo trì hạ tầng vật lý.
    * **Economies of Scale (Lợi thế quy mô):**
        * *Quy tắc 1/4 chi phí:* Nhà cung cấp mua sắm phần cứng với số lượng khổng lồ -> giá rẻ hơn nhiều so với cá nhân mua lẻ.
        * Hiệu quả năng lượng và quản lý tốt hơn.
* **Nhược điểm:**
    * **Provider Lock-in (Sự ràng buộc):** Khó chuyển đổi sang nhà cung cấp khác do phụ thuộc công nghệ độc quyền.
    * **Compliance:** Phải tuân thủ chính sách của nhà cung cấp.

---

## 2. Mô hình lai & Chiến lược đa nền tảng

### 🔗 Hybrid Cloud (Đám mây lai)
> **Công thức:** Hybrid = Private Cloud + Public Cloud

* **Mục tiêu:** Giải quyết bài toán "trung dung" giữa Bảo mật và Chi phí.
* **Use Case điển hình:**
    * *Dữ liệu nhạy cảm (Tài chính, KH):* Lưu ở **Private Cloud** (An toàn, tuân thủ luật).
    * *Web server/Ứng dụng public:* Chạy trên **Public Cloud** (Tận dụng khả năng scale).
    * *Cloud Bursting:* Khi Private quá tải, đẩy phần dư thừa sang Public.

### Multi-Cloud (Đa đám mây)
> **Triết lý:** "Không bỏ tất cả trứng vào một giỏ" (Don't put all eggs in one basket).

* **Định nghĩa:** Sử dụng dịch vụ của **nhiều nhà cung cấp Public Cloud khác nhau** cùng lúc (Ví dụ: Dùng AWS cho Computing nhưng dùng Google cho AI/Data).
* **Lợi ích:**
    * Giảm thiểu rủi ro **Vendor Lock-in**.
    * Tăng độ sẵn sàng (Nếu AWS sập, vẫn còn Azure).
    * Tối ưu chi phí (Chọn dịch vụ rẻ nhất của từng bên).

---

## 3. Các gã khổng lồ (Hyperscalers)
*Các đơn vị vận hành Data Center quy mô siêu lớn, thống trị thị trường.*

| Nhà cung cấp           | Năm ra mắt | Đặc điểm nổi bật                                                                  |
| :--------------------- | :--------- | :-------------------------------------------------------------------------------- |
| **AWS (Amazon)**       | **2006**   | **Người tiên phong (Pioneer).** Thị phần lớn nhất, danh mục dịch vụ đa dạng nhất. |
| **Microsoft Azure**    | **2010**   | Mạnh về tích hợp với hệ sinh thái Windows/Enterprise.                             |
| **Google Cloud (GCP)** | -          | Đi sau nhưng mạnh về Data, AI, Machine Learning và Container (Kubernetes).        |

---

## 4. Khung quyết định chiến lược (Decision Framework)
Khi lựa chọn mô hình, doanh nghiệp phải cân nhắc sự đánh đổi (Trade-off) giữa 3 yếu tố:

1.  **Kiểm soát (Control):** Private > Hybrid > Public.
2.  **Chi phí (Cost):** Public thường tối ưu hơn nhờ Pay-as-you-go, nhưng Private rẻ hơn nếu tải ổn định và lâu dài.
3.  **Rủi ro (Risk):**
    * *Private:* Rủi ro vận hành (hỏng hóc tự chịu).
    * *Public:* Rủi ro bảo mật (dữ liệu nằm nhà người khác) và Lock-in.
---


# Pool Câu Hỏi Trắc Nghiệm - Cloud Computing Chapter 3
## Chủ đề: Các Loại Hình Đám Mây và Nhà Cung Cấp

### Câu 1
**Trong mô hình Private Cloud (Đám mây riêng), lợi ích chính của việc áp dụng công nghệ đám mây vào trung tâm dữ liệu nội bộ thay vì dùng máy chủ vật lý truyền thống là gì?**
A. Để bán lại tài nguyên cho công chúng.
B. Để tránh tình trạng một số máy chủ bị quá tải (oversubscribed) trong khi các máy chủ khác lại nhàn rỗi (underutilized) bằng cách cân bằng tải trên toàn bộ hạ tầng.
C. Để loại bỏ hoàn toàn bộ phận IT.
D. Để sử dụng kết nối Internet công cộng thay vì mạng nội bộ.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Khi sử dụng Private Cloud, tổ chức có thể tránh được việc tài nguyên bị sử dụng mức thấp hoặc quá tải cục bộ bằng cách phân phối tính toán trên tất cả các máy chủ vật lý thông qua ảo hóa,.

---

### Câu 2
**Yếu tố kỹ thuật nào giúp các nhà cung cấp Public Cloud giảm chi phí thiết bị mạng xuống mức thấp hơn nhiều so với doanh nghiệp tự đầu tư?**
A. Sử dụng cáp mạng giá rẻ.
B. Sử dụng các thiết bị chuyển mạch "whitebox" (phần cứng cơ bản) kết hợp với công nghệ Software Defined Networking (SDN) để điều khiển.
C. Không sử dụng Router.
D. Mua lại thiết bị cũ từ các công ty phá sản.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Thay vì dùng switch đắt tiền, nhà cung cấp dùng "whitebox switches" (chỉ gồm phần cứng chuyển gói cơ bản) và dùng phần mềm SDN để cấu hình và điều khiển, giúp giảm chi phí đầu tư thiết bị,.

---

### Câu 3
**Thuật ngữ "AIops" trong bối cảnh các dịch vụ nâng cao của Public Cloud đề cập đến điều gì?**
A. Robot thay thế con người cắm dây mạng.
B. Sử dụng Trí tuệ nhân tạo (AI) và Học máy (ML) để tự động hóa việc vận hành trung tâm dữ liệu (ví dụ: phát hiện bất thường, giám sát lưu lượng).
C. Dịch vụ chatbot chăm sóc khách hàng.
D. Một loại phần cứng máy chủ mới.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** AIops (Artificial Intelligence operations) là việc sử dụng AI/ML để tự động hóa quản lý phần cứng và phần mềm trung tâm dữ liệu, giúp phát hiện các bất thường như lưu lượng sai lệch hoặc gai tải.

---

### Câu 4
**"Vendor Lock-in" (Sự khóa chặt của nhà cung cấp) là thuật ngữ dùng để chỉ chiến lược nào của các nhà cung cấp đám mây?**
A. Khóa cửa trung tâm dữ liệu để đảm bảo an ninh.
B. Sử dụng các ưu đãi (dịch vụ độc quyền, miễn phí một số tính năng) và rào cản kỹ thuật để làm cho việc khách hàng chuyển sang nhà cung cấp khác trở nên khó khăn hoặc tốn kém.
C. Cam kết không tăng giá trong 10 năm.
D. Yêu cầu khách hàng ký hợp đồng giấy.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Lock-in đề cập đến các biện pháp khuyến khích và trở ngại mà nhà cung cấp sử dụng để ngăn cản khách hàng chuyển sang nhà cung cấp khác (ví dụ: dịch vụ chỉ chạy trên hạ tầng của họ),.

---

### Câu 5
**Đâu là một lợi thế quan trọng của Private Cloud (On-premises) so với Public Cloud đối với các ứng dụng nhạy cảm về độ trễ?**
A. Private Cloud luôn có phần cứng mạnh hơn.
B. Private Cloud đặt tại chỗ (on-premises) giúp giảm độ trễ mạng do khoảng cách địa lý ngắn hơn so với việc kết nối đến Public Cloud.
C. Private Cloud không cần kết nối mạng.
D. Private Cloud sử dụng cáp quang tốc độ ánh sáng.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Vì nằm trong khuôn viên công ty (on-premises), độ trễ giữa nhân viên và trung tâm dữ liệu Private Cloud thấp hơn đáng kể so với độ trễ đến Public Cloud, đặc biệt nếu Public Cloud ở xa về mặt địa lý.

---

### Câu 6
**Mô hình Hybrid Cloud (Đám mây lai) giải quyết vấn đề "Computational overflow" (Tràn tải tính toán) như thế nào?**
A. Tự động mua thêm máy chủ vật lý khi quá tải.
B. Sử dụng Private Cloud cho nhu cầu cơ bản và đẩy phần xử lý dư thừa sang Public Cloud trong các đợt cao điểm.
C. Ngừng phục vụ khách hàng khi quá tải.
D. Xóa bớt dữ liệu cũ để giải phóng chỗ trống.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Trong đợt cao điểm, nếu Private Cloud không đủ tài nguyên, tổ chức có thể gửi phần tính toán tràn (overflow) sang Public Cloud thay vì phải mua thêm máy chủ chỉ để phục vụ cao điểm.

---

### Câu 7
**Mục tiêu chính của chiến lược "Multi-cloud" (Đa đám mây) mà các doanh nghiệp lớn áp dụng là gì?**
A. Để nhận được nhiều quà tặng khuyến mãi từ các nhà cung cấp.
B. Để tận dụng các dịch vụ Public Cloud trong khi tránh bị phụ thuộc vào một nhà cung cấp duy nhất (tránh Lock-in).
C. Để tăng độ phức tạp cho bộ phận IT.
D. Để kết hợp giữa Private và Public Cloud.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Các tổ chức lớn cố gắng tránh phụ thuộc vào một nhà cung cấp duy nhất (Lock-in). Multi-cloud nghĩa là trở thành khách hàng của nhiều hơn một nhà cung cấp Public Cloud.

---

### Câu 8
**Thuật ngữ "Hyperscalers" dùng để chỉ nhóm đối tượng nào?**
A. Các công ty khởi nghiệp về công nghệ đám mây.
B. Các thiết bị cân bằng tải siêu tốc.
C. Các công ty sở hữu và vận hành các cơ sở hạ tầng đám mây quy mô khổng lồ (như AWS, Azure, Google Cloud).
D. Các phần mềm mở rộng quy mô tự động.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Hyperscalers là thuật ngữ chỉ các công ty sở hữu và vận hành các cơ sở vật chất đám mây lớn nhất (dựa trên mức chi tiêu khổng lồ cho hạ tầng dữ liệu), ví dụ như Amazon, Microsoft, Google.

---

### Câu 9
**Tại sao một số tổ chức trong các ngành được kiểm soát chặt chẽ (regulated industries) bắt buộc phải sử dụng Private Cloud hoặc Hybrid Cloud thay vì hoàn toàn dùng Public Cloud?**
A. Vì Public Cloud quá đắt đỏ.
B. Vì quy định pháp lý yêu cầu kiểm soát vị trí lưu trữ, truyền tải dữ liệu và quyền truy cập phần cứng/phần mềm mà Public Cloud không đáp ứng được.
C. Vì Public Cloud không hỗ trợ hệ điều hành Windows.
D. Vì họ không tin tưởng vào Internet.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Đối với các tổ chức trong ngành được quy định, các quy tắc có thể yêu cầu tổ chức phải kiểm soát việc đặt và truyền dữ liệu cũng như phần cứng/phần mềm sử dụng. Private Cloud cho phép tuân thủ các quy tắc này.

---

### Câu 10
**Một thách thức kỹ thuật lớn khi áp dụng chiến lược Multi-cloud là gì?**
A. Không có đủ nhân viên biết sử dụng máy tính.
B. Các dịch vụ có sẵn từ nhà cung cấp này có thể không tương thích hoặc không có sẵn ở nhà cung cấp khác, đòi hỏi phải viết phần mềm chuyển đổi.
C. Chi phí Internet tăng gấp đôi.
D. Các nhà cung cấp từ chối phục vụ khách hàng dùng nhiều dịch vụ.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Việc sử dụng nhiều nhà cung cấp tạo ra thách thức: dịch vụ từ nhà cung cấp này có thể không khớp với nhà cung cấp kia, đòi hỏi tổ chức phải viết phần mềm chuyên biệt để chuyển đổi hoặc kết hợp dữ liệu.
Chào bạn, tiếp tục với **Chương 3: Types Of Clouds And Cloud Providers**, tôi đã soạn thảo bộ câu hỏi trắc nghiệm **nâng cao**.

Các câu hỏi này tập trung vào sự giao thoa giữa **công nghệ** (như SDN, Whitebox switches, AIops) và **chiến lược kinh doanh** (Lock-in, Hyperscale economics), đòi hỏi khả năng phân tích sâu hơn là chỉ ghi nhớ khái niệm đơn thuần.

Dưới đây là nội dung định dạng Markdown để bạn lưu vào Obsidian:

---
# Pool Câu Hỏi Trắc Nghiệm Nâng Cao - Cloud Computing Chapter 3

## Chủ đề: Phân tích Kỹ thuật Hạ tầng, Chiến lược Hyperscale và Vendor Lock-in

### Câu 11
**Các nhà cung cấp Hyperscale đạt được lợi thế chi phí khổng lồ không chỉ nhờ mua sắm số lượng lớn mà còn nhờ thay đổi kiến trúc phần cứng mạng. Sự kết hợp giữa "Whitebox switches" và công nghệ "Software Defined Networking (SDN)" giúp họ giảm chi phí như thế nào?**
A. Whitebox switches có phần cứng đắt tiền hơn nhưng không cần phần mềm quản lý.
B. Họ sử dụng các switch giá rẻ chỉ có phần cứng chuyển mạch cơ bản (bare-metal), loại bỏ chi phí phần mềm tích hợp sẵn đắt đỏ, và đẩy toàn bộ việc điều khiển thông minh lên một máy chủ chạy phần mềm SDN tập trung.
C. SDN cho phép loại bỏ hoàn toàn các switch vật lý và thay thế bằng cáp nối trực tiếp.
D. Whitebox switches tự động sửa lỗi phần cứng mà không cần con người can thiệp.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Thay vì dùng switch thương mại đắt tiền tích hợp sẵn phần mềm quản lý phức tạp, nhà cung cấp dùng "whitebox switches" (chỉ gồm phần cứng chuyển gói cơ bản) và dùng phần mềm SDN chạy trên máy tính riêng biệt để cấu hình và điều khiển, giúp giảm sâu chi phí đầu tư thiết bị,.

---

### Câu 12
**Tại sao các nhà cung cấp đám mây lại tích cực cung cấp các dịch vụ nâng cao (Advanced Services) như AI/ML phân tích dữ liệu, Serverless, hay các công cụ giám sát mạng độc quyền với giá rất rẻ hoặc miễn phí?**
A. Vì các dịch vụ này không tốn tài nguyên để vận hành.
B. Đây là một chiến thuật tạo ra "Vendor Lock-in" mềm: khi khách hàng tích hợp sâu các dịch vụ độc quyền này vào quy trình nghiệp vụ, việc chuyển sang nhà cung cấp khác sẽ đòi hỏi viết lại phần mềm và đào tạo lại nhân viên, gây tốn kém và khó khăn.
C. Vì các dịch vụ này là mã nguồn mở nên nhà cung cấp không được phép thu phí.
D. Để giúp khách hàng dễ dàng di chuyển sang đối thủ cạnh tranh.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Các dịch vụ nâng cao thường liên kết chặt chẽ với hạ tầng của nhà cung cấp. Việc sử dụng chúng tạo ra "Lock-in" (sự khóa chặt), khiến khách hàng khó chuyển đi vì dịch vụ đó không chạy trên hạ tầng của nhà cung cấp khác hoặc đòi hỏi nỗ lực chuyển đổi lớn,.

---

### Câu 13
**Khái niệm "Computational overflow" (Tràn tải tính toán) trong mô hình Hybrid Cloud giải quyết bài toán kinh tế nào cho doanh nghiệp?**
A. Bài toán lưu trữ dữ liệu backup dài hạn giá rẻ.
B. Bài toán tối ưu hóa chi phí đầu tư (Capex) cho các đợt cao điểm ngắn hạn: Doanh nghiệp không cần mua server vật lý chỉ để phục vụ một vài đợt cao điểm trong năm, mà có thể đẩy phần tải dư thừa đó sang Public Cloud và trả tiền theo mức dùng (Opex).
C. Bài toán bảo mật dữ liệu tuyệt đối bằng cách không bao giờ dùng Public Cloud.
D. Bài toán giảm độ trễ mạng xuống mức 0.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Thay vì mua thêm máy chủ vật lý (sẽ bị nhàn rỗi hầu hết thời gian) chỉ để phục vụ đợt cao điểm, tổ chức có thể gửi phần tính toán tràn (overflow) sang Public Cloud. Điều này giúp tối ưu hóa chi phí bằng cách kết hợp Private Cloud cho tải nền và Public Cloud cho tải đỉnh.

---

### Câu 14
**Định nghĩa về "Hyperscale" (Quy mô siêu lớn) đã thay đổi như thế nào từ thời kỳ đầu của Internet sang kỷ nguyên Cloud Computing hiện đại?**
A. Chuyển từ đo lường dung lượng ổ cứng sang đo lường tốc độ CPU.
B. Chuyển từ đo lường số lượng người dùng đồng thời (ví dụ: người dùng Search, Social) sang đo lường mức chi tiêu khổng lồ (hàng tỷ USD) cho hạ tầng máy chủ và thiết bị mạng mỗi năm.
C. Chuyển từ đo lường số lượng nhân viên IT sang đo lường số lượng robot.
D. Định nghĩa không thay đổi, luôn dựa trên số lượng trang web được lưu trữ.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Các định nghĩa ban đầu đo lường số lượng người dùng đồng thời. Tuy nhiên, sự chuyển dịch sang điện toán đám mây đa mục đích đã làm thước đo người dùng trở nên không phù hợp. Định nghĩa hiện tại dựa trên mức chi tiêu cho hạ tầng dữ liệu (ví dụ: Google, Amazon, Microsoft chi hàng chục tỷ USD mỗi năm).

---

### Câu 15
**Thách thức kỹ thuật lớn nhất ngăn cản một doanh nghiệp thực hiện chiến lược "Multi-cloud" (Đa đám mây) một cách trơn tru là gì?**
A. Các nhà cung cấp đám mây chặn kết nối Internet của nhau.
B. Sự không tương thích về API, định dạng dữ liệu và các dịch vụ quản lý giữa các nhà cung cấp, đòi hỏi doanh nghiệp phải viết phần mềm trung gian (middleware) hoặc adapter để chuyển đổi.
C. Thiếu hụt nguồn cung điện năng toàn cầu.
D. Doanh nghiệp không thể có nhiều hơn một tài khoản email.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Các dịch vụ có sẵn từ nhà cung cấp này có thể không tương thích hoặc không có sẵn ở nhà cung cấp khác. Điều này đòi hỏi tổ chức phải viết phần mềm chuyên biệt để chuyển đổi hoặc kết hợp dữ liệu, tạo ra sự phức tạp lớn.

---

### Câu 16
**Trong bối cảnh AIops, công nghệ Machine Learning (Học máy) giúp nhà cung cấp đám mây giải quyết vấn đề "Anomalies" (Bất thường) trong hạ tầng mạng như thế nào?**
A. Bằng cách tự động gửi email xin lỗi khách hàng trước khi sự cố xảy ra.
B. Bằng cách phân tích luồng dữ liệu thời gian thực để phát hiện các mẫu lưu lượng sai lệch (ví dụ: mis-routed traffic) hoặc các gai tải (spikes) không giải thích được mà con người khó phát hiện kịp thời.
C. Bằng cách tăng giá dịch vụ tự động khi mạng chậm.
D. Bằng cách thay thế toàn bộ router bằng phần mềm AI.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** AIops sử dụng ML để giám sát lưu lượng mạng và phát hiện các bất thường như lưu lượng bị định tuyến sai hoặc các đợt tăng tải đột biến không giải thích được, giúp tự động hóa việc vận hành trung tâm dữ liệu.

---

### Câu 17
**Tại sao "Cloud-to-cloud migration services" (Dịch vụ di chuyển từ đám mây này sang đám mây khác) lại là một công cụ cạnh tranh chiến lược của các nhà cung cấp Public Cloud?**
A. Vì nó giúp các nhà cung cấp hợp tác vui vẻ với nhau.
B. Vì nó giúp phá vỡ rào cản gia nhập (entry barrier) đối với khách hàng đang bị "Lock-in" ở đối thủ, lôi kéo họ chuyển sang sử dụng dịch vụ của mình dễ dàng hơn.
C. Vì nó là yêu cầu bắt buộc của luật pháp quốc tế.
D. Vì nó giúp tăng độ trễ mạng để thu thêm phí.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Để lôi kéo khách hàng từ nhà cung cấp khác, các provider cung cấp dịch vụ di chuyển đám mây sang đám mây. Đây là chiến lược để tấn công vào cơ sở khách hàng của đối thủ cạnh tranh bằng cách giảm thiểu khó khăn khi chuyển đổi.

---

### Câu 18
**Xét về độ trễ (latency), lợi thế "On-premises" (Tại chỗ) của Private Cloud so với Public Cloud có ý nghĩa nhất trong trường hợp nào?**
A. Khi doanh nghiệp chỉ cần gửi email hàng ngày.
B. Khi doanh nghiệp có nhiều chi nhánh nhỏ rải rác toàn cầu truy cập vào một server duy nhất.
C. Khi doanh nghiệp cần xử lý các tác vụ thời gian thực (real-time) hoặc tương tác liên tục giữa nhân viên và máy chủ nằm ngay trong khuôn viên công ty, tránh được độ trễ đường truyền Internet đường dài.
D. Khi doanh nghiệp muốn tiết kiệm tiền điện.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Độ trễ giữa nhân viên và trung tâm dữ liệu Private Cloud (nằm trong khuôn viên - on-premises) thấp hơn đáng kể so với độ trễ đến Public Cloud, đặc biệt nếu Public Cloud ở xa về mặt địa lý. Điều này quan trọng cho các ứng dụng nhạy cảm về độ trễ.
