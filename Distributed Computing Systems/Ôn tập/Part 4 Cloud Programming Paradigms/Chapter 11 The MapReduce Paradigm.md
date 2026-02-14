
### 1. Vấn đề của lập trình song song truyền thống

Trong các siêu máy tính truyền thống, quy trình xử lý song song thường gồm 3 bước cơ bản,:

1. **Split (Chia nhỏ):** Chia dữ liệu thành N phần.
2. **Compute (Tính toán):** Giao mỗi phần cho một CPU xử lý song song.
3. **Combine (Kết hợp):** Một bộ xử lý duy nhất thu thập kết quả từ N phần và tổng hợp lại.

- **Hạn chế:** Bước "Combine" trở thành nút thắt cổ chai (bottleneck) vì chỉ có một bộ xử lý thực hiện, không thể xử lý nổi lượng dữ liệu khổng lồ của Big Data.

### 2. Giải pháp: Mô hình MapReduce

MapReduce là một mô hình lập trình (được Google tiên phong) nhằm giải quyết nút thắt trên bằng cách **song song hóa cả bước Combine**.

Quy trình MapReduce gồm 5 bước khái niệm (Data flow),:

1. **Split:** Chia dữ liệu đầu vào thành các phần nhỏ.
2. **Map:** Các bộ xử lý "Map" xử lý dữ liệu thô và tạo ra các cặp kết quả trung gian `<key, value>`.
3. **Shuffle (Tráo đổi):** Đây là bước quan trọng nhất. Hệ thống tự động phân phối lại dữ liệu dựa trên `key`. Tất cả dữ liệu có cùng `key` sẽ được đẩy về cùng một nơi.
4. **Reduce:** Các bộ xử lý "Reduce" nhận nhóm dữ liệu (đã được Shuffle) và thực hiện tổng hợp song song.
5. **Merge:** Hợp nhất các kết quả cuối cùng.

> **Ví dụ kinh điển (Word Count):** Đếm số lần xuất hiện của mỗi từ trong một thư viện sách khổng lồ.
> 
> - **Map:** Đọc sách, gặp chữ "Cloud" thì xuất ra `<Cloud, 1>`.
> - **Shuffle:** Gom tất cả các cặp có key là "Cloud" về một máy, tất cả key "Server" về một máy khác.
> - **Reduce:** Máy nhận được danh sách `[<Cloud, 1>, <Cloud, 1>, ...]` sẽ cộng tổng lại thành `<Cloud, 10500>`.

### 3. Apache Hadoop & HDFS

Chương này giới thiệu **Hadoop** là công cụ nguồn mở phổ biến nhất để thực hiện MapReduce. Hadoop gồm 2 thành phần chính hoạt động song kiếm hợp bích:

#### A. HDFS (Hadoop Distributed File System) - Lưu trữ

Hệ thống tệp phân tán được thiết kế để lưu trữ dữ liệu khổng lồ trên phần cứng phổ thông.

- **Block lớn:** Chia file thành các khối rất lớn (mặc định 128MB) thay vì vài KB như hệ điều hành thường.
- **Replication (Sao chép):** Mỗi khối được lưu thành 3 bản sao trên các máy khác nhau để chịu lỗi (Fault Tolerance).
- **Kiến trúc:**
    - **NameNode (Master):** Quản lý mục lục, biết block nào nằm ở đâu (metadata).
    - **DataNode (Slave):** Nơi thực sự chứa dữ liệu.

#### B. MapReduce Engine - Tính toán

- **JobTracker (Master):** Nhận công việc từ người dùng, chia nhỏ và phân phối cho máy con.
- **TaskTracker (Slave):** Thực hiện lệnh Map hoặc Reduce cụ thể.

### 4. Nguyên lý cốt lõi: Data Locality (Định vị dữ liệu)

Một khái niệm cực kỳ quan trọng trong chương này là **"Colocation of computation"** (Đặt tính toán cùng chỗ với dữ liệu).

- Thay vì tải dữ liệu (hàng Petabyte) về máy chủ để xử lý (tốn băng thông mạng khủng khiếp).
- Hadoop gửi đoạn mã chương trình (vài KB/MB) đến trực tiếp **DataNode** đang chứa dữ liệu đó để chạy.

### Tóm lại

Chương 11 giải thích cách chúng ta chuyển từ việc "cố gắng làm một máy tính chạy nhanh hơn" sang việc "chia nhỏ bài toán để hàng nghìn máy tính cùng làm". Nó là nền tảng cho kỷ nguyên Big Data.

---

# Phân Tích Sâu Chương 11: The MapReduce Paradigm

### 1. Vấn đề gốc rễ: Nút thắt cổ chai của xử lý song song

Trước khi có MapReduce, việc xử lý dữ liệu lớn trên cụm máy tính (Cluster) gặp phải một thách thức lớn ở bước **Combine** (Kết hợp).

- **Mô hình cũ:** Bạn chia dữ liệu thành $N$ phần (Split), giao cho $N$ máy xử lý song song (Compute). Tuy nhiên, để ra kết quả cuối cùng, bạn thường phải dồn hết kết quả về **một máy duy nhất** để tổng hợp.
- **Hệ quả:** Máy "Combine" trở thành nút thắt cổ chai (bottleneck), không thể mở rộng (scale) khi dữ liệu tăng lên hàng Petabyte.
- **Giải pháp của MapReduce:** Thay vì dùng 1 máy để combine, MapReduce **song song hóa cả bước Combine** thông qua cơ chế _Shuffle_ và _Reduce_.

### 2. Giải phẫu quy trình MapReduce (The Mechanics)

MapReduce không chỉ là "chia việc", nó là một quy trình chuyển đổi dữ liệu chặt chẽ gồm 5 bước. Điểm mấu chốt nằm ở cách dữ liệu di chuyển giữa các bước.

#### A. Bước 1 & 2: Split & Map (Cục bộ hóa)

- **Split:** Dữ liệu đầu vào được chia logic thành các phần nhỏ (splits).
- **Map:** Hàm `Map(k1, v1) -> list(k2, v2)` xử lý dữ liệu thô và tạo ra các cặp khóa-giá trị trung gian (intermediate key-value pairs).
    - _Chi tiết sâu:_ Bước này diễn ra hoàn toàn song song và độc lập. Quan trọng nhất, code Map được gửi đến nơi có dữ liệu (**Data Locality**) để chạy, tránh nghẽn mạng,.

#### B. Bước 3: Shuffle (Trái tim của MapReduce)

Đây là bước "ma thuật" và cũng là nơi tốn băng thông nhất. Hệ thống phải đảm bảo **toàn bộ dữ liệu có cùng một Key ($k2$) từ tất cả các máy Map phải được chuyển về cùng một máy Reduce**.

