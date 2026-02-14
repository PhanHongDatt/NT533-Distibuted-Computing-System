# Kiến thức

### Tổng quan Chương 8: Lưu trữ Ảo (Virtual Storage)

Chương 8 mô tả cách các trung tâm dữ liệu chuyển dịch từ việc sử dụng ổ cứng gắn trực tiếp trên máy chủ (Direct Attached Storage - DAS) sang các cơ chế **lưu trữ từ xa (Remote Storage)**. Mục tiêu là tách biệt khả năng tính toán (Compute) và lưu trữ (Storage), cho phép chúng mở rộng độc lập và tăng cường độ tin cậy.

Có hai mô hình trừu tượng chính cho lưu trữ bền vững:

1. **Giao diện hướng Khối (Block-oriented interface):** Dùng cho ổ cứng vật lý (đọc/ghi theo block cố định).
2. **Giao diện hướng Byte (Byte-oriented interface):** Dùng cho hệ thống tệp (File System) cho phép thao tác mở/đọc/ghi file tùy ý.

Dưới đây là phân tích chi tiết các thuật ngữ:

---

### 1. NAS (Network Attached Storage)

**Định nghĩa:** NAS là hệ thống lưu trữ tệp từ xa (Remote File Storage) sử dụng phần cứng và phần mềm chuyên dụng, được thiết kế tối ưu cho hiệu năng cao và độ bền trong trung tâm dữ liệu.

- **Cơ chế hoạt động:** NAS hoạt động dựa trên giao diện **Tệp (File interface)**. Máy khách (Client) gửi yêu cầu mở, đọc, hoặc ghi một tệp cụ thể thông qua mạng. Hệ thống tệp thực sự nằm trên máy chủ NAS, không phải trên máy khách.
- **Đặc điểm:**
    - Sử dụng giao thức chia sẻ file qua mạng (như NFS).
    - Trình bày dữ liệu dưới dạng các thư mục và tệp tin phân cấp.
- **Ưu điểm:**
    - **Chia sẻ dễ dàng:** Rất tốt cho việc nhiều ứng dụng hoặc nhiều người dùng cùng truy cập và chia sẻ dữ liệu.
    - **Hỗ trợ Container:** Container có thể dễ dàng gắn (mount) các thư mục từ NAS để lưu trữ dữ liệu bền vững.
- **Nhược điểm:**
    - Phụ thuộc vào Hệ điều hành (OS) vì mỗi OS có định dạng hệ thống tệp khác nhau.
    - Hiệu năng có thể thấp hơn SAN do overhead của giao thức tệp.

### 2. SAN (Storage Area Network)

**Định nghĩa:** SAN là hệ thống cung cấp quyền truy cập đĩa từ xa (Remote Disk Access) sử dụng giao diện hướng Khối (Block-oriented interface).

- **Cơ chế hoạt động:** SAN cung cấp cho mỗi máy khách (như một máy ảo - VM) một **đĩa ảo (virtual disk)**. Đối với Hệ điều hành của VM, ổ đĩa này trông giống hệt như một ổ cứng vật lý gắn cục bộ, nhưng thực tế dữ liệu được truyền qua mạng.
- **Đặc điểm:**
    - Máy khách gửi yêu cầu đọc/ghi các khối dữ liệu (ví dụ: block 0, block 10) thay vì tên file.
    - Trước đây SAN dùng mạng riêng biệt (như Fibre Channel), nhưng hiện nay xu hướng là sử dụng chung mạng Ethernet của Data Center (Hyper-Converged Infrastructure - HCI).
- **Ưu điểm:**
    - **Linh hoạt tuyệt đối:** Hoạt động với bất kỳ Hệ điều hành nào (Windows, Linux đều có thể format đĩa ảo theo cách riêng của chúng).
    - **Hiệu năng cao:** Do thao tác ở mức block thấp nhất.
- **Nhược điểm:** Khó chia sẻ dữ liệu giữa các máy chủ khác nhau (vì đĩa được gán riêng cho một VM) và Container không thể sử dụng trực tiếp nếu không qua một lớp trung gian.

### 3. So sánh NAS và SAN

|Tiêu chí|NAS (Network Attached Storage)|SAN (Storage Area Network)|
|:--|:--|:--|
|**Giao diện**|File (Byte-oriented)|Block (Block-oriented)|
|**Đơn vị dữ liệu**|Tệp tin (Files), Thư mục (Directories)|Khối đĩa (Disk Blocks)|
|**Hệ thống tệp (File System)**|Nằm trên thiết bị NAS (Server side)|Nằm trên máy khách/VM (Client side)|
|**Khả năng tương thích**|Phụ thuộc vào OS (cần driver tương thích FS)|Tương thích mọi OS (vì là đĩa thô)|
|**Ứng dụng chính**|Chia sẻ file, dùng cho Container|Ổ đĩa hệ thống cho VM, Database hiệu năng cao|

### 4. Mapping (Trong bối cảnh SAN/Virtual Disk)

**Khái niệm:** Mapping là cơ chế cốt lõi giúp SAN tạo ra ảo giác về một "ổ đĩa liền mạch" cho máy khách.

- **Vấn đề:** Máy chủ lưu trữ (Storage Server) có hàng trăm ổ đĩa vật lý. Máy ảo (Client) cần một ổ đĩa ảo (ví dụ 100GB). Các khối dữ liệu của ổ đĩa ảo này có thể nằm rải rác lộn xộn trên các ổ đĩa vật lý khác nhau của server.
- **Cơ chế Virtual Disk Map:** Server duy trì một bảng ánh xạ (Map).
    - Khi Client yêu cầu ghi vào **Block 0** của đĩa ảo.
    - Server tra bảng Map và xác định Block 0 của Client tương ứng với **Block 21043** trên **Đĩa vật lý số 27** của Server.
    - Server thực hiện ghi vào vị trí vật lý đó.
