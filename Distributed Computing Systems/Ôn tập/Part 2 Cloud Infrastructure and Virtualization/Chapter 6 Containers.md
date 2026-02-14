
# Kiến thức
## 1.  "Cô lập": Namespaces & Cgroups
Container "nhẹ" vì dùng chung Kernel. Nhưng làm sao dùng chung mà tiến trình (process) này không "nhìn thấy" tiến trình kia? Câu trả lời nằm ở 2 tính năng của Linux Kernel:

### A. Linux Namespaces (Không gian tên) - Sự cô lập góc nhìn
> [!NOTE] Analogy (Ẩn dụ)
> Hãy tưởng tượng Kernel là một tòa chung cư. Namespaces biến mỗi căn hộ thành một thế giới riêng. Người ở căn hộ A không biết sự tồn tại của người ở căn hộ B, dù họ sống cùng một tòa nhà.

Container sử dụng các loại Namespace sau để đánh lừa tiến trình bên trong:
* **PID Namespace:** Cô lập danh sách tiến trình. Bên trong Container, tiến trình chính luôn thấy mình là **PID 1** (Process ID 1 - vua của mọi process), dù bên ngoài máy chủ thật nó có thể là PID 12345.
* **NET Namespace:** Cô lập mạng. Mỗi container có địa chỉ IP riêng, bảng định tuyến (routing table) riêng, cổng (port) riêng. Cổng 80 ở Container A không đụng hàng cổng 80 ở Container B.
* **MNT Namespace (Mount):** Cô lập hệ thống file. Container chỉ nhìn thấy thư mục gốc `/` của riêng nó.

### B. Cgroups (Control Groups) - Sự quản lý tài nguyên
> [!NOTE] Analogy (Ẩn dụ)
> Nếu Namespaces là những bức tường ngăn cách, thì Cgroups là đồng hồ điện nước. Nó giới hạn mức tiêu thụ của từng căn hộ.

* **Chức năng:** Giới hạn lượng CPU, RAM, Disk I/O mà một container (hoặc nhóm container) được phép dùng.
* **Ý nghĩa sống còn:** Ngăn chặn vấn đề **"Noisy Neighbor"** (Hàng xóm ồn ào). Nếu không có Cgroups, một container bị lỗi vòng lặp vô hạn có thể ngốn sạch 100% CPU của máy chủ vật lý, làm sập toàn bộ các container khác.

---

## 2. Kiến trúc lưu trữ: Layered File System (Hệ thống file phân lớp)
Đây là phần khó hiểu nhất nhưng "ăn tiền" nhất của Docker.

> [!QUESTION] Tại sao 100 container Ubuntu chạy cùng lúc chỉ tốn dung lượng bằng 1 container?

### Cơ chế Union File System (UnionFS)
Docker Image không phải là một khối liền mạch. Nó giống như một chiếc **bánh xếp tầng (Pancake Stack)**.

1.  **Read-only Layers (Các lớp chỉ đọc):**
    * Khi bạn tải một Image (ví dụ: Ubuntu), nó bao gồm các lớp cơ sở (Base OS files). Các lớp này là **Bất biến (Immutable)** - không thể chỉnh sửa.
    * *Sự kỳ diệu:* Nếu bạn chạy 100 container từ image Ubuntu này, **cả 100 container đều dùng chung (share) các lớp Read-only này**. Chúng không bị copy ra 100 lần. -> **Tiết kiệm đĩa cực đại.**

2.  **Writable Layer (Lớp ghi được):**
    * Khi một Container khởi chạy, Docker đặt một lớp mỏng "có thể ghi" lên trên cùng của chồng bánh.
    * Mọi dữ liệu sinh ra trong quá trình chạy (log file, temp file) đều nằm ở lớp mỏng này.

### Cơ chế Copy-on-Write (CoW)
* *Tình huống:* Điều gì xảy ra nếu Container muốn chỉnh sửa một file `/etc/config` nằm ở lớp Read-only bên dưới?
* *Giải quyết:*
    1.  Docker tìm file đó ở lớp dưới.
    2.  Nó **Copy** file đó lên lớp Writable trên cùng.
    3.  Container thực hiện chỉnh sửa trên bản copy đó (**Write**).
    4.  Bản gốc ở dưới vẫn nguyên vẹn (cho các container khác dùng).

👉 **Kết luận:** Cơ chế này giúp việc khởi tạo container nhanh siêu tốc (vì không cần copy toàn bộ hệ điều hành) và tiết kiệm dung lượng đĩa.

---

## 3. Kiến trúc Docker: Client - Host - Registry
Để phân tích sâu luồng hoạt động, hãy mổ xẻ điều gì xảy ra khi bạn gõ lệnh: `docker run hello-world`

1.  **Docker Client (CLI):**
    * Nhận lệnh `docker run`.
    * Chuyển lệnh này thành REST API call gửi đến Docker Daemon.
2.  **Docker Daemon (Server):**
    * Kiểm tra xem Image `hello-world` đã có ở máy chưa (Local Cache).
    * Nếu chưa -> Gọi ra **Docker Registry** (Docker Hub) để tải về.
3.  **Tạo Container:**
    * Daemon yêu cầu Kernel cấp phát Namespaces và Cgroups.
    * Daemon xếp chồng các lớp Image (UnionFS) và thêm lớp Writable.
    * Daemon cấp IP ảo (qua Bridge Network).
    * Khởi chạy tiến trình.

---

## 4. Những hiểu lầm tai hại (Myths vs. Facts) - Dành cho bài thi tự luận

