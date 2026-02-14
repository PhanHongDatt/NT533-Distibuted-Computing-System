
# Kiến thức
---

## 1. Tư duy cốt lõi: Overlay vs. Underlay
Để hiểu VXLAN, bạn bắt buộc phải phân biệt được hai khái niệm này:

* **Underlay Network (Mạng nền):** Là hạ tầng vật lý gồm Switch, Router, dây cáp, chạy giao thức IP bình thường. Nhiệm vụ duy nhất: Chuyển gói tin từ Server A sang Server B.
* **Overlay Network (Mạng phủ):** Là mạng ảo được tạo ra **nằm trên** Underlay. Các máy ảo (VM) cảm thấy như chúng đang nối với nhau bằng một sợi dây trực tiếp, dù thực tế chúng có thể nằm cách xa nhau cả vòng trái đất.
    * *Công nghệ lõi:* **Tunneling (Đường hầm).**

---

## 2. Phân tích sâu: Network Address Translation (NAT)

Trong Cloud, NAT không chỉ đơn giản là "chia sẻ internet" như cục Wifi ở nhà bạn.

### Vấn đề của Cloud (The Problem)
* **Overlapping IP Addresses (Trùng lặp địa chỉ IP):** Khách hàng A muốn dùng dải IP `192.168.1.x`. Khách hàng B cũng muốn dùng `192.168.1.x`.
* Nếu đưa gói tin này xuống Router vật lý, Router sẽ "loạn" vì không biết `192.168.1.1` là của ông A hay ông B.

### Cơ chế NAT trong Cloud
Cloud sử dụng NAT để cô lập và định tuyến giao tiếp:

1.  **Outbound NAT (Source NAT):**
    * Khi VM (IP Private) muốn truy cập Internet.
    * Router ảo thay thế IP nguồn `10.0.0.5` thành IP Public của Cloud Provider.
2.  **Inbound NAT (Destination NAT / Port Forwarding):**
    * Khi từ Internet muốn truy cập vào Web Server trên VM.
    * Cloud Provider nhận yêu cầu ở IP Public, sau đó dịch và chuyển tiếp vào IP Private của VM.

> [!IMPORTANT] NAT vs. Encapsulation
> NAT thay đổi nội dung Header (đổi IP). Còn **VXLAN** (sắp nói dưới đây) thì **không đổi IP** mà "bọc" gói tin lại. Cloud hiện đại dùng VXLAN để giải quyết vấn đề trùng IP nội bộ hiệu quả hơn NAT truyền thống.

---

## 3. Tại sao VLAN thất bại trong Cloud?
Trước khi có VXLAN, người ta dùng VLAN (802.1Q). Nhưng VLAN có 2 giới hạn tử huyệt:

1.  **Giới hạn 4096 ID:** VLAN ID chỉ có 12-bit. Tối đa tạo được 4096 mạng con. Cloud Provider có hàng triệu khách hàng -> **Không đủ.**
2.  **Phụ thuộc Switch vật lý:** Muốn tạo VLAN, phải cấu hình trên switch thật. Điều này đi ngược lại nguyên tắc "Tự động hóa" của Cloud.

👉 **Giải pháp:** VXLAN ra đời.

---

## 4. Deep Dive: VXLAN (Virtual Extensible LAN) 

**VXLAN** là công nghệ **Overlay** tiêu chuẩn của thế giới Cloud hiện nay.

### A. Cấu trúc gói tin VXLAN (The Packet Structure)
Hãy tưởng tượng VXLAN giống như việc gửi thư (Gói tin gốc) bằng cách bỏ nó vào một phong bì lớn hơn (Gói tin UDP).

* **Inner Frame (Gói tin bên trong):** Là gói tin Ethernet gốc mà máy ảo (VM) gửi đi. (VD: VM A gửi cho VM B).
* **VXLAN Header:** Chứa **VNI (VXLAN Network Identifier)**.
    * VNI dài **24-bit**.
    * Khả năng tạo ra **16 triệu** mạng LAN ảo (so với 4096 của VLAN).
* **Outer Header (Gói tin bên ngoài):** Là đầu mục UDP/IP để các Router vật lý biết cách chuyển phong bì này đi trên mạng Underlay.

### B. Cơ chế hoạt động (How it works?)
Mọi phép màu nằm ở **VTEP (VXLAN Tunnel Endpoint)** - thường nằm ngay trên **vSwitch** (Switch ảo) của Server vật lý.

**Kịch bản:** VM-1 (Host A) gửi tin cho VM-2 (Host B). Cả hai đều thuộc VNI 100.

1.  **Gửi (Encapsulation):**
    * VM-1 gửi gói tin Ethernet thông thường.
    * Gói tin đi đến vSwitch. vSwitch (đóng vai trò VTEP) phát hiện: "À, gói này cần đi đến Host B".
    * VTEP **đóng gói** gói tin gốc vào trong gói UDP. Nó dán nhãn `VNI = 100` vào.
    * Nó gửi gói UDP này ra mạng vật lý với địa chỉ đích là Host B.

2.  **Truyền (Transport):**
    * Các Router/Switch vật lý chỉ nhìn thấy gói UDP bên ngoài. Chúng chuyển nó đi như mọi gói tin bình thường khác. Chúng không quan tâm bên trong là gì.