- MapReduce framework tự động xử lý việc này: Sắp xếp (Sort), gom nhóm và truyền tải qua mạng.
- _Ví dụ:_ Nếu bạn đếm từ (Word Count), tất cả các cặp `<"Cloud", 1>` từ hàng nghìn máy Map sẽ được đẩy về duy nhất máy Reduce chịu trách nhiệm cho vần "C",.

#### C. Bước 4 & 5: Reduce & Merge (Tổng hợp song song)

- **Reduce:** Hàm `Reduce(k2, list(v2)) -> list(v3)` nhận vào một Key và một danh sách các giá trị tương ứng để xử lý (ví dụ: cộng tổng).
- Vì có nhiều Key khác nhau, ta có thể chạy nhiều máy Reduce song song (Partitioning by key).

### 3. Hadoop & HDFS: Cặp bài trùng (Symbiosis)

Chương 11 nhấn mạnh rằng MapReduce không thể hoạt động hiệu quả nếu thiếu một hệ thống tệp tin hỗ trợ nó, đó là **HDFS (Hadoop Distributed File System)**. Hai thành phần này được thiết kế "đo ni đóng giày" cho nhau.

#### Tại sao HDFS lại thiết kế "kỳ lạ" như vậy?

HDFS không giống NTFS hay EXT4. Nó hy sinh tính năng để đổi lấy hiệu năng cho MapReduce:

1. **Block Size khổng lồ (64MB/128MB):**
    - _Lý do:_ Để giảm chi phí seek (tìm kiếm) trên đĩa cứng. MapReduce chủ yếu đọc tuần tự (streaming read). Block lớn giúp thời gian đọc dữ liệu chiếm tỉ trọng lớn hơn thời gian tìm đầu đọc,.
2. **Chỉ có Read & Append:**
    - _Lý do:_ MapReduce không cần sửa đổi (update) một dòng ở giữa file. Nó đọc vào, xử lý và ghi ra file mới. Việc loại bỏ tính năng "sửa file" giúp HDFS đơn giản hóa việc quản lý tính nhất quán (consistency) khi có nhiều bản sao,.
3. **Rack Awareness (Nhận thức vị trí):**
    - HDFS biết rõ server nào nằm cùng Rack. Khi sao chép dữ liệu (mặc định 3 bản), nó sẽ đặt bản 1 ở máy cục bộ, bản 2 ở Rack khác, bản 3 ở máy khác cùng Rack với bản 2. Điều này giúp chịu lỗi (mất cả Rack vẫn còn data) và tối ưu băng thông khi đọc,.

### 4. Khả năng chịu lỗi (Fault Tolerance)

Đây là điểm mạnh nhất của mô hình này khi chạy trên hàng nghìn máy chủ "rẻ tiền" (commodity hardware) vốn rất dễ hỏng.

- **Worker Failure (Máy con chết):** Master (JobTracker) ping các worker định kỳ. Nếu worker chết, Master đơn giản là gán tác vụ đó cho một worker khác (nơi có chứa bản sao dữ liệu của tác vụ đó). Cơ chế này hoàn toàn tự động.
- **Speculative Execution (Thực thi dự phòng):** Nếu một máy chạy quá chậm (do ổ cứng lỗi, CPU yếu), Hadoop sẽ âm thầm chạy một bản sao của tác vụ đó trên máy khác. Máy nào xong trước thì lấy kết quả, giết tác vụ còn lại.

### 5. Hạn chế và Đánh đổi (Trade-offs)

Phân tích sâu cần nhìn nhận cả mặt yếu,:

- **Độ trễ cao (High Latency):** MapReduce không dành cho các truy vấn thời gian thực (online). Nó là xử lý theo lô (Batch processing). Bạn không thể dùng nó để chạy website.
- **Small Files Problem:** HDFS cực dở khi chứa hàng triệu file nhỏ (dưới 128MB) vì NameNode phải giữ metadata trong RAM, gây quá tải bộ nhớ Master.
- **Data Motion:** Dù cố gắng Data Locality, nhưng bước Shuffle vẫn tạo ra lưu lượng mạng ngang (East-West traffic) rất lớn trong Data Center.

### Tóm lại

Chương 11 cho thấy sự chuyển dịch tư duy từ **"Scale-up"** (mua máy mạnh hơn) sang **"Scale-out"** (chia nhỏ bài toán).

- **MapReduce** là giải thuật để chia nhỏ tính toán.
- **HDFS** là giải pháp để chia nhỏ dữ liệu.
- Sự kết hợp này cho phép xử lý dữ liệu ở quy mô mà không một siêu máy tính đơn lẻ nào làm được.
Dựa trên các tài liệu đã cung cấp (đặc biệt là _The Cloud Computing Book - Chapter 11_, các slide về _Big Data Solutions_, _Hadoop_, _MapReduce_, và _Spark_), tôi sẽ phân tích sâu về Apache Hadoop, đánh giá ưu nhược điểm của nó trong mô hình MapReduce và so sánh với các công nghệ khác.

### 1. Phân tích sâu kiến trúc Apache Hadoop

Hadoop không chỉ là một phần mềm đơn lẻ mà là một hệ sinh thái mã nguồn mở (Java-based) cho phép xử lý phân tán các tập dữ liệu lớn trên các cụm máy tính (clusters) sử dụng phần cứng phổ thông (commodity hardware),.

Hadoop giải quyết bài toán Big Data bằng cách chia nhỏ vấn đề (chia để trị) thông qua hai thành phần cốt lõi hoạt động cộng sinh chặt chẽ,:

#### A. HDFS (Hadoop Distributed File System) - Tầng lưu trữ

Khác với các hệ thống tệp thông thường, HDFS được thiết kế đặc thù cho việc xử lý hàng loạt (batch processing) với băng thông dữ liệu cao:

- **Block Size khổng lồ:** HDFS chia tệp thành các khối (blocks) rất lớn, mặc định là **128MB** (so với vài KB của OS thường),. Mục đích là để giảm chi phí tìm kiếm (seek time) trên đĩa cứng, tối ưu cho việc đọc tuần tự (streaming access).
- **Master/Slave Architecture:**
    - **NameNode (Master):** Quản lý metadata (tên file, quyền truy cập, vị trí block). Nó giữ toàn bộ metadata trong RAM để truy xuất nhanh, nhưng điều này tạo ra giới hạn về số lượng file (vấn đề "Small file problem"),.
    - **DataNode (Slave):** Nơi thực sự lưu trữ dữ liệu. Nó thực hiện đọc/ghi theo yêu cầu và báo cáo trạng thái (heartbeat) về NameNode.