| Hiểu lầm (Myth) | Sự thật (Fact) - Phân tích sâu |
| :--- | :--- |
| "Container an toàn như VM" | **SAI.** Container dùng chung Kernel. Nếu hacker khai thác được lỗi của Kernel (Kernel Panic/Exploit), hắn có thể tấn công xuống Host OS và đánh sập tất cả Container khác. VM an toàn hơn vì có Hardware Isolation. |
| "Docker là Container duy nhất" | **SAI.** Docker chỉ là công cụ phổ biến nhất. Công nghệ lõi là **LXC** (Linux Containers). Ngày nay có các runtime khác như *containerd, CRI-O, Podman*. Docker chỉ làm cho việc dùng Container dễ dàng hơn thôi. |
| "Dữ liệu trong Container là vĩnh cửu" | **SAI.** Container được thiết kế để **Ephemeral** (Ngắn hạn). Khi xóa container, lớp Writable bị xóa theo -> Mất dữ liệu. Muốn lưu dữ liệu lâu dài, phải dùng **Volumes** (cơ chế mount thư mục từ máy thật vào container). |

# Pool Câu Hỏi Trắc Nghiệm - Cloud Computing Chapter 6

## Chủ đề: Khái niệm Container & Docker (Định nghĩa & Từ khóa)

### Câu 1
**Sự khác biệt cơ bản nhất về mặt kiến trúc giữa "Container" và "Virtual Machine" (VM) là gì?**
A. Container chạy chậm hơn VM.
B. Container yêu cầu một Guest OS (Hệ điều hành khách) đầy đủ cho mỗi ứng dụng, trong khi VM thì không.
C. Container chia sẻ hạt nhân (kernel) của hệ điều hành máy chủ và không cần Guest OS riêng biệt, trong khi mỗi VM chạy một Guest OS đầy đủ.
D. Container chỉ chạy được trên Windows.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Nhược điểm chính của VM là overhead của việc chạy một hệ điều hành khách đầy đủ. Container giải quyết vấn đề này bằng cách chạy như một tiến trình bị cô lập trên hệ điều hành máy chủ, tránh được overhead của Guest OS,.

---

### Câu 2
**Cơ chế nào trong hệ điều hành Linux được sử dụng để tạo ra sự cô lập (isolation) cho các Container, đảm bảo tiến trình trong container này không nhìn thấy tiến trình của container khác?**
A. Linux Firewalls.
B. Linux Namespaces.
C. Hypervisor.
D. Virtual Memory.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Các cơ chế cô lập trong Linux được gọi là **Namespaces** (ví dụ: PID namespace, Network namespace). Chúng ngăn chặn một ứng dụng truy cập thông tin về các ứng dụng bị cô lập khác,.

---

### Câu 3
**Trong thuật ngữ Docker, thuật ngữ nào dùng để chỉ một tệp tin (file) chứa mã nguồn và tất cả các thành phần phụ thuộc (dependencies) cần thiết để chạy một ứng dụng, và tệp này có tính chất "immutable" (bất biến)?**
A. Docker Container.
B. Docker Daemon.
C. Docker Image.
D. Dockerfile.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** **Docker Image** là một tệp chứa mã nhị phân và các phụ thuộc. Khi đã được xây dựng, image trở nên bất biến (immutable) - nó không thay đổi bất kể bao nhiêu lần nó được chạy,.

---

### Câu 4
**Tệp văn bản chứa các chỉ dẫn (instructions) để tự động xây dựng một Docker Image được gọi là gì?**
A. Makefile.
B. Dockerfile.
C. ImageBuilder.
D. ContainerList.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Lập trình viên tạo một tệp văn bản có tên là **Dockerfile** chứa các đặc tả về cách xây dựng một image.

---

### Câu 5
**Thành phần phần mềm nào trong kiến trúc Docker chạy dưới dạng một tiến trình nền (background daemon) và chịu trách nhiệm quản lý các container cũng như image?**
A. Docker CLI (Command Line Interface).
B. Docker Hub.
C. dockerd (Docker daemon).
D. Kubernetes.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Các container hoạt động dưới sự kiểm soát của một ứng dụng được gọi là **Docker daemon (dockerd)**. Nó chạy nền và quản lý cả các container đang chạy lẫn các image lưu trên đĩa,.

---

### Câu 6
**Thuật ngữ "Layer" (Lớp) trong Docker Image đề cập đến điều gì?**
A. Một lớp bảo mật mạng.
B. Một mảnh phần mềm (piece of software) được thêm vào image trong quá trình xây dựng.
C. Lớp vật lý của mô hình OSI.
D. Một loại container đặc biệt.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Docker sử dụng thuật ngữ **Layer** để chỉ mỗi phần mềm mà lập trình viên thêm vào image khi nó được xây dựng.

---

### Câu 7
**Lệnh nào sau đây được sử dụng để chuyển đổi một Dockerfile thành một Docker Image?**
A. `docker run`
B. `docker start`
C. `docker build`
D. `docker pull`

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Lệnh `docker build` đọc Dockerfile và tuân theo các chỉ dẫn để xây dựng một image,.

---

### Câu 8
**Trong Dockerfile, từ khóa nào BẮT BUỘC phải xuất hiện đầu tiên để chỉ định hệ điều hành cơ sở (Base OS)?**
A. `RUN`
B. `IMPORT`
C. `FROM`
D. `START`

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Từ khóa **FROM** chỉ định hệ điều hành cơ sở cho image và phải xuất hiện đầu tiên trong Dockerfile (ví dụ: `FROM alpine`).

---

### Câu 9
**"Docker Hub" đóng vai trò gì trong hệ sinh thái Docker?**
A. Là phần mềm quản lý container trên máy tính cá nhân.
B. Là một registry (kho lưu trữ) chứa các phần mềm và image mã nguồn mở đã được xây dựng sẵn để người dùng tải về.
C. Là một thiết bị phần cứng kết nối mạng.
D. Là ngôn ngữ lập trình để viết container.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Docker Hub là một **registry** rộng lớn chứa các phần mềm mã nguồn mở sẵn sàng để sử dụng, cho phép lập trình viên chia sẻ và tải về các ứng dụng đã đóng gói.

---