3.  **Nhận (Decapsulation):**
    * Gói tin đến Host B. vSwitch tại Host B (VTEP) nhận lấy.
    * Nó **bóc tách** lớp vỏ UDP và VXLAN Header ra.
    * Nó đọc VNI 100 và biết phải chuyển cho VM-2.
    * VM-2 nhận được gói tin Ethernet gốc y hệt như VM-1 vừa gửi.

> [!NOTE] Ẩn dụ
> **VXLAN giống như đi máy bay:**
> * Bạn (Gói tin gốc) bước vào sân bay (VTEP).
> * Bạn được xếp vào một cái máy bay (Gói UDP VXLAN).
> * Máy bay bay trên trời (Mạng vật lý Underlay).
> * Đến nơi, bạn bước ra khỏi máy bay.
> * Bạn vẫn là bạn, không bị thay đổi gì cả (IP giữ nguyên).

---

## 5. Ứng dụng thực tế (Real-world Applications)

### 1. Multi-tenancy trong Public Cloud (AWS VPC, Google VPC)
* Khi bạn tạo một VPC trên AWS, thực chất AWS đang tạo cho bạn một **VNI** riêng biệt.
* Nhờ VXLAN, gói tin của bạn (Tenant A) và gói tin của đối thủ (Tenant B) có thể chạy chung trên một dây cáp quang mà hoàn toàn không nhìn thấy nhau, kể cả khi hai bạn đặt trùng địa chỉ IP Private.

### 2. Kubernetes Networking (Container Network Interface - CNI)
* Các plugin mạng như **Flannel** hay **Calico** (chế độ IP-in-IP hoặc VXLAN) dùng kỹ thuật này để nối các Pods nằm trên các Node khác nhau.
* Giúp mọi Pod trong cluster Kubernetes giao tiếp được với nhau qua một mạng phẳng (Flat Network) dù hạ tầng vật lý bên dưới rất phức tạp.

### 3. Live Migration qua Layer 3
* Ngày xưa, muốn di chuyển máy ảo (Live Migration), hai máy chủ vật lý phải cùng nằm trong một mạng Layer 2 (cùng Switch).
* Với VXLAN, vì gói tin được đóng gói thành UDP (Layer 3), bạn có thể di chuyển máy ảo từ Server ở Hà Nội vào Server ở Sài Gòn (miễn là có kết nối IP) mà không bị mất kết nối mạng.

---

# Pool Câu Hỏi Cơ Bản & Khái Niệm - Cloud Computing Chapter 7

## Chủ đề: Khái niệm Mạng ảo, VLAN và VXLAN

### Câu 1
**Hai mục tiêu xung đột nhau (conflicting goals) mà một nhà thiết kế mạng trung tâm dữ liệu phải cân bằng là gì?**
A. Tốc độ cao và Chi phí thấp.
B. Kết nối phổ quát (Universal connectivity) và Giao tiếp an toàn, cô lập (Safe, isolated communication).
C. Phần cứng và Phần mềm.
D. Mạng có dây và Mạng không dây.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Một mặt, mạng phải đảm bảo mọi VM/Container có thể giao tiếp với nhau bất kể vị trí (universal connectivity). Mặt khác, mỗi khách hàng (tenant) cần sự đảm bảo rằng hệ thống của họ bị cô lập và an toàn khỏi những người thuê khác,.

---

### Câu 2
**Thuật ngữ nào dùng để chỉ một mạng ảo được tạo ra bằng cách cấu hình các switch để hạn chế giao tiếp (không tồn tại vật lý nhưng hoạt động như một mạng riêng)?**
A. Underlay Network.
B. Physical Network.
C. Overlay Network.
D. Subnet Mask.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** "Overlay network" là thuật ngữ chỉ mạng ảo được tạo ra bằng cách cấu hình switch, nằm trên nền tảng mạng vật lý,.

---

### Câu 3
**Giới hạn lớn nhất của công nghệ VLAN (Virtual Local Area Network) khiến nó không phù hợp cho các trung tâm dữ liệu đám mây quy mô lớn là gì?**
A. Nó làm chậm tốc độ mạng.
B. Nó chỉ hỗ trợ tối đa 4096 mạng ảo (do thẻ VLAN tag chỉ có 12-bit).
C. Nó không hỗ trợ kết nối Internet.
D. Nó yêu cầu cáp quang đắt tiền.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Mỗi thẻ VLAN bao gồm 12 bit, giới hạn công nghệ này ở mức 4096 VLAN. Con số này không đủ cho các trung tâm dữ liệu lớn với hàng nghìn khách hàng.

---

### Câu 4
**Công nghệ nào được thiết kế để mở rộng quy mô của VLAN, cho phép tạo ra hơn 16 triệu mạng ảo bằng cách sử dụng các gói tin Internet (IP packets)?**
A. VPN.
B. VXLAN (Virtual Extensible LAN).
C. HTTP.
D. FTP.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** VXLAN mở rộng công nghệ VLAN lên quy mô đủ cho các trung tâm dữ liệu lớn nhất (hơn 16 triệu mạng ảo) bằng cách sử dụng gói tin Internet để đóng gói.

---