- **Cơ chế sao chép (Replication):** Mặc định mỗi block được sao chép ra **3 bản** trên các node khác nhau để chịu lỗi. Hadoop có khả năng nhận biết topology mạng (Rack-aware), giúp nó đặt các bản sao ở các rack khác nhau để phòng ngừa sự cố mất điện/mạng toàn rack,.

#### B. MapReduce & YARN - Tầng tính toán & Điều phối

- **MapReduce Paradigm:** Là mô hình lập trình để xử lý song song. Quy trình gồm: _Split $\rightarrow$ Map $\rightarrow$ Shuffle $\rightarrow$ Reduce $\rightarrow$ Merge_,.
- **Data Locality (Định vị dữ liệu):** Đây là nguyên lý "vàng" của Hadoop. Thay vì di chuyển dữ liệu đến nơi có CPU (gây nghẽn mạng), Hadoop di chuyển mã lệnh (Map tasks) đến trực tiếp DataNode đang chứa dữ liệu đó để xử lý cục bộ,.
- **YARN (Yet Another Resource Negotiator):** Trong Hadoop 2.x trở đi, việc quản lý tài nguyên được tách khỏi MapReduce và giao cho YARN. YARN gồm _ResourceManager_ (quản lý toàn cục) và _NodeManager_ (quản lý từng node), giúp Hadoop chạy được nhiều loại ứng dụng khác ngoài MapReduce,.

---

### 2. Ưu và Nhược điểm của Hadoop trong mô hình MapReduce

#### Ưu điểm (Pros)

1. **Khả năng mở rộng tuyến tính (Scalability):** Có thể mở rộng từ một vài server lên hàng nghìn server (Scale-out) một cách dễ dàng. Hiệu năng tăng tỷ lệ thuận với số lượng node,.
2. **Khả năng chịu lỗi cao (Fault Tolerance):**
    - Về dữ liệu: Nếu một ổ cứng hỏng, HDFS tự động phục hồi từ các bản sao khác.
    - Về tính toán: Nếu một TaskTracker (worker) bị chậm hoặc chết, Master sẽ khởi động lại tác vụ đó trên một node khác chứa bản sao dữ liệu tương ứng,.
3. **Chi phí thấp (Cost Effectiveness):** Chạy trên phần cứng phổ thông (commodity hardware) rẻ tiền, không cần siêu máy tính hay storage chuyên dụng (SAN/NAS),.
4. **Tính linh hoạt (Flexibility):** Hỗ trợ nhiều ngôn ngữ lập trình. Dù viết bằng Java, Hadoop hỗ trợ _Hadoop Streaming_ cho phép viết Map/Reduce bằng Python, C++, Ruby, v.v.,.

#### Nhược điểm (Cons)

1. **Độ trễ cao (High Latency):** Hadoop MapReduce được thiết kế cho xử lý theo lô (batch). Nó không phù hợp cho các truy vấn thời gian thực (real-time) hoặc các tác vụ yêu cầu phản hồi tức thì (mili giây),.
2. **I/O Bottleneck (Nút thắt nhập xuất):** Cơ chế của MapReduce buộc phải ghi kết quả trung gian (giữa bước Map và Reduce) xuống đĩa cứng (HDFS) để đảm bảo an toàn. Việc đọc/ghi đĩa liên tục làm chậm đáng kể quá trình xử lý, đặc biệt với các thuật toán lặp (iterative algorithms) như Machine Learning,.
3. **Vấn đề tệp nhỏ (Small File Problem):** HDFS hoạt động kém hiệu quả với hàng triệu tệp nhỏ (nhỏ hơn block size) vì NameNode bị quá tải bộ nhớ khi quản lý metadata cho quá nhiều tệp.
4. **Khó lập trình:** Viết một chương trình MapReduce nguyên thủy rất phức tạp và khó debug. (Tuy nhiên các công cụ như Pig/Hive đã giảm bớt gánh nặng này).

---

### 3. So sánh Hadoop với các công nghệ khác

#### A. Hadoop MapReduce vs. Apache Spark

Đây là sự so sánh quan trọng nhất trong hệ sinh thái Big Data hiện đại.

| Tiêu chí               | Hadoop MapReduce                                                                      | Apache Spark                                                                                                 |
| :--------------------- | :------------------------------------------------------------------------------------ | :----------------------------------------------------------------------------------------------------------- |
| **Cơ chế xử lý**       | **Disk-based:** Ghi kết quả trung gian xuống đĩa cứng HDFS sau mỗi bước.              | **In-memory:** Sử dụng RDD (Resilient Distributed Datasets) để lưu trữ và xử lý dữ liệu ngay trên RAM,.      |
| **Tốc độ**             | Chậm hơn do I/O đĩa.                                                                  | Nhanh hơn **100 lần** trong bộ nhớ và **10 lần** trên đĩa so với Hadoop.                                     |
| **Trường hợp sử dụng** | Tốt cho xử lý hàng loạt (Batch processing) dữ liệu cực lớn mà không cần kết quả ngay. | Tuyệt vời cho Machine Learning (ML), xử lý luồng (Streaming), và truy vấn tương tác (Interactive analytics). |
| **Độ phức tạp**        | Code dài dòng, khó viết.                                                              | API cấp cao, dễ sử dụng hơn (Java, Scala, Python, R).                                                        |
| **Sự phụ thuộc**       | Thường dùng HDFS để lưu trữ.                                                          | Có thể chạy độc lập, nhưng thường dùng HDFS làm tầng lưu trữ bên dưới (Spark không có file system riêng).    |

#### B. Hadoop vs. RDBMS (Hệ quản trị CSDL quan hệ)

|Tiêu chí|RDBMS (SQL)|Hadoop (NoSQL/Batch)|
|:--|:--|:--|
|**Dữ liệu**|Cấu trúc rõ ràng (Structured), lược đồ tĩnh (Schema-on-write).|Đa dạng: Cấu trúc, bán cấu trúc, phi cấu trúc (text, log, video). Lược đồ động (Schema-on-read).|
|**Mở rộng**|**Scale-up (Vertical):** Nâng cấp phần cứng máy chủ mạnh hơn (tốn kém, có giới hạn).|**Scale-out (Horizontal):** Thêm nhiều máy rẻ tiền vào cụm.|
|**Truy cập**|Tương tác nhanh, độ trễ thấp, hỗ trợ ACID mạnh.|Xử lý lô, độ trễ cao, tính nhất quán cuối cùng (eventual consistency).|

#### C. Hadoop vs. Microservices

Mặc dù hai khái niệm này thuộc về hai phạm trù khác nhau (Data Processing vs. Application Architecture), nhưng trong ngữ cảnh hiện đại:

- **Hadoop:** Là một hệ thống nguyên khối (monolithic) khổng lồ tập trung vào dữ liệu.
- **Microservices:** Tập trung vào chia nhỏ ứng dụng. Tuy nhiên, kiến trúc Microservices gặp khó khăn khi phải xử lý dữ liệu phân tán lớn (vấn đề Distributed Data) và thường phải kết hợp với các hệ thống Big Data để xử lý phân tích,.

### Tổng kết

Hadoop MapReduce là nền tảng tiên phong cho kỷ nguyên Big Data, cực kỳ mạnh mẽ về độ bền và chi phí cho việc lưu trữ/xử lý lô lượng dữ liệu Petabyte. Tuy nhiên, do hạn chế về tốc độ I/O đĩa, nó đang dần nhường chỗ cho **Apache Spark** trong các tác vụ cần tốc độ cao, xử lý thời gian thực và Machine Learning, mặc dù HDFS vẫn là lớp lưu trữ nền tảng phổ biến cho cả hai.

# 🐘 Chapter 11: The MapReduce Paradigm & Hadoop

**Tags:** #CloudComputing #Chapter11 #MapReduce #Hadoop #BigData #HDFS
**Số lượng:** 50 câu
**Mức độ:** Khó & Cực khó (Deep Dive)
**Tài liệu tham chiếu:** The Cloud Computing Book (Douglas Comer)

---

## Phần 1: Nguyên lý Xử lý Song song & MapReduce (Concepts)

### Câu 1
Tại sao việc chuyển các ứng dụng truyền thống (Legacy Apps) lên chạy trên máy ảo (VM) trong đám mây vẫn chưa được gọi là "Cloud-Native"?
A. Vì VM chạy chậm hơn máy vật lý.
B. Vì ứng dụng đó không được thiết kế để tận dụng các đặc tính của đám mây như tính co giãn (elasticity) và xử lý song song (parallelism).
C. Vì ứng dụng đó viết bằng ngôn ngữ lập trình cũ.
D. Vì đám mây không hỗ trợ các hệ điều hành cũ.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Cloud-native software là phần mềm được thiết kế *cụ thể* cho môi trường đám mây để khai thác tối đa lợi thế như khả năng mở rộng (scalability), độ tin cậy thông qua song song hóa, thay vì chỉ "bê nguyên" (lift and shift) ứng dụng cũ lên VM.

### Câu 2
Ba bước cơ bản (conceptual steps) để giải quyết một vấn đề bằng xử lý song song là gì?
A. Read, Process, Write.
B. Partitioning (Chia nhỏ), Parallel Computation (Tính toán song song), Combination (Kết hợp kết quả).
C. Map, Shuffle, Reduce.
D. Input, Output, Error.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Đây là mô hình tổng quát: Chia bài toán lớn thành các mảnh nhỏ (Split/Partition), xử lý đồng thời các mảnh (Compute), và gom kết quả lại (Combine). MapReduce là một hiện thực hóa của mô hình này.

### Câu 3
Trong mô hình MapReduce, bước nào chịu trách nhiệm điều phối và chuyển dữ liệu từ các node thực hiện hàm Map sang các node thực hiện hàm Reduce?
A. Split.
B. Merge.
C. Shuffle.
D. Combine.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** **Shuffle** là quá trình phân phối lại dữ liệu: các kết quả trung gian (intermediate results) có cùng khóa (key) từ nhiều node Map khác nhau sẽ được gom lại và chuyển đến cùng một node Reduce.

### Câu 4
Nếu một bài toán có thể giải quyết nhanh chóng bằng một máy tính duy nhất, việc áp dụng MapReduce sẽ dẫn đến điều gì?
A. Tốc độ xử lý tăng lên gấp nhiều lần.
B. Tiết kiệm tài nguyên CPU.
C. Hiệu năng giảm do chi phí quản lý (overhead) không cần thiết.
D. Kết quả chính xác hơn.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** MapReduce sinh ra overhead (chia nhỏ, truyền mạng, khởi động task). Với dữ liệu nhỏ, thời gian xử lý overhead còn lớn hơn thời gian tính toán thực tế.

### Câu 5
Hàm **Map** trong MapReduce thực hiện nhiệm vụ gì về mặt logic?
A. Gom nhóm các dữ liệu giống nhau.
B. Xử lý một phần dữ liệu đầu vào và tạo ra các cặp khóa-giá trị (key-value pairs) trung gian.
C. Tính tổng các giá trị.
D. Chia file đầu vào thành các khối nhỏ.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Hàm Map nhận input (thường là một phần của dữ liệu gốc), xử lý và xuất ra danh sách các cặp `(k, v)`. Ví dụ: Đếm từ thì Map xuất ra `(word, 1)`.

### Câu 6
Trong ví dụ đếm từ (Word Count) kinh điển, input của hàm **Reduce** là gì?
A. Toàn bộ văn bản gốc.
B. Một từ duy nhất và tổng số lần xuất hiện của nó.
C. Một từ (khóa) và danh sách các con số 1 (list of counts) từ các hàm Map gửi tới.
D. Danh sách các file kết quả.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** Sau bước Shuffle, Reduce nhận được `(key, [list_of_values])`. Ví dụ: `("Cloud", [1, 1, 1, 1])`. Nhiệm vụ của Reduce là cộng tổng danh sách đó lại.

### Câu 7
Tại sao các thuật toán cơ sở dữ liệu (DBMS) hiện đại đôi khi được coi là đối thủ của MapReduce?
A. Vì DBMS rẻ hơn.
B. Vì DBMS hiện đại cũng hỗ trợ xử lý song song (Parallel Query Processing) và có thể hiệu quả hơn MapReduce cho các dữ liệu có cấu trúc.
C. Vì DBMS không cần lưu trữ đĩa.
D. Vì MapReduce không hỗ trợ SQL.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Các hệ quản trị CSDL song song (Parallel RDBMS) đã tối ưu hóa việc query dữ liệu lớn từ lâu. MapReduce mạnh hơn ở chỗ xử lý dữ liệu *phi cấu trúc* (unstructured data) hoặc bán cấu trúc.

### Câu 8
Vấn đề lớn nhất về hiệu năng (Performance Bottleneck) khi thực hiện xử lý song song trên quy mô lớn thường là gì?
A. Tốc độ CPU chậm.
B. Thiếu RAM.
C. Communication Overhead (Chi phí truyền tải dữ liệu qua mạng).
D. Lỗi phần mềm.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** Trong các tác vụ I/O-bound, việc di chuyển hàng Terabyte dữ liệu từ nơi lưu trữ đến nơi tính toán (và giữa các node Map/Reduce) tốn rất nhiều thời gian, có thể làm lu mờ lợi ích của việc tính toán song song.