### Câu 10
**Trạng thái của một Container được gọi là "Ephemeral" (Tạm thời/Phù du). Điều này có nghĩa là gì đối với dữ liệu lưu trong file system của container khi container đó tắt đi?**
A. Dữ liệu sẽ tự động được sao lưu lên đám mây.
B. Dữ liệu vẫn tồn tại vĩnh viễn và có thể truy cập bởi container khác.
C. Các thay đổi đối với file local sẽ không được lưu lại (persist) cho lần chạy sau, trừ khi dùng cơ chế lưu trữ ngoài.
D. Container không thể ghi dữ liệu.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Trừ khi lập trình viên kết nối container với bộ nhớ bền vững bên ngoài, các thay đổi đối với tệp tin cục bộ khi container chạy sẽ không được lưu lại sau khi container kết thúc (container is immutable).
Chào bạn, để tiếp tục nâng cao mức độ khó cho **Chương 6: Containers**, tôi đã biên soạn bộ câu hỏi ở mức độ **vận dụng (Application)**.

# Pool Câu Hỏi Trắc Nghiệm Vận Dụng - Cloud Computing Chapter 6

## Chủ đề: Kiến trúc Docker, Dockerfile Logic & Xử lý sự cố

### Câu 11
**Tình huống:** Bạn viết một Dockerfile cho ứng dụng Python. Bạn sử dụng lệnh `RUN python app.py` để khởi chạy ứng dụng. Tuy nhiên, khi bạn thực hiện lệnh `docker build`, quá trình build bị treo (hang) vô thời hạn và không bao giờ tạo ra được image. Nguyên nhân kỹ thuật là gì?
A. Do bạn chưa cài đặt Python trong Base Image.
B. Do lệnh `RUN` được thực thi trong quá trình build image. Nếu `app.py` là một server chạy liên tục (long-running process), quá trình build sẽ đợi ứng dụng kết thúc thì mới sang bước tiếp theo, dẫn đến bị treo.
C. Do Docker daemon bị lỗi kết nối mạng.
D. Do lệnh `RUN` chỉ dùng để cài đặt thư viện, không dùng để chạy lệnh.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Từ khóa `RUN` chỉ định việc chạy một chương trình để thêm một layer mới vào image trong quá trình **xây dựng (build time)**. Nếu bạn chạy một ứng dụng server không bao giờ tắt tại bước này, quá trình build sẽ đợi mãi mãi. Để chạy ứng dụng khi container khởi động, phải dùng `ENTRYPOINT` hoặc `CMD`,.

---

### Câu 12
**Tình huống:** Một kỹ sư muốn tạo một Docker image cho ứng dụng web. Trong Dockerfile, anh ta sử dụng `CMD ["/bin/ping", "localhost"]` để giữ container chạy nền kiểm tra. Tuy nhiên, khi người dùng chạy lệnh `docker run my-image /bin/bash`, họ thấy rằng lệnh ping không được thực thi mà thay vào đó họ truy cập vào shell bash. Tại sao?
A. Vì Docker ưu tiên lệnh `/bin/bash` do nó là shell mặc định.
B. Vì lệnh `CMD` trong Dockerfile chỉ cung cấp các đối số mặc định (default arguments) hoặc lệnh mặc định. Nếu người dùng cung cấp lệnh khác tại thời điểm chạy (`docker run`), lệnh trong `CMD` sẽ bị ghi đè hoàn toàn.
C. Vì lệnh ping bị lỗi.
D. Vì `ENTRYPOINT` chưa được thiết lập.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Mục đích chính của `CMD` là cung cấp các đối số mặc định cho `ENTRYPOINT` hoặc một lệnh mặc định. Các giá trị này có thể bị ghi đè (overridden) khi container được khởi động nếu người dùng nhập lệnh khác vào `docker run`.

---

### Câu 13
**Tình huống:** Bạn đang thiết kế một container để xử lý dữ liệu log. Yêu cầu là container phải luôn luôn thực thi file `process_logs.py` mỗi khi khởi động, nhưng người dùng có thể truyền thêm các tham số khác nhau (ví dụ: `--verbose` hoặc `--quiet`) khi chạy `docker run`. Cặp lệnh Dockerfile nào dưới đây giải quyết tốt nhất yêu cầu này?
A. `RUN ["python", "process_logs.py"]`
B. `CMD ["python", "process_logs.py"]`
C. `ENTRYPOINT ["python", "process_logs.py"]` kết hợp với `CMD []` (để chứa tham số mặc định nếu cần).
D. `ADD process_logs.py /app/`

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** `ENTRYPOINT` chỉ định chương trình bắt buộc phải chạy khi container bắt đầu. Các tham số từ `CMD` hoặc từ dòng lệnh `docker run` sẽ được nối vào sau `ENTRYPOINT` làm đối số. Cách này đảm bảo `process_logs.py` luôn chạy và nhận tham số động,.

---

### Câu 14
**Tình huống:** Một ứng dụng trong Container ghi dữ liệu quan trọng vào thư mục `/app/data`. Sau khi container bị tắt và xóa đi (removed), người quản trị phát hiện toàn bộ dữ liệu trong `/app/data` đã biến mất vĩnh viễn. Để khắc phục vấn đề này cho lần chạy sau mà không thay đổi mã nguồn ứng dụng, giải pháp kiến trúc nào cần được áp dụng trong Dockerfile hoặc khi chạy container?
A. Sử dụng lệnh `COPY` để sao chép dữ liệu ra ngoài trước khi tắt.
B. Sử dụng cơ chế `VOLUME` (trong Dockerfile) hoặc bind mount (khi chạy) để kết nối thư mục `/app/data` của container với hệ thống lưu trữ bền vững bên ngoài (persistent storage).
C. Tăng dung lượng RAM cho container.
D. Chuyển sang sử dụng Virtual Machine.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Container có tính chất "immutable" và "ephemeral". Các thay đổi đối với file cục bộ sẽ không được lưu lại sau khi container kết thúc trừ khi lập trình viên kết nối container với bộ nhớ bền vững bên ngoài thông qua cơ chế `VOLUME` hoặc mount,.