- **Lợi ích:** Client hoàn toàn không biết về sự phức tạp bên dưới. Điều này cho phép Server tối ưu hóa không gian, thực hiện RAID, hoặc di chuyển dữ liệu mà không ảnh hưởng đến Client.

### 5. Object (Lưu trữ Đối tượng - Object Storage)

**Định nghĩa:** Đây là mô hình lưu trữ thứ ba, ra đời để giải quyết các hạn chế của NAS và SAN trong môi trường Cloud. Nó lưu trữ dữ liệu dưới dạng các "đối tượng" (objects) tùy ý thay vì block hay file trong thư mục phân cấp.

- **Đặc điểm nổi bật:**
    1. **Lưu trữ đối tượng tùy ý:** Có thể là ảnh, video, tài liệu, không giới hạn định dạng. Các đối tượng được gom vào các "Buckets" (thùng chứa) thay vì thư mục lồng nhau.
    2. **Truy cập phổ quát (Universal Accessibility):** Sử dụng giao diện **RESTful API** (giao thức HTTP) thay vì giao thức ổ đĩa hay tệp tin. Điều này có nghĩa là bất kỳ ứng dụng nào, chạy trên bất kỳ OS nào, ở bất kỳ đâu trên Internet đều có thể truy cập được thông qua URL.
    3. **Độc lập với Hệ điều hành:** Không phụ thuộc vào File System của OS như NAS, cũng không cần mount như SAN.
- **Ví dụ điển hình:** Amazon S3.

### 6. Lưu trữ Ảo (Virtual Storage) - Tổng hợp

Trong ngữ cảnh chương này, **Lưu trữ ảo** là thuật ngữ bao trùm việc trừu tượng hóa các thiết bị lưu trữ vật lý.

- Nó thay thế ổ đĩa cục bộ (Local Storage) bằng **Lưu trữ từ xa (Remote Storage)**.
- Nó sử dụng phần mềm để tạo ra các "Đĩa ảo" (thông qua SAN/Block storage) hoặc "Hệ thống tệp ảo" (thông qua NAS) để cung cấp cho Máy ảo hoặc Container.
- **Kết luận:** Nhờ lưu trữ ảo, dữ liệu không còn bị trói buộc vào một máy chủ vật lý cụ thể, cho phép các tính năng quan trọng của đám mây như **Live Migration** (di chuyển VM mà không mất dữ liệu) và **High Availability** (dữ liệu vẫn an toàn khi server tính toán bị hỏng).

---
# Câu hỏi ôn tập

## Phần 1: Khái niệm Cơ bản về Lưu trữ (Persistent Storage Concepts)

### Câu 1
Thuật ngữ "Persistent storage" (Lưu trữ bền vững) hoặc "Non-volatile storage" ám chỉ điều gì?
A. Dữ liệu bị mất khi tắt nguồn điện.
B. Cơ chế lưu trữ giữ lại dữ liệu ngay cả khi nguồn điện bị ngắt.
C. Lưu trữ chỉ dùng cho bộ nhớ cache của CPU.
D. Dữ liệu được lưu trữ tạm thời trên RAM.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Persistent storage (Lưu trữ bền vững) là cơ chế lưu trữ giữ lại dữ liệu sau khi nguồn điện đã bị loại bỏ (ngược lại với Volatile storage như RAM).

### Câu 2
Hai dạng trừu tượng hóa chính của lưu trữ bền vững (persistent storage abstractions) mà hệ điều hành cung cấp cho người dùng là gì?
A. Bit và Byte.
B. Named files (Tệp có tên) và Hierarchical directories (Thư mục phân cấp).
C. TCP và IP.
D. CPU và RAM.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Hệ điều hành che giấu sự phức tạp của phần cứng và cung cấp hai sự trừu tượng hóa trực quan cho người dùng: tệp có tên (files) và thư mục phân cấp (directories/folders).

### Câu 3
Giao diện phần cứng ổ đĩa (Disk Hardware Interface) hoạt động theo cơ chế nào?
A. Byte-oriented interface (Hướng Byte).
B. Object-oriented interface (Hướng Đối tượng).
C. Block-oriented interface (Hướng Khối).
D. Stream-oriented interface (Hướng Luồng).

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** Phần cứng ổ đĩa vật lý cung cấp một giao diện hướng khối (block-oriented), chỉ có thể lưu trữ và truy xuất các khối dữ liệu có kích thước cố định (ví dụ: 512 bytes hoặc 4KB).

### Câu 4
Đặc điểm nào sau đây là **ĐÚNG** khi nói về thao tác ghi (Write) trên giao diện hướng khối (Block-oriented interface)?
A. Có thể ghi đè lên một bit đơn lẻ mà không ảnh hưởng các bit khác trong khối.
B. Có thể ghi đè một nửa khối.
C. Không thể ghi một phần khối; toàn bộ khối phải được thay thế bằng dữ liệu mới.
D. Khối có kích thước thay đổi tùy ý.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** Phần cứng không thể ghi một phần của khối (partial block write). Khi muốn thay đổi dù chỉ 1 bit, hệ thống phải đọc cả khối lên, sửa đổi trong RAM, rồi ghi đè lại toàn bộ khối đó xuống đĩa.

