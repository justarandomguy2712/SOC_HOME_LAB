# Kiến trúc

## Trạng thái hiện tại (đến hết Phase 3)

| Thành phần | Chi tiết |
|---|---|
| Máy chủ SIEM | VM `Wazuh-SIEM`, Ubuntu Server 22.04.5, Wazuh 4.14 all-in-one (manager, indexer, dashboard, filebeat) |
| Endpoint | VM `Win10-Endpoint`, Windows 10 Pro, Wazuh Agent (agent id 001, tên `win10-endpoint`), Sysmon |
| Luồng log | Windows Event Log (Security) và Sysmon/Operational, qua Wazuh Agent, tới manager (1514/tcp), vào indexer, hiện trên dashboard |
| Truy cập quản trị | Trình duyệt trên host tới `https://10.10.10.10` (443/tcp). Không công khai ra Internet |

## Quyết định thiết kế

| Quyết định | Lý do |
|---|---|
| Một SIEM duy nhất (Wazuh) | RAM 12 GB không đủ cho nhiều nền tảng trùng chức năng |
| Phase 1-4 dùng một mạng (VMnet10) | Giảm điểm lỗi khi pipeline SIEM chưa ổn định. VMnet11/12 và pfSense để Phase 5 |
| Tắt Vulnerability Detection | Module tải kho CVE làm đầy đĩa 28 GB (xem báo cáo, mục sự cố). Không cần cho các kịch bản đã chọn |
| NIC NAT tạm thời | Chỉ để tải gói và agent. Sẽ gỡ khỏi VM Windows ở Phase 5 |
| GNS3 không nằm trong đường chính | VMnet đủ để phân vùng mạng; GNS3 tốn thêm RAM |

## Bản đồ nguồn log

| Nguồn | Kênh | Sự kiện đã thấy trên dashboard |
|---|---|---|
| Sysmon | `Microsoft-Windows-Sysmon/Operational` | ID 1 (process), ID 11 (file create) |
| Windows Security | `Security` | 4625 (đăng nhập thất bại), 4624 (thành công), 4672 |
| Log hệ thống Wazuh server | `auth.log` | Alert SSH đăng nhập sai (kiểm thử Phase 2) |

Sơ đồ: [../diagrams/architecture.mmd](../diagrams/architecture.mmd)