---

### Câu 15
**Tình huống:** Bạn đang chạy một Docker container trên một máy chủ Linux. Bạn muốn container này có thể nhìn thấy và quản lý tất cả các tiến trình (processes) đang chạy trên máy chủ vật lý (Host OS), giống như một ứng dụng giám sát hệ thống. Bạn cần thay đổi cấu hình Namespace nào khi chạy container?
A. Sử dụng `Network Namespace` riêng biệt.
B. Chia sẻ `PID Namespace` của Host với Container (sử dụng cờ `--pid=host`).
C. Sử dụng `Mount Namespace` mặc định.
D. Không thể thực hiện được vì Container luôn luôn bị cô lập tuyệt đối.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Mặc định, Linux Namespaces (cụ thể là PID namespace) cô lập các tiến trình để container không thấy tiến trình của Host. Tuy nhiên, Docker cho phép chia sẻ namespace nếu cần thiết. Việc chia sẻ PID namespace cho phép container nhìn thấy các tiến trình của Host,.

---

### Câu 16
**Tình huống:** Trong quy trình CI/CD, bạn cần tối ưu hóa thời gian build Docker image. Bạn nhận thấy rằng mỗi khi thay đổi một dòng code nhỏ trong file `app.py`, Docker lại tải lại toàn bộ các thư viện dependencies (ví dụ: `npm install` hoặc `pip install`) tốn rất nhiều thời gian. Dựa trên cơ chế "Layer caching" của Docker, bạn nên sắp xếp lại Dockerfile như thế nào?
A. Đặt lệnh `COPY . .` (copy toàn bộ source code) lên dòng đầu tiên, sau đó mới đến lệnh cài đặt dependencies.
B. Đặt lệnh cài đặt dependencies (ví dụ `COPY requirements.txt` và `RUN pip install`) lên trước, sau đó mới đến lệnh `COPY` source code ứng dụng.
C. Gộp tất cả vào một dòng `RUN` duy nhất.
D. Không sử dụng Dockerfile mà build thủ công.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Docker cache các layer. Nếu một layer thay đổi, các layer sau nó phải build lại. Source code thay đổi thường xuyên hơn dependencies. Do đó, nên copy file định nghĩa dependencies và cài đặt chúng trước (tạo layer ít thay đổi), sau đó mới copy source code (layer hay thay đổi). Điều này giúp tận dụng cache cho bước cài đặt thư viện,.

---

### Câu 17
**Tình huống:** Bạn muốn tải một file nén `.tar.gz` từ một URL từ xa và giải nén nó vào trong image tại đường dẫn `/usr/src/app`. Bạn muốn thực hiện việc này chỉ bằng một lệnh duy nhất trong Dockerfile để giảm số lượng layer. Bạn nên dùng chỉ thị nào?
A. `COPY http://example.com/file.tar.gz /usr/src/app`
B. `RUN curl http://example.com/file.tar.gz`
C. `ADD http://example.com/file.tar.gz /usr/src/app`
D. `IMPORT http://example.com/file.tar.gz`

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Từ khóa `ADD` mạnh hơn `COPY`. Nó cho phép chỉ định URL từ xa và có khả năng tự động giải nén (decompress) các file archive vào đích đến, giúp thực hiện tác vụ trong một bước. Tuy nhiên, sách cũng lưu ý `COPY` thường được khuyến nghị hơn nếu không cần tính năng giải nén tự động này để tránh image bị phình to không cần thiết,.

---

### Câu 18
**Tình huống:** Một hệ thống sử dụng kiến trúc Microservices được triển khai bằng Docker. Dịch vụ A cần gọi Dịch vụ B. Cả hai đều chạy trên cùng một máy chủ vật lý nhưng trong các container khác nhau. Tại sao Dịch vụ A không thể gọi Dịch vụ B thông qua `localhost` (ví dụ: `localhost:8080`) mặc dù chúng ở cùng trên một máy mẹ?
A. Vì chúng sử dụng chung PID namespace.
B. Vì mỗi container có một `Network Namespace` riêng biệt, nghĩa là chúng có ngăn xếp mạng (IP, cổng, interface) riêng. `localhost` của Container A là riêng biệt với `localhost` của Container B.
C. Vì Docker chặn tất cả traffic nội bộ.
D. Vì file system của chúng khác nhau.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Linux Namespaces (cụ thể là Network namespace) tạo ra sự cô lập về mạng. Mỗi container có địa chỉ IP và bảng định tuyến riêng. `localhost` trong container chỉ tham chiếu đến chính nó, không phải container khác hay máy chủ host,.

---

### Câu 19
**Tình huống:** Bạn cần chạy một ứng dụng Legacy yêu cầu hệ điều hành Windows. Máy chủ hiện có của bạn chạy Linux và đã cài đặt Docker Engine. Bạn có thể sử dụng Docker để chạy ứng dụng Windows này trên máy chủ Linux đó không (giả sử không dùng ảo hóa lồng nhau phức tạp)?
A. Có, vì Docker hỗ trợ mọi hệ điều hành.
B. Không, vì Container chia sẻ kernel của hệ điều hành máy chủ (Host OS). Container Windows cần kernel Windows, không thể chạy trực tiếp trên kernel Linux.
C. Có, chỉ cần dùng lệnh `FROM windows` trong Dockerfile.
D. Có, nhưng sẽ chạy rất chậm.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Kiến trúc Container dựa vào việc chia sẻ hạt nhân (kernel) của Host OS. Do đó, container phải tương thích với kernel của host. Một container Windows không thể chạy trên Linux host kernel và ngược lại (trừ khi có lớp ảo hóa VM hỗ trợ bên dưới),.

---