### Câu 9
Thuật ngữ "I/O-bound computation" ám chỉ điều gì?
A. Chương trình bị lỗi Input/Output.
B. Thời gian đọc/ghi dữ liệu (từ đĩa hoặc mạng) chiếm phần lớn thời gian thực thi, nhiều hơn thời gian tính toán của CPU.
C. Chương trình chỉ chạy trên thiết bị I/O.
D. CPU hoạt động 100% công suất.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Với Big Data, hầu hết các tác vụ là I/O-bound (chờ đọc dữ liệu). MapReduce và Hadoop được thiết kế để tối ưu cho loại tác vụ này.

### Câu 10
Đâu là thứ tự đúng của chuỗi xử lý trong MapReduce?
A. Map -> Reduce -> Shuffle -> Split -> Merge.
B. Split -> Map -> Shuffle -> Reduce -> Merge.
C. Split -> Shuffle -> Map -> Reduce -> Merge.
D. Map -> Split -> Shuffle -> Merge -> Reduce.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** > 1. **Split:** Chia input. 
> 2. **Map:** Xử lý song song. 
> 3. **Shuffle:** Chuyển dữ liệu giữa các máy (sort & group). 
> 4. **Reduce:** Tổng hợp. 
> 5. **Merge:** Ghi kết quả cuối cùng.

---

## Phần 2: Kiến trúc Hadoop & HDFS

### Câu 11
Hai thành phần chính cấu tạo nên Apache Hadoop là gì?
A. MapReduce Engine và HDFS (Hadoop Distributed File System).
B. Docker và Kubernetes.
C. SQL và NoSQL.
D. Java và Python.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: A**
> **Giải thích:** Hadoop gồm 2 phần cốt lõi: **HDFS** để lưu trữ dữ liệu phân tán và **Processing Component** (thường là MapReduce hoặc YARN sau này) để xử lý dữ liệu đó.

### Câu 12
Trong kiến trúc HDFS, **NameNode** đóng vai trò gì?
A. Lưu trữ nội dung thực tế của các file.
B. Thực hiện tính toán MapReduce.
C. Lưu trữ Metadata (siêu dữ liệu): Tên file, thư mục, và bản đồ vị trí các block (file nào nằm ở DataNode nào).
D. Sao lưu dữ liệu dự phòng.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** NameNode là "bộ não" quản lý không gian tên (namespace). Nó không chứa dữ liệu file (block data), chỉ chứa thông tin chỉ đường.

### Câu 13
Kích thước mặc định của một Block trong HDFS (phiên bản hiện đại) thường là bao nhiêu?
A. 4 KB (giống ổ cứng thường).
B. 64 KB.
C. 128 MB (hoặc lớn hơn).
D. 1 GB.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** Block size của HDFS rất lớn (64MB hoặc 128MB) để giảm thiểu chi phí tìm kiếm (seek time) trên đĩa và tối ưu cho việc đọc tuần tự lượng dữ liệu lớn.

### Câu 14
Nếu một **DataNode** bị hỏng ổ cứng, dữ liệu có bị mất không và tại sao?
A. Có, vì dữ liệu chỉ lưu một nơi.
B. Không, nhờ cơ chế **Replication** (Sao chép). Mặc định mỗi block được lưu 3 bản trên các node khác nhau.
C. Không, vì NameNode chứa bản sao lưu.
D. Có, trừ khi dùng RAID phần cứng.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** HDFS đảm bảo độ tin cậy bằng phần mềm. Mặc định `replication_factor = 3` (1 bản local, 1 bản ở rack khác, 1 bản ở node khác cùng rack đó).

### Câu 15
Điểm yếu chí tử (Single Point of Failure) trong kiến trúc Hadoop đời đầu (Hadoop 1.x) là gì?
A. DataNode.
B. NameNode.
C. MapReduce Job.
D. Network Switch.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Nếu NameNode chết, toàn bộ Cluster ngừng hoạt động vì không ai biết dữ liệu nằm ở đâu (dù DataNode vẫn sống). Các phiên bản sau (Hadoop 2.x+) đã có High Availability (HA) NameNode.

### Câu 16
Khái niệm **"Rack Awareness"** (Nhận thức về tủ Rack) trong Hadoop có ý nghĩa gì khi lưu trữ dữ liệu?
A. Hadoop chỉ lưu dữ liệu trên một Rack duy nhất để chạy nhanh.
B. Hadoop cố gắng phân tán các bản sao (replicas) của một block sang các Rack khác nhau để đề phòng trường hợp nguyên một Rack bị mất điện hoặc hỏng switch.
C. Hadoop không quan tâm đến vị trí vật lý.
D. Hadoop yêu cầu các Rack phải có màu khác nhau.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Để tối ưu khả năng chịu lỗi (Fault Tolerance), HDFS không đặt cả 3 bản sao trên cùng 1 Rack. Nếu Rack đó cháy, dữ liệu vẫn còn ở Rack khác.

### Câu 17
Mô hình phần cứng điển hình cho một Cluster Hadoop là gì?
A. Sử dụng siêu máy tính đắt tiền.
B. Sử dụng SAN (Storage Area Network) tập trung.
C. Sử dụng hàng ngàn máy chủ phổ thông (commodity hardware), mỗi máy có ổ cứng cục bộ (DAS) riêng.
D. Không dùng ổ cứng, chỉ dùng RAM.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** Hadoop thiết kế để chạy trên phần cứng rẻ tiền. Triết lý là "Di chuyển tính toán đến nơi có dữ liệu" (Move computation to data), nên mỗi node vừa tính toán vừa lưu trữ.

### Câu 18
HDFS hỗ trợ tốt nhất cho loại thao tác ghi dữ liệu nào?
A. Random Write (Ghi ngẫu nhiên vào giữa file).
B. Update (Cập nhật sửa đổi dữ liệu cũ).
C. Append-only (Chỉ ghi nối thêm vào cuối file) hoặc ghi một lần đọc nhiều lần (Write-once-read-many).
D. Transactional Write (Ghi giao dịch phức tạp).

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** HDFS không phải là hệ thống file đa năng (như NTFS/Ext4). Nó tối ưu cho MapReduce: Ghi luồng dữ liệu lớn một lần và không cho phép sửa đổi nội dung đã ghi (chỉ cho phép append trong một số trường hợp).

