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

>[!Giải pháp] Thay vì xây dựng hệ thống máy chủ phân tán, tiến hành xây đựng hệ thống **datacenter**
>**DataCenter**: nơi tập hợp hàng trăm ngàn máy chủ, được xếp chồng trên các tủ rack với nhau