### Câu 5
**Cơ chế "Encapsulation" (Đóng gói) trong VXLAN hoạt động như thế nào?**
A. Nó nén dữ liệu lại để truyền nhanh hơn.
B. Nó đặt toàn bộ gói tin Ethernet (của máy ảo) vào bên trong một gói tin Internet (UDP/IP) để truyền đi trên mạng vật lý.
C. Nó mã hóa dữ liệu để bảo mật.
D. Nó chia nhỏ gói tin thành các bit.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** VXLAN đặt gói tin Ethernet bên trong một gói tin Internet (IP) để chuyển đi. Để gửi gói tin Internet kết quả, nó lại đặt vào trong một gói Ethernet "bên ngoài".

---

### Câu 6
**Để phân phối các gói tin broadcast hoặc multicast hiệu quả đến nhiều VM trong cùng một mạng ảo mà không gây quá tải mạng, VXLAN sử dụng công nghệ gì?**
A. IP Multicast.
B. Flooding.
C. Peer-to-peer.
D. Satellite communication.

> [!success]- Đáp án & Giải thích
> **Đáp án:** A
> **Giải thích:** VXLAN sử dụng công nghệ multicast để phân phối gói tin hiệu quả. Thay vì gửi nhiều bản sao riêng lẻ, nó gửi một bản sao duy nhất qua mạng và mạng sẽ phân phối đến các đích đến.

---

### Câu 7
**Phần mềm nào chạy trên máy chủ (server) đóng vai trò kết nối các VM với mạng vật lý và thực hiện các quy tắc chuyển tiếp gói tin giống như một switch vật lý?**
A. Virtual Disk.
B. Hypervisor Firewall.
C. Virtual Network Switch (ví dụ: Open vSwitch).
D. Load Balancer.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Một phần mềm được gọi là "virtual network switch" cho phép server gán IP riêng cho từng VM và chuyển tiếp gói tin giữa các VM và mạng trung tâm dữ liệu,.

---

### Câu 8
**Công nghệ nào cho phép nhiều Container chia sẻ một địa chỉ IP duy nhất của máy chủ (Host OS) một cách an toàn bằng cách sử dụng các địa chỉ IP riêng (private IP) bên trong?**
A. DHCP.
B. DNS.
C. NAT (Network Address Translation).
D. VPN.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** NAT cho phép nhiều máy tính (hoặc container) chia sẻ một địa chỉ IP công khai. NAT thay thế địa chỉ IP riêng của container bằng IP của host khi gửi tin ra ngoài,.

---

## Chủ đề: Quản lý Mạng, SDN và Programmable Networks

### Câu 9
**Giao thức nào được sử dụng để ngăn chặn hiện tượng lặp vòng (cycles/loops) gói tin trong mạng Ethernet có cấu trúc Leaf-Spine?**
A. HTTP.
B. STP (Spanning Tree Protocol).
C. FTP.
D. SMTP.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Kiến trúc Leaf-Spine có nhiều vòng lặp. STP (Spanning Tree Protocol) chạy khi switch khởi động để phát hiện và thiết lập quy tắc ngăn chặn gói tin lặp vòng mãi mãi.

---

### Câu 10
**"Software Defined Networking" (SDN) tách biệt chức năng điều khiển mạng khỏi thiết bị phần cứng bằng cách sử dụng thành phần phần mềm nào?**
A. SDN Controller.
B. SDN Driver.
C. SDN Monitor.
D. SDN Switch.

> [!success]- Đáp án & Giải thích
> **Đáp án:** A
> **Giải thích:** SDN sử dụng một máy tính chuyên dụng chạy phần mềm gọi là "SDN Controller" để cấu hình và giám sát các switch mạng theo các chính sách cấp cao.

---

### Câu 11
**Giao thức chuẩn nào định nghĩa cách giao tiếp giữa SDN Controller và các thiết bị switch (để cài đặt các quy tắc chuyển tiếp - forwarding rules)?**
A. OpenFlow.
B. OpenStack.
C. OpenSource.
D. OpenConnect.

> [!success]- Đáp án & Giải thích
> **Đáp án:** A
> **Giải thích:** Giao thức OpenFlow là một thành phần chính của công nghệ SDN, định nghĩa dạng và ý nghĩa của các thông điệp truyền giữa controller và switch.

---

### Câu 12
**Trong mô hình SDN thế hệ 1 (sử dụng OpenFlow), điều gì xảy ra khi một switch nhận được một gói tin không khớp với bất kỳ quy tắc nào trong bảng chuyển tiếp (Table miss)?**
A. Switch tự động học và chuyển tiếp.
B. Switch gửi gói tin đó về SDN Controller để xin chỉ thị (Exception handling).
C. Switch xóa gói tin ngay lập tức.
D. Switch khởi động lại.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Hạn chế chính của các quy tắc chuyển tiếp tĩnh là việc xử lý ngoại lệ: một gói tin không khớp với quy tắc nào phải được gửi về controller để xử lý (slow path processing).

---

### Câu 13
**Ngôn ngữ lập trình nào được sử dụng trong SDN thế hệ 2 (Programmable Networks) để nạp chương trình vào trực tiếp trong switch, giúp switch tự xử lý các gói tin?**
A. Java.
B. Python.
C. P4.
D. C++.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Thế hệ thứ hai của SDN sử dụng một ngôn ngữ lập trình đặc biệt tên là P4 để tạo ra mạng lập trình được (programmable network), cho phép cài đặt chương trình vào switch.

---

