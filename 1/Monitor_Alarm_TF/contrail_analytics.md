Contrail (Tungsten Fabric) là hệ thống phân tán gồm các compute node, control node, configuration node, database node, web UI node và analytic node. Analytic node có trách nhiệm thu thập thông tin trạng thái hệ thống, usage statistic, debug information từ tất cả software module trên các node của hệ thống. Analytic node lưu trữ dữ liệu toàn bộ hệ thống trên database dựa trên Apache Cassandra - hệ thống quản lý cơ sở dữ liệu phân tán. Cơ sở dữ liệu được truy vấn thông qua ngôn ngữ giống SQL và REST APIs.

Thông tin trạng thái hệ thống thu thập bởi analytic node được tổng hợp trên các node, được hiện thị trên giao diện cho phép người dùng cập nhật thông tin dễ dàng. 

Debug information được thu thập bởi analytics node gồm
- System log (syslog) message: thông điệp gỡ lỗi, thông tin được tạo bởi software component của hệ thống.
- Object log message: lưu trữ các thay đổi tác động lên các đối tượng hệ thống như VM, virtual network, service instance, virtual router, BGP peer, routing instance
- Trace message: lưu trữ các hoạt động được thu thập cục bộ bởi các software component và gửi tới analytic node theo yêu cầu 

Statistic information liên quan đến flow, CPU, memory usage được thu thập bởi analytic node hiển thị trên giao diện người dùng, cung cấp lịch sử dữ liệu. Các truy vấn được xử lý qua REST APIs.

Analytics data được ghi lại trong cơ sở dữ liệu. Các dữ liệu sẽ được xóa sau khoảng thời gian mặc định (TTL) 48h. Giá trị mặc định có thể được thay đổi theo giá trị *database_ttl* trong cấu hình cluster.
 