### Câu 20
**Tình huống:** Một nhà phát triển sử dụng `dockerd` (Docker daemon) thông qua RESTful API để triển khai hàng loạt container từ xa. Điều này ám chỉ mô hình quản lý nào thường thấy trong các hệ thống đám mây lớn?
A. Quản lý thủ công (Manual management).
B. Sử dụng phần mềm điều phối (Orchestration software) thay vì con người gõ lệnh CLI.
C. Sử dụng Desktop as a Service.
D. Sử dụng mạng ngang hàng.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** `dockerd` cung cấp RESTful API chủ yếu để cho các ứng dụng (như phần mềm Orchestration - ví dụ Kubernetes) gửi yêu cầu quản lý container một cách tự động, thay vì để con người tương tác qua CLI,.


# Pool Câu Hỏi Trắc Nghiệm Vận Dụng Cao & Bẫy - Cloud Computing Chapter 6

## Chủ đề: Docker Internals, Linux Namespaces, và Dockerfile Optimization

### Câu 1
**Tình huống:** Một kỹ sư DevOps xây dựng một Docker image dựa trên `FROM ubuntu:latest` (dung lượng khoảng 70MB). Sau đó, anh ta chạy container này trên một máy chủ sử dụng hệ điều hành Red Hat Enterprise Linux (RHEL). Khi anh ta gõ lệnh `uname -r` bên trong container Ubuntu này để kiểm tra phiên bản Kernel, kết quả trả về sẽ là gì?**
A. Phiên bản Kernel của Ubuntu (do image là Ubuntu).
B. Phiên bản Kernel của RHEL (Hệ điều hành máy chủ).
C. Một phiên bản Kernel ảo hóa trung gian do Docker Engine tạo ra.
D. Lệnh sẽ báo lỗi vì Container không có quyền truy cập thông tin Kernel.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Đây là sự khác biệt cốt lõi giữa VM và Container. Container **chia sẻ kernel** của hệ điều hành máy chủ (Host OS). `FROM ubuntu` chỉ cung cấp **base operating system** (các thư viện, tệp tin nhị phân ở user space, file system), nó hoạt động như một lớp đệm (shim layer) chứ không chứa Kernel riêng. Do đó, container Ubuntu chạy trên RHEL sẽ sử dụng Kernel của RHEL,.

---

### Câu 2
**Tình huống:** Bạn đang tối ưu hóa dung lượng của Docker Image. Trong `Dockerfile`, bạn thực hiện như sau:
- Layer 1: `RUN wget http://example.com/big-file.zip` (Tải file nặng 1GB).
- Layer 2: `RUN unzip big-file.zip` (Giải nén ra file 2GB).
- Layer 3: `RUN rm big-file.zip` (Xóa file gốc 1GB).
**Kết quả về kích thước cuối cùng của Image sẽ như thế nào so với việc gộp cả 3 lệnh trên vào cùng một dòng `RUN`?**
A. Kích thước giống nhau, Docker tự động dọn dẹp file rác.
B. Kích thước Image ở trường hợp chia 3 Layers sẽ lớn hơn rất nhiều (vẫn chứa 1GB của file zip dù đã xóa ở Layer 3).
C. Kích thước Image ở trường hợp chia 3 Layers sẽ nhỏ hơn vì Docker nén từng layer riêng biệt.
D. Lệnh `rm` ở Layer 3 sẽ xóa hoàn toàn dữ liệu vật lý của Layer 1 khỏi đĩa cứng.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Docker Image là tập hợp các Layer **bất biến (immutable)**. Khi bạn xóa một file ở Layer 3, file đó chỉ bị "ẩn" đi đối với container final, nhưng dữ liệu vật lý của nó vẫn nằm vĩnh viễn ở Layer 1 để đảm bảo tính tái sử dụng. Để giảm dung lượng, phải gộp tải - giải nén - xóa vào cùng một lệnh `RUN` (cùng một layer).

---

### Câu 3
**Tình huống:** Trong Dockerfile, bạn sử dụng lệnh: `ENTRYPOINT ["/bin/ping"]`. Khi người dùng chạy container bằng lệnh: `docker run my-image google.com`, điều gì thực sự xảy ra bên trong container?
A. Container sẽ chạy lệnh `/bin/ping` và bỏ qua đối số `google.com`.
B. Container sẽ chạy lệnh `/bin/ping` với đối số mặc định từ `CMD` (nếu có), bỏ qua `google.com`.
C. Container sẽ chạy lệnh `/bin/ping` và nối chuỗi `google.com` vào phía sau, tạo thành lệnh `/bin/ping google.com`.
D. Docker sẽ báo lỗi vì `ENTRYPOINT` không nhận tham số từ dòng lệnh `docker run`.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Khi sử dụng `ENTRYPOINT` (ở dạng exec form như ví dụ), tất cả các đối số được cung cấp sau lệnh `docker run` (ở đây là `google.com`) sẽ được **nối thêm (appended)** vào sau lệnh được định nghĩa trong `ENTRYPOINT`. Đây là cách thiết kế container hoạt động như một tệp thực thi (executable).

---

### Câu 4
**Tình huống:** Bạn chạy một ứng dụng web trong Docker container. Ứng dụng này cần ghi log liên tục. Vì container có tính chất "Ephemeral" (ngắn hạn/phù du), bạn lo ngại dữ liệu log sẽ mất khi container bị xóa. Bạn quyết định sử dụng cơ chế `VOLUME` trong Dockerfile. Điều này thực chất làm gì về mặt hệ thống tệp?**
A. Nó sao chép dữ liệu log lên đám mây của Docker Hub.
B. Nó tạo ra một "lỗ hổng" (mount point) trong hệ thống tệp của container, kết nối trực tiếp thư mục đó với một vùng lưu trữ bền vững bên ngoài (trên Host OS hoặc Storage Server), bỏ qua lớp writable layer tạm thời của container.
C. Nó khóa file log để không ai có thể xóa được.
D. Nó tăng dung lượng ổ cứng cho container.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** `VOLUME` chỉ định một điểm gắn kết (mount point) nơi hệ thống tệp bên ngoài (external file system) có thể kết nối vào. Trừ khi được kết nối với bộ nhớ bền vững (persistent storage) thông qua cơ chế này, mọi thay đổi file trong container sẽ mất đi khi container tắt. Volume giúp dữ liệu tồn tại độc lập với vòng đời container,.

