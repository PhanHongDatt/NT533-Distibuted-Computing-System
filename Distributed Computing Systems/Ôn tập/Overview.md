
# Trắc nghiệm
MapReduce: gồm những bước nào.
Mục tiêu của kiến trúc phân tán.
Trong cloud architecuture gồm những thành phần nào?
Hệ thống phân tán gồm những ưu và nhược điểm gì
Trình tự kĩ thuật của mapreduce:có những bước nào, tracking cái job ntn?...
Hadoop: namenode, ...datanode
Trong thực tế có bao nhiêu kiến trúc cloud: private, public, hybrid, multicloud.
Hiểu luồng hoạt dộng của spark. 
Replication hoạt động như nào? mục đích của nó
Kubernetes docker

# Tự luận
Phần tự luận sẽ có 3 câu hỏi với các dạng như sau
**2 dạng chính:**
+ Đầu tiên là câu hỏi dạng đưa tình huống, có thể là đưa một hệ thống hoặc triển khai một hệ thống nào đó và yêu cầu cần phải đề xuất sử dụng công nghệ gì, triển khai như thế nào, phân tích ưu nhược và lý do sử dụng công nghệ đó. Một số công nghệ có thể được đề cập như devops, microservices ....
+ Câu thứ 2 thường sẽ rơi vào dạng so sánh công nghệ, kỹ thuật, so sánh ưu và nhược điểm của công nghệ đó, quá trình triển khai và vận hành, hiệu suất...
+ Một dạng câu hỏi cũng cần lưu Kiến trúc hoạt động của các công nghệ quan trọng như: MapReduce, Docker, Hadoop, Kubernetes, OpenStack...
+ Câu thứ 3 (Optional) Mô tả lại đồ án và đóng góp trong đồ án.
	+ Nên làm theo hướng vẽ lại sơ đồ đồ án, luồng của đồ án, sau đó giải thích vai trò trong đồ án và đóng góp cá nhân trong quá trình thực hiện đồ án và kết quả mang lại
	+ Lưu ý: Câu này nên làm vừa, không nên làm quá dài