### Câu 14
**Lợi ích chính của việc sử dụng ngôn ngữ P4 so với OpenFlow truyền thống là gì?**
A. Giảm giá thành phần cứng.
B. Giảm nhu cầu gửi các gói tin ngoại lệ (exceptions) về Controller, do switch có thể tự xử lý cục bộ.
C. Tăng băng thông vật lý của dây cáp.
D. Hỗ trợ Wifi tốt hơn.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Việc sử dụng một chương trình (P4) có nghĩa là switch có thể xử lý các ngoại lệ tại chỗ (locally) mà không cần gửi mỗi ngoại lệ về cho controller.

---

### Câu 15
**Trong ngữ cảnh SDN, thuật ngữ "Slow path processing" ám chỉ điều gì?**
A. Quá trình truyền tin qua cáp đồng.
B. Quá trình gói tin được xử lý bởi phần cứng switch.
C. Quá trình gói tin phải được gửi về SDN Controller để xử lý (do không khớp quy tắc).
D. Quá trình khởi động switch.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Chú thích cuối trang xác định thuật ngữ "slow path processing" dùng để chỉ việc các gói tin được gửi đến controller để xử lý.

---

### Câu 16
**Thuật ngữ "Underlay Network" đề cập đến lớp nào trong kiến trúc mạng ảo hóa?**
A. Lớp phần mềm tạo mạng ảo.
B. Lớp ứng dụng của người dùng.
C. Mạng vật lý cơ sở cung cấp kết nối thực tế.
D. Lớp bảo mật.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Thuật ngữ "underlay network" đề cập đến mạng vật lý cơ bản cung cấp các kết nối mạng thực tế cho các thực thể trong mạng ảo,.

---

### Câu 17
**Ba thành phần chính để tạo nên một SDN Controller bao gồm: Ứng dụng quản lý, Controller app, và...?**
A. Người dùng.
B. Hệ điều hành (thường là Linux) chạy trên máy tính điều khiển.
C. Màn hình hiển thị.
D. Bàn phím.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Cách tiếp cận SDN sử dụng một máy tính chuyên dụng chạy hệ điều hành thông thường (thường là Linux), ứng dụng SDN controller, và ứng dụng quản lý.

---

### Câu 18
**Khi cấu hình switch bằng OpenFlow, các quy tắc chuyển tiếp (forwarding rules) dựa trên thông tin gì của gói tin?**
A. Nội dung file đính kèm.
B. Các trường trong Header của gói tin (ví dụ: IP nguồn/đích, cổng TCP, VLAN tag).
C. Tên người gửi.
D. Thời gian gửi.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Một quy tắc chuyển tiếp sử dụng các mục trong tiêu đề gói tin (packet header) để quyết định nơi gói tin sẽ được gửi đến (ví dụ: VLAN tag, IP source/destination, TCP port),.

---

### Câu 19
**Tại sao việc quản lý mạng trong Cloud Data Center lại phức tạp hơn mạng truyền thống?**
A. Vì dây cáp dài hơn.
B. Vì sự kết hợp của: kết nối phức tạp (Leaf-Spine), nhiều lớp ảo hóa (VLAN/VXLAN), và sự di chuyển tùy ý của các VM/Container.
C. Vì sử dụng nhiều điện hơn.
D. Vì không có nhân viên IT.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Việc cấu hình và vận hành mạng trung tâm dữ liệu đặc biệt khó khăn do các kết nối phức tạp, nhiều cấp độ ảo hóa, vị trí tùy ý của các thực thể, và việc di chuyển VM,.

---

### Câu 20
**Một "Virtual Network" (Mạng ảo) trong đám mây thực chất là gì?**
A. Một mạng lưới cáp ảo vô hình.
B. Một cấu hình trên các switch để hạn chế giao tiếp, tạo ảo giác về một mạng riêng biệt cho khách hàng.
C. Một loại virus máy tính.
D. Một kết nối Wifi.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Nhà cung cấp tưởng tượng ra một tập hợp các mạng độc lập và cấu hình mỗi switch để giữ lưu lượng của mạng này tách biệt với mạng khác. Mạng ảo chỉ đạt được bằng cách hạn chế tập hợp các VM/Container nhận được gói tin.
Chào bạn, tiếp tục với **Chương 7: Virtual Networks**, đây là bộ câu hỏi ở mức độ **vận dụng cao (High Application)** và **xử lý sự cố thực tiễn**.

Các câu hỏi này yêu cầu bạn không chỉ nhớ khái niệm mà phải hiểu sâu về luồng dữ liệu (packet flow), sự tương tác giữa phần cứng/phần mềm và các điểm nghẽn (bottlenecks) trong kiến trúc mạng đám mây.

---

# Pool Câu Hỏi Trắc Nghiệm Vận Dụng Cao & Thực Tiễn - Cloud Computing Chapter 7

## Chủ đề: SDN, VXLAN, Network Overheads và Troubleshooting

