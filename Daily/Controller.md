

# Managing Distributed Systems
+ Định kì => tốn tài nguyên
+ Chia cụm => đỡ tốn nhưng chưa tối ưu
+ Khi nào có vấn đề => giải quyết vấn đề => Không hỏi thăm định kì để check
# Managing Cloud Applications
Quản trị trên cloud khác với phân tán vì công nghệ thay đổi liên tục
Nếu cài các công cụ quản lý => cần phải có các agent trên các node được quản lý => thông báo đến server (Zabbix)
IOT => phân tán => hạn chế cơ chế định kì => dùng cơ chế trigger (khi có sự thay đổi thì mới gửi thông tin về)
Vì cloud có tính chất thay đổi liên tục nên không thể sử dụng cơ chế spoiling => bắt buộc sử dụng cơ chế agent (quản lý sự kiện) để kiểm soát 

# Control Loops
Hysteresis: Được hiểu là độ trễ giữa điều khiển và hành động (điều khiển nhanh nhưng hành động được thực hiện chậm hơn) => Khi controler điều khiển quá nhanh => sai => cần thêm độ trễ để có thể xác định đúng vấn đề => giải quyết
VD: Auto Scalling => xác định traffic => chờ xem có tăng không => scale nếu không thì thôi.
Cụ thể hơn là không thể đưa ra được một cấu hình chung cho tất cả các máy tính vì mỗi máy có hiệu năng khác nhau => cần thích ứng phù hợp cho các máy => nên dùng phần cứng đồng bộ


# Kubernetes Controllers
Auto Scaling : Scale down cần phải có thời gian chờ để xem có traffic nào phát sinh hay không => nếu scale down quá nhanh thì sẽ k tối ưu cho xử lý traffic
Scaleup thì độ trễ thấp nhất để giải quyết vấn đề nhanh chóng (~ 0s).
=> Khẩn cấp => làm liền => không khẩn cấp thì từ từ
# Tổng kết
Serverless => sử dụng sự kiện => giảm tài nguyên 
Nhược: độ trễ lớn vcl => nên sử dụng cơ chế bất đồng bộ => những phần mềm dùng serverless nên dùng bất đồng bộ 
Cách khắc phục: chia nhỏ các container để tối ưu được tốc độ khởi động (đủ nhỏ nhưng không quá phân tán vì khó quản lý => đi ngược nguyên tắc quản lý) để có tốc độ khởi động nhanh nhất (tuy nhiên chưa đi đến được realtime)