---

### Câu 5
**Tình huống:** Một hacker đã xâm nhập vào được một Docker Container đang chạy với quyền root (bên trong container). Tuy nhiên, hắn nhận thấy rằng hắn không thể nhìn thấy các tiến trình (processes) của các container khác hoặc của máy chủ Host, và cũng không thể tắt (kill) các tiến trình đó. Cơ chế bảo mật nào của Linux Kernel chịu trách nhiệm chính cho việc ngăn chặn tầm nhìn và tác động này?**
A. File System Permissions (chmod).
B. Control Groups (cgroups).
C. Process ID (PID) Namespace.
D. Docker Daemon Firewall.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** **PID Namespace** cho phép mỗi ứng dụng bị cô lập sử dụng một tập hợp ID tiến trình riêng (bắt đầu từ 1). Nó đảm bảo rằng nếu một ứng dụng cố gắng trích xuất thông tin về một process ID, kết quả chỉ tham chiếu đến các tiến trình trong cùng môi trường cô lập đó, ngăn chặn việc truy cập thông tin về các ứng dụng khác trên cùng Host,.

---

### Câu 6
**Tình huống:** Bạn đang viết một Dockerfile và sử dụng lệnh `RUN apt-get update`. Bạn nhận thấy rằng quá trình build diễn ra thành công. Tuy nhiên, khi đồng nghiệp của bạn build lại image đó trên máy của họ vào tuần sau, Docker **không** tải các gói cập nhật mới nhất mà lại sử dụng các gói cũ đã tải từ lần build trước đó của bạn (giả sử họ dùng chung cache server hoặc registry). Đây là tính năng gì và làm sao để bắt buộc Docker tải lại?**
A. Đây là lỗi mạng; cần reset modem.
B. Đây là tính năng **Layer Caching** (hoặc Immutability). Vì dòng lệnh `RUN` không đổi, Docker tái sử dụng layer cũ. Để sửa, cần thay đổi nội dung lệnh hoặc dùng flag `--no-cache`.
C. Đây là tính năng **Base OS**. Phải đổi Base OS mới update được.
D. Dockerfile không hỗ trợ update phần mềm.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Docker Image là bất biến (immutable) và sử dụng cơ chế cache các layer. Nếu nội dung chỉ thị trong Dockerfile không đổi, Docker sẽ sử dụng layer đã được build trước đó thay vì chạy lại lệnh. Điều này đảm bảo **reproducible execution** (thực thi tái lập được) nhưng có thể gây ra việc không nhận được bản vá bảo mật mới nếu không quản lý cache đúng cách.

---

### Câu 7
**Tình huống:** Bạn cần chạy một ứng dụng Legacy yêu cầu cấu hình IP tĩnh và cần giao tiếp trực tiếp với các thiết bị phần cứng đặc biệt trên mạng LAN của công ty mà không qua NAT. Trong Docker, bạn nên chọn chế độ mạng (Network Mode) nào để container **chia sẻ hoàn toàn** ngăn xếp mạng (IP, Port) với máy chủ Host?**
A. Bridge Mode (Mặc định).
B. Host Mode (`--network host`).
C. Overlay Mode.
D. None Mode.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Container có thể **clone (sao chép/dùng chung)** địa chỉ IP của Host. Trong chế độ này, container không nhận IP riêng mà chia sẻ IP và cổng mạng của Host OS. Điều này loại bỏ sự cô lập mạng nhưng cần thiết cho một số ứng dụng đặc biệt.

---

### Câu 8
**Tình huống:** Bạn sử dụng lệnh `ENTRYPOINT ["/bin/sh", "-c", "echo hi"]` (Shell form) thay vì `ENTRYPOINT ["/bin/echo", "hi"]` (Exec form). Vấn đề tiềm ẩn lớn nhất về quản lý tín hiệu (Unix signals) khi muốn dừng container này là gì?**
A. Container sẽ không bao giờ chạy được.
B. Lệnh `docker stop` sẽ gửi tín hiệu SIGTERM tới `/bin/sh` (shell), nhưng shell thường không chuyển tiếp tín hiệu đó tới tiến trình con (`echo`). Kết quả là container sẽ không dừng ngay lập tức mà phải đợi Docker engine `kill` sau thời gian timeout (thường là 10s).
C. Container sẽ chạy nhanh hơn.
D. Không thể truyền tham số vào.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Sách khuyến nghị sử dụng **non-shell option** (Exec form) vì chương trình sẽ chạy với **Process ID 1** và sẽ nhận được các tín hiệu Unix (như SIGTERM để tắt nhẹ nhàng). Nếu dùng shell, shell chiếm PID 1 và có thể không chuyển tín hiệu cho ứng dụng thực sự, làm hỏng quy trình tắt container.

# Pool Câu Hỏi Troubleshooting & Vận Dụng Thực Tế - Cloud Computing Chapter 6

## Chủ đề: Dockerfile Logic, Data Persistence, và Runtime Errors