### Câu 5
Ngược lại với ổ đĩa, Hệ thống tệp (File System) cung cấp giao diện dạng nào cho ứng dụng?
A. Block-oriented interface.
B. Byte-oriented interface.
C. Sector-oriented interface.
D. Cylinder-oriented interface.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Hệ thống tệp cung cấp giao diện hướng byte (byte-oriented), cho phép ứng dụng đọc/ghi dữ liệu ở bất kỳ vị trí byte nào và với số lượng byte tùy ý (không cần quan tâm đến block).

---

## Phần 2: Local vs Remote Storage & NAS

### Câu 6
Thuật ngữ DAS (Directly Attached Storage) đồng nghĩa với loại lưu trữ nào?
A. Remote storage (Lưu trữ từ xa).
B. Cloud storage (Lưu trữ đám mây).
C. Local storage (Lưu trữ cục bộ).
D. Object storage (Lưu trữ đối tượng).

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** Lưu trữ cục bộ (Local storage) là thiết bị được kết nối trực tiếp với máy tính qua bus (như SATA, PCIe), còn được gọi là DAS.

### Câu 7
NAS là viết tắt của thuật ngữ nào?
A. Network Area Storage.
B. Network Attached Storage.
C. Node Attached System.
D. New Access Storage.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** NAS là viết tắt của **Network Attached Storage**.

### Câu 8
NAS (Network Attached Storage) cung cấp loại truy cập lưu trữ nào?
A. Remote Block Access (Truy cập khối từ xa).
B. Remote File Access (Truy cập tệp từ xa).
C. Remote Memory Access (Truy cập bộ nhớ từ xa).
D. Local Disk Access.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** NAS hoạt động ở cấp độ File (File-level storage). Nó cung cấp dịch vụ lưu trữ tệp qua mạng.

### Câu 9
Giao thức tiêu chuẩn nào thường được sử dụng bởi các hệ thống NAS để chia sẻ tệp?
A. HTTP.
B. NFS (Network File System).
C. BGP.
D. SMTP.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** **NFS (Network File System)** là giao thức chuẩn để chia sẻ tệp qua mạng trên môi trường Linux/Unix (tương đương SMB/CIFS trên Windows).

### Câu 10
Một nhược điểm chính của NAS là gì?
A. Không thể chia sẻ file.
B. Khó sử dụng.
C. Phụ thuộc vào Hệ điều hành (OS-specific).
D. Không hỗ trợ mạng Ethernet.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** NAS thường phụ thuộc vào ngữ nghĩa của hệ thống tệp (File System semantics) của hệ điều hành (ví dụ quyền hạn file, định dạng đường dẫn), gây khó khăn khi truy cập chéo nền tảng.

---

## Phần 3: SAN (Storage Area Network)

### Câu 11
SAN là viết tắt của thuật ngữ nào?
A. Storage Area Network.
B. Server Area Network.
C. Simple Access Node.
D. System And Network.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: A**
> **Giải thích:** SAN là viết tắt của **Storage Area Network**.

### Câu 12
SAN cung cấp loại giao diện lưu trữ nào cho máy khách?
A. Byte-oriented / File interface.
B. Block-oriented / Disk interface.
C. Object-oriented interface.
D. Web interface.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** SAN cung cấp lưu trữ ở cấp độ Khối (Block-level). Máy tính kết nối vào SAN sẽ nhìn thấy tài nguyên lưu trữ như một ổ đĩa thô (Raw Disk).

### Câu 13
Đối với một Máy ảo (VM), ổ đĩa được cung cấp bởi SAN trông giống như gì?
A. Một thư mục chia sẻ trên mạng.
B. Một website.
C. Một ổ đĩa vật lý gắn cục bộ (Local disk).
D. Một cơ sở dữ liệu.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** Ảo thuật của SAN là làm cho ổ đĩa từ xa (Remote Disk) xuất hiện trước hệ điều hành như thể nó là một ổ đĩa cục bộ (Local Physical Disk).

### Câu 14
Cơ chế nào giúp SAN server quản lý việc cấp phát các khối đĩa cho máy khách?
A. File Allocation Table.
B. Virtual Disk Map (Bản đồ đĩa ảo).
C. Routing Table.
D. DNS.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Server sử dụng ID của khách hàng để tra cứu trong "Virtual disk map", ánh xạ các khối ảo (virtual blocks) mà khách hàng nhìn thấy tới các khối vật lý (physical blocks) thực sự trên hệ thống lưu trữ.

### Câu 15
"Hyper-Converged Infrastructure" (HCI) trong bối cảnh lưu trữ ám chỉ điều gì?
A. Sử dụng mạng cáp quang riêng biệt cho lưu trữ.
B. Sử dụng mạng Ethernet tiêu chuẩn chung cho cả lưu lượng ứng dụng và lưu lượng lưu trữ (SAN).
C. Loại bỏ hoàn toàn ổ cứng.
D. Chỉ sử dụng NAS.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** HCI hội tụ hạ tầng bằng cách sử dụng chung một mạng Ethernet tốc độ cao cho cả lưu lượng mạng thông thường và lưu lượng lưu trữ (thay vì xây mạng Fibre Channel riêng cho SAN như trước kia).

---

## Phần 4: Object Storage & Thuật ngữ khác

### Câu 16
Object Storage (Lưu trữ đối tượng) sử dụng giao diện nào để truy cập dữ liệu?
A. SCSI commands.
B. NFS protocol.
C. RESTful interface (HTTP).
D. IDE cables.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** Object Storage sử dụng giao diện web **RESTful API** (dựa trên HTTP methods: GET, PUT, DELETE) để truy cập dữ liệu.