### Câu 19
Khi NameNode khởi động, nó lấy thông tin về vị trí các block (Block locations) từ đâu?
A. Đọc từ ổ cứng của chính NameNode.
B. Nhận báo cáo (Block Report) từ tất cả các DataNode gửi về.
C. Hỏi người quản trị.
D. Tự quét toàn bộ mạng.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Metadata file (fsimage) trên đĩa của NameNode chỉ lưu cấu trúc thư mục. Vị trí block thực tế được DataNode báo cáo động khi hệ thống khởi động. Điều này giúp hệ thống linh hoạt khi DataNode thêm/bớt.

### Câu 20
Trong Hadoop, **"Data Locality"** (Tính cục bộ của dữ liệu) là nguyên tắc gì?
A. Luôn di chuyển dữ liệu đến node có CPU mạnh nhất.
B. Ưu tiên lập lịch thực hiện Map Task ngay trên node đang chứa block dữ liệu cần xử lý (để tránh truyền qua mạng).
C. Lưu dữ liệu tại máy của người dùng.
D. Chỉ xử lý dữ liệu ở một vùng địa lý.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Đây là chìa khóa hiệu năng của Hadoop. Thay vì kéo 1TB dữ liệu về máy chủ tính toán (tốn băng thông), Hadoop gửi đoạn code nhỏ (vài KB) đến máy đang giữ 1TB dữ liệu đó để chạy.

---

## Phần 3: Cơ chế hoạt động & Tối ưu hóa (Deep Dive)

### Câu 21
Tại sao kích thước Split (phần dữ liệu chia nhỏ cho mỗi Map task) nên bằng với kích thước HDFS Block (ví dụ 128MB)?
A. Để cho đẹp.
B. Để đảm bảo mỗi Map task chỉ xử lý dữ liệu cục bộ trên node đó, tránh việc một task phải đọc một nửa block ở node này và một nửa block ở node kia qua mạng.
C. Để giảm số lượng Map task.
D. HDFS bắt buộc phải làm vậy.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Tối ưu hóa Data Locality. Nếu Split size > Block size, Map task sẽ phải kéo phần dữ liệu thừa từ node khác về -> Phá vỡ tính cục bộ.

### Câu 22
Hadoop xử lý thế nào khi một DataNode chứa dữ liệu đang chạy Map Task thì bị chết (fail)?
A. Map Task đó bị hủy và công việc thất bại hoàn toàn.
B. Master node phát hiện lỗi, tìm một DataNode khác đang giữ bản sao (replica) của block dữ liệu đó, và khởi động lại Map Task trên node mới này.
C. Hệ thống chờ DataNode đó sống lại.
D. Người quản trị phải can thiệp thủ công.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Đây là tính năng **Self-healing** và **Fault Tolerance**. Nhờ cơ chế Replication, dữ liệu luôn có bản dự phòng để tính toán lại.

### Câu 23
Quá trình **Shuffle** trong Hadoop MapReduce bao gồm các thao tác nào?
A. Chỉ copy dữ liệu.
B. Copy, Sort (Sắp xếp) và Merge. Dữ liệu từ các Map task được sắp xếp theo Key trước khi gửi đến Reduce.
C. Xóa dữ liệu tạm.
D. Mã hóa dữ liệu.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Shuffle là giai đoạn tốn kém nhất. Nó đảm bảo Reduce nhận được dữ liệu đã được gom nhóm: tất cả values của cùng một Key sẽ nằm liền nhau (do đã sort).

### Câu 24
Nếu input data là một file nén dạng **GZIP**, điều gì sẽ xảy ra với tính năng Parallelism của MapReduce?
A. Không ảnh hưởng gì.
B. Hiệu năng giảm vì GZIP không hỗ trợ split (non-splittable). Toàn bộ file GZIP phải được xử lý bởi duy nhất 1 Map Task, làm mất khả năng song song.
C. Hadoop tự động giải nén song song.
D. File GZIP sẽ bị xóa.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Đây là câu hỏi thực tiễn quan trọng. Một số định dạng nén (như GZIP) không thể chia nhỏ để đọc từ giữa file. Do đó Hadoop buộc phải dùng 1 Map cho cả file to -> Nghẽn cổ chai. Nên dùng các định dạng splittable như BZIP2 hoặc LZO (có index).

### Câu 25
Trong Hadoop, **Speculative Execution** (Thực thi đầu cơ) giải quyết vấn đề gì?
A. Dự đoán kết quả trước khi chạy.
B. Xử lý vấn đề "Straggler" (Task chạy quá chậm). Hadoop sẽ chạy thêm một bản sao của task chậm đó trên node khác. Cái nào xong trước thì lấy, cái kia hủy.
C. Tiết kiệm điện năng.
D. Chạy task vào giờ thấp điểm.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Đôi khi một node bị lỗi đĩa hoặc quá tải khiến task chạy rất chậm (nhưng không chết). Cơ chế này giúp job không bị kéo dài vô tận chỉ vì 1 task chậm.

### Câu 26
Khi sử dụng Hadoop với các hệ thống lưu trữ tách biệt (như NAS hoặc Object Storage S3) thay vì HDFS cục bộ, nhược điểm chính là gì?
A. Không thể chạy được MapReduce.
B. Mất tính năng **Data Locality**. Dữ liệu phải di chuyển qua mạng từ kho lưu trữ đến server tính toán, gây nghẽn mạng.
C. Dữ liệu bị mất an toàn.
D. Code phức tạp hơn.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Kiến trúc Hadoop cổ điển (HDFS) gắn liền disk với compute node. Khi dùng S3 hay NAS, compute và storage tách rời -> Phải truyền dữ liệu qua mạng (Network bottleneck).

### Câu 27
Kết quả trung gian (Intermediate output) của hàm Map được lưu ở đâu?
A. Trên HDFS (có 3 bản replication).
B. Trên ổ cứng cục bộ (Local Disk) của node chạy Map task.
C. Trên RAM của NameNode.
D. Gửi thẳng sang Reduce mà không lưu.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Kết quả trung gian là tạm thời (temporary), chỉ cần thiết cho đến khi job xong. Lưu trên HDFS (với 3 bản copy) là lãng phí tài nguyên IO/Network không cần thiết.

### Câu 28
Nếu số lượng **Reduce Task** được cấu hình là 0, chuyện gì xảy ra?
A. Lỗi chương trình.
B. Job chỉ chạy giai đoạn Map (Map-only job), kết quả của Map được ghi thẳng ra HDFS mà không qua Shuffle/Sort.
C. Job không chạy gì cả.
D. Hadoop tự động set về 1.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Map-only job rất phổ biến cho các tác vụ ETL đơn giản (ví dụ: lọc dữ liệu, đổi định dạng) không cần gom nhóm (aggregation).