### Câu 1
**Tình huống:** Bạn đang quản lý một mạng Data Center sử dụng công nghệ VXLAN để tạo mạng ảo cho các khách hàng (tenants). Mạng vật lý (Underlay) là Ethernet tiêu chuẩn với MTU (Maximum Transmission Unit) mặc định là 1500 bytes. Một khách hàng phàn nàn rằng tốc độ mạng giữa hai máy ảo (VM) của họ rất chậm, mặc dù băng thông vật lý vẫn còn dư thừa. Khi kiểm tra, bạn thấy CPU của các thiết bị đầu cuối VXLAN (VTEP) tăng cao. Nguyên nhân kỹ thuật có khả năng nhất là gì?**
A. Do VXLAN sử dụng giao thức TCP để đảm bảo tin cậy, gây ra độ trễ bắt tay (handshake latency).
B. Do cơ chế đóng gói (encapsulation) của VXLAN thêm header vào gói tin gốc. Nếu gói tin gốc có kích thước 1500 bytes, gói tin VXLAN kết quả sẽ lớn hơn 1500 bytes, buộc phải bị phân mảnh (fragmentation) để truyền qua mạng vật lý, gây tốn tài nguyên CPU để xử lý phân mảnh và lắp ghép.
C. Do VXLAN giới hạn băng thông của mỗi mạng ảo ở mức 10 Mbps để đảm bảo công bằng.
D. Do bảng định tuyến của SDN Controller bị quá tải.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** VXLAN hoạt động bằng cách đặt gói tin Ethernet ban đầu vào bên trong một gói tin Internet (IP/UDP) để truyền đi. Việc này thêm các header "bên ngoài" làm tăng kích thước tổng thể. Nếu không cấu hình Jumbo Frames (tăng MTU) trên mạng vật lý, các gói tin sẽ bị phân mảnh, làm giảm hiệu suất nghiêm trọng do overhead xử lý.

---

### Câu 2
**Tình huống:** Trong kiến trúc SDN thế hệ 1 (sử dụng OpenFlow với các quy tắc chuyển tiếp tĩnh), điều gì sẽ xảy ra khi một switch nhận được một gói tin từ một ứng dụng mới mà tiêu đề (header) của nó **không khớp** với bất kỳ quy tắc nào hiện có trong bảng chuyển tiếp (flow table)? Rủi ro bảo mật/hiệu năng là gì?**
A. Switch sẽ tự động drop gói tin đó để đảm bảo an toàn; rủi ro là ứng dụng mới không hoạt động được.
B. Switch sẽ gửi gói tin đó về SDN Controller để xử lý (Slow path processing). Nếu kẻ tấn công gửi hàng triệu gói tin lạ (DDoS), SDN Controller sẽ bị quá tải và tê liệt khả năng điều khiển toàn bộ mạng.
C. Switch sẽ tự động học địa chỉ MAC và chuyển tiếp như một switch truyền thống; rủi ro là vòng lặp mạng (loop).
D. Switch sẽ sử dụng AI tích hợp để đoán đích đến của gói tin.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Hạn chế chính của các quy tắc chuyển tiếp tĩnh là việc xử lý ngoại lệ: một gói tin không khớp với quy tắc nào phải được gửi về Controller để xử lý. Quy trình này gọi là "slow path processing". Nếu số lượng ngoại lệ quá lớn (ví dụ do tấn công), Controller sẽ trở thành nút cổ chai.

---

### Câu 3
**Tình huống:** Bạn cần triển khai một ứng dụng Container yêu cầu hiệu suất mạng cực cao và muốn loại bỏ hoàn toàn chi phí (overhead) của việc chuyển đổi địa chỉ mạng (NAT). Bạn chấp nhận rủi ro về việc xung đột cổng (port conflict). Giải pháp cấu hình mạng nào cho Container là phù hợp nhất theo lý thuyết trong sách?**
A. Sử dụng chế độ "Bridge Mode" mặc định của Docker.
B. Sử dụng chế độ "Clone Host IP" (hoặc Host networking), cho phép container chia sẻ trực tiếp địa chỉ IP và ngăn xếp mạng của hệ điều hành máy chủ (Host OS).
C. Sử dụng VXLAN để tạo mạng overlay riêng.
D. Gán cho mỗi container một địa chỉ IP công khai riêng biệt từ ISP.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Container có thể "clone" địa chỉ IP của host. Nếu một container sử dụng cùng địa chỉ IP với Host OS, nó bỏ qua lớp NAT. Tuy nhiên, việc sử dụng cùng địa chỉ IP có nghĩa là việc sử dụng mạng của container có thể xung đột với các ứng dụng khác (ví dụ: tranh chấp cổng 80).

---

### Câu 4
**Tình huống:** Để giải quyết vấn đề "Slow path processing" của SDN thế hệ 1, công nghệ SDN thế hệ 2 (Programmable Networks) sử dụng ngôn ngữ lập trình P4 để làm gì?**
A. Để lập trình lại Controller chạy nhanh hơn.
B. Để nạp một chương trình máy tính vào trực tiếp trong switch, giúp switch tự xử lý các gói tin ngoại lệ tại chỗ (locally) mà không cần gửi về Controller.
C. Để mã hóa toàn bộ gói tin nhằm tăng cường bảo mật.
D. Để thay thế hoàn toàn giao thức IP bằng giao thức P4.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Thế hệ thứ hai của SDN sử dụng ngôn ngữ P4 để tạo ra mạng lập trình được. Việc sử dụng một chương trình (P4) có nghĩa là switch có thể xử lý các ngoại lệ tại chỗ (locally) mà không cần gửi mỗi ngoại lệ về cho controller.

---