### Câu 17
Trong Object Storage, các đối tượng (objects) được gom nhóm vào đâu (tương tự như thư mục)?
A. Directories.
B. Buckets.
C. Blocks.
D. Sectors.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Khái niệm container chứa các đối tượng trong Object Storage được gọi là **Buckets**.

### Câu 18
Amazon S3 là ví dụ điển hình của loại hình lưu trữ nào?
A. Local Storage.
B. Block Storage.
C. Object Storage.
D. File Storage.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** Amazon **S3** (Simple Storage Service) là dịch vụ **Object Storage** nổi tiếng nhất.

### Câu 19
Ưu điểm nổi bật của Object Storage so với NAS là gì?
A. Tốc độ truy cập nhanh hơn RAM.
B. Độc lập với hệ điều hành (OS independent).
C. Chỉ lưu được văn bản.
D. Yêu cầu phần cứng đắt tiền.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Do dùng giao thức HTTP/REST, Object Storage hoàn toàn **độc lập với hệ điều hành**. Bất kỳ thiết bị nào có kết nối Internet (Mobile, Web app, Server Linux/Windows) đều truy cập được.

### Câu 20
Công nghệ RAID (Redundant Array of Independent Disks) được sử dụng trong các hệ thống lưu trữ nhằm mục đích gì?
A. Tăng dung lượng RAM.
B. Tăng tính bền vững (durability) và khả năng chịu lỗi bằng cách sao chép dữ liệu trên nhiều đĩa.
C. Giảm tiêu thụ điện năng.
D. Làm mát hệ thống.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** RAID (Redundant Array of Independent Disks) giúp tăng độ an toàn dữ liệu (Durability) và tính sẵn sàng (Availability) bằng cách lưu trữ dự phòng trên nhiều đĩa cứng vật lý.Chào bạn, đây là bộ **30 câu hỏi trắc nghiệm** cho **Chương 8: Virtual Storage** với mức độ **Vận dụng, Phân tích, và Thực tiễn**. Bộ câu hỏi này được thiết kế với các "bẫy" kỹ thuật, yêu cầu bạn phải hiểu sâu về cơ chế hoạt động bên dưới (under-the-hood) chứ không chỉ thuộc lòng định nghĩa.

Nội dung được trình bày dưới dạng **Markdown Obsidian** để bạn dễ dàng copy vào vault của mình.

---


# Pool Câu Hỏi Vận Dụng Cao & Thực Tiễn - Chapter 8: Virtual Storage

## Phần 1: Cơ chế Block vs. File (The Mechanics)

### Câu 1
**Tình huống:** Một ứng dụng cơ sở dữ liệu (Database) cần thực hiện thay đổi **1 byte** dữ liệu nằm ở giữa một tệp tin dữ liệu khổng lồ 100GB. Hệ thống lưu trữ bên dưới sử dụng giao diện **Block-oriented** (ví dụ: SAN). Về mặt kỹ thuật, điều gì thực sự xảy ra tại lớp phần cứng lưu trữ?
A. Ổ đĩa tìm đến đúng byte đó và ghi đè giá trị mới lên, không ảnh hưởng đến các byte xung quanh.
B. Hệ điều hành đọc toàn bộ tệp 100GB vào RAM, sửa 1 byte, và ghi lại toàn bộ 100GB xuống đĩa.
C. Phần cứng đĩa đọc toàn bộ **Block** chứa byte đó (ví dụ 4KB) vào bộ nhớ đệm, sửa byte đó, sau đó ghi đè lại toàn bộ Block đó xuống đĩa.
D. Hệ thống tạo ra một bản sao (snapshot) mới của dữ liệu chỉ chứa 1 byte thay đổi.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Giao diện đĩa là **Block-oriented**. Phần cứng không thể ghi một phần của khối. Để thay đổi một byte, nó phải đọc toàn bộ khối, sửa đổi và ghi lại toàn bộ khối đó. Đây là chi tiết quan trọng về hiệu năng I/O.

### Câu 2
**Tình huống:** Bạn đang thiết kế một ứng dụng yêu cầu hiệu năng I/O cực cao. Bạn so sánh giữa việc dùng NAS và SAN. Tại sao SAN thường có độ trễ thấp hơn và hiệu năng cao hơn NAS cho các tác vụ ghi dữ liệu ngẫu nhiên (random write)?
A. Vì SAN sử dụng cáp quang (Fiber Channel) còn NAS luôn dùng cáp đồng.
B. Vì SAN xử lý ở mức **Block**, không tốn chi phí (overhead) cho việc quản lý Metadata của hệ thống tệp (như cập nhật inode, directory structure) trên đường truyền mạng như NAS.
C. Vì SAN nén dữ liệu trước khi gửi.
D. Vì NAS phải mã hóa dữ liệu SSL/TLS bắt buộc.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Trong mô hình NAS, request gửi đi là file request, server phải xử lý logic hệ thống tệp (metadata updates). Trong mô hình SAN, client (OS của VM) quản lý file system, chỉ gửi các lệnh block I/O thô qua mạng, giảm bớt overhead xử lý tại phía server lưu trữ.

### Câu 3
**Tình huống:** Một hệ điều hành gửi lệnh ghi dữ liệu vào "Block 0" và "Block 1" của ổ đĩa ảo (Virtual Disk) được cung cấp bởi SAN. Thực tế trên các ổ đĩa vật lý tại Data Center, hai khối dữ liệu này có nhất thiết phải nằm liền kề nhau không?
A. Có, để đảm bảo tốc độ đọc tuần tự (sequential read).
B. Không, chúng có thể nằm trên hai ổ đĩa vật lý hoàn toàn khác nhau.
C. Có, nhưng chúng có thể bị đảo ngược thứ tự.
D. Không, nhưng chúng bắt buộc phải nằm trên cùng một ổ đĩa vật lý.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Cơ chế **Mapping Virtual Disks to Physical Disks** cho phép Server ánh xạ các block ảo của client tới bất kỳ block vật lý nào rảnh rỗi, kể cả trên các đĩa khác nhau. Client hoàn toàn không biết về sự phân mảnh này.

