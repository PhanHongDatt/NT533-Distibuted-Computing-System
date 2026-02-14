
**Tài liệu tham chiếu:** The Cloud Computing Book (Douglas Comer - Sec 5.7 to 5.18)

---

## 1. Tháp tin cậy & Quyền hạn (Trust Hierarchy)
> [!NOTE] Nguyên lý cốt lõi
> Để đảm bảo an toàn, hệ thống phân chia quyền lực theo tầng. "Cấp dưới muốn làm việc lớn thì phải xin phép cấp trên".

* **Cơ chế:**
    * **App (Least Trust):** Quyền thấp nhất. Muốn truy cập phần cứng -> Gọi OS.
    * **Guest OS:** Quyền trung bình. Tưởng mình nắm phần cứng, nhưng khi ra lệnh cho Bus/IO -> Bị **Hypervisor** chặn lại (Trap).
    * **Hypervisor (Most Trust):** Quyền cao nhất (Root mode). Kiểm soát thực sự phần cứng vật lý.
* **Quy tắc:** Nếu một thực thể cố làm việc vượt quá quyền hạn -> CPU sẽ chuyển quyền điều khiển (return control) cho tầng cao hơn xử lý.

---

## 2. Ảo hóa I/O (Virtual I/O Devices)
*Vấn đề nan giải: OS nào cũng được lập trình để độc chiếm thiết bị (màn hình, ổ cứng, mạng), nhưng Cloud thì phải chia sẻ.*

### Cách giải quyết
Hypervisor tạo ra các **Thiết bị ảo (Virtual Devices)** bằng phần mềm
_Cơ chế là đánh lừa các máy ảo bằng những driver ảo hóa để các máy ảo gần như tưởng mình đang được tương tác trực tiếp với các thiết bị thật, tuy nhiên bản chất thì thông qua Hypervisor_
* Với Guest OS: Nó nhìn thấy một thiết bị thật, giao tiếp qua Bus thật.
* Thực tế: Hypervisor "đứng giữa" (Interception), nhận lệnh từ Guest OS và chuyển tiếp ra phần cứng thật hoặc gửi qua mạng.

###  Hai chiến lược thiết kế thiết bị ảo (Rất quan trọng)
| Chiến lược                                               | Mô tả                                                                                                  | Ưu/Nhược điểm                                                                                                                               |
| :------------------------------------------------------- | :----------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------ |
| **1. Emulation**<br>(Giả lập thiết bị cũ)                | Tạo phần mềm giả lập y hệt một card mạng/ổ cứng thực tế đã tồn tại (VD: Card Intel E1000).             | **Ưu:** Tương thích mọi OS (vì OS đã có sẵn driver cũ).<br>**Nhược:** Chậm, phức tạp vì phải mô phỏng cả những giao thức phần cứng rắc rối. |
| **2. Invent New Device**<br>(Tạo thiết bị mới hoàn toàn) | Thiết kế một chuẩn thiết bị ảo mới, tối ưu riêng cho môi trường ảo hóa. **(Ví dụ kinh điển: virtIO)**. | **Ưu:** Hiệu năng cực cao, thiết kế sạch đẹp, bỏ qua các bước rườm rà của phần cứng cũ.<br>**Nhược:** Cần cài Driver riêng cho Guest OS.    |

> [!TIP] Mở rộng cho bài thi
> **virtIO** là từ khóa "vàng" ở mục này. Nó là đại diện cho kỹ thuật *Paravirtualization I/O* (Hợp tác giữa Guest và Hypervisor để tăng tốc).

---

## 3. Máy ảo là một "Đối tượng Kỹ thuật số" (Digital Object)
* **Khái niệm:** Vì VM được tạo thành hoàn toàn từ phần mềm (Code, Data, Memory State, CPU Registers), nên ta có thể biến cả cái máy tính đang chạy thành một **tập hợp các bytes**.
* **Hệ quả:**
    * Có thể lưu VM thành file (Snapshot).
    * Có thể copy file đó sang máy khác (Cloning).
    * Có thể gửi file đó qua mạng (Migration).
* **Ví dụ Virtual Disk:** Ổ cứng của VM thực chất là các lệnh gửi qua mạng đến hệ thống lưu trữ (Storage) của Data Center.

---

## 4. Di chuyển nóng (Live Migration) 


**Vấn đề:** Làm sao di chuyển VM từ Server A -> Server B mà **không được tắt máy** (Downtime gần như bằng 0)?

**Giải pháp 3 Pha (Three-Phase Approach):**

1.  **Phase 1: Pre-copy (Copy trước)**
    * VM vẫn chạy bình thường trên Server A.
    * Hypervisor copy toàn bộ RAM sang Server B.
    * *Vấn đề nảy sinh:* Trong lúc copy, VM vẫn đang viết dữ liệu mới vào RAM. Những trang nhớ bị thay đổi này gọi là **Dirty Pages**.
2.  **Phase 2: Stop-and-copy (Dừng và copy)**
    * Tạm ngưng VM trên Server A (trong tíc tắc).
    * Chỉ copy nốt các **Dirty Pages** (số lượng rất ít) sang Server B.