# Chi tiết ôn tập
---
## [[Chapter 1 The Motivations For Cloud]]
+ Chapter này không cần ôn quá sâu, chỉ cần đọc sơ để biết được sự phổ biến của cloud cũng như lợi ích của cloud mang lại
## [[Chapter 2 Elastic Computing And Its Advantages]]
Chapter này tập trung vào việc phân tích ưu nhược điểm của Elastic cũng như đề cập đến một số kiến trúc phổ biến và cần lưu ý để phân biệt được
+ IaaS [[Chapter 2 Elastic Computing And Its Advantages#Câu 5]]
+ PaaS (Platform as a Service)[[Chapter 2 Elastic Computing And Its Advantages#Câu 6]]
+ SaaS (Software as a Service) [[Chapter 2 Elastic Computing And Its Advantages#Câu 7]]
+ DaaS (Desktop as a Service):  DaaS cung cấp toàn bộ trải nghiệm máy tính để bàn (desktop) từ xa, bao gồm hệ điều hành, tệp tin và ứng dụng, thay vì chỉ một ứng dụng đơn lẻ. (Hiểu đơn giản là dùng một máy tính để tiến hành truy cập một máy tính trên đám mây sau đó tiến hành thao tác trên máy tính đó)
+ Pay as Services 
+ Elastic Cpmputing [[Chapter 2 Elastic Computing And Its Advantages#Câu 1]]
+ Multi tenant (Đa thuê bao) [[Chapter 2 Elastic Computing And Its Advantages#Câu 2]]
+ Logical Isolation (Cô lập Logic)[[Chapter 2 Elastic Computing And Its Advantages#Câu 4]]
+ **Guaranteed synchronization** (Đồng bộ hóa được đảm bảo): Với SaaS, chỉ có một bản sao của mỗi tệp tồn tại trên đám mây, mọi thay đổi đều được áp dụng cho bản sao này -> cơ chế đồng bộ khi truy cập nhiều thiết bị
+ **Thin Client**:Thiết bị không cần có bộ xử lý mạnh mẽ vì các tác vụ xử lý và lưu trữ được thực hiện trên đám mây, các thin client chỉ đóng vai trò cổng kết nối đến các máy tính thực sự xử lý các tác vụ đó
## [[Chapter 3 Types of Clouds and Cloud Providers]]
Chapter này đề cập rất nhiều đến khai niệm về Cloud, cần phải hiểu được các kiến trúc cloud khác nhau cũng như là các lợi ích cũng như các mô hình cloud khác nhau
+ **Private Cloud**
+ **Public Cloud**
+ **Advatages of Public Cloud**
+ **Advatages of Private Cloud**
+ **Compare**
+ **Hybrid Cloud**
+ **Multi-Cloud**
+ **Hyperscalers**
---
## [[Chapter 4 Data Center Infrastructure and Equipment]]
Chapter này tập trung vào công nghệ của Data Center và kiến trúc, cần nắm được các thuật ngữ xuất hiện trong chapter này
+ **Rack**: Tủ chứa thiết bị (Cao ~6,5 ft rộng ~2ft) (1ft cỡ 30cm)
	+ Unit (U): Đơn vị đo chiều cao tròn tủ  rack. 1U = 1,75 inches. 1 rack = 42 U (Tiêu chuẩn)
+ **POD**
+ 
## [[Chapter 5 Virtual Machines]]
Tập trung vào các khái niệm về công nghệ ảo hóa như:
+ Ảo hóa một phần
+ Ảo hóa toàn phần
+ Ảo hóa dựa trên hệ điều hành
+ Ảo hóa dựa trên phần cứng
+ Ảo hóa thiết bị đầu vào
+ Ảo hóa Containerzation
Phân loại các công nghệ hỏa hóa
	+ Phân loại dựa trên kỹ thuật ảo hóa
		+ Ảo hóa 1 phần (Partical / Para - vitualization)
			+ Chạy nhanh nhưng cần phải sửa OS để thích hợp
		+ Ảo hóa toàn phần (Full vitualization)
			+ Nguyên bản, tương thích tuyệt đối nhưng chạy chậm, cần bẫy để mô phỏng lệnh
	+ Phân loại dựa trên vị trí của Hypervisor
		+ Ảo hóa dựa trên phần cứng ( Hardware-based / Bare-metal) ***Type 1 Hypervisor***, dạng này khả phổ biến với những đại diện như VMware ESXi, Hyper - V, KVM, được dùng cho các Cloud Data Center vì hiệu năng cao và quản lý tài nguyên tốt nhất
		+ Ảo hóa dựa trên hệ điêù hành (Operating System-based/ Hosted) ***Type 2 Hypervisor***, Hypervisor chạy như một ứng dụng phần mềm trên một hệ điều hành thông thường (host OS), được thấy phổ biến qua VMware workstation,V Virtual Box, được dùng cho các máy tính cá nhân. Hiệu năng thấp hơn Type 1 vì phải đi qua một lớp HOST OS
	+ Ảo hóa thành phần
		+ Ảo hóa thiết bị đầu ra đầu vào (Vitual I/O Devices)
## [[Chapter 6 Containers]]
Trong chapter này tập trung vào cấu trúc Container với các kiến thức như sau:
+ So sánh Container với VM
+ **Docker
+ **Component
+ **Ưu điểm của Container**
## [[Chapter 7 Virtual Networks]]
Chủ yếu vào phần Mạng ảo và NAT
## [[Chapter 8 Virtual Storage]]
Phân biệt các khái niệm và so sánh với nhau
+ NAS
+ SAN
+ Lưu trữ ảo
+ Mapping
+ Object
## [[Chapter 9 Automation]]


## [[Chapter 10 Orchestration Automated Replication and Parallelism]]
*Đây là Chapter rất quan trọng, sẽ là Chapter ddongsg góp nhiều vào trong cấu trúc đề thi cuối kì cả phần tự luận và trắc nghiệm nên cần ôn kĩ càng*
Tập trung vào ôn phần Kubernetes và các khái niệm liên quan đến Kubernetes 


## [[Chapter 11 The MapReduce Paradigm]]
*Đây là Chapter rất quan trọng, sẽ là Chapter ddongsg góp nhiều vào trong cấu trúc đề thi cuối kì cả phần tự luận và trắc nghiệm nên cần ôn kĩ càng*
Tập trung vào phần MapReduce, Hadoop, HDFS
Triển khai các hệ thống đó
So sánh 
Ưu nhược
## [[Chapter 12 Microservices]]
Chapter này chủ yếu là nắm được khái niệm và đưa ra được ưu nhược của MicroServices và Monolithic, chú ý thêm Proxy, Deadlock 

## [[Chapter 13 Controller Based Management Software]]
Tìm hiểu về các thuật ngữ trong Chapter:
+ Custom Resources & Custom Controller
+ Loop
+ Kube Controller
+ Mesh
## [[Chapter 14 Serverless Computing and Event Processing]]
*Chapter này đề cập đến công nghệ khá mới hiện nay là Serverless, chính vì thế mà khả năng câu này sẽ rơi vào phần tự luận vì xu thế hiện nay*
+ Công nghệ mới
+ Tính cần thiết
+ So sánh
+ Triển khai
+ Khó khăn
## [[Chapter 15 DevOps]]
*Không phải phần trọng tâm*
Tìm hiểu một số thuật ngữ sau
+ CI
+ CD
+ Sandbox
+ Canary
+ Blue/Green
## [[Chapter 16 Edge Computing and IIOT]]
*Vì hệ thống phân tán có ứng dụng cao trong hệ thống edge computing và IOT nên có khả năng sẽ rơi vào phần tự luận => CẦN TẬP TRUNG*
Cần nắm được công nghệ xu thế IOT và Edge
Khả năng sẽ ra bài toán trong Edge Computing từ đó giải quyết vấn đề

## [[Chapter 17 Cloud Security and Privacy]]
Tìm hiểu các thuật ngữ trong Cloud Security


# Checklist
Dựa trên các tài liệu về Cloud Computing mà bạn đã cung cấp, dưới đây là **checklist ôn tập toàn diện** được phân loại theo các chủ đề chính:

### Phần 1: Khái Niệm Cơ Bản và Động Lực Áp Dụng

| Mục Tiêu Ôn Tập                      | Chi Tiết Cần Ghi Nhớ                                                                                                                                                                                                                                                                      | Nguồn |
| :----------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---- |
| **Định nghĩa Cloud Computing**       | Hiểu rõ định nghĩa: sự chuyển đổi CNTT từ sản phẩm sang dịch vụ. Mô hình ứng dụng hoặc dịch vụ chạy (một phần hoặc toàn bộ) trên hạ tầng của bên thứ ba, người thuê chỉ trả tiền cho những gì họ sử dụng.                                                                                 |       |
| **Các cấp độ tiến hóa**              | Nắm được sự chuyển đổi từ Sản phẩm (Product) sang Dịch vụ (Service).                                                                                                                                                                                                                      |       |
| **Động lực thúc đẩy**                | Hai động lực chính: Sự ổn định hiệu suất đơn luồng dẫn đến chuyển sang tính toán song song, và những thay đổi về chi phí vận hành và bảo trì hạ tầng.                                                                                                                                     |       |
| **Kiểm soát tài nguyên**             | Khả năng người thuê điều chỉnh linh hoạt lượng tài nguyên sử dụng, bao gồm tăng trưởng từng bước và **mở rộng quy mô động (Dynamic scaling)**.                                                                                                                                            |       |
| **Các loại Đám mây (Cloud Types)**   | Phân biệt **Đám mây Riêng (Private cloud)** (sử dụng nội bộ một tổ chức), **Đám mây Công cộng (Public cloud)** (dịch vụ thương mại, nhiều khách hàng), **Đám mây Lai (Hybrid-cloud)** (sử dụng cả riêng và công cộng), và **Đa Đám mây (Multi-cloud)** (sử dụng nhiều đám mây khác nhau). |       |
| **Xu hướng sử dụng**                 | Hầu hết các doanh nghiệp sử dụng ít nhất một đám mây; 78% sử dụng mô hình Lai giữa công cộng và riêng tư.                                                                                                                                                                                 |       |
| **Lợi ích kinh tế của Public Cloud** | **Quy mô kinh tế (Economies of scale)** (mua sắm số lượng lớn, giảm chi phí); **Chuyên môn (Expertise)** (tiếp cận đội ngũ chuyên gia đa lĩnh vực); **Dịch vụ nâng cao (Advanced services)** (giám sát dựa trên AI, cập nhật tự động).                                                    |       |

### Phần 2: Mô Hình Dịch Vụ Đám Mây (XaaS)

| Mục Tiêu Ôn Tập                        | Chi Tiết Cần Ghi Nhớ                                                                                                                                                                                               | Nguồn |
| :------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---- |
| **IaaS (Infrastructure as a Service)** | Cung cấp tài nguyên ảo hóa tương đương tài nguyên vật lý (máy ảo, mạng ảo). Người dùng có **toàn quyền kiểm soát (complete control)** hệ điều hành và ứng dụng. Ví dụ: AWS, GCE, Azure.                            |       |
| **PaaS (Platform as a Service)**       | Cung cấp môi trường để xây dựng và triển khai phần mềm (trình biên dịch, thư viện). Khách hàng tập trung hoàn toàn vào ứng dụng; không kiểm soát các lớp thấp hơn (HĐH cố định). Ví dụ: Google App Engine, Heroku. |       |
| **FaaS (Function as a Service)**       | Là một danh mục con của PaaS. Ứng dụng là các hàm **không trạng thái (stateless)** và **hướng sự kiện (Event-driven)**. Khách hàng chỉ cần cung cấp mã hàm; backend vô hình (**serverless architecture**).         |       |
| **SaaS (Software as a Service)**       | Mô hình thuê bao. Ứng dụng chạy trên trình duyệt hoặc điện thoại (thin clients), dữ liệu và tính toán nặng vẫn nằm trên đám mây. Ví dụ: Office 365, Google Workspace.                                              |       |
| **HaaS (Humans as a Service)**         | Nguyên tắc crowdsourcing; sử dụng trí tuệ con người cho các công việc máy tính khó xử lý. Ví dụ: Amazon Mechanical Turk.                                                                                           |       |
| **DaaS (Desktop as a Service)**        | Còn gọi là Cloud Operating Systems. Ứng dụng và dữ liệu người dùng nằm trên máy chủ, người dùng chỉ cần trình duyệt.                                                                                               |       |

### Phần 3: Công Nghệ Nền Tảng (Ảo Hóa và Container)

| Mục Tiêu Ôn Tập                           | Chi Tiết Cần Ghi Nhớ                                                                                                                                                                                                                | Nguồn |
| :---------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---- |
| **Tính toán Đàn hồi (Elastic Computing)** | Khả năng tăng/giảm tài nguyên thuê theo nhu cầu. Được hỗ trợ bằng cách cấp phát tài nguyên hạt mịn (fine-grained allocations) và phân bổ lại liền mạch.                                                                             |       |
| **Vai trò của Ảo hóa**                    | Là công nghệ hỗ trợ máy ảo (VM) và container, cho phép chia sẻ tài nguyên và **cô lập (isolation)** giữa những người thuê.                                                                                                          |       |
| **Đặc tính Máy ảo**                       | Tạo/hủy nhanh chóng bằng phần mềm; **Chia sẻ vật lý (Physical sharing)** (nhiều VM trên cùng server); **Cô lập logic (Logical isolation)** (VM không tương tác với nhau, dữ liệu an toàn).                                          |       |
| **Các phương pháp Ảo hóa**                | Phân biệt **Mô phỏng phần mềm (Software emulation)**, **Bán ảo hóa (Paravirtualization)** (sửa đổi HĐH khách), và **Ảo hóa hoàn toàn (Full virtualization)** (HĐH chạy không cần sửa đổi, cần hỗ trợ phần cứng).                    |       |
| **Di chuyển Máy ảo (VM Migration)**       | Quá trình chuyển VM sang server khác (dừng - chuyển - khởi động lại). **Di chuyển trực tiếp (Live migration)** giúp VM tiếp tục chạy trong hầu hết quá trình, giảm thiểu gián đoạn.                                                 |       |
| **Container vs. VM**                      | VM cung cấp sự linh hoạt hoàn toàn nhưng có overhead (thời gian khởi động HĐH, cấp phát bộ nhớ tĩnh). Container **nhẹ hơn (more lightweight)**, khởi động/dừng nhanh hơn VM, phù hợp hơn cho các ứng dụng đám mây có tính động cao. |       |

### Phần 4: Hạ Tầng và Trung Tâm Dữ Liệu

|Mục Tiêu Ôn Tập|Chi Tiết Cần Ghi Nhớ|Nguồn|
|:--|:--|:--|
|**Cấu trúc Data Center**|Thiết bị được đặt trong **Racks**. Racks được tổ chức thành **Rows** (có lối đi giữa). Rows được nhóm lại thành **Pods**.||
|**Nguồn điện và Làm mát**|Cần nguồn điện dư thừa, pin và máy phát điện. Sử dụng khu vực chứa nhiệt (**heat containment zones**) (lối đi nóng/lạnh). Có thể dùng **làm mát bằng chất lỏng (Liquid cooling)**.||
|**Mạng Data Center**|Racks có **Top-of-Rack (ToR)** switches. Kiến trúc mạng thường là **Fat Trees** để tổng hợp lưu lượng ở các lớp cao hơn.||
|**Mạng ảo (Virtual Networks)**|Mục tiêu: Kết nối phổ quát và giao tiếp an toàn, cô lập. Công nghệ sử dụng: **VLAN** (gắn thẻ gói Ethernet, 12-bit identifier, 4096 VLANs); **VXLAN** (sử dụng encapsulation để hỗ trợ hàng chục ngàn mạng ảo).||
|**Các loại Lưu trữ**|Phân biệt **Block storage** (VM có ổ đĩa ảo, truy cập qua mạng); **File storage (NAS)** (giao diện tệp, dễ chia sẻ); và **Object storage (S3)** (lưu trữ đối tượng tùy ý, khả năng mở rộng cao, độc lập với HĐH).||
|**Lưu trữ từ xa vs. Cục bộ**|Lưu trữ từ xa được truy cập trên máy chủ khác thông qua mạng. Lưu trữ cục bộ (directly attached storage) được truy cập qua bus (ví dụ: PCI Express).||

### Phần 5: Tính Toán Phân Tán và Dữ Liệu Lớn

| Mục Tiêu Ôn Tập              | Chi Tiết Cần Ghi Nhớ                                                                                                                                                                                                             | Nguồn |
| :--------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---- |
| **Đặc điểm Big Data (3 Vs)** | **Volume** (Khối lượng lớn); Variety (Đa dạng định dạng: Text, images, audio, streaming, v.v.); **Velocity** (Tốc độ tạo và xử lý nhanh).                                                                                        |       |
| **MapReduce Paradigm**       | Mô hình lập trình song song để xử lý lượng lớn dữ liệu. Gồm 3 giai đoạn: **Map stage** (xử lý dữ liệu đầu vào, tạo các đoạn nhỏ), **Shuffle stage** (sắp xếp và nhóm kết quả), và **Reduce stage** (tổng hợp kết quả cuối cùng). |       |
| **Hadoop**                   | Khung nguồn mở (Apache, viết bằng Java) cho lưu trữ và tính toán phân tán. Các module cốt lõi: HDFS (Distributed Storage), MapReduce (Distributed Computation), YARN (Job scheduling và quản lý tài nguyên).                     |       |
| **HDFS**                     | Hệ thống tệp phân tán, có khả năng chịu lỗi cao (fault-tolerant). Lưu trữ các tệp lớn bằng cách chia thành các khối (block) và nhân bản chúng.                                                                                   |       |
| **Apache Spark**             | Công nghệ tính toán cluster nhanh, mở rộng mô hình MapReduce để sử dụng hiệu quả hơn cho các loại tính toán khác. Tính năng chính: **Tính toán In-Memory (In-memory cluster computing)**.                                        |       |
| **NoSQL Databases**          | Cung cấp cơ chế lưu trữ và truy xuất dữ liệu khác với mô hình quan hệ. **Ưu điểm:** Khả năng mở rộng theo chiều ngang (Horizontally Scalable), lược đồ động (Dynamic Schema), phù hợp cho dữ liệu phân cấp.                      |       |
| **Các loại NoSQL**           | Key-Value, Document, Column-Family, Graph. Ví dụ: **MongoDB** (Document DB); **Cassandra** (Column-oriented DB, kiến trúc phân tán, không có điểm lỗi duy nhất).                                                                 |       |

### Phần 6: Tự Động Hóa và Ứng Dụng Cloud Native

|Mục Tiêu Ôn Tập|Chi Tiết Cần Ghi Nhớ|Nguồn|
|:--|:--|:--|
|**Microservices**|Phân tách chức năng ứng dụng thành các chương trình nhỏ, mỗi chương trình xử lý một chức năng đơn lẻ hoặc vài chức năng liên quan. Cho phép nâng cấp độc lập.||
|**Nhược điểm Microservices**|Lỗi cascading (một dịch vụ lỗi có thể gây ra lỗi cho nhiều dịch vụ khác), phức tạp quản lý, trùng lặp chức năng.||
|**Tự động hóa (Automation)**|Cần thiết vì quy mô cực lớn và sự thay đổi liên tục của các dịch vụ. Các tác vụ có thể tự động hóa: Quản lý tài nguyên ảo (VM, disk, network, container), giám sát, tối ưu hóa (cân bằng tải, giảm thiểu điện năng), và phục hồi an toàn.||
|**Điều phối (Orchestration)**|Tự động hóa hoàn toàn vòng đời dịch vụ, bao gồm triển khai, cấu hình, cập nhật, phục hồi sau lỗi và tự động mở rộng quy mô (Autoscaling). Các khối xây dựng (**building blocks**) (ví dụ: container, VM) có tính chất tạm thời (**Ephemeral**).||
|**Kubernetes (K8s)**|Nền tảng nguồn mở, mở rộng để quản lý khối lượng công việc và dịch vụ container hóa. K8s quản lý **pods** (đơn vị nhỏ nhất có thể quản lý, chứa một hoặc nhiều container).||
|**DevOps**|Mô hình văn hóa/tổ chức, kết hợp nhóm phát triển (Dev), QA, và vận hành (Ops). Dựa trên tự động hóa và điều phối. Yêu cầu: **Tích hợp liên tục (CI)** và **Phân phối liên tục (CD)**.||

### Phần 7: Bảo Mật, Tính Toán Biên và Tính Bền Vững

|Mục Tiêu Ôn Tập|Chi Tiết Cần Ghi Nhớ|Nguồn|
|:--|:--|:--|
|**Thách thức Bảo mật trên Cloud**|**Thiếu kiểm soát và khả năng hiển thị** (khó điều tra lỗi); **Hạ tầng chia sẻ** (tăng rủi ro nếu cơ chế cô lập không hoàn hảo); **Môi trường thực thi động** (Orchestration, autoscaling là yếu tố phức tạp).||
|**Bảo mật truy cập từ xa**|Mọi truy cập trong môi trường đám mây đều là truy cập từ xa. Cần: Mã hóa đầu cuối (**End-to-end encryption**); **Quản lý danh tính (Identity management)**; **VPN (Virtual Private Network)**.||
|**Tính toán Biên (Edge Computing)**|Đưa tính toán và lưu trữ đến gần người dùng hơn (gần ISP, tháp di động, hoặc gateway thông minh). Mục tiêu: Giảm **độ trễ đầu cuối (end-to-end latency)** cho các ứng dụng nhạy cảm (gaming, xe tự lái).||
|**Tính bền vững (Green Cloud Computing)**|Mục tiêu: Giảm thiểu tác động môi trường. Cloud Computing thường hiệu quả năng lượng hơn vận hành On-Premises nhờ **Dynamic Provisioning** và **Multi-Tenancy**.||
|**Đo lường Carbon Footprint**|Do không thể đo lường trực tiếp, sử dụng **Carbon Proxies** (metrics tương quan với phát thải carbon như Điện/Năng lượng, Chi phí, Tải CPU, Tải Mạng).||
|**Chiến lược mở rộng quy mô (Scaling)**|Các chiến lược để tránh **Overprovisioning**: **Pro-Active** (dựa trên dự đoán nhu cầu); **Random** (cân bằng tải cao điểm ngẫu nhiên); **Demand Shaping** (điều chỉnh nhu cầu theo nguồn cung); **Demand Shifting** (chuyển khối lượng công việc sang thời điểm/khu vực có lượng carbon thấp hơn).||

---

**Ghi chú về các Luật và Nguyên tắc cơ bản:**

- **Định luật Moore (Moore's Law):** Không phải luật tự nhiên, mà là quy tắc dựa trên quan sát thực nghiệm rằng số lượng bóng bán dẫn trên một mạch tích hợp tăng gấp đôi sau một chu kỳ nhất định.
- **Định luật Amdahl (Amdahl's Law):** Xác định khả năng tăng tốc tối đa (SpeedUp) của chương trình khi sử dụng nhiều CPU. Sự tăng tốc bị giới hạn bởi phần chương trình chạy tuần tự (sequential portion).
- **Định luật Gustafson (Gustafson's Law):** Đề xuất rằng khi quy mô vấn đề tăng lên cùng với số lượng CPU, phần tuần tự trở nên ít quan trọng hơn, và SpeedUp có thể tăng gần như tuyến tính với số lượng CPU.

Việc nắm vững các khái niệm và mối liên hệ giữa các phần này sẽ giúp bạn có cái nhìn tổng thể và sâu sắc về lĩnh vực Cloud Computing.
# BSide
Nên dành mỗi câu trắc nghiệm/phút
Không nên làm từ đầu tới cuối => vì hệ thống trộn đề
Nên dò => thường đề câu này là đáp án câu khác