### Câu 4
**Tình huống (Bẫy):** Bạn có một file server (NAS) chạy Linux format ổ đĩa dạng `ext4`. Bạn muốn mount ổ đĩa này vào một máy khách chạy Windows. Máy khách Windows có cần phải cài đặt driver `ext4` để đọc dữ liệu không?
A. Có, vì Windows mặc định dùng NTFS, không hiểu `ext4`.
B. Không, vì máy khách Windows giao tiếp qua giao thức chia sẻ file (như NFS/SMB), không giao tiếp trực tiếp với định dạng đĩa vật lý.
C. Có, nhưng chỉ cần cài trên Hypervisor.
D. Không, vì NAS sẽ tự động format lại ổ đĩa thành NTFS khi Windows kết nối.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Đây là điểm mạnh của **NAS**. File System nằm trên thiết bị NAS (Server-side). Client chỉ gửi lệnh "đọc file A". Client không cần quan tâm đĩa của Server format kiểu gì, miễn là cả hai hiểu chung giao thức mạng (NFS/SMB).

---

## Phần 2: NAS vs. SAN vs. Object (So sánh & Lựa chọn)

### Câu 5
**Tình huống:** Một công ty truyền thông cần lưu trữ hàng tỷ bức ảnh thumbnail cho website. Các ảnh này được viết một lần (write once) và đọc nhiều lần (read many) bởi hàng triệu người dùng toàn cầu thông qua trình duyệt web. Giải pháp lưu trữ nào là tối ưu nhất về chi phí và khả năng truy cập?
A. Tạo một file server NAS khổng lồ và share qua NFS.
B. Sử dụng SAN để cấp ổ cứng cho Web Server.
C. Sử dụng **Object Storage** (như Amazon S3).
D. Lưu trữ trực tiếp trên ổ cứng cục bộ của từng Web Server.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** **Object Storage** hỗ trợ "arbitrary number of objects", cung cấp "universal accessibility" qua giao diện RESTful (HTTP) phù hợp cho web, và có khả năng mở rộng (scalability) tốt hơn NAS cho số lượng file cực lớn.

### Câu 6
**Tình huống:** Bạn đang cấu hình một cụm máy chủ cơ sở dữ liệu Oracle RAC (Real Application Clusters) yêu cầu nhiều máy chủ cùng truy cập vào một vùng dữ liệu chung với tốc độ block-level (khối). Công nghệ nào phù hợp nhất?
A. NAS, vì nó hỗ trợ chia sẻ file.
B. SAN, vì nó cung cấp truy cập block-level và cho phép (với phần mềm quản lý cluster) nhiều server cùng truy cập một LUN (Logical Unit Number).
C. Object Storage, vì nó rẻ.
D. Local DAS, vì nó nhanh nhất.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Mặc dù SAN thường gán đĩa riêng cho 1 client, nhưng trong các trường hợp Cluster Database cao cấp, SAN hỗ trợ shared block storage (điều mà NAS không tối ưu bằng về hiệu năng database, và Object store không hỗ trợ giao diện block).

### Câu 7
**Tình huống (So sánh):** Điều gì là trở ngại lớn nhất khi bạn muốn sử dụng **NAS** để lưu trữ ổ đĩa hệ điều hành (boot drive) cho một máy ảo (VM) chạy Windows, so với việc dùng SAN?
A. Tốc độ NAS quá chậm để boot.
B. Windows không thể boot từ một đường dẫn mạng (network path) nếu không có các thiết lập đặc biệt (như iSCSI - vốn là giao thức SAN), trong khi SAN hiển thị như một ổ đĩa cục bộ thực thụ.
C. NAS không hỗ trợ file lớn.
D. NAS hay bị mất dữ liệu.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** SAN cung cấp **"remote disk interface"**. Hệ điều hành nhìn thấy nó như một ổ đĩa vật lý local chưa format, nên có thể cài đặt OS và boot bình thường. NAS cung cấp file system, OS cần phải boot xong mới mount được NAS (trừ các trường hợp đặc biệt như PXE/Netboot nhưng phức tạp hơn).

### Câu 8
**Tình huống:** Một Container cần truy cập vào dữ liệu cấu hình dùng chung. Bạn sử dụng `VOLUME` để mount dữ liệu. Tại sao việc sử dụng **NAS** (như NFS) lại dễ dàng hơn cho Container so với **SAN**?
A. Vì Container không thể giao tiếp với SAN.
B. Vì SAN yêu cầu Container phải format ổ đĩa, trong khi Container thường là ephemeral (ngắn hạn) và cần truy cập file ngay lập tức. NAS cho phép nhiều container cùng đọc/ghi file trên một thư mục chia sẻ mà không lo hỏng file system.
C. Vì NAS rẻ hơn.
D. Vì Docker chỉ hỗ trợ driver NFS.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** SAN cấp block device. Nếu 2 container cùng mount 1 block device và cùng format/ghi, sẽ hỏng dữ liệu (trừ khi dùng Cluster FS). NAS quản lý file system ở phía server, nên nhiều container có thể mount và chia sẻ file an toàn và dễ dàng.

---