3.  **Phase 3: Post-copy (Copy sau/Khôi phục)**
    * Chuyển trạng thái CPU (Registers) sang Server B.
    * VM tiếp tục chạy trên Server B ngay tại điểm nó bị dừng.


---

## 5. Hosted Hypervisor (Type 2)
* **Khác biệt:** Chạy trên nền một OS khác (Host OS) dưới dạng một ứng dụng (User space process).
* **Cơ chế:** Map (ánh xạ) các yêu cầu I/O của máy ảo sang các dịch vụ (System calls) của Host OS.
* **Lợi ích:** Cho phép chạy song song nhiều OS (Linux trên Mac, Windows trên Linux) để testing/dev mà không cần reboot (khác với Multiboot).

---

# Phân loại Công nghệ Ảo hóa (Virtualization Types)

## 1. Phân loại dựa trên kỹ thuật ảo hóa (Techniques)

###  Ảo hóa một phần (Partial / Para-virtualization)

> _Trong sách dùng thuật ngữ: **Para-virtualization**_

- **Cơ chế:** Hệ điều hành khách (Guest OS) **biết** nó đang chạy trong môi trường ảo. Do đó, mã nguồn của Guest OS được sửa đổi (modified) để thay thế các lệnh gọi phần cứng phức tạp bằng các lệnh gọi trực tiếp tới Hypervisor (gọi là _Hypercalls_).
    
- **Đặc điểm:**
    
    - **Ưu điểm:** Hiệu năng cao (gần như máy thật) do loại bỏ được gánh nặng giả lập (emulation overhead).
        
    - **Nhược điểm:** Phải sửa đổi nhân hệ điều hành (Kernel). Khó thực hiện với các OS đóng mã nguồn như Windows (trừ khi cài thêm driver đặc biệt).
        

###  Ảo hóa toàn phần (Full Virtualization)

- **Cơ chế:** Hệ điều hành khách chạy **nguyên bản** (unmodified), không hề biết nó đang bị ảo hóa. Hypervisor giả lập hoàn toàn phần cứng bên dưới.
    
- **Đặc điểm:**
    
    - **Ưu điểm:** Tương thích tuyệt đối. Chạy được mọi hệ điều hành (Windows cũ, Linux cũ...).
        
    - **Nhược điểm:** Trước đây rất chậm do phải dùng phần mềm để "bẫy" (trap) và mô phỏng từng lệnh CPU (Software Emulation). _Tuy nhiên, ngày nay nhờ sự hỗ trợ của phần cứng (mục 3 dưới đây), hiệu năng đã cực tốt._
        

---

## 2. Phân loại dựa trên vị trí của Hypervisor (Architecture)

### Ảo hóa dựa trên phần cứng (Hardware-based / Bare-metal)

> _Thuật ngữ chuyên ngành: **Type 1 Hypervisor**_

- **Cơ chế:** Phần mềm ảo hóa (Hypervisor) chạy **trực tiếp** trên phần cứng vật lý, không cần hệ điều hành chủ nào cả.
    
- **Đại diện:** VMware ESXi, Microsoft Hyper-V, KVM (thực tế KVM biến Linux thành Type 1).
    
- **Ứng dụng:** Dùng cho **Cloud Data Center** (AWS, Google Cloud) vì hiệu năng cao nhất và quản lý tài nguyên tốt nhất.
    

### Ảo hóa dựa trên hệ điều hành (Operating System-based / Hosted)

> _Thuật ngữ chuyên ngành: **Type 2 Hypervisor**_

- **Cơ chế:** Hypervisor chạy như một **ứng dụng phần mềm** trên một hệ điều hành thông thường (gọi là Host OS).
    
- **Đại diện:** VMware Workstation, Oracle VirtualBox.
    
- **Ứng dụng:** Dùng cho **Personal Computer (Laptop)**.
    
    - _Ví dụ:_ Bạn dùng Laptop chạy Windows (Host OS), cài VirtualBox (Hosted Hypervisor) để chạy máy ảo Ubuntu (Guest OS).
        
- **Lưu ý:** Hiệu năng thấp hơn Type 1 vì mọi lệnh phải đi qua thêm một lớp Host OS.
    

---

## 3. Ảo hóa thành phần (Components)

###  Ảo hóa thiết bị đầu vào/ra (Virtual I/O Devices)

- **Vấn đề:** Ổ cứng, Card mạng, Bàn phím là tài nguyên vật lý hữu hạn, nhưng hàng chục VM đều muốn dùng chung.
    
- **Giải pháp:** Hypervisor tạo ra các **Thiết bị ảo (Virtual Devices)** bằng phần mềm.
    
    - Khi Guest OS gửi lệnh ghi dữ liệu vào "Ổ cứng ảo", Hypervisor chặn lại (Intercept) và chuyển tiếp lệnh đó xuống ổ cứng thật hoặc gửi qua mạng (SAN/NAS).
        
- **Công nghệ nổi bật:** **VirtIO** (Chuẩn giao tiếp I/O ảo hóa giúp tăng tốc độ bằng cách cho phép Guest OS hợp tác với Hypervisor).
    

---

## 4. Xu hướng hiện đại: Containerization (Chương 6)