### Câu 5
**Tình huống:** Một hệ thống VXLAN sử dụng cơ chế Multicast để xử lý các gói tin Broadcast của máy ảo (VM). Nếu một VM gửi một gói tin Broadcast (ví dụ: ARP request), VXLAN sẽ xử lý gói tin này như thế nào để tối ưu hóa băng thông mạng vật lý?**
A. Nó gửi gói tin đó đến SDN Controller, và Controller sẽ gửi unicast đến từng VM khác.
B. Nó sao chép gói tin thành hàng nghìn bản (unicast) và gửi lần lượt đến tất cả các máy chủ khác trong Data Center.
C. Nó đóng gói gói tin đó vào một gói tin Multicast IP duy nhất và gửi vào mạng vật lý. Các switch vật lý sẽ chịu trách nhiệm nhân bản gói tin này đến các nhánh cần thiết.
D. Nó chặn gói tin Broadcast và trả lời giả lập ngay lập tức.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** VXLAN sử dụng công nghệ multicast để phân phối gói tin hiệu quả. Thay vì gửi nhiều bản sao riêng lẻ, nó gửi một bản sao duy nhất qua mạng (dưới dạng gói tin multicast) và mạng sẽ phân phối đến các đích đến.

---

### Câu 6
**Tình huống:** Trong mô hình mạng Overlay/Underlay, khi một VM A (trên Server 1) gửi dữ liệu cho VM B (trên Server 2) thông qua VXLAN. Từ góc độ của một switch vật lý nằm giữa Server 1 và Server 2 (Underlay network), gói tin đang di chuyển có đặc điểm gì?**
A. Switch chỉ nhìn thấy địa chỉ MAC/IP của VM A và VM B.
B. Switch nhìn thấy đây là một gói tin UDP/IP thông thường, với địa chỉ nguồn là IP của Server 1 và địa chỉ đích là IP của Server 2. Nội dung bên trong (Ethernet frame của VM) là dữ liệu (payload) và switch không quan tâm đến nó.
C. Switch nhìn thấy gói tin được mã hóa hoàn toàn và không thể định tuyến được.
D. Switch nhìn thấy thẻ VLAN tag 12-bit của gói tin.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** VXLAN đặt toàn bộ gói tin Ethernet (của VM) vào bên trong một gói tin Internet (IP) để chuyển đi. Để gửi gói tin Internet kết quả, nó lại đặt vào trong một gói Ethernet "bên ngoài". Mạng underlay chỉ xử lý gói tin bên ngoài này dựa trên địa chỉ IP của các máy chủ vật lý (VTEP).

---

### Câu 7
**Tình huống:** Một công ty sử dụng Container và muốn các Container của họ có địa chỉ IP riêng biệt nhưng vẫn có thể giao tiếp ra Internet thông qua địa chỉ IP công khai duy nhất của máy chủ vật lý. Công nghệ nào cho phép thực hiện điều này bằng cách thay thế địa chỉ IP riêng của Container bằng IP của Host khi gói tin đi ra ngoài?**
A. VXLAN.
B. DHCP.
C. NAT (Network Address Translation).
D. DNS.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** NAT cho phép nhiều máy tính (hoặc container) chia sẻ một địa chỉ IP. NAT chặn mỗi gói tin đi ra, thay thế địa chỉ IP riêng bằng địa chỉ IP của host, và thực hiện ngược lại khi nhận phản hồi.

---

### Câu 8
**Tình huống:** Bạn đang thiết kế mạng cho một Cloud Data Center quy mô lớn. Bạn chọn kiến trúc Leaf-Spine để tối ưu hóa lưu lượng Đông-Tây (East-West traffic). Tại sao kiến trúc này lại hỗ trợ tốt cho việc di chuyển máy ảo (VM Migration) và tính linh hoạt của SDN hơn so với kiến trúc phân cấp (Hierarchical) truyền thống?**
A. Vì nó sử dụng cáp quang đắt tiền hơn.
B. Vì nó cung cấp kết nối phổ quát (universal connectivity) với độ trễ thấp và băng thông đồng đều giữa bất kỳ cặp rack nào, cho phép đặt VM ở bất kỳ đâu mà không lo ngại về nút thắt cổ chai ở các tầng trên cùng như mô hình cây (tree).
C. Vì nó loại bỏ hoàn toàn nhu cầu sử dụng switch.
D. Vì nó chỉ hỗ trợ giao thức IPv6.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Mạng phải đảm bảo kết nối phổ quát để nhà cung cấp có quyền tự do đặt VM và container trên bất kỳ máy chủ vật lý nào. Kiến trúc Leaf-Spine (được mô tả trong Ch 4 và liên hệ ở Ch 7) cung cấp nhiều đường dẫn và băng thông cao cho lưu lượng Đông-Tây, hỗ trợ tốt cho việc đặt vị trí tùy ý và di chuyển VM,.
Chào bạn, tôi hiểu rằng 8 câu hỏi có thể chưa đủ để bao quát hết các khía cạnh phức tạp của **Chương 7: Virtual Networks**.

Dưới đây là **10 câu hỏi bổ sung** (từ câu 9 đến câu 18), tiếp tục duy trì mức độ **vận dụng và thực tiễn**, khai thác thêm các khía cạnh về **VLAN/VXLAN, Virtual Switches, và Quy trình tự động hóa mạng** được đề cập trong tài liệu.

---

# Pool Câu Hỏi Bổ Sung - Cloud Computing Chapter 7

