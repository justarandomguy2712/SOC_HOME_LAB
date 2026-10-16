# Mạng

## Đã triển khai

| VMnet | Loại | Subnet | DHCP | Dùng cho |
|---|---|---|---|---|
| VMnet10 | Host-only | 10.10.10.0/24 | Tắt | Wazuh (10.10.10.10), Windows (10.10.10.20); host là 10.10.10.1 |
| NAT mặc định của VMware | NAT | 192.168.136.0/24 | Bật | `ens33` của Wazuh và NIC 2 của Windows, chỉ để tải gói |

Các VMnet cũ của lab trước (192.168.30.x và 192.168.136.x) được giữ nguyên, không chỉnh sửa. Dải 10.10.10.0/24 không trùng mạng nhà (192.168.1.x).

## Luồng được phép

| Nguồn | Đích | Cổng | Mục đích | Đã xác minh |
|---|---|---|---|---|
| Windows | Wazuh | 1514/tcp | Gửi sự kiện từ agent | Có (`Test-NetConnection`, agent Active) |
| Windows | Wazuh | 1515/tcp | Đăng ký agent | Có (`Test-NetConnection`) |
| Host | Wazuh | 443/tcp | Dashboard | Có |
| Host | Wazuh | 22/tcp | SSH quản trị | Có |
| Wazuh nội bộ | | 9200, 55000 | Indexer, API | Không mở ra ngoài (chưa kiểm tra bằng quét cổng) |

## Kế hoạch Phase 5 (chưa triển khai)

| VMnet | Subnet | Vùng |
|---|---|---|
| VMnet11 | 10.10.20.0/24 | Enterprise LAN (Windows) |
| VMnet12 | 10.10.30.0/24 | Attack (Kali) |
| pfSense | 10.10.10.1/10.10.20.1/10.10.30.1 (dự kiến) | Định tuyến và firewall giữa các vùng |

Lưu ý: VM ở các VMnet khác nhau không tự liên lạc. Mọi đường đi phải qua pfSense và có rule cho phép.