Đây là sự tiến hóa tiếp theo của ảo hóa, khắc phục nhược điểm "nặng nề" của máy ảo truyền thống (VM).

### Ảo hóa Container (Containerization)

> _Thuật ngữ chuyên ngành: **OS-level Virtualization (Ảo hóa cấp hệ điều hành)**_

- **Sự khác biệt cốt lõi với VM:**
    
    - **VM:** Ảo hóa phần cứng. Mỗi VM phải "gánh" một bộ nhân hệ điều hành (Guest OS) riêng -> Tốn RAM, khởi động lâu (vài phút).
        
    - **Container:** Ảo hóa **Hệ điều hành**. Tất cả Container **dùng chung một nhân (Kernel)** của Host OS, chỉ đóng gói thư viện (Libs) và ứng dụng (App).
        
- **Đặc điểm:**
    
    - **Siêu nhẹ (Lightweight):** Kích thước tính bằng Megabytes (thay vì Gigabytes như VM).
        
    - **Khởi động cực nhanh:** Tính bằng mili-giây (vì không cần boot OS).
        
    - **Đại diện:** **Docker** (công cụ tạo container), **Kubernetes** (công cụ quản lý container).

| **Tiêu chí**                | **Virtual Machine (VM)**            | **Container**                       |
| --------------------------- | ----------------------------------- | ----------------------------------- |
| **Cấp độ ảo hóa**           | Hardware Virtualization (Phần cứng) | OS Virtualization (Hệ điều hành)    |
| **Thành phần**              | App + Libs + **Guest OS**           | App + Libs (Dùng chung Host Kernel) |
| **Kích thước**              | Nặng (GB)                           | Nhẹ (MB)                            |
| **Khởi động**               | Chậm (Vài phút)                     | Nhanh (Giây/Mili-giây)              |
| **Tính cô lập (Isolation)** | Cao (An toàn hơn)                   | Thấp hơn (Do dùng chung Kernel)     |
Chào bạn, tiếp tục với **Chương 5: Virtual Machines**, tôi đã soạn thảo bộ câu hỏi tập trung vào các **khái niệm cốt lõi** và **từ khóa quan trọng**.

Chương này giải thích cơ chế hoạt động của máy ảo, phân biệt các loại ảo hóa và cách chúng đạt được hiệu suất cao (gần như thiết bị thật) trong môi trường đám mây.

Dưới đây là nội dung định dạng Markdown để bạn lưu vào Obsidian:

---

# Pool Câu Hỏi Trắc Nghiệm - Cloud Computing Chapter 5

## Chủ đề: Khái niệm Máy ảo (VM) và Công nghệ Ảo hóa

### Câu 1
**Ba phương pháp tiếp cận chính đối với công nghệ ảo hóa được đề cập trong chương này là gì?**
A. Software emulation, Para-virtualization, và Full virtualization.
B. Hardware emulation, Network virtualization, và Storage virtualization.
C. Partial virtualization, Total virtualization, và Zero virtualization.
D. Type 1, Type 2, và Type 3 virtualization.

> [!success]- Đáp án & Giải thích
> **Đáp án:** A
> **Giải thích:** Các công nghệ được sử dụng để triển khai máy ảo có thể được chia thành ba loại lớn: Mô phỏng phần mềm (Software emulation), Bán ảo hóa (Para-virtualization), và Ảo hóa toàn phần (Full virtualization).

---

### Câu 2
**Nhược điểm lớn nhất của phương pháp "Software Emulation" (Mô phỏng phần mềm) khiến nó không phù hợp làm hình thức ảo hóa chính trong trung tâm dữ liệu là gì?**
A. Nó không hỗ trợ đồ họa.
B. Nó thêm chi phí (overhead) đáng kể dẫn đến tốc độ thực thi chậm.
C. Nó chỉ chạy được trên Windows.
D. Nó yêu cầu phần cứng quá đắt tiền.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Mặc dù mang lại tính di động cao, mô phỏng phần mềm có nhược điểm lớn là thêm chi phí đáng kể, dẫn đến việc thực thi chậm, khiến nó không hấp dẫn để sử dụng làm hình thức ảo hóa chính trong trung tâm dữ liệu.

---

### Câu 3
**Đặc điểm chính phân biệt "Para-virtualization" (Bán ảo hóa) so với ảo hóa toàn phần là gì về mặt yêu cầu đối với hệ điều hành khách (Guest OS)?**
A. Guest OS phải là Windows.
B. Mã nguồn của Guest OS phải được sửa đổi (thay thế các chỉ thị đặc quyền bằng các lệnh gọi hypervisor) trước khi có thể sử dụng.
C. Guest OS không cần thay đổi gì cả.
D. Guest OS phải chạy trên đĩa mềm.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Để ngăn hệ điều hành chiếm quyền kiểm soát phần cứng, Para-virtualization yêu cầu mã của hệ điều hành phải được thay đổi (altered) trước khi sử dụng. Các chỉ thị đặc quyền phải được thay thế bằng các lệnh gọi đến chức năng của hypervisor.

---

