# vRouter
Tungsten Fabric vRouter được cài đặt trên mỗi host chạy workloads (VM hoặc container), vRouter xử lý chuyển tiếp packet, thực hiện chính sách mạng và bảo mật.
vRouter giao tiếp với controller thông qua XMPP (một messaging protocol). vRouter thực hiện IRB (Integrated Bridgine and Routing) chức năng của router vật lý. 
vRouter gồm 2 thành phần: vRouter Agent và vRouter Forwarder kết nối với nhau qua ptk0. vRouter chạy trên mỗi host, kết nối với Tungsten Fabric qua overlay network
- vRouter agent chứa VRFs và ACLs nhận request từ Tungsten Fabric controller, sau đó gửi tới vRouter Forwarder. vRouter agent chạy trên user space của máy host. 
- vRouter Forwarder chạy như một kernel module trong user space khi DPDK được sử dụng, hoặc trong một network interface card có thể lập trình "smart NIC"



# OpenvSwitch

## Architecture

OpenvSwitch gồm ba thành phần chính:
- database server (ovsdb-server)
- daemon (ovs-vswitchd)
- kernel module

![](images/ovs-architecture.png)

- Controller là một OpenFlow Controller (Ryu) hoặc một OVS database manager.
- ovsdb-server là thành phần cấu hình database, khôi phục lại thông tin cấu hình bridge, interface sau khi khởi động lại.
- ovs-vswitchd thành phần chính của Open vSwitch xử lý các flow setup
- kernel module: cải thiện hiệu suất của OVS.




# So sánh vRouter với OpenvSwitch
- OVS sử dụng nhiều table với kiến trúc phân cấp: ofproto table, dpcls/Megaflow (subtables), EMC; vRouter sử dụng một huge hash table, tất cả forwarding thread sử dụng chung một flow table.

- Tunnel Types hỗ trợ: OVS hỗ trợ VXLAN, GRE; vRouter hỗ trợ cả VXLAN, MPLSoGRE và MPLSoUDP.

- ARP processing: OVS chỉ broadcast gói tin ARP trừ khi openflow table thực hiện xử lý; vRouter không thực hiện broadcast gói tin ARP, vRouter sử dụng VRRP hoặc host MAC để phản hồi ARP request, thực hiện định tuyến ARP request nếu địa chỉ MAC đích là MAC host, vRouter cũng thực hiện l2 switch bằng cách flooding hoặc forwarding.

# So sánh OpenStack OVS với Tungsten Fabric vRouter
- Control Plane
  - OpenStack OVS: ML2 plugin
  - Tungsten Fabric vRouter: XMPP từ Controller tới vRouter Agent 
- L2 Forwarding
  - OpenStack OVS: kết hợp OVS và Linux Bridge Driver
  - Tungsten Fabric vRouter: được xử lý bình thường
- L2 Gateway
  - OpenStack OVS: neutron cung cấp khả năng kết nối mạng tới L2 Gateway bên ngoài, hỗ trợ đóng gói VXLAN 
  - Tungsten Fabric vRouter: hỗ trợ Native BGP EVPN Layer 2/3, hỗ trợ: MPLSoUDP, MPLSoGRE, VXLAN
- L3 Routing and Protocols
  - OpenStack OVS: 
    - Central routing: thông qua networking node, không phù hợp môi trường nhà khai thác
    - DVR (Distributed Virtual Router) dựa trên OVS ban đầu chỉ hỗ trợ định tuyến giữa các server, hỗ trợ chậm với traffic giữa client và server.
    - Protocol: Static Routes, Neutron BGPVPN
  - Tungsten Fabric vRouter: 
    - Distributed NAT, Floating IP, Security Group hõ trợ sẵn vRouter
    - Protocol: BGP L3VPN, BGP EVPN
- SNAT
  - OpenStack OVS: cả DVR, SNAT xử lý tập trung
  - Tungsten Fabric vRouter: thực hiện SNAT phân tán hoàn toàn
- Security Groups
  - OpenStack OVS: thực hiện với iptable
  - Tungsten Fabric vRouter: xử lý bởi vRouter và phân tán
- DHCP
  - OpenStack OVS: dựa trên triển khai mạng giới hạn mở rộng
  - Tungsten Fabric vRouter: xử lý bởi vRouter và phân tán 
- Floating IP
  - OpenStack OVS: Centralized L3 Agent
  - Tungsten Fabric vRouter: dựa trên SNAT và phân tán hoàn toàn
- L3VPN Gateway Support
  - OpenStack OVS: mặc định không hỗ trợ
  - Tungsten Fabric vRouter: BGP L3VPN và Device Manager cho phép workflow tự động kết nối VM và Container với L3VPN network 

- Native Network Load Balancing
  - OpenStack OVS: không có sẵn
  - Tungsten Fabric vRouter: L3 ECMP




















