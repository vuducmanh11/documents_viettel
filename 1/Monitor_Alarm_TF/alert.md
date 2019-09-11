# Contrail Alerts

## Contrail Alerts Overview

[link-reference](https://github.com/Juniper/contrail-controller/wiki/Contrail-Alerts-Overview)

### Summary

Alert được cung cấp trên mỗi UVE. Contrail Analytic sẽ bắn (hoặc gỡ) alert sử dụng python-coded 'rules', nó sẽ kiểm tra thông tin từ UVE và object's config. Một số rule được tạo sẵn, các rule khác có thể được thêm bởi python plugin stevedore.

Contrail Analytics API cung cấp 
- Quyền đọc Alert một phần của UVE GET API
- Alert acknowledgement sử dụng POST request
- UVE và Alert streaming sử dụng Server-Sent Events 

#### Alert format 

GET http://<analytics-ip>:8081/analytics/alarms

```
{
    analytics-node: [
        {
            name: "nodec40",
            value: {
                UVEAlarms: {
                    alarms: [
                        {
                            any_of: [
                                {
                                    all_of: [
                                        {
                                            json_operand1_value: ""PROCESS_STATE_STOPPED"",
                                            rule: {
                                                oper: "!=",
                                                operand1: {
                                                    keys: [
                                                        "NodeStatus",
                                                        "process_info",
                                                        "process_state"
                                                    ]
                                                },
                                                operand2: {
                                                    json_value: ""PROCESS_STATE_RUNNING""
                                                }
                                            },
                                            json_vars: {
                                                NodeStatus.process_info.process_name: "contrail-topology"
                                            }
                                        }
                                    ]
                                }
                            ],
                            severity: 3,
                            ack: false,
                            timestamp: 1457395052889182,
                            token: "eyJ0aW1lc3RhbXAiOiAxNDU3Mzk1MDUyODg5MTgyLCAiaHR0cF9wb3J0IjogNTk5NSwgImhvc3RfaXAiOiAiMTAuMjA0LjIxNy4yNCJ9",
                            type: "ProcessStatus"
                        }
                    ]
                }
            }
        }
    ]
}
``` 
- "any_of": có thể thêm biểu thức điều kiện với các alarm rule 
*[ [rule1 AND rule2 AND ... AND ruleN] ... OR [rule11 AND rule22 AND ... AND ruleNN] ]*
- "ack": biểu thị alert có được enable hay không
- "token": được sử dụng bởi client để yêu cầu acknowledgement

### Analytics APIs for Alerts

1. GET http://<analytics-ip>:<rest-api-port>/analytics/uves/control-node/aXXsYY&cfilt=UVEAlarms
Lấy danh sách cảnh báo trên Control Node aXXsYY. Nó có sẵn cho tất cả UVE table-types

2. GET http://<analytics-ip>:<rest-api-port>/analytics/alarms
Lấy danh sách tất cả alarms trong hệ thống  

3. POST http://<analytics-ip>:<rest-api-port>/analytics/alarms/acknowledge
Body: {“table”: <object-type>, “name”: <key>, “type”: <alarm type>, “token”: <token>} 
Cho phép user acknowledge alarm. Có thể acknowledge/un-ack alarm sử dụng API 1,2 với tham số "ackFilt=True" "ackFilt=False"

### Analytics APIs for SSE streams 

4. GET http://<analytics-ip>:<rest-api-port>/analytics/uve-stream?tablefilt=control-node

Cung cấp một SSE-based stream của UVE update cho Control Node alarm. Nó có sẵn cho tất cả UVE table-types. Nếu không truyền "tablefilt", tất cả các UVE sẽ được cung cấp

5. GET http://<analytics-ip>:<rest-api-port>/analytics/alarms-stream?tablefilt=control-node
Tương tự API4 nhưng chỉ cung cấp phần Alert của UVE thay vì cung cấp toàn bộ nội dung của UVE.

### Contrail Alarm Notification

Script *contrail-alarm-notify* ( trong thư mục /usr/bin ) được sử dụng để gửi email cho alarm. Script sẽ nhận cập nhật real-time từ /analytics/alarms-stream API và gửi email tới người nhận. 

```
contrail-alarms-notify --smtp-server smtp.juniper.net --smtp-server-port 25 --sender-email xyz@juniper.net --receiver-email-list someone@juniper.net someone@gmail.com
```