## Phần 3: Kiến trúc & Hạ tầng (Infrastructure)

### Câu 9
**Tình huống:** Trong kiến trúc **Hyper-Converged Infrastructure (HCI)**, lưu lượng truy cập lưu trữ (Storage Traffic) di chuyển như thế nào so với kiến trúc SAN truyền thống?
A. Nó di chuyển trên một mạng cáp quang (Fibre Channel) riêng biệt hoàn toàn.
B. Nó di chuyển trên cùng hạ tầng mạng Ethernet (Leaf-Spine) với lưu lượng ứng dụng (Application Traffic).
C. Nó chỉ di chuyển nội bộ trong server, không ra mạng.
D. Nó sử dụng sóng Wifi.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** HCI tận dụng sự phát triển của Ethernet tốc độ cao (100Gbps+) và kiến trúc Leaf-Spine để hợp nhất (converge) mạng lưu trữ và mạng dữ liệu thành một, loại bỏ mạng chuyên dụng đắt tiền.

### Câu 10
**Tình huống (Bẫy):** Bạn xóa một file 10GB trên máy ảo đang sử dụng SAN. Dung lượng đĩa vật lý tại SAN Server có được giải phóng ngay lập tức về 0 không?
A. Có, ngay lập tức.
B. Không hẳn. Hệ điều hành trên VM chỉ đánh dấu file là "đã xóa" trong bảng định dạng file system của nó. SAN Server chỉ nhìn thấy các thao tác Block, nó không hiểu khái niệm "xóa file" trừ khi có lệnh TRIM/UNMAP được gửi xuống.
C. Không bao giờ giải phóng được.
D. Có, nhưng sau 24h.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Đây là hệ quả của việc tách biệt **File Interface** (ở Client) và **Block Interface** (ở Server). Server lưu trữ Block không hiểu cấu trúc File System của Client.

### Câu 11
**Tình huống:** Tại sao trong các Data Center hiện đại, người ta thường đặt các thiết bị lưu trữ (Storage Appliances) rải rác (ví dụ: mỗi Pod một thiết bị) thay vì gom tất cả vào một căn phòng duy nhất?
A. Để tiết kiệm tiền thuê phòng.
B. Để giảm độ trễ mạng (Latency) và tránh tắc nghẽn cổ chai (Bottleneck) khi lưu lượng đi qua các tầng mạng cao hơn (Spine/Super Spine).
C. Để làm mát dễ hơn.
D. Để bảo mật tốt hơn.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Việc truy cập remote storage tạo ra "East-West traffic". Đặt storage gần server (trong cùng Pod) giúp giảm số lượng switch phải đi qua, giảm độ trễ.

---

## Phần 4: Object Storage & Cloud Native (Xu thế mới)

### Câu 12
**Tình huống:** Bạn muốn sửa đổi dòng thứ 500 trong một file log 1GB đang lưu trên **Object Storage** (S3). Bạn có thể dùng lệnh `seek` để nhảy đến vị trí đó và ghi đè dữ liệu không?
A. Có, Object Storage hỗ trợ đầy đủ các lệnh như POSIX file system.
B. Không, Object Storage coi đối tượng là bất biến (immutable) ở mức độ nội dung. Bạn phải tải file về, sửa đổi, và upload lại toàn bộ file mới (PUT overwrite).
C. Có, nhưng sẽ rất chậm.
D. Không, bạn chỉ có thể xóa file.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Giao diện của Object Storage là RESTful (GET, PUT, DELETE, POST). `PUT` thay thế toàn bộ đối tượng. Không có thao tác `write` một phần hay `seek` như Block/File storage.

### Câu 13
**Tình huống:** Tại sao **Object Storage** lại được coi là "Universal Accessibility" (Truy cập phổ quát) hơn so với NAS/SAN?
A. Vì nó nhanh hơn.
B. Vì nó dùng giao thức HTTP/REST API. Bất kỳ thiết bị nào (Mobile, Server, Laptop) chạy bất kỳ hệ điều hành nào, miễn là có kết nối Internet đều có thể truy cập được mà không cần mount hay cài driver đặc biệt.
C. Vì nó miễn phí.
D. Vì nó không cần mật khẩu.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Object store sử dụng giao diện RESTful, độc lập với hệ điều hành, cho phép truy cập từ bất cứ đâu.

### Câu 14
**Tình huống:** Trong Object Storage, khái niệm "Buckets" tương đương với cái gì trong hệ thống tệp truyền thống, nhưng có điểm gì khác biệt cốt lõi?
A. Tương đương với File, nhưng to hơn.
B. Tương đương với Folder/Directory, nhưng cấu trúc của Object Storage là **phẳng (flat)**, không cho phép lồng nhau phức tạp (sub-folders thực chất chỉ là ảo ảnh qua tên key).
C. Tương đương với Ổ đĩa (Drive).
D. Tương đương với RAM.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Object store nhóm các đối tượng vào các "buckets". Tuy nhiên, khác với thư mục phân cấp, cấu trúc của nó phẳng để đảm bảo khả năng mở rộng (scalability).

---

## Phần 5: Troubleshoot & Vận hành (Operational)

### Câu 15
**Tình huống:** Một máy ảo (VM) đang chạy thì máy chủ vật lý bị hỏng. Dữ liệu trên ổ đĩa C: (Virtual Disk) của VM đó có bị mất không?
A. Có, vì nó nằm trên máy chủ vật lý bị hỏng.
B. Không, vì ổ đĩa C: thực chất là một tệp ảnh đĩa nằm trên hệ thống SAN/NAS từ xa. VM có thể được khởi động lại trên một máy chủ vật lý khác và kết nối lại với ổ đĩa đó.
C. Có, trừ khi đã backup ra USB.
D. Không, vì dữ liệu nằm trên RAM.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Đây là lợi ích cốt lõi của việc tách biệt Compute (Server) và Storage (Remote Storage). Dữ liệu được an toàn và hỗ trợ tính năng **VM Migration** hoặc **High Availability**.

