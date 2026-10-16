# Hướng dẫn triển khai (Phase 1-3)

Ghi chú: các lệnh dưới đây là các lệnh đã chạy trong quá trình xây lab. Phiên bản phần mềm thay đổi theo thời gian; đối chiếu với tài liệu chính thức trước khi dùng lại.

## Phase 1: Preparation

| # | Việc | Lệnh hoặc thao tác | Xác minh |
|---|---|---|---|
| 1 | Kiểm tra máy | `Get-CimInstance Win32_Processor`, `Get-PSDrive` | CPU, RAM, dung lượng trống |
| 2 | Thư mục lab | `D:\SOC-Lab\VMs`, `ISO`, `repo` | Thư mục tồn tại |
| 3 | VMnet10 | Virtual Network Editor: Add Network, Host-only, `10.10.10.0/255.255.255.0`, tắt DHCP | `ipconfig` có `10.10.10.1`; ping từ VM thành công |
| 4 | VM Wazuh | 2 vCPU, 4096 MB RAM, đĩa 30 GB (không allocate trước), NIC1 VMnet10, NIC2 NAT | Settings hiện 2 NIC |
| 5 | Ubuntu Server 22.04.5 | Bản Ubuntu Server (không minimized). `ens34` Manual `10.10.10.10/24`, không gateway. `ens33` DHCP. Cài OpenSSH | `ip -br a` |
| 6 | Mở rộng `/` | `sudo lvextend -r -l +100%FREE /dev/ubuntu-vg/ubuntu-lv`; nếu `df -h /` chưa tăng: `sudo resize2fs /dev/ubuntu-vg/ubuntu-lv` | `df -h /` hiện khoảng 28 GB |
| 7 | Cập nhật và snapshot | `sudo apt update && sudo apt -y upgrade`, shutdown, snapshot `clean-base-p1` | Snapshot có trong Snapshot Manager |

## Phase 2: Wazuh all-in-one

| # | Việc | Lệnh | Xác minh |
|---|---|---|---|
| 1 | Tải script (nhánh 4.14) | `curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh` | File tồn tại |
| 2 | Tắt Vulnerability Detection ngay khi `ossec.conf` xuất hiện (chạy ở phiên SSH khác, trước khi cài) | `until sudo test -f /var/ossec/etc/ossec.conf; do sleep 1; done; sudo sed -i '/<vulnerability-detection>/,/<\/vulnerability-detection>/ s#<enabled>yes</enabled>#<enabled>no</enabled>#' /var/ossec/etc/ossec.conf && echo DONE` | `grep -A3 "<vulnerability-detection>" /var/ossec/etc/ossec.conf` hiện `<enabled>no</enabled>` |
| 3 | Cài | `sudo bash ./wazuh-install.sh -a` (chạy trong `tmux`) | Cuối log in thông tin truy cập |
| 4 | Kiểm tra dịch vụ | `sudo systemctl is-active wazuh-manager wazuh-indexer wazuh-dashboard filebeat` | 4 dòng `active` |
| 5 | Theo dõi đĩa khi cài | `watch -n 10 "df -h / \| tail -1; du -sh /var/ossec"` | `/var/ossec` không vượt vài GB |
| 6 | Dashboard | `https://10.10.10.10` (chứng chỉ tự ký) | Đăng nhập được |

Mật khẩu `admin` do installer sinh ra, lưu trong trình quản lý mật khẩu, không đưa vào repo.

## Phase 3: Windows endpoint

| # | Việc | Thao tác | Xác minh |
|---|---|---|---|
| 1 | VM Windows 10 Pro | 2 vCPU, 3 GB, 40 GB, NIC1 VMnet10, NIC2 NAT; tài khoản local | Vào được desktop |
| 2 | Đặt IP | `New-NetIPAddress -InterfaceAlias "<NIC VMnet10>" -IPAddress 10.10.10.20 -PrefixLength 24` | `ipconfig` |
| 3 | Kiểm tra mạng | `Test-NetConnection 10.10.10.10 -Port 1514` và `-Port 1515` | `TcpTestSucceeded : True` |
| 4 | Wazuh Agent | Dashboard > Agents > Deploy new agent (Windows, MSI, server `10.10.10.10`, name `win10-endpoint`); dán khối lệnh vào PowerShell Admin; `NET START WazuhSvc` | Agent trạng thái Active |
| 5 | Sysmon | `Invoke-WebRequest -Uri "https://download.sysinternals.com/files/Sysmon.zip" -OutFile "C:\Lab\Sysmon.zip"`; giải nén; tải `sysmonconfig-export.xml` của SwiftOnSecurity; `Sysmon64.exe -accepteula -i sysmonconfig.xml` | `Get-Service Sysmon64` Running |
| 6 | Cho Wazuh đọc Sysmon | Thêm khối `<localfile>` (xem `configs/wazuh/`), `Restart-Service WazuhSvc` | Sự kiện Sysmon xuất hiện trong Discover |
| 7 | Sự kiện thử | `whoami`, `hostname`, đăng nhập sai | Có ID 1, 11 và 4625 trong Discover |

Lỗi đã gặp khi dán lệnh nhiều dòng: xem mục sự cố trong báo cáo.
