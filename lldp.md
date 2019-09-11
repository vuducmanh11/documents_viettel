# LLDP - Link Layer Discovery Protocol

LLDP là một chuẩn định nghĩa messages, được đóng gói trong Ethernet frame với mục đích tạo giao thức cung cấp thông tin giữa các thiết bị trong mạng LAN. Các message được truyền định kỳ mặc định 30s trên mỗi port. LLDP là một chuẩn tương tự CDP (Cisco Discovery Protocol), nhưng độc lập với nhà cung cấp, được sử dụng để xác định network topology, khắc phục sự cố và tự động quản lý mạng.

## Benefits of LLDP

LLDP cung cấp các lợi ích

- Sử dụng đơn giản trong môi trường multi-vendor
- Discovery chính xác physical network topology, đơn giản khắc phục sự cố trong mạng doanh nghiệp
- Cho phép discovery của các vị trí trong môi trường multi-vendor
- Cung cấp device capability và hỗ trợ option system name, description và management address
- Cung cấp thông tin có thể được sử dụng để phát hiện duplex và speed mismatch
- Discover thiết bị cấu hình lỗi hoặc unreachable IP

## Frame format

Trong LLDP, mỗi thiết bị gửi thông tin từ interface sau mỗi khoảng được fixed, trong khuông dạng Ethernet frame. Mỗi fram chứa một LLDP Data Unit (LLDPDU). Mỗi LLDPDU là một chuỗi các cấu trúc thể hiện type-length-value (TLV). EtherType field được set 0x88cc. Mỗi LLDP frame bắt đầu với Chassis ID, port ID và TTL hoặc hop limit. Frame kết thúc với một special TLV gọi là end of LLDPDU mà cả type, length field được set 0. LLDP chỉ định cho phép các tổ chức định nghĩa và endcode TLV của họ.  Chúng được gọi Organizationally Specific TLVs và bắt đầu với LLDP TLV Type 127.

### LLDPDU types

Có hai kiểu LLDPDU: Normal và shutdown advisory. Normal lldpdu cung cấp thông tin quản lý về local device tới device neighbor. TLV được truyền bắt buộc hoặc tùy chọn. Trong khi đó, khi port disable, LLDP bị disable hoặc switch được reboot. Một lldp shutdown frame được chuyển tới neighboring unit, xác định LLDP information không còn hợp lệ.

## Structure of LLDP Messages

LLDP thay đổi thông tin thông qua đơn vị của dữ liệu gọi là LLDPDU. Các dữ liệu đơn vị gồm TLVs và mỗi trường TLV tương ứng type và length xác định. LLDP chuẩn IEEE 801.1AB có 3 TLV bắt buộc mở đầu một LLDPU theo thứ tự
- Type 1 = Chassis ID (Identifies the device)
- Type 2 = Port ID (Identifies the port)
- Type 3 = Time to live (thời gian thiết bị nhận xác định gói tin còn hợp lệ hay không)

Theo các TLV bắt buộc, một LLDPDU có thể bổ sung, tùy chọn TLVs
- Type 4 = Port description
- Type 5 = System name
- Type 6 = System description
- Type 7 = System capabilities
- Type 8 = Management address
- Type 0 = End of LLDPDU
## LLDP operation modes

Một LLDP agent hoạt động một trong 3 mode

- **Transmit-only mode**: agent chỉ chuyển thông tin về khả năng và current status của local system
- **Receive-only mode**: agent chỉ nhận thông tin về khả năng và current status của remote system
- **Transmit and Receive mode**: agent có thể chuyển khả năng và current status của local system và nhận khả năng và current status của remote system.

Bất cứ khi nào timing counter hết hạn hoặc LLDP information được thay đổi, LLDP agent gửi LLDP frame tới hàng xóm đã enable LLDP. LLDP manager lấy thông tin bên trong MIB (Management Information Base) và định dạng nó thành TLVs và chèn vào LLDPDU. Khi một agent nhận được LLDPDU này, nó kiểm tra để chắc chắn đúng thứ tự chuỗi bắt buộc TLVs, xác thực các option, nếu có lỗi sẽ drop. Khi TLV hợp lệ, nó được lưu trữ trong neighbor database. 

## LLDP Media Endpoint Devices (LLDP-MED)

LLDP-MED là một extension cho LLDP. Giao thức này đặc biệt thường sử dụng để hỗ trợ ứng dụng Voice Over IP (VOIP). LLDP-MED cho phép network discovery giữa các thiết bị kết nối mạng và media endpoints như softphone, IP telephone, VOIP gateway và conference bridge. Mặc định, network device chỉ gửi LLDP packet tới khi nó nhận LLDP-MED packet từ các endpoint device. Nó sẽ tiếp tục gửi LLDP-MED packet cho đến khi remote device kết nối tới thiết bị có khả năng LLDP-MED. Nó hỗ trợ các TLV
- LLDP-MED capabilities TLV
- Network policy TLV
- Power management TLV
- Inventory management TLV
- Location TLV