### Câu 16
**Tình huống:** Bạn đang vận hành một hệ thống Hadoop MapReduce. Tại sao HDFS (Hadoop Distributed File System) lại chia file thành các **Block** rất lớn (ví dụ 128MB) thay vì 4KB như đĩa thông thường?
A. Để tiết kiệm dung lượng đĩa.
B. Để giảm kích thước Metadata mà NameNode phải quản lý (ít block hơn = ít metadata hơn) và tối ưu hóa cho việc đọc tuần tự (streaming reads) khối lượng lớn.
C. Vì Java không hỗ trợ file nhỏ.
D. Để tăng độ trễ.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** HDFS được thiết kế cho "Large streaming reads". Block lớn giúp giảm overhead của metadata trên Master (NameNode) và tăng throughput cho các tác vụ xử lý dữ liệu lớn.

### Câu 17
**Tình huống:** Khi sử dụng **Live Migration** để chuyển một VM từ Server A sang Server B. Tại sao quá trình này yêu cầu cả hai Server phải cùng truy cập được vào một hệ thống lưu trữ chung (Shared Storage like SAN/NAS)?
A. Để copy RAM nhanh hơn.
B. Vì Live Migration chỉ copy trạng thái CPU và RAM (memory state). Ổ cứng (Disk state) quá lớn để copy tức thời. Việc dùng Shared Storage giúp VM tại Server B tiếp tục truy cập dữ liệu ngay lập tức mà không cần copy dữ liệu đĩa.
C. Luật an ninh mạng yêu cầu vậy.
D. Để Server A điều khiển Server B.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** VM Migration chuyển đổi VM thành digital object (RAM, CPU state). Dữ liệu đĩa thường được để lại trên remote storage để tránh việc phải copy lượng dữ liệu khổng lồ, giúp quá trình migrate diễn ra nhanh chóng (live).

### Câu 18
**Tình huống:** Ứng dụng của bạn báo lỗi "Disk Full", nhưng khi kiểm tra trên SAN storage array, dung lượng vật lý vẫn còn trống rất nhiều. Nguyên nhân có thể là gì?
A. Ổ đĩa vật lý bị hỏng.
B. Dung lượng của **Virtual Disk** được cấp phát (allocated) cho VM đó đã hết (ví dụ cấp 100GB đã dùng hết 100GB), dù SAN pool còn hàng TB. Cần phải mở rộng (extend) Virtual Disk.
C. Do cáp mạng bị lỏng.
D. Do CPU quá tải.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Server SAN cung cấp một "ổ đĩa ảo" với kích thước cố định cho Client. Client chỉ nhìn thấy kích thước đó, không nhìn thấy tổng dung lượng của SAN.

### Câu 19
**Tình huống:** Bạn sử dụng Docker Container để chạy một Database. Bạn dùng lệnh `docker run` mà không khai báo `-v` (volume). Sau khi restart container, dữ liệu database biến mất. Tại sao?
A. Hacker đã xóa.
B. Docker bị lỗi.
C. Vì Container File System là **Ephemeral** (phù du). Các thay đổi trên lớp R/W layer của container sẽ mất đi khi container bị xóa/tạo lại. Cần dùng **Volume** để trỏ xuống lớp lưu trữ bền vững (Persistent Storage).
D. Vì Database chưa commit.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** "Unless a programmer connects a container to permanent storage, changes made to local files when a container runs will not be saved".

### Câu 20
**Tình huống:** Trong hệ thống **RAID** (thường dùng trong NAS/SAN), mục đích chính của việc ghi dữ liệu dư thừa (redundancy) là gì?
A. Để tăng gấp đôi dung lượng lưu trữ.
B. Để tăng khả năng chịu lỗi (**Fault Tolerance**) và độ bền (**Durability**). Nếu một ổ cứng hỏng, dữ liệu vẫn còn và hệ thống vẫn hoạt động.
C. Để làm chậm hệ thống.
D. Để tiết kiệm điện.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** RAID (Redundant Array of Independent Disks) đặt các bản sao dữ liệu dư thừa lên nhiều đĩa để hệ thống tiếp tục hoạt động nếu một đĩa bị hỏng.

---

## Phần 6: Tổng hợp & Đánh đố (Tricky)

### Câu 21
**Thuật ngữ nào mô tả việc mạo danh một thiết bị lưu trữ vật lý bằng phần mềm để hệ điều hành khách (Guest OS) nghĩ rằng nó đang có một ổ cứng riêng?**
A. Disk Encapsulation.
B. Virtual Disk / Block Storage.
C. Object Mapping.
D. File Masquerading.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Virtual disks (block storage) cung cấp giao diện đĩa từ xa nhưng hành xử "exactly like a local disk" đối với phần mềm trên server.

### Câu 22
**Sự khác biệt chính giữa "Scale-Up" (Vertical) và "Scale-Out" (Horizontal) trong bối cảnh lưu trữ là gì?**
A. Scale-Up là thêm ổ cứng vào một máy chủ lưu trữ hiện có; Scale-Out là thêm nhiều máy chủ lưu trữ (node) vào cụm lưu trữ.
B. Scale-Up là mua ổ cứng to hơn; Scale-Out là mua ổ cứng nhỏ hơn.
C. Scale-Up dùng cho NAS; Scale-Out dùng cho SAN.
D. Không có sự khác biệt.

