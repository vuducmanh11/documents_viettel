# Tungsten Fabric Architecture

### Introduction

Mô tả Tungsten Fabric cung cấp platform virtual networking có khả năng mở rộng làm việc với đa dạng VM và container orchestrator, có thể tích hợp với mạng vật lý và cơ sở hạ tầng tính toán. Tungsten fabric sử dụng chuẩn networking như BGP EVPN control plane và VXLAN overlay để kết nối liền mạch workload nởi orchestrator domain khác nhau (VM quản lý bởi VMware vCenter và container bởi Kubernetes).

Tungsten Fabric cung cấp một highly scalable virtual networking platform được thiest kế hỗ trợ multi-tenant network trong các môi trường lơn nhất trong khi hỗ trợ đồng thời nhiều orchestrator.

Vì có rất ít datacenter triển khai thực sự "greenfield", gần như luôn có yêu cầu tích hợp workload đã triển khai trên cơ sở hạ tầng mới với workload và mạng được triển khai trước nó. Sau đây mô tả tập hợp các kịch bản cho triển khai new cloud infrastructure sẽ được triển khai và nơi các cơ sở hạ tầng đã có.

### Use Cases

- Enable Platform-as-a-Service và Software-as-a-Service với khả năng mở rộng và mềm dẻo cao trong các datacenter được quản lý bởi OpenStack
- Virtual networking với hệ thống quản lý bở Kubernete container bao gồm Red Hat OpenShift
- Cho phép môi trường ảo(mới hoặc exist) chạy trên VMware vCenter sử dụng Tungsten Fabric virtual networking giữa các VM
- Kết nối Tungsten Fabric virtual network tới mạng vật lý sử dụng gateway router với BGP peering với networking overlay và trực tiếp thông qua mạng bên dưới data center

Các use case có thể được triển khai trong bất kỳ sự kết hợp để quyết định yêu cầu đặc biệt trong nhiều trường hợp triển khai. Các thuộc tính chính của Tungsten Fabric được minh họa như sau:

![Tungsten Fabric Features](images/TFA_feature_set.png)

- Virtual networking sử dụng encapsulation tunnel giữa các virtualize host
- Plugin cho open-source orchestrator cho các VM và container
- Các chính sách an toàn dựa trên ứng dụng theo tag
- Tích hợp VMware orchestration stack
- Kết nối tới external network sử dụng BGP, SNAT và thông qua overlay network

Vì cùng controller và thành phần chuyển tiếp được sử dụng trong mọi thực hiện, Tungsten Fabric cung cấp một giao diện thống nhất để quản lý connectivity trong tất cả  môi trường support, có thể kết nối liền mạch giữa workloads được quản lý bởi các orchestrator khác nhau, dù VM hay container và tới đích trong external network.

### Key Features of Tungsten Fabric

Tungsten Fabric quản lý và thực hiện virtual networking trong môi trường cloud sử dụng OpenStack và Kubernete orchestrator. Tungsten Fabric sử dụng overlay netwok giữa vRoutes chạy trên mỗi host. Nó đã được chứng minh, công nghệ mạng dựa trên chuẩn ngày nay hỗ trợ mạng diện rộng của đa số nhà cung cấp dịch vụ trên thế giới, nhưng tái sử dụng để làm việc với virtualize workload và cloud automation trong data center có thể bao gồm từ phạm vi trung tâm dữ liệu quy mô lớn tới telco POPs nhỏ hơn. Nó cung cấp nhiều tính năng tăng cường qua thực hiện native networking của orchestrator bao gồm:

- Highly scalable, multi-tenant networking
- Multi-tenant IP address management
- DHCP, ARP proxies to avoid flooding into networks
- Efficient edge replication for broadcast and multicast traffic
- Local, per-tenant DNS resolution
- Distributed firewall with access control lists
- Application-based security policies
- Distributed load balancing across hosts
- Network address translation (1:1 floating IPs and distributed SNAT)
- Service chaining with virtual network functions
- Dual stack IPv4 and IPv6
- BGP peering with gateway routers
- BGP as a Service (BGPaaS) for distribution of routes between privately managed customer networks and service provider networks

## How Tungsten Fabric Works

### Tungsten Fabric Working with An Orchestrator

Tungsten Fabric controller tích hợp với cloud management system như OpenStack hoặc Kubernete. Chức năng của nó đảm bảo khi VM hoặc container được tạo, nó cung cấp kết nối mạng theo network và security policy trong controller hoặc orchestrator.
Tungsten Fabric bao gồm hai phần chính:
- Tungsten Fabric Controller: tập dịch vụ phần mềm chứa model của network và network policy, thường chạy trên một ít server high availability
- Tungsten Fabric vRouter: cài đặt trên mỗi host (VM hoặc container), vRouter xử lý packet forwarding và thực hiện chính sách mạng và bảo mật.

Triển khai chung của Tungsten Fabric như sau:

![Tungsten Fabric private cloud](images/TFA_private_cloud.png)

Tungsten Fabric controller tích hợp với một orchestrator qua một software plugin thực hiện networking service của orchestrator. Vd: Tungsten Fabric plugin cho OpenStack thực hiện Neutron API và kube-network-manager và các thành phần CNI (container network interface) lắng nghe sự kiện mạng liên quan sử dụng Kubernete k8s API.
Tungsten Fabric vRouter thay thế Linux bridge và IP table hoặc Open vSwitch networking trên compute host, và controller cấu hình vRouter để thực hiện chính sách mạng và bảo mật mong muốn.