## Chủ đề: Virtual Switching, Network Architecture & Automation

### Câu 9
**Tình huống:** Một công ty khởi nghiệp bắt đầu với quy mô nhỏ sử dụng công nghệ VLAN truyền thống để phân chia mạng cho các phòng ban. Khi công ty phát triển thành một nhà cung cấp dịch vụ đám mây (Cloud Provider) với hàng nghìn khách hàng, hệ thống mạng đột ngột không thể tạo thêm mạng riêng ảo mới cho khách hàng thứ 4097. Giới hạn kỹ thuật nào của chuẩn 802.1Q VLAN đã gây ra vấn đề này?**
A. Do băng thông switch không đủ.
B. Do thẻ VLAN (VLAN tag) chỉ có độ dài 12-bit, giới hạn số lượng mạng ảo tối đa là 4096 (2^12).
C. Do dây cáp mạng bị quá tải.
D. Do xung đột địa chỉ IP.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Công nghệ VLAN gán thẻ (tag) cho các gói tin Ethernet. Mỗi thẻ bao gồm một định danh 12-bit, điều này giới hạn công nghệ ở mức tối đa 4096 VLAN. Con số này không đủ cho các trung tâm dữ liệu lớn với hàng nghìn người thuê.

---

### Câu 10
**Tình huống:** Trên một máy chủ vật lý đang chạy 10 máy ảo (VM) thuộc 10 khách hàng khác nhau. Khi VM 1 gửi gói tin cho VM 2 (cùng nằm trên máy chủ này), gói tin đó có cần phải đi ra khỏi máy chủ vật lý tới switch ToR (Top-of-Rack) rồi quay lại không? Và thành phần phần mềm nào chịu trách nhiệm chuyển tiếp gói tin này ngay bên trong máy chủ?**
A. Có, gói tin phải ra switch vật lý; thành phần xử lý là NIC driver.
B. Không, gói tin được chuyển tiếp nội bộ bởi "Virtual Network Switch" (ví dụ: Open vSwitch) chạy trên Hypervisor, giúp cô lập và định tuyến traffic giữa các VM.
C. Không, các VM tự kết nối với nhau qua bộ nhớ chia sẻ (Shared Memory).
D. Có, gói tin phải đi lên Router trung tâm.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Hypervisor sử dụng một phần mềm gọi là "virtual network switch" (như Open vSwitch). Nó kết nối các VM với nhau và với mạng vật lý, cho phép chuyển tiếp gói tin giữa các VM ngay trên server mà không cần ra ngoài, đồng thời đảm bảo quy tắc cô lập.

---

### Câu 11
**Tình huống:** Bạn đang khắc phục sự cố mạng cho một ứng dụng phân tán. Bạn bắt được một gói tin trên dây cáp vật lý nối giữa hai switch. Bạn thấy đây là một gói tin UDP với địa chỉ IP nguồn và đích là của hai máy chủ vật lý. Tuy nhiên, khi giải mã phần dữ liệu (payload) của gói UDP này, bạn lại thấy bên trong chứa một Ethernet frame hoàn chỉnh khác với địa chỉ MAC và IP của các máy ảo. Kỹ thuật này được gọi là gì?**
A. Network Address Translation (NAT).
B. Packet Sniffing.
C. Encapsulation (Đóng gói) - Cơ chế cốt lõi của các giao thức Overlay như VXLAN, GRE.
D. Deep Packet Inspection.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Các công nghệ ảo hóa mạng như GRE, VXLAN sử dụng cơ chế "Encapsulation". Gói tin của mạng ảo được đóng gói bên trong gói tin của mạng vật lý để truyền đi. Gói tin mang một định danh để xác định nó thuộc về mạng ảo nào,.

---

### Câu 12
**Tình huống:** Khi một VM được thực hiện "Live Migration" từ Server A sang Server B nằm ở một Pod khác trong Data Center. Ngoài việc sao chép dữ liệu RAM, hệ thống quản lý mạng tự động (Automated Network Management) phải thực hiện hành động quan trọng nào để đảm bảo VM vẫn nhận được gói tin tại vị trí mới mà không bị gián đoạn kết nối?**
A. Thay đổi địa chỉ IP của VM.
B. Cập nhật lại các quy tắc chuyển tiếp (forwarding rules) trên toàn bộ hệ thống switch (hoặc SDN Controller) để hướng luồng dữ liệu về vị trí vật lý mới (Server B) thay vì Server A.
C. Khởi động lại VM để nhận mạng mới.
D. Yêu cầu người dùng kết nối lại VPN.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Một trong những thách thức của quản lý mạng DC là sự di chuyển của VM. Khi VM di chuyển, hệ thống phải cập nhật thông tin chuyển tiếp qua mạng để cho phép các thực thể khác tiếp tục tiếp cận được VM đó sau khi di chuyển,.

---

### Câu 13
**Tình huống:** Để triển khai công nghệ VXLAN thành công trong Data Center, ngoài việc cấu hình các đầu cuối (VTEP), người quản trị mạng bắt buộc phải kích hoạt tính năng gì trên các thiết bị switch/router của mạng vật lý (Underlay) để hỗ trợ việc tìm kiếm (discovery) và broadcast trong mạng ảo?**
A. IP Multicast.
B. Spanning Tree Protocol (STP).
C. DHCP Snooping.
D. Port Security.