> [!success]- Đáp án & Giải thích
> **Đáp án:** A
> **Giải thích:** Dựa trên khái niệm chung về Scalability trong chương, Scale-up là thêm tài nguyên vào thiết bị hiện có, Scale-out là thêm thiết bị mới vào cụm.

### Câu 23
**Tại sao các hệ thống File System phân tán (như NFS) thường gặp vấn đề về "Consistency" (tính nhất quán) khi nhiều client cùng ghi vào một file?**
A. Vì mạng quá chậm.
B. Vì cơ chế caching tại phía Client (để tăng tốc độ) có thể khiến Client A không nhìn thấy ngay lập tức những gì Client B vừa ghi lên Server.
C. Vì Server không đủ RAM.
D. Vì giao thức NFS không hỗ trợ khóa file.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Vấn đề cache coherence (tính nhất quán của bộ nhớ đệm) là một thách thức lớn trong thiết kế hệ thống phân tán, nơi consistency liên quan đến locking, caching và synchronization.

### Câu 24
**Trong ngữ cảnh Cloud, thuật ngữ "Thin Client" liên quan như thế nào đến mô hình SaaS và Lưu trữ?**
A. Thin Client là máy chủ chứa dữ liệu.
B. Thin Client là thiết bị người dùng có cấu hình thấp, không cần lưu trữ dữ liệu cục bộ nhiều vì toàn bộ dữ liệu và ứng dụng đã nằm trên Cloud Storage/SaaS.
C. Thin Client là một loại ổ cứng mỏng.
D. Thin Client là phần mềm quản lý mạng.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** "Because processing and storage occur in a cloud data center... a device used to access a SaaS service does not need a powerful processor, large memory, or storage... Industry uses the term **thin client**".

### Câu 25
**Giao thức nào sau đây KHÔNG phải là giao thức dùng cho Network Attached Storage (NAS)?**
A. NFS (Network File System).
B. SMB (Server Message Block).
C. iSCSI (Internet Small Computer Systems Interface).
D. AFP (Apple Filing Protocol).

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** NFS và SMB là giao thức chia sẻ file (NAS). iSCSI là giao thức block-level dùng để truyền tải lệnh SCSI qua IP, đặc trưng của **SAN**.

### Câu 26
**"Data Locality" trong HDFS có ý nghĩa gì đối với hiệu năng xử lý Big Data?**
A. Dữ liệu luôn được giữ tại máy của người dùng.
B. Di chuyển tính toán (computation) đến nơi có dữ liệu (data node) thay vì di chuyển lượng lớn dữ liệu đến nơi có tính toán, giúp giảm tải băng thông mạng.
C. Dữ liệu được lưu trên một đĩa duy nhất.
D. Dữ liệu được nén lại.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** "Move compute to data" - Cluster có băng thông hạn chế, việc di chuyển code (nhỏ) đến nơi có data (lớn) hiệu quả hơn nhiều.

### Câu 27
**Điều gì xảy ra với hiệu suất (Performance) khi ta sử dụng cơ chế "Snapshots" (Copy-on-write) quá nhiều trên hệ thống lưu trữ?**
A. Hiệu suất tăng lên do đọc nhanh hơn.
B. Hiệu suất ghi (Write performance) có thể bị giảm vì mỗi lần ghi vào block cũ, hệ thống phải copy block cũ sang chỗ khác trước khi ghi đè (hoặc ghi vào chỗ mới và cập nhật metadata).
C. Không ảnh hưởng gì.
D. Dung lượng đĩa tăng lên gấp đôi ngay lập tức.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Copy-on-write là một cách đơn giản và nhanh để tạo snapshot, nhưng "a request to write to a file creates a new copy", điều này ngụ ý overhead cho các tác vụ ghi.

### Câu 28
**Trong kiến trúc SAN, thành phần nào chịu trách nhiệm duy trì "Virtual Disk Map" (Bản đồ ánh xạ từ Block ảo sang Block vật lý)?**
A. Client (Máy ảo).
B. SAN Switch.
C. SAN Storage Server (Disk Array Controller).
D. Hypervisor.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** "The server uses the client’s unique ID to form a virtual disk map... The server maintains a mapping". Client không hề biết về bản đồ này.

### Câu 29
**"Object Relational Impedance Mismatch" là động lực thúc đẩy sự phát triển của loại cơ sở dữ liệu nào, thường đi kèm với các mô hình lưu trữ mới?**
A. SQL Database.
B. NoSQL Database.
C. Flat File System.
D. Tape Storage.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Động lực cho NoSQL bao gồm "limiting the object-relational impedance mismatch" - sự không tương thích giữa mô hình đối tượng trong code và mô hình bảng trong DB quan hệ.

### Câu 30
**Công nghệ nào cho phép một doanh nghiệp tận dụng các ổ cứng cũ rải rác trên hàng trăm server để tạo thành một "Storage Pool" khổng lồ, thống nhất, có khả năng chịu lỗi mà không cần mua thiết bị SAN chuyên dụng đắt tiền?**
A. RAID card phần cứng.
B. Software-Defined Storage (SDS) (như Ceph, vSAN) chạy trên hạ tầng siêu hội tụ (HCI).
C. USB Hub.
D. Tape Library.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Đây là mô tả của **Hyper-Converged Infrastructure (HCI)** và xu hướng chuyển dịch từ phần cứng chuyên dụng sang phần mềm chạy trên commodity hardware, sử dụng mạng Ethernet hội tụ.