### Câu 4
**Thành phần phần mềm chịu trách nhiệm tạo và quản lý các máy ảo (VM) được gọi là gì?**
A. Supervisor.
B. Hypervisor.
C. Manager.
D. Controller.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Phần mềm chịu trách nhiệm tạo và quản lý các máy ảo được gọi là hypervisor. Chúng ta coi hypervisor như là thành phần kiểm soát phần cứng bên dưới.

---

### Câu 5
**Tại sao các ứng dụng chạy trong môi trường "Full Virtualization" (Ảo hóa toàn phần) có thể đạt được tốc độ gần như phần cứng thật (native speed)?**
A. Vì Hypervisor ép xung nhịp CPU lên cao hơn.
B. Vì hầu hết các chỉ thị (instructions) của ứng dụng được CPU thực thi trực tiếp mà không cần qua phần mềm trung gian mô phỏng.
C. Vì máy ảo bỏ qua các bước kiểm tra bảo mật.
D. Vì ứng dụng được viết lại bằng ngôn ngữ máy.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Khía cạnh quan trọng nhất của công nghệ VM là chi phí thấp (low overhead). Hầu hết các chỉ thị, đặc biệt là trong mã ứng dụng, thực thi "natively" (trực tiếp trên phần cứng) với tốc độ tối đa,.

---

### Câu 6
**Trong kiến trúc VM, "Privilege Levels" (Các mức đặc quyền) của bộ xử lý được mở rộng như thế nào so với máy tính truyền thống?**
A. Chỉ có một mức duy nhất cho tất cả.
B. Thêm một mức ưu tiên thấp hơn cho Virus.
C. Thêm một mức đặc quyền mới dành cho Hypervisor (Hypervisor mode), bên cạnh Kernel mode (cho OS) và User mode (cho Apps).
D. Loại bỏ Kernel mode.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Khi máy chủ chạy hypervisor và các VM, bộ xử lý sử dụng ba mức đặc quyền: một cho hypervisor, một thứ hai cho hệ điều hành (kernel mode), và một thứ ba cho các ứng dụng (user mode).

---

### Câu 7
**Khái niệm "Virtual I/O Device" (Thiết bị nhập/xuất ảo) hoạt động như thế nào đối với Hệ điều hành khách (Guest OS)?**
A. Guest OS nhìn thấy thiết bị như một thiết bị vật lý thật và giao tiếp qua bus ảo, nhưng thực tế Hypervisor chặn và xử lý các yêu cầu này bằng phần mềm.
B. Guest OS biết đó là thiết bị giả và từ chối hoạt động.
C. Guest OS kết nối trực tiếp với thiết bị vật lý bỏ qua Hypervisor.
D. Guest OS không cần thiết bị I/O.

> [!success]- Đáp án & Giải thích
> **Đáp án:** A
> **Giải thích:** Từ quan điểm của hệ điều hành, thiết bị ảo không thể phân biệt được với thiết bị thật vì giao tiếp với thiết bị ảo sử dụng các hoạt động bus giống hệt như giao tiếp với thiết bị vật lý.

---

### Câu 8
**Tính năng "VM Migration" (Di chuyển máy ảo) dựa trên nguyên lý cốt lõi nào của máy ảo?**
A. Máy ảo rất nhẹ.
B. Máy ảo là một "Digital Object" (Đối tượng kỹ thuật số), toàn bộ trạng thái, bộ nhớ và thiết bị của nó chỉ là dữ liệu phần mềm nên có thể chuyển thành các bytes và gửi qua mạng.
C. Máy ảo có chân và tự di chuyển.
D. Máy ảo sử dụng trí tuệ nhân tạo.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Vì VM được thực hiện bằng phần mềm, tất cả các mảnh của VM có thể được thu thập lại thành một đối tượng kỹ thuật số (digital object). Hypervisor có thể chuyển đổi VM thành các bytes và gửi qua mạng đến một máy chủ mới,.

---

### Câu 9
**Ba giai đoạn của quy trình "Live Migration" (Di chuyển trực tiếp) giúp giảm thiểu thời gian ngừng hoạt động (downtime) là gì?**
A. Stop, Copy, Start.
B. Pre-copy, Stop-and-copy, Post-copy.
C. Copy RAM, Copy Disk, Copy CPU.
D. Freeze, Move, Thaw.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Kỹ thuật được sử dụng chia việc di chuyển thành ba giai đoạn: Phase 1: pre-copy (sao chép bộ nhớ khi VM vẫn chạy), Phase 2: stop-and-copy (tạm dừng để chép phần thay đổi), Phase 3: post-copy (gửi trạng thái còn lại).

---

### Câu 10
**"Hosted Hypervisor" (ví dụ: VMware Workstation, VirtualBox chạy trên laptop cá nhân) khác với Hypervisor trong Data Center (Type 1) ở điểm nào?**
A. Nó chạy trực tiếp trên phần cứng.
B. Nó chạy như một ứng dụng (Application) trên nền một hệ điều hành chủ (Host OS) thông thường.
C. Nó mạnh hơn Type 1.
D. Nó không thể chạy Windows.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Một dạng công nghệ VM thay thế cho phép hypervisor chạy trên một hệ điều hành thông thường (ví dụ: trên laptop của người dùng). Người dùng khởi chạy "Hosted hypervisor" như một ứng dụng chạy cùng với các ứng dụng khác.