> [!success]- Đáp án & Giải thích
> **Đáp án:** A
> **Giải thích:** VXLAN sử dụng IP multicast để xử lý các gói tin broadcast/multicast của mạng ảo. Do đó, hạ tầng mạng vật lý phải được cấu hình để hỗ trợ định tuyến multicast hoạt động chính xác.

---

### Câu 14
**Tình huống:** Trong quy trình triển khai (deployment workflow) một VM mới được mô tả trong sách, bước nào sau đây liên quan trực tiếp đến việc cấu hình mạng và thường được tự động hóa để tránh lỗi con người?**
A. Lắp đặt thêm dây cáp mạng mới vào server.
B. Cấu hình Hypervisor để gán địa chỉ IP cho VM và cấu hình Virtual Switch để chuyển tiếp gói tin cho VM đó.
C. Mua thêm địa chỉ IP từ nhà cung cấp internet (ISP).
D. Cài đặt trình duyệt web cho VM.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Trong quy trình triển khai mẫu (Hình 9.1 và nội dung liên quan), các bước bao gồm: gán địa chỉ IP cho VM và cấu hình mạng (virtual switch/hypervisor) để chuyển tiếp gói tin. Đây là các bước cần tự động hóa.

---

### Câu 15
**Tình huống:** Một hacker thuê một VM trên cùng một máy chủ vật lý với VM của nạn nhân. Hacker đưa card mạng ảo của mình vào chế độ "Promiscuous mode" nhằm nghe lén (sniff) traffic của nạn nhân. Tại sao hành động này thường thất bại trong môi trường Cloud chuẩn?**
A. Vì hacker không có quyền admin trong VM của hắn.
B. Vì Virtual Switch trên Hypervisor đã được quản lý bởi nhà cung cấp (Cloud Provider) và được cấu hình để cô lập traffic, chỉ chuyển tiếp gói tin đến đúng đích (destination MAC), ngăn chặn việc một VM nhìn thấy traffic của VM khác (trừ broadcast được phép).
C. Vì các VM sử dụng cáp quang riêng biệt.
D. Vì hacker không biết mật khẩu wifi.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Virtual switch bên trong server cô lập các mạng ảo. Virtual switch được quản lý bởi nhà cung cấp đám mây, đảm bảo các quy tắc chuyển tiếp được thực thi nghiêm ngặt để traffic của tenant này không lọt sang tenant kia,.

---

### Câu 16
**Tình huống:** Bạn nghe thuật ngữ "Hyper-converged infrastructure" (Hạ tầng siêu hội tụ) trong thiết kế Data Center hiện đại. Về mặt kiến trúc mạng, điều này có ý nghĩa gì đối với lưu lượng truy cập Storage (Lưu trữ) và Application (Ứng dụng)?**
A. Chúng chạy trên hai hệ thống cáp riêng biệt hoàn toàn để bảo mật.
B. Chúng sử dụng chung một hạ tầng mạng Ethernet tiêu chuẩn (general-purpose networks) thay vì sử dụng mạng SAN (Fibre Channel) riêng biệt như trước đây.
C. Lưu lượng Storage được truyền qua wifi.
D. Ứng dụng không cần lưu trữ dữ liệu.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Các triển khai hiện tại thường sử dụng Ethernet tiêu chuẩn cho cả lưu lượng ứng dụng và lưu lượng lưu trữ. Xu hướng này được gọi là hạ tầng siêu hội tụ (Hyper-converged infrastructure).

---

### Câu 17
**Tình huống:** Một container cần giao tiếp ra Internet để tải bản cập nhật. Container này có địa chỉ IP riêng (Private IP) là 172.17.0.2. Tuy nhiên, máy chủ vật lý chỉ có một địa chỉ IP công khai (Public IP). Cơ chế nào cho phép container này gửi gói tin ra ngoài Internet và nhận phản hồi về đúng tiến trình của nó?**
A. VXLAN.
B. VLAN Tagging.
C. NAT (Network Address Translation) kết hợp với Port Address Translation (PAT).
D. Load Balancing.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** NAT (Network Address Translation) được sử dụng để xử lý việc chuyển tiếp giữa các thực thể có địa chỉ riêng (như container) và mạng bên ngoài. NAT thay thế địa chỉ IP riêng bằng địa chỉ IP của host khi gửi tin ra ngoài và ngược lại.

---

### Câu 18
**Tình huống:** Tại sao việc cấu hình mạng Data Center thủ công (bằng tay) được coi là bất khả thi (impossible) và bắt buộc phải dùng Tự động hóa (Automation)?**
A. Vì con người gõ phím quá chậm.
B. Vì sự kết hợp của: số lượng thiết bị khổng lồ, cấu trúc kết nối phức tạp (như Fat Tree/Leaf-Spine), nhiều lớp ảo hóa chồng chéo (Overlay/Underlay), và tính chất động của VM (tạo/xóa/di chuyển liên tục).
C. Vì các switch không có giao diện dòng lệnh.
D. Vì chi phí thuê nhân viên quá cao.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Cấu hình và vận hành mạng trung tâm dữ liệu đặc biệt khó khăn do các kết nối phức tạp, nhiều cấp độ ảo hóa, vị trí tùy ý của các thực thể, và việc di chuyển VM. Do đó, cần các công cụ tự động hóa để quản lý,.
