Tungsten Fabric thu thập thông tin từ hạ tầng cloud (compute, network, storage) và các workload(VM, container) chạy trên nó để tạo điều kiện vận hành giám sát, troubleshoot. Dữ liệu được thu thập với nhiều dạng như: syslog, structured message (Sandesh), Ipfix, Sflow và SNMP. Các object như vRouter, physical host, virtual machine, interface, virtual machine, interface, virtual network, các chính sách được mô hình hóa như User Visible Entities (UVEs) và các thuộc tính cho UVE có thể đến từ nhiều nguồn với các định dạng khác nhau. 

Kiến trúc cho việc thu thập phân tích được thể hiện hình sau:

![Architecture for analytics collection](images/TFA_analytics.png)

Các nguồn dữ liệu có thể được cấu hình với địa chỉ IP của collector hoặc có một load balancer cho các collector. Analytic node chuẩn hóa dữ liệu đến theo một định dạng chung, gửi dữ liệu tổng hợp được đến Cassandra database thông qua Kafka service. API URL 

## Analytics REST API

Dữ liệu phân tích được thu thập bởi Tungsten Fabric có thể truy cập thông qua REST API thông qua port 8082 của địa chỉ external virtual IP. Thông tin cấu hình và vận hành được lưu trong các object User Visible Entities (UVEs), được tổng hợp từ các TF component. 
HTTP GET query truy vấn danh sách table trong analytics database, API và schema.

HTTP POST query được sử dụng để truy vấn dữ liệu lưu trữ trong bảng theo thời gian. Câu truy vấn gồm định dạng JSON chứa table, file, condition, start time, end time để lấy dữ liệu.

Analytic API có thể được sử dụng để cấu hình và lấy các alarm dựa trên threshold crossing event cho bất kỳ chuỗi thời gian nào trên analytic database. 
Server-sent event (SSE) stream có thể được cấu hình EVU hoặc alarm in analytic database. 
