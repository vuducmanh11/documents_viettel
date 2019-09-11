## High Availability for Analytics 
Contrail hỗ trợ multi instance analytics cho tính sẵn có cao và load balancing.
Contrail analytics gồm hai thành phần
- **contrail-collector**: nhận status, logs, flow information từ Contrail processing element (generators) và lưu trữ chúng 

Mọi generator được kết nối tới một trong các **contrail-collector** instance tại mọi thời điểm. Nếu instance fail (or shutdown), tất cả các generator được kết nối tới nó tự động di chuyển tới instance đang hoạt động khác, thường mất khoảng vài giây. Một số thông điệp có thể mất trong quá trình movement. UVEs có khả năng phục hồi message loss, do đó state trong UVE đồng bộ với state trong generator.

- **contrail-opserver**: cung cấp external API để UVEs lấy thông tin và query log và flow 

Mỗi analytics component expose một northbound REST API đại diện bởi **contrail-opserver** service do đó một analytic component failure hay một **contrail-opserver** service fail không ảnh hưởng hoạt động của các instance khác. 
Các cách quản lý kết nối với các **contrail-opserver** endpoint
  - Định kỳ poll **contrail-opserver** service trên các analytics node để xác định list endpoint hoạt động sau đó thực hiện API request từ một hoặc nhiều endpoint hoạt động
  - Contrail user interface sử dụng cùng northbound REST API để hiển thị trên dashboards, và tương tác với bất kỳ **contrail-opserver** high ava

 