# Pool Câu Hỏi Trắc Nghiệm Vận Dụng & Đánh Đố - Cloud Computing Chapter 5

## Chủ đề: Cơ chế hoạt động sâu của Hypervisor, Live Migration và Tối ưu hóa I/O

### Câu 21
**Trong quy trình "Live Migration" (Di chuyển nóng) gồm 3 giai đoạn, điều gì sẽ xảy ra về mặt kỹ thuật nếu Máy ảo (VM) đang chạy một ứng dụng ghi dữ liệu vào RAM (memory-intensive) với tốc độ nhanh hơn tốc độ truyền tải mạng giữa máy chủ nguồn và máy chủ đích?**
A. Quá trình di chuyển sẽ thành công ngay lập tức nhờ cơ chế nén dữ liệu.
B. Giai đoạn 1 (Pre-copy) sẽ không bao giờ kết thúc hoặc giai đoạn 2 (Stop-and-copy) sẽ kéo dài quá lâu, gây gián đoạn dịch vụ đáng kể (downtime).
C. Hypervisor sẽ tự động tắt VM để bảo vệ dữ liệu.
D. VM sẽ bị phân mảnh và chạy trên cả hai máy chủ cùng lúc.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Trong giai đoạn 1 (pre-copy), bộ nhớ được sao chép khi VM vẫn chạy. Nếu VM thay đổi các trang nhớ (tạo ra "dirty pages") nhanh hơn tốc độ mạng có thể chuyển chúng đi, thì việc đồng bộ hóa sẽ không bao giờ hoàn tất hoặc buộc hệ thống phải chuyển sang giai đoạn 2 (dừng VM) với lượng dữ liệu còn lại rất lớn, gây ra thời gian ngưng trệ dài.

---

### Câu 22
**Một nhà phát triển muốn tối ưu hóa hiệu suất I/O cho VM. Thay vì để Hypervisor mô phỏng một ổ đĩa vật lý thương mại (như SCSI hay IDE cũ kỹ) vốn có giao diện phức tạp, họ chọn phương án "Invent a new, imaginary device" (như virtIO). Lợi thế cốt lõi của phương án này được đề cập trong sách là gì?**
A. Nó cho phép VM truy cập trực tiếp vào phần cứng vật lý mà không qua Hypervisor.
B. Nó giúp che giấu các chi tiết phần cứng lộn xộn, cung cấp một thiết kế "sạch" và hiệu quả hơn để viết driver, thậm chí có thể tối ưu hóa bằng cách tạo ra các block ảo kích thước lớn.
C. Nó giúp VM chạy được trên mọi hệ điều hành mà không cần cài driver.
D. Nó biến ổ cứng thành RAM để tăng tốc độ.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Sách nêu rõ việc tạo ra một thiết bị ảo tưởng tượng (imaginary device) cho phép lập trình viên tạo ra một thiết kế đẹp và sạch (clean design), tránh các chi tiết phần cứng lộn xộn của thiết bị thương mại, và có thể tăng hiệu quả (ví dụ: tạo đĩa ảo với block lớn).

---

### Câu 23
**Xét về mô hình "Privilege Levels" (Các mức đặc quyền), khi một ứng dụng (App) chạy trên một VM (Full Virtualization) thực hiện một lệnh tính toán cộng trừ đơn giản (ADD instruction), CPU sẽ thực thi lệnh này ở chế độ nào và với tốc độ ra sao?**
A. CPU thực thi ở chế độ Hypervisor Mode với tốc độ chậm do phải qua lớp mô phỏng.
B. CPU thực thi ở chế độ Kernel Mode vì App nằm trong VM.
C. CPU thực thi trực tiếp ở chế độ User Mode với tốc độ phần cứng (native speed), không qua can thiệp của Hypervisor hay OS.
D. CPU từ chối thực thi vì App không có quyền truy cập phần cứng.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Trong ảo hóa toàn phần, các chỉ thị (đặc biệt là mã ứng dụng không đặc quyền) được thực thi "natively" (trực tiếp) trên phần cứng ở tốc độ tối đa. Ứng dụng vẫn chạy ở User mode (mức tin cậy thấp nhất), và phần cứng ngăn cản nó can thiệp vào OS hay Hypervisor,.

---

### Câu 24
**Sự khác biệt tinh tế nhất giữa "Hosted Hypervisor" (Loại 2, ví dụ: chạy trên laptop) và "Bare-metal Hypervisor" (Loại 1, chạy trong Data Center) về cách xử lý yêu cầu I/O mạng từ máy ảo là gì?**
A. Hosted Hypervisor cho phép VM sở hữu hoàn toàn card mạng vật lý.
B. Hosted Hypervisor phải chuyển tiếp (map) các yêu cầu I/O từ Guest OS sang các dịch vụ của Host OS, trong khi Loại 1 kiểm soát phần cứng trực tiếp.
C. Hosted Hypervisor không hỗ trợ kết nối Internet.
D. Hosted Hypervisor sử dụng kết nối Wifi, còn Loại 1 dùng dây cáp.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Hosted Hypervisor chạy như một ứng dụng trên Host OS. Nó phải sắp xếp một ánh xạ (mapping) giữa các yêu cầu I/O của Guest OS và các dịch vụ của Host OS. Ví dụ, nó chuyển tiếp yêu cầu mạng tới Host OS để xử lý thay vì điều khiển trực tiếp phần cứng như Hypervisor loại 1.