### Câu 1
**Tình huống (Troubleshoot):** Một lập trình viên viết Dockerfile để chạy ứng dụng Python. Anh ta sử dụng lệnh `RUN python app.py` với mong muốn ứng dụng sẽ khởi động ngay khi container chạy. Tuy nhiên, khi thực hiện lệnh `docker build`, quá trình build bị treo (hangs) và không bao giờ hoàn thành. Dựa trên định nghĩa về các lệnh trong Dockerfile, nguyên nhân là gì?**
A. Do máy chủ không kết nối được Internet để tải Python.
B. Do lệnh `RUN` được thực thi tại thời điểm xây dựng image (build time). Nếu `app.py` là một tiến trình chạy liên tục (như web server), quá trình build sẽ đợi tiến trình này kết thúc mới sang bước tiếp theo, dẫn đến việc bị treo vô hạn.
C. Do chưa cấu hình cổng mạng (EXPOSE).
D. Do lệnh `RUN` chỉ dùng để cài đặt gói phần mềm, không chạy được script.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Sách xác định rõ từ khóa `RUN` chỉ định việc chạy một chương trình để thêm một layer mới vào image trong quá trình **xây dựng**. Lỗi phổ biến là nhầm lẫn `RUN` (build time) với `CMD` hoặc `ENTRYPOINT` (run time). Nếu chạy một ứng dụng server tại `RUN`, Docker sẽ đợi nó thoát (exit) để commit layer, nhưng server thì không bao giờ thoát.

---

### Câu 2
**Tình huống (Vận dụng):** Bạn cần tạo một Docker image sao cho container **luôn luôn** chạy lệnh kiểm tra sức khỏe hệ thống `/bin/healthcheck` khi khởi động. Tuy nhiên, bạn muốn cho phép người dùng có thể truyền thêm các tham số tùy biến (ví dụ: `--verbose` hoặc `--quick`) vào cuối lệnh này khi họ gõ `docker run`. Cặp chỉ thị nào trong Dockerfile giải quyết tốt nhất yêu cầu này?**
A. Dùng `RUN ["/bin/healthcheck"]`.
B. Chỉ dùng `CMD ["/bin/healthcheck"]`.
C. Dùng `ENTRYPOINT ["/bin/healthcheck"]` kết hợp với `CMD []` (để chứa tham số mặc định nếu cần).
D. Dùng `ADD /bin/healthcheck`.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** `ENTRYPOINT` chỉ định chương trình bắt buộc phải chạy khi container bắt đầu. `CMD` có mục đích chính là cung cấp các đối số mặc định cho `ENTRYPOINT`. Khi kết hợp, lệnh trong `ENTRYPOINT` sẽ chạy cố định, và các tham số từ người dùng (khi `docker run`) sẽ được nối vào sau, cho phép tùy biến tham số mà không thay đổi lệnh chính.

---

### Câu 3
**Tình huống (Troubleshoot):** Một ứng dụng Web chạy trong container tạo ra các file log quan trọng tại thư mục `/var/log/myapp`. Sau khi container bị xóa (removed) để cập nhật phiên bản mới, toàn bộ file log cũ đã biến mất. Dựa trên tính chất "Immutable" (Bất biến) và "Ephemeral" (Phù du) của container, bạn đã quên cấu hình điều gì để giữ lại dữ liệu này?**
A. Quên sử dụng lệnh `COPY` để sao chép log ra ngoài trước khi tắt.
B. Quên cấu hình `EXPOSE` cho thư mục log.
C. Quên sử dụng cơ chế `VOLUME` (hoặc bind mount) để kết nối thư mục `/var/log/myapp` với hệ thống lưu trữ bền vững bên ngoài (persistent storage).
D. Quên cài đặt cơ sở dữ liệu bên trong container.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Trừ khi lập trình viên kết nối container với bộ nhớ bền vững bên ngoài (persistent storage), các thay đổi đối với tệp tin cục bộ khi container chạy sẽ không được lưu lại sau khi container kết thúc. Cơ chế `VOLUME` chỉ định một điểm gắn kết (mount point) để kết nối với hệ thống tệp bên ngoài nhằm lưu trữ dữ liệu lâu dài.

---

### Câu 4
**Tình huống (Thực tế):** Khi tối ưu hóa kích thước của Docker Image, bạn thấy một Dockerfile có cấu trúc như sau:
1. `RUN wget http://example.com/big-file.zip`
2. `RUN unzip big-file.zip`
3. `RUN rm big-file.zip`
**Tại sao kích thước Image cuối cùng vẫn rất lớn mặc dù bạn đã xóa file zip ở bước 3?**
A. Vì Docker không bao giờ xóa dữ liệu thực sự.
B. Vì mỗi lệnh `RUN` tạo ra một layer riêng biệt. File zip vẫn tồn tại ở Layer 1 dù bị "ẩn" đi ở Layer 3. Để giảm dung lượng, cần gộp 3 lệnh này vào cùng một dòng `RUN`.
C. Vì lệnh `rm` không hoạt động trong Docker.
D. Vì file zip được lưu trong cache của CPU.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Docker sử dụng khái niệm **Layer** (lớp) cho mỗi phần mềm thêm vào image. Image là tập hợp các layer bất biến. Khi xóa file ở layer sau, dữ liệu vật lý của layer trước vẫn còn. Để tối ưu, cần thực hiện tải, giải nén và xóa trong cùng một lệnh (một layer) để file rác không bị lưu vào lịch sử image.

---

### Câu 5
**Tình huống (Troubleshoot):** Bạn chạy lệnh `docker run -d my-web-server`. Lệnh trả về một mã ID dài (container ID) nhưng khi bạn kiểm tra bằng `docker ps`, bạn không thấy container đó đang chạy. Khi dùng `docker ps -a`, bạn thấy container đã ở trạng thái `Exited (0)`. Nguyên nhân phổ biến nhất liên quan đến kiến trúc tiến trình của container là gì?**
A. Do Docker Daemon bị lỗi.
B. Do ứng dụng bên trong container (ví dụ web server) chạy dưới dạng background daemon (forking to background) thay vì chạy foreground. Khi tiến trình chính (PID 1) trả về/thoát, Docker coi như container đã hoàn thành nhiệm vụ và tắt nó.
C. Do thiếu RAM.
D. Do chưa map port.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Một container là một instance của image đang chạy. Container hoạt động giống như một tiến trình. Nếu lệnh khởi động (CMD/ENTRYPOINT) chạy xong và thoát (hoặc chạy ngầm và trả lại điều khiển ngay lập tức), container sẽ dừng lại. Ví dụ trong sách: lệnh `echo` in ra xong sẽ thoát, làm container thoát theo. Ứng dụng trong container phải chạy ở chế độ foreground để giữ container sống.

