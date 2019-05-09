
## Overview: Underlay Overlay Mapping using Contrail Analytics

Ngày nay các cloud data center gồm nhiều server kết nối với nhau cung cấp khả năng tính toán và lưu trữ để chạy hàng loạt ứng dụng. Các server được kết nối tới TOR switch và lần lượt kết nối spine router. Việc triển khai cloud thường được chia sẻ bởi nhiều tenant, mỗi tenant cần nhiều mạng riêng. Các mạng rieng được cung cấp bởi overlay network được tạo bởi tunnel (gre, ip-in-ip, mac-in-mac) qua kết nối bên dưới hoặc vật lý. 

Luồng dữ liệu trên overlay network, Contrail có thể cung cấp thống kê và visualize các traffic trên underlay network. 

## Underlay Overlay Analytics Available in Contrail 

Các số liệu thống kê mà Contrail cung cấp của overlay underlay traffic:
- Hiển thị topology của underlay network 
Giao diện của physical underlay network hiển thị các kết nối giữa các server và VM trên server.
- Hiển thị chi tiết của bất kỳ phần tử nào trong topology
Có thể hiển thị chi tiết pRouter, vRouter, VM và các liên kết giữa chúng, hiển thị ...
- Hiển thị underlay path của overlay flow
Có thể lấy underlay path từ overlay flow và map path trong topology view 

## Architecture and Data Collection

Tổng hợp dữ liệu để ánh xạ  overlay flow với underlay path được xử lý qua các bước bởi Contrail module. 
Các bước cần thiết 
1. SNMP collector module thăm dò các physical router
SNMP collector module nhận ủy quyền và cấu hình của physical router từ Contrail config module, thăm dò tất cả physical router sử dụng SNMP protocol( Simple Network Management Protocol tập các giao thức cho phép kiểm tra các thiết bị mạng). Collector upload dữ liệu tới Contrail Analytic collector. SNMP information được lưu trữ trên pRouter UVEs (physical router user visible entity) 
2. IPfix và sFlow thu thập thông số flow 
Physical router được cấu hình để gửi thông số tới collector, sử dụng một trong các collection protocol: Internet Protocol Flow Information Export (IPFIX) hoặc sFlow ( industry standard for sampled flow of packet export at Layer 2)
3. Topology module read SNPM information
Contrail topology module đọc thông tin SNMP từ pRouter UVE từ analytic API, compute neighbor list, và ghi thông tin hàng xóm tới pRouter UVE. Neighbor list được sử dụng bởi WebUI để hiển thị topology vật lý
4. Contrail user interface read and display topology và statics
Contrail user interface module lấy thông tin topology từ Contrail analytic và hiển thị physical topology. Thông tin lưu trữ  trên analytic hiển thị biểu đồ link statistic, ánh xạ của overlay flow trên underlay network.

## New Processes/Services for Underlay Overlay Mapping 
Các service của TF (OpenContrail) hiển thị với command

```
contrail-status
```