---

### Câu 25
**Trong ngữ cảnh ảo hóa, tại sao kỹ thuật "Para-virtualization" (Bán ảo hóa) lại yêu cầu phải sửa đổi mã nguồn của Hệ điều hành khách (Guest OS), trong khi Full Virtualization thì không?**
A. Vì Para-virtualization chạy trên phần cứng cũ không hỗ trợ ảo hóa.
B. Để thay thế các chỉ thị đặc quyền (privileged instructions) trong OS bằng các lời gọi trực tiếp đến hàm của Hypervisor (Hypercalls), ngăn OS chiếm quyền kiểm soát phần cứng.
C. Để làm cho giao diện đồ họa đẹp hơn.
D. Vì Para-virtualization chỉ hỗ trợ Linux.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Để ngăn hệ điều hành chiếm quyền kiểm soát phần cứng trong môi trường Para-virtualization, mã của hệ điều hành phải được thay đổi. Các chỉ thị đặc quyền phải được thay thế bằng các lệnh gọi đến chức năng của hypervisor,.

---

### Câu 26
**Tình huống:** Một máy chủ vật lý đang chạy 10 máy ảo. Hypervisor phát hiện một máy ảo (VM A) đang cố gắng thực hiện một lệnh truy cập trực tiếp vào thanh ghi điều khiển của phần cứng (hardware control register). Dựa trên cơ chế "Trust Hierarchy" (Phân cấp tin cậy), hành động nào sẽ xảy ra tiếp theo?**
A. Phần cứng cho phép lệnh thực thi vì VM A là một hệ điều hành.
B. Phần cứng chặn lệnh này, gây ra một ngoại lệ (exception) và chuyển quyền điều khiển cho Hypervisor để xử lý.
C. Máy chủ vật lý sẽ tự khởi động lại.
D. Lệnh được chuyển cho VM kế bên (VM B) xử lý.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Khi một thực thể (ở đây là Guest OS trong VM) cố gắng thực hiện một thao tác vượt quá mức độ tin cậy của nó (truy cập phần cứng trực tiếp), bộ xử lý sẽ trả quyền điều khiển cho phần mềm ở mức tin cậy cao hơn tiếp theo, trong trường hợp này là Hypervisor.

---

### Câu 27
**Công nghệ nào cho phép một Hosted Hypervisor (chạy trên laptop) có thể chia sẻ tệp tin (share files) giữa Host OS và Guest OS một cách hiệu quả mà không cần qua mạng?**
A. Sử dụng USB ảo.
B. Sử dụng cơ chế ánh xạ (mapping) I/O để tạo ra các thư mục chia sẻ (shared folders), cho phép ứng dụng trong VM truy cập dữ liệu trên Host như một ổ đĩa cục bộ.
C. Sao chép dữ liệu vào RAM của CPU.
D. Gửi email từ Guest OS ra Host OS.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Hosted hypervisor có thể sắp xếp để một số tệp hoặc hệ thống tệp được chia sẻ giữa hệ điều hành khách và chủ. Điều này cung cấp một cách hiệu quả để các ứng dụng trong VM chia sẻ dữ liệu với các ứng dụng chạy trên máy chủ.

---

### Câu 28
**Khái niệm "VM as a Digital Object" (VM là một đối tượng kỹ thuật số) mang lại khả năng quản trị quan trọng nào cho trung tâm dữ liệu ngoài việc di chuyển (migration)?**
A. Khả năng biến VM thành robot vật lý.
B. Khả năng "Snapshot" (chụp ảnh nhanh) trạng thái của VM để sao lưu hoặc phục hồi (rollback) về điểm thời gian trước đó khi gặp sự cố.
C. Khả năng tăng tốc độ CPU lên vô hạn.
D. Khả năng chạy VM mà không cần điện.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Vì toàn bộ VM (bộ nhớ, trạng thái CPU, đĩa ảo) đều là dữ liệu phần mềm, chúng có thể được thu thập thành một đối tượng kỹ thuật số. Điều này cho phép lưu lại trạng thái (snapshot), sao chép, hoặc di chuyển dễ dàng như xử lý một file,.
Chào bạn, tiếp tục với **Chương 5: Virtual Machines**, tôi đã thiết kế bộ câu hỏi ở mức độ **vận dụng cao và thực tiễn**.


# Pool Câu Hỏi Thực Tế & Vận Dụng Cao - Cloud Computing Chapter 5

## Chủ đề: Tối ưu hóa Hiệu năng VM, Xử lý sự cố Di chuyển (Migration) và Kiến trúc Bảo mật