### Câu 29
Tại sao NameNode nên có nhiều RAM?
A. Để chạy MapReduce nhanh hơn.
B. Để lưu trữ toàn bộ Metadata của hệ thống file trong bộ nhớ (In-memory) giúp truy xuất nhanh.
C. Để cache dữ liệu người dùng.
D. Để chạy giao diện đồ họa.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Giới hạn số lượng file trong HDFS phụ thuộc vào dung lượng RAM của NameNode (vì mỗi object file/block tốn khoảng 150 bytes RAM metadata).

### Câu 30
Combiner trong MapReduce là gì?
A. Là hàm Reduce chạy trên phía Map (Mini-Reducer).
B. Là công cụ gộp file.
C. Là bước cuối cùng của Job.
D. Là phần mềm nén dữ liệu.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: A**
> **Giải thích:** Combiner giúp giảm lượng dữ liệu gửi qua mạng trong bước Shuffle bằng cách thực hiện tính tổng cục bộ ngay sau Map. Ví dụ: Map ra `(the, 1), (the, 1)`, Combiner gộp thành `(the, 2)` trước khi gửi đi.

---

## Phần 4: So sánh & Thực tiễn (Practical & Tricky)

### Câu 31
Sự khác biệt lớn nhất giữa **Hadoop MapReduce** và **Apache Spark** là gì?
A. Spark không dùng HDFS.
B. MapReduce ghi kết quả trung gian xuống đĩa (Disk-based), trong khi Spark cố gắng xử lý và lưu trữ dữ liệu trong bộ nhớ (In-memory processing).
C. Spark chỉ chạy trên Windows.
D. MapReduce nhanh hơn Spark.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Spark nhanh hơn MapReduce (có thể tới 100 lần) nhờ tránh được thao tác Disk I/O nặng nề giữa các bước xử lý. Spark giữ dữ liệu trong RAM (RDDs).

### Câu 32
Trong trường hợp nào ta **KHÔNG** nên sử dụng Hadoop/MapReduce?
A. Xử lý Petabytes log file.
B. Xử lý giao dịch thời gian thực (Real-time Transaction/OLTP) cần độ trễ thấp (low latency).
C. Phân tích dữ liệu lịch sử.
D. Xây dựng chỉ mục tìm kiếm (Search Index).

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** MapReduce là xử lý theo lô (Batch Processing), có độ trễ khởi động cao. Nó không phù hợp cho các ứng dụng cần phản hồi tức thì (ms) như giao dịch ngân hàng hay web app tương tác.

### Câu 33
Hadoop hỗ trợ viết MapReduce bằng ngôn ngữ nào?
A. Chỉ Java.
B. Java là native, nhưng hỗ trợ các ngôn ngữ khác (Python, C++, Ruby...) qua cơ chế **Hadoop Streaming** và **Pipes**.
C. Chỉ C++.
D. Chỉ SQL.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Dù Hadoop viết bằng Java, Hadoop Streaming cho phép dùng bất kỳ ngôn ngữ nào có thể đọc/ghi từ standard input/output (stdin/stdout) để viết Map/Reduce.

### Câu 34
YARN (Yet Another Resource Negotiator) được sinh ra trong Hadoop 2.0 để làm gì?
A. Thay thế HDFS.
B. Tách biệt chức năng quản lý tài nguyên (Resource Management) khỏi chức năng xử lý dữ liệu, cho phép Hadoop chạy nhiều loại ứng dụng khác ngoài MapReduce (như Spark, Storm).
C. Làm giao diện web đẹp hơn.
D. Tăng dung lượng lưu trữ.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Trước YARN, Hadoop chỉ chạy được MapReduce. YARN biến Hadoop thành một hệ điều hành dữ liệu, nơi MapReduce chỉ là một ứng dụng chạy trên đó.

### Câu 35
Lệnh `hadoop fs -put` thực hiện điều gì?
A. Tải một file từ HDFS về máy local.
B. Upload một file từ máy local lên HDFS.
C. Xóa file trên HDFS.
D. Di chuyển file trong HDFS.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Đây là lệnh cơ bản để đưa dữ liệu vào "kho" HDFS. Quá trình này sẽ chia file thành block và phân tán chúng đi.

### Câu 36
"Straggler" trong thuật ngữ MapReduce ám chỉ điều gì?
A. Một node bị hỏng hoàn toàn.
B. Một task chạy chậm hơn hẳn các task khác, làm kéo dài thời gian hoàn thành của cả Job.
C. Một loại virus.
D. Dữ liệu rác.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** "Straggler" là kẻ tụt hậu. Nó quyết định tốc độ của cả đoàn tàu. Cơ chế Speculative Execution sinh ra để xử lý nó.

### Câu 37
Tại sao nói Hadoop MapReduce tuân theo mô hình "Schema-on-Read" thay vì "Schema-on-Write" (như RDBMS)?
A. Vì Hadoop không có Schema.
B. Vì Hadoop kiểm tra cấu trúc dữ liệu khi ghi vào.
C. Vì Hadoop cho phép lưu dữ liệu thô (raw) vào kho mà không cần định nghĩa cấu trúc trước. Cấu trúc chỉ được áp dụng (parse) khi chương trình đọc dữ liệu lên để xử lý.
D. Vì Hadoop chỉ đọc dữ liệu.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: C**
> **Giải thích:** Điều này mang lại sự linh hoạt cực lớn. Bạn có thể dump mọi loại log vào HDFS. Sau này muốn phân tích trường nào thì viết code parse trường đó (khác với SQL phải `CREATE TABLE` định nghĩa cột trước khi `INSERT`).

### Câu 38
InputSplit khác Block như thế nào?
A. Block là đơn vị lưu trữ vật lý, InputSplit là đơn vị xử lý logic.
B. Chúng là một.
C. Block lớn hơn InputSplit.
D. InputSplit nằm trên đĩa, Block nằm trên RAM.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: A**
> **Giải thích:** Block là cách HDFS cắt file (cứng nhắc 128MB). InputSplit là cách MapReduce nhìn dữ liệu. Thông thường 1 Split = 1 Block, nhưng lập trình viên có thể tùy chỉnh InputFormat để 1 Split bao gồm nhiều Block hoặc 1 phần Block.

### Câu 39
Điều gì xảy ra nếu bạn cố gắng sửa đổi (update) một dòng dữ liệu ở giữa một file 100GB trên HDFS?
A. HDFS thực hiện rất nhanh.
B. Không thể thực hiện trực tiếp. Bạn phải đọc file ra, sửa đổi, và ghi lại thành file mới (hoặc append nếu logic cho phép).
C. HDFS chỉ cho phép sửa 1 byte.
D. File sẽ bị corrupt.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Mô hình Write-Once-Read-Many của HDFS không hỗ trợ random write/update để tối ưu hóa thông lượng đọc tuần tự (throughput).

