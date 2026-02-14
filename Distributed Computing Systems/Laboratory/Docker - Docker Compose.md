** Trạng thái **: #Đang-thực-hiện
**Giảng viên**: #Le-Anh-Tuan

# Docker là gì?
+ Docker là một nền tảng phần mềm, cho phép đóng gói một ứng dụng với tất cả các thành phần cần thiết để triển khai phần mềm như:
	+ thư viện
	+ hệ điều hành
+ Đơn vị tiêu chuẩn là một [container] 
# Mô hình Docker và So sánh Docker vs Vitual Machine
+ Quá trình xâu dựng Docker bao gồm: 
	+ Build: là quá trình mà dev tiến hành xây dựng các ứng dụng sử dụng Docker containers
	+ Ship: là quá trình mà sau khi app đã hoàn thành, lúc này container sẽ được đóng gói và tiến hành gửi đến bất kì đâu mà không cần lo lắng về libs, dependencies....
	+ Run: là quá trình triển khai ứng dụng trên một nền tảng mới như Clouds... mà không cần quan tâm là có đủ gói, môi trường hay không
![[Pasted image 20251031222324.png]]
## So sánh kiến trúc giữa Virtual Machine và Docker
* So sánh kiến trúc hạ tầng giữa VM và Docker*

| Thành phần                         | Virtural Machines                       | Docker                                              |
| ---------------------------------- | --------------------------------------- | --------------------------------------------------- |
| Hạ tầng vật lý<br>(Infrastructure) | Đều chạy trên phần cứng, máy chủ vật lý |                                                     |
| Hệ điều hành chủ <br>(Host OS)     | Có<br>Là nền tảng cho Hyper V           | Có<br>Là nền tảng cho Docker Engine                 |
| Lớp trung gian <br>                | HyperV (VMware, Virtual Box)            | Docker Engine                                       |
| Hệ điều hành khách<br>(Guest OS)   | Mỗi máy ảo 1 hệ điều hành riêng         | Các container chia sẻ kernel chung với Host OS      |
| Ứng dụng                           | Chạy riêng trong VM, tách biệt          | Chạy trong container, chia sẻ kernel, tách biệt nhẹ |

>[!Note] Giải thích phần chia sẻ Kernal của Docker: Docker không ảo hóa toàn bộ hệ điều hành như máy ảo, mà nó sử dụng Kernel của Host thật, tiến hành thực thi thông qua quá trình "ảo hóa tiến trình" chứ không "ảo hóa hệ điều hành như máy ảo".Đối với Docker chạy ubuntu trên windows, kernel sẽ được dùng thông qua WSL (Windows Sub for Linux) để chạy cách tiến trình.
>Mô hình
>[Hardware]
   │
[Windows Kernel]
   │
[WSL2 VM] → (chứa Linux Kernel)
   │
[Docker Engine (trong WSL2)]
   ├── Container 1 (Ubuntu)
   ├── Container 2 (Debian)
   └── Container 3 (Alpine)

>[!Kết luận] Chính vì không cần ảo hóa cả hệ điều hành khách (Gust OS) nên về mặt phần cứng, Docker sẽ tối ưu hơn, tốc độ cũng nhanh hơn. Bên cạnh đó việc đóng gói các thư viện và dependencies khiến cho Docker có tính tái sử dụng cao. Tuy nhiên việc chia sẻ Kernel với Host OS đặt ra vấn đề bảo mật với các hệ thống yêu cầu độ an toàn cao.

## Các khái niệm cơ bản trong Docker
+ **Container**: một đơn vị thực thi, đóng gói môi trường + thư viện + ứng dụng
+ **Image**: Snapshoot có tên (tagimage) của container tại từng lớp
+ **Layer**: Một mảnh của container image, tương ứng với mỗi **dòng** trong **Dockerfile**, ở mỗi Layer ta có thể tạo ra một Container Image
+ **Dockerfile**: file text định nghĩa cấu trúc Docker Image, tiến hành thêm môi trường, thư viện thực thi mà ứng dụng chạy trong container cần (OS, libs, dependencies).
![[Pasted image 20251031235419.png]]

# Các câu lệnh làm việc với Docker
## Khởi chạy container
Để tiến hành khởi chạy một container, thực hiện câu lệnh sau:
`bash docker run <tham số> <image>`
+ Một số tham số phổ biến của câu lệnh  [[Tham số phổ biến Docker]]
+ Trong đó, có thể kể đến hai tham số khá phổ biến khi chạy Docker đó là tham số --name và -it
	+ Với tham số --name, dev chủ động khởi tạo và chạy một Docker có tên container sau tham số --name, dev có thể dễ dàng quản lý các container hơn và tường minh hơn thay vì để Docker tự tạo tên cho container.
	+ Với tham số -it khi đi chung với nhau, sẽ tiến hành tạo một powershell kết nối trực tiếp vào Docker, thực hiện truyền lệnh trực tiếp vào container. Nói đơn giản có thể hình dung là mình đang SSH vào container.
+ 