### Câu 1
**Tình huống:** Bạn đang quản lý một máy chủ cơ sở dữ liệu (Database Server) chạy trên máy ảo. Ứng dụng này thực hiện ghi dữ liệu vào bộ nhớ (RAM) liên tục với tốc độ rất cao. Khi bạn cố gắng thực hiện "Live Migration" để chuyển VM này sang máy chủ vật lý khác để bảo trì, bạn nhận thấy quá trình này bị "treo" ở giai đoạn đầu tiên rất lâu và không thể hoàn thành. Dựa trên cơ chế 3 pha của Live Migration, nguyên nhân kỹ thuật là gì?**
A. Do máy chủ đích không có đủ ổ cứng để chứa dữ liệu.
B. Do tốc độ ghi thay đổi dữ liệu vào RAM (dirty pages) của ứng dụng nhanh hơn tốc độ mạng có thể truyền tải các trang nhớ đó sang máy chủ mới trong pha "Pre-copy".
C. Do Hypervisor bị lỗi phần mềm.
D. Do pha "Post-copy" không tìm thấy CPU tương thích.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Trong pha 1 (pre-copy), bộ nhớ được sao chép trong khi VM vẫn chạy. Hệ thống theo dõi các trang nhớ bị thay đổi (dirty pages). Nếu VM thay đổi bộ nhớ nhanh hơn tốc độ mạng chuyển đi, quá trình đồng bộ hóa sẽ không bao giờ hội tụ để chuyển sang pha tiếp theo, khiến quá trình di chuyển bị kéo dài vô tận hoặc thất bại.

---

### Câu 2
**Tình huống:** Một nhà phát triển phàn nàn rằng hiệu suất mạng của máy ảo rất thấp. Khi kiểm tra cấu hình, bạn thấy máy ảo đang sử dụng driver mô phỏng card mạng "Intel E1000" (một thiết bị phần cứng cũ). Để tối ưu hóa hiệu suất I/O mà không cần thay đổi phần cứng vật lý, bạn nên đề xuất chuyển sang loại thiết bị ảo nào và tại sao?**
A. Chuyển sang mô hình "Software Emulation" hoàn toàn để tăng tính tương thích.
B. Chuyển sang sử dụng "Imaginary Device" (như virtIO) vì nó được thiết kế sạch (clean design), tránh các chi tiết phần cứng lộn xộn của thiết bị thương mại và tối ưu hóa việc truyền dữ liệu giữa Guest OS và Hypervisor.
C. Gán trực tiếp (Passthrough) toàn bộ card mạng vật lý cho VM, dù điều này làm mất khả năng chia sẻ card mạng cho các VM khác.
D. Tăng số lượng vCPU cho máy ảo.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Sách đề xuất phương pháp "Invent a new, imaginary device". Thay vì mô phỏng một thiết bị phần cứng cũ với giao diện phức tạp, thiết bị ảo tưởng tượng (như virtIO) cung cấp một thiết kế hiệu quả, tránh các chi tiết phần cứng lộn xộn, giúp viết driver dễ dàng hơn và tăng hiệu suất I/O.

---

### Câu 3
**Tình huống:** Một công ty muốn chuyển đổi hệ điều hành cũ (Legacy OS) từ máy chủ vật lý lên đám mây. Hệ điều hành này là một phiên bản Windows cũ đã ngừng hỗ trợ (closed source) và không thể sửa đổi mã nguồn. Kỹ sư hệ thống đề xuất sử dụng công nghệ "Para-virtualization" để có hiệu suất cao nhất. Đề xuất này có khả thi không?**
A. Có, vì Para-virtualization hỗ trợ mọi hệ điều hành.
B. Không, vì Para-virtualization yêu cầu phải sửa đổi mã nguồn của Guest OS (thay thế lệnh đặc quyền bằng hypercalls), điều này không thể thực hiện được với OS đóng mã nguồn.
C. Có, nhưng cần cài đặt thêm driver.
D. Không, vì Windows cũ không chạy được trên CPU hiện đại.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Para-virtualization yêu cầu mã của hệ điều hành phải được thay đổi (altered) trước khi sử dụng để thay thế các chỉ thị đặc quyền bằng các lệnh gọi đến hypervisor. Do đó, nó không phù hợp với các hệ điều hành đóng mã nguồn hoặc không thể sửa đổi,.

---

### Câu 4
**Tình huống:** Trong một cuộc kiểm tra bảo mật, một hacker đã chiếm được quyền kiểm soát cấp độ "Kernel Mode" bên trong một máy ảo (Guest OS). Hacker này cố gắng thực hiện một lệnh truy cập trực tiếp vào vùng nhớ vật lý thuộc về một máy ảo khác trên cùng host. Cơ chế "Trust Hierarchy" của ảo hóa sẽ ngăn chặn điều này như thế nào?**
A. Guest OS sẽ tự động phát hiện và chặn hacker.
B. Các lệnh truy cập phần cứng nhạy cảm từ Kernel Mode (của Guest OS) sẽ bị CPU chặn lại vì thiếu quyền (do đang chạy dưới quyền Hypervisor Mode), quyền điều khiển sẽ được trả về cho Hypervisor để xử lý và từ chối.
C. Hacker sẽ thành công vì Kernel Mode là cấp độ quyền lực cao nhất.
D. Máy chủ sẽ tự động tắt nguồn.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Trong mô hình ảo hóa, có 3 mức đặc quyền: Hypervisor, Kernel (Guest OS), và User (App). Hypervisor chạy ở mức tin cậy cao nhất. Nếu Guest OS (mức tin cậy thấp hơn) cố gắng thực hiện hành động vượt quá quyền hạn (như truy cập bộ nhớ không thuộc về nó), phần cứng sẽ trả quyền điều khiển về cho Hypervisor,.