### Câu 40
Amazon EMR (Elastic MapReduce) là ví dụ của mô hình dịch vụ nào?
A. IaaS.
B. PaaS (Platform as a Service) chuyên dụng cho Big Data.
C. SaaS.
D. FaaS.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** EMR cung cấp nền tảng Hadoop/Spark được cài sẵn, quản lý giúp người dùng, người dùng chỉ cần submit job hoặc cấu hình cluster -> PaaS.

---

## Phần 5: Advanced & Tình huống (Scenario)

### Câu 41
Một Job MapReduce đếm từ (WordCount) chạy trên 1TB dữ liệu. Kết quả output chỉ có 1 file duy nhất `part-r-00000`. Điều này nghĩa là gì?
A. Hệ thống bị lỗi.
B. Job chỉ cấu hình 1 Reducer duy nhất.
C. Dữ liệu đầu vào chỉ có 1 từ.
D. MapReduce tự động gộp file.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Số lượng file output bằng số lượng Reducer. Nếu chỉ có 1 file, nghĩa là toàn bộ dữ liệu từ các Map (sau shuffle) đã bị dồn vào 1 Reducer duy nhất. Đây là "bad practice" với dữ liệu lớn vì node đó sẽ quá tải.

### Câu 42
Partitioner trong MapReduce quyết định điều gì?
A. Chia file đầu vào.
B. Quyết định cặp (key, value) nào sẽ được gửi đến Reducer nào.
C. Quyết định thứ tự chạy task.
D. Quyết định nơi lưu file.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Mặc định Partitioner dùng `Hash(key) % num_reducers`. Điều này đảm bảo tất cả các record có cùng Key sẽ luôn về cùng 1 Reducer.

### Câu 43
Vấn đề "Data Skew" (Lệch dữ liệu) trong Reduce phase xảy ra khi nào?
A. Khi dữ liệu quá ít.
B. Khi một Key xuất hiện quá nhiều lần so với các Key khác (ví dụ từ "the" chiếm 80% văn bản), khiến một Reducer phải xử lý lượng dữ liệu khổng lồ trong khi các Reducer khác ngồi chơi.
C. Khi ổ cứng bị đầy.
D. Khi mạng bị chậm.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Data Skew là kẻ thù của xử lý song song. Thời gian hoàn thành Job = thời gian của Reducer chậm nhất (xử lý nhiều nhất).

### Câu 44
Heartbeat trong Hadoop dùng để làm gì?
A. Để đồng bộ đồng hồ.
B. Để DataNode báo cáo cho NameNode biết "tôi còn sống" và báo cáo trạng thái block.
C. Để Map nói chuyện với Reduce.
D. Để người quản trị kiểm tra hệ thống.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Nếu NameNode không nhận được Heartbeat từ DataNode trong khoảng thời gian quy định (mặc định 10 phút), nó coi node đó đã chết và bắt đầu quy trình sao chép dữ liệu (re-replicate) sang node khác.

### Câu 45
DistCp (Distributed Copy) dùng để làm gì?
A. Copy file trong máy local.
B. Copy dữ liệu song song giữa các Cluster Hadoop (hoặc trong cùng Cluster) bằng cách sử dụng MapReduce.
C. Copy file cấu hình.
D. Backup NameNode.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** DistCp biến việc copy file thành một Map-only Job, tận dụng hàng trăm node để copy dữ liệu song song, đạt tốc độ cực cao.

### Câu 46
SafeMode của NameNode là trạng thái gì?
A. Trạng thái an toàn, không cho phép hacker xâm nhập.
B. Trạng thái Read-only (Chỉ đọc). NameNode vào SafeMode khi mới khởi động để kiểm tra đủ số lượng block replica an toàn trước khi cho phép ghi/xóa.
C. Trạng thái tắt máy.
D. Trạng thái chạy chậm.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Khi mới boot, NameNode chưa biết DataNode nào còn sống. Nó chờ nhận Block Report. Trong lúc này, nó cấm mọi thao tác sửa đổi namespace.

### Câu 47
Tại sao HDFS không phù hợp để lưu trữ hàng tỉ file kích thước nhỏ (Small files problem)?
A. Vì tổng dung lượng quá lớn.
B. Vì mỗi file (dù nhỏ) cũng chiếm một entry trong bộ nhớ RAM của NameNode. Hàng tỉ file sẽ làm tràn RAM NameNode.
C. Vì DataNode không đọc được file nhỏ.
D. Vì tốc độ mạng không đủ.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Đây là giới hạn kiến trúc. Giải pháp là gộp các file nhỏ thành file lớn (SequenceFile/HAR) hoặc dùng giải pháp khác như HBase.

### Câu 48
Zookeeper đóng vai trò gì trong hệ sinh thái Hadoop (đặc biệt là HA NameNode)?
A. Quản lý thú trong sở thú.
B. Dịch vụ điều phối phân tán (Distributed Coordination Service), dùng để bầu chọn Active NameNode và quản lý Failover tự động.
C. Lưu trữ dữ liệu.
D. Chạy MapReduce.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Zookeeper đảm bảo tại một thời điểm chỉ có duy nhất 1 NameNode làm Active, tránh tình trạng "Split-Brain" (2 não cùng điều khiển gây hỏng dữ liệu).

### Câu 49
Secondary NameNode là gì?
A. Là NameNode dự phòng (Backup), thay thế ngay khi Primary chết.
B. Là node phụ trợ, định kỳ gộp (merge) Edit Logs vào FsImage để giảm tải cho Primary NameNode khi khởi động lại. Nó KHÔNG phải là node dự phòng nóng (Hot Standby).
C. Là DataNode đặc biệt.
D. Là node quản lý Job.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Rất nhiều người nhầm lẫn tên gọi này. Secondary NameNode thực chất là "Checkpoint Node". Để có HA thực sự, phải dùng Standby NameNode.

### Câu 50
Cơ chế "Data Integrity" (Toàn vẹn dữ liệu) trong HDFS hoạt động ra sao khi đọc dữ liệu?
A. Không kiểm tra gì cả.
B. Sử dụng Checksum (CRC32). Khi ghi, HDFS tính checksum lưu kèm data. Khi đọc, client tính lại và so sánh. Nếu lệch -> Báo lỗi và đọc từ replica khác.
C. So sánh với NameNode.
D. Dùng phần mềm diệt virus.

> [!SUCCESS]- Đáp án & Giải thích
> **Đáp án: B**
> **Giải thích:** Đảm bảo dữ liệu không bị hỏng hóc âm thầm (bit rot) do lỗi ổ cứng hoặc đường truyền.