---

### Câu 6
**Tình huống (Vận dụng):** Bạn muốn tải một file cấu hình từ một URL từ xa và đặt nó vào trong image. Bạn phân vân giữa `COPY` và `ADD`. Theo khuyến nghị thực tế (best practice) được đề cập trong sách để tránh làm image bị phình to không cần thiết do các file nén tự động bung ra, bạn nên chọn phương án nào?**
A. Luôn dùng `ADD` vì nó mạnh hơn.
B. Tải file về máy local, xử lý (giải nén/lọc), sau đó dùng `COPY` để đưa vào image.
C. Dùng `ADD` trực tiếp URL vào image.
D. Dùng `RUN` để mở trình duyệt tải file.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** `ADD` có khả năng tự động giải nén (decompress) các file archive, điều này đôi khi dẫn đến việc tạo ra các file thừa không mong muốn trong image. Sách khuyến nghị phương pháp: tải file về máy cục bộ, mở archive, xóa file không cần thiết, và sau đó sử dụng từ khóa `COPY` để sao chép vào image.

---

### Câu 7
**Tình huống (An ninh/Vận dụng):** Một hacker chiếm được quyền kiểm soát một tiến trình bên trong container. Hắn cố gắng liệt kê các tiến trình đang chạy trên hệ điều hành máy chủ (Host OS) để tìm điểm yếu tấn công tiếp theo. Tuy nhiên, lệnh `ps aux` bên trong container chỉ hiển thị một vài tiến trình (thường là chính nó và các tiến trình con). Cơ chế nào của Linux Kernel đã ngăn chặn hacker nhìn thấy toàn bộ hệ thống?**
A. Chroot Jail.
B. PID Namespace (Process ID Namespace).
C. Hypervisor Firewall.
D. Docker Hub Security.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Cơ chế **PID Namespace** cô lập các định danh tiến trình. Nó đảm bảo rằng nếu một ứng dụng (ở đây là ứng dụng trong container) cố gắng trích xuất thông tin về process ID, kết quả chỉ tham chiếu đến các tiến trình trong cùng môi trường cô lập đó, ngăn chặn việc truy cập thông tin về các ứng dụng khác (hoặc Host OS).

---

### Câu 8
**Tình huống (Thực tế):** Bạn gõ lệnh `docker run` tại terminal. Thông báo lỗi xuất hiện: *"Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?"*. Điều này cho thấy mối quan hệ gì giữa lệnh `docker` và `dockerd`?**
A. Lệnh `docker` và `dockerd` là một chương trình duy nhất, lỗi do file bị hỏng.
B. Lệnh `docker` là client (giao diện dòng lệnh), nó cần giao tiếp với `dockerd` (daemon chạy nền) để thực thi lệnh. Lỗi này nghĩa là service chạy nền đang bị tắt.
C. Lỗi do bạn chưa cắm dây mạng.
D. Lỗi do Docker Hub bị sập.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Docker Engine gồm 2 phần chính: `dockerd` (daemon quản lý container/image chạy nền) và `docker` (chương trình giao diện dòng lệnh CLI). Người dùng tương tác với `docker`, và `docker` gửi yêu cầu đến `dockerd`. Nếu `dockerd` không chạy, CLI sẽ không thể thực hiện tác vụ,.

---

### Câu 9
**Tình huống (Cấu hình):** Bạn có một file Dockerfile với dòng `EXPOSE 80`. Khi chạy container này, bạn thử truy cập vào `http://localhost:80` trên máy chủ (Host) nhưng không kết nối được. Tại sao `EXPOSE` không tự động mở cổng ra bên ngoài?**
A. `EXPOSE` chỉ là một dạng tài liệu (documentation) để báo cho người dùng biết container *dự định* dùng cổng nào. Để truy cập được từ Host, bạn phải dùng cờ `-p` (publish) khi chạy lệnh `docker run` để ánh xạ cổng.
B. Do tường lửa của Linux chặn.
C. Do `EXPOSE` bị lỗi thời, phải dùng `VOLUME`.
D. Do container chưa khởi động xong.

> [!success]- Đáp án & Giải thích
> **Đáp án:** A
> **Giải thích:** `EXPOSE` chỉ định số hiệu cổng giao thức mà container được *thiết kế* để sử dụng. Tuy nhiên, nó không tự động map cổng đó ra host. Việc kết nối thực sự ra thế giới bên ngoài (publish port) phải được thực hiện khi khởi chạy container (thường qua tham số trong lệnh run hoặc qua orchestration).

---

### Câu 10
**Tình huống (Khái niệm Base OS):** Bạn build một Docker image từ `FROM ubuntu:latest`. Bạn chạy image này trên một máy chủ sử dụng hệ điều hành CentOS. Khi bạn gõ lệnh kiểm tra phiên bản Kernel bên trong container (`uname -r`), bạn sẽ thấy phiên bản Kernel của Ubuntu hay CentOS?**
A. Kernel của Ubuntu (do FROM ubuntu).
B. Kernel của CentOS (Host OS).
C. Kernel riêng của Docker.
D. Không có Kernel.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Container chia sẻ hạt nhân (kernel) của hệ điều hành máy chủ. Base OS (Ubuntu) trong Dockerfile chỉ đóng vai trò là một lớp đệm (shim layer) cung cấp các thư viện và file hệ thống (user space), chứ không chứa kernel riêng. Do đó, nó sẽ hiển thị kernel của Host OS (CentOS).