---

### Câu 5
**Tình huống:** Bạn cần sao lưu (backup) toàn bộ trạng thái của một máy ảo đang chạy, bao gồm cả nội dung trong RAM và trạng thái các thanh ghi CPU, để có thể phục hồi lại chính xác thời điểm đó nếu quá trình nâng cấp phần mềm thất bại. Tính chất nào của VM cho phép thực hiện việc này dễ dàng so với máy chủ vật lý?**
A. Tính chất "VM as a Digital Object".
B. Tính chất "Live Migration".
C. Tính chất "Para-virtualization".
D. Tính chất "Virtual I/O".

> [!success]- Đáp án & Giải thích
> **Đáp án:** A
> **Giải thích:** Vì VM được thực hiện hoàn toàn bằng phần mềm, tất cả các mảnh của VM (bộ nhớ, trạng thái CPU, thiết bị ảo) có thể được thu thập lại thành một đối tượng kỹ thuật số (digital object). Điều này cho phép lưu snapshot toàn bộ trạng thái VM vào đĩa như một file,.

---

### Câu 6
**Tình huống:** Một nhà phát triển phần mềm muốn tạo một môi trường test cô lập trên laptop cá nhân (chạy Windows) để chạy thử ứng dụng trên Linux. Họ cài đặt VMware Workstation (Hosted Hypervisor). Khi máy ảo Linux gửi một gói tin mạng ra Internet, Hosted Hypervisor xử lý gói tin này khác với Bare-metal Hypervisor (Type 1) như thế nào?**
A. Nó gửi trực tiếp ra card mạng vật lý.
B. Nó phải chuyển tiếp (map) yêu cầu I/O đó sang hệ điều hành chủ (Windows Host OS) và nhờ Windows thực hiện việc gửi gói tin, thay vì điều khiển phần cứng trực tiếp.
C. Nó tạo ra một card mạng vật lý mới.
D. Nó không thể kết nối mạng.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Hosted hypervisor chạy như một ứng dụng trên Host OS. Nó phải sắp xếp một ánh xạ (mapping) giữa các yêu cầu I/O của Guest OS và các dịch vụ của Host OS. Ví dụ, nó chuyển tiếp yêu cầu mạng tới Host OS để xử lý.

---

### Câu 7
**Tình huống:** Trong pha 2 "Stop-and-copy" của quá trình Live Migration, tại sao thời gian dừng (downtime) của VM lại rất ngắn, mặc dù VM có thể có dung lượng RAM rất lớn (ví dụ 64GB)?**
A. Vì toàn bộ 64GB đã được nén lại còn vài KB.
B. Vì pha này không sao chép dữ liệu.
C. Vì phần lớn dữ liệu (64GB) đã được sao chép trong pha 1 (Pre-copy) khi VM đang chạy. Pha 2 chỉ sao chép phần nhỏ các trang nhớ bị thay đổi (dirty pages) trong quá trình pha 1 diễn ra.
D. Vì VM sử dụng chung RAM với máy chủ đích.

> [!success]- Đáp án & Giải thích
> **Đáp án:** C
> **Giải thích:** Pha 1 (pre-copy) sao chép toàn bộ bộ nhớ khi VM vẫn chạy. Pha 2 chỉ tạm dừng VM để sao chép những thay đổi (dirty pages) xảy ra sau pha 1. Vì lượng thay đổi thường nhỏ so với tổng dung lượng, thời gian dừng là rất ngắn.

---

### Câu 8
**Tình huống:** Một ứng dụng chạy trong VM thực hiện hàng ngàn phép tính cộng trừ (CPU intensive). Người quản trị lo ngại rằng việc chạy trong VM sẽ làm chậm ứng dụng do các lớp ảo hóa. Dựa trên kiến trúc "Full Virtualization", bạn giải thích thế nào để trấn an họ?**
A. Ứng dụng sẽ chạy chậm hơn 50% do phải qua Hypervisor.
B. Hầu hết các chỉ thị của ứng dụng (user mode instructions) được CPU thực thi trực tiếp (natively) trên phần cứng mà không cần sự can thiệp của Hypervisor hay OS, nên tốc độ gần như tương đương máy vật lý.
C. Hypervisor sẽ ép xung CPU để bù đắp hiệu năng.
D. Ứng dụng chạy nhanh hơn do được ảo hóa.

> [!success]- Đáp án & Giải thích
> **Đáp án:** B
> **Giải thích:** Đặc điểm quan trọng nhất của VM là chi phí thấp (low overhead). Hầu hết các chỉ thị, đặc biệt là mã ứng dụng, thực thi "natively" (trực tiếp trên phần cứng) với tốc độ tối đa. Hypervisor chỉ can thiệp khi có lệnh đặc quyền,.
