# Báo cáo SOC Home Lab: Phase 1 đến Phase 4 (đang dở)

Ngày lập: 16/10/2026. Phạm vi: Phase 1-3 hoàn thành, Phase 4 mới xong kịch bản S01 ở mức rule đã bắn.
Nguyên tắc: mọi số liệu ghi "đã đo" có nguồn là kết quả lệnh hoặc ảnh chụp trong quá trình làm lab. Mục nào chưa đo ghi rõ là chưa đo.

## 1. Tóm tắt

| Hạng mục | Kết quả |
|---|---|
| Mục tiêu | Dựng SOC Home Lab chạy trên máy 12 GB RAM để thu log, viết rule, điều tra |
| Đã hoàn thành | Wazuh all-in-one, Windows endpoint với Wazuh Agent và Sysmon, pipeline log tới dashboard, rule tùy chỉnh S01 bắn được |
| Sự cố đáng chú ý | Cài Wazuh lần đầu làm đầy đĩa 28 GB do module Vulnerability Detection; đã khắc phục |
| Chưa làm | Kịch bản S02-S06, pfSense, Suricata, Kali, hunting, incident report |
| Chưa kết luận | S01 Pass hay Fail: thiếu số lần sai tại thời điểm bắn và ảnh dashboard |

## 2. Môi trường

| Mục | Giá trị |
|---|---|
| CPU | Intel Core i5-12450HX, 8 nhân, 12 luồng |
| RAM | 11,7 GB |
| Ổ đĩa | SSD Micron 512 GB; ổ C trống 70 GB, ổ D trống 52,2 GB lúc bắt đầu; VM đặt ở D |
| Ảo hóa | VMware Workstation; VBS (Virtualization-based security) báo `Running` trên host. Chưa tắt Memory integrity, chưa thấy VM chậm đáng kể |
| VM Wazuh | Ubuntu Server 22.04.5, 2 vCPU, 4 GB RAM, đĩa 30 GB (LV `/` mở rộng lên 28 GB) |
| VM Windows | Windows 10 Pro (ISO bản đa phiên bản, không phải Enterprise Evaluation), 2 vCPU, 3 GB RAM, 40 GB, chưa kích hoạt |
| SIEM | Wazuh 4.14 (cài bằng `wazuh-install.sh -a` nhánh 4.14); số phiên bản chi tiết chưa ghi lại |

## 3. Kiến trúc và mạng

| VMnet | Subnet | Thành phần |
|---|---|---|
| VMnet10 (host-only, tắt DHCP) | 10.10.10.0/24 | Wazuh 10.10.10.10, Windows 10.10.10.20, host 10.10.10.1 |
| NAT | 192.168.136.0/24 | NIC phụ của hai VM, chỉ để tải gói |

Luồng chính: Wazuh Agent (Windows) gửi sự kiện tới 10.10.10.10:1514 (đăng ký qua 1515); quản trị qua `https://10.10.10.10`. Sơ đồ: `diagrams/architecture.mmd`.

Giả định và thay đổi so với thiết kế ban đầu:

| Thiết kế ban đầu | Thực tế | Lý do |
|---|---|---|
| Ba VMnet (10, 11, 12) từ đầu | Chỉ VMnet10 | Giảm điểm lỗi; VMnet11/12 chuyển sang Phase 5 |
| Windows 10 Enterprise Evaluation | Windows 10 Pro | ISO sẵn có là bản đa phiên bản |
| Đĩa Wazuh 30 GB, `/` ~28 GB | Đúng như vậy sau khi mở rộng LV | |

## 4. Phase 1: Preparation

| Việc | Kết quả (đã đo) |
|---|---|
| Kiểm tra phần cứng | Như bảng mục 2 |
| Dải IP | Host có 192.168.1.x, 192.168.30.x, 192.168.136.x; không trùng 10.10.10.0/24 |
| VMnet10 | `ping 10.10.10.1` từ VM Wazuh: 3/3, 0% mất gói |
| Ubuntu | `ens34` = `10.10.10.10/24`, `ens33` = DHCP từ NAT |
| Mở rộng `/` | `lvextend` đổi LV từ 14 GiB lên 28 GiB; `df` vẫn báo 14G cho đến khi chạy `resize2fs` |
| Snapshot | `clean-base-p1` (người dùng xác nhận đã tạo) |
| Thử revert snapshot | Thực hiện thật ở Phase 2 khi cài Wazuh lỗi; khôi phục thành công |

Tiêu chí: VM khởi động ổn định, mạng đúng thiết kế, khôi phục được: **Đạt**.

## 5. Phase 2: Wazuh Deployment

### 5.1. Sự cố: đĩa đầy khi cài lần đầu

| Mục | Chi tiết |
|---|---|
| Hiện tượng | Installer treo ở bước `Updating the internal users`; console báo `systemd-journald: Failed to create new user journal: No space left on device`; `df -h /` hiện 28G dùng 28G, 100% |
| Chẩn đoán | `du` cho thấy `/var/ossec` khoảng 18 GB: `tmp` 9,1 GB, `queue/vd/feed` 7,8 GB. Sau khi dừng dịch vụ, `/` tụt về 8,1 GB |
| Nguyên nhân | Module Vulnerability Detection của manager tải và giải nén kho CVE khi khởi động. Kết luận này dựa trên vị trí thư mục (`queue/vd`) và thời điểm; chưa đối chiếu bằng tài liệu chính thức |
| Lỗi đi kèm | `securityadmin` báo `SocketTimeoutException`: hệ quả của đĩa đầy chứ không phải lỗi cấu hình chứng chỉ |
| Giả thuyết đã loại | File bị xóa nhưng còn bị giữ mở (`lsof +L1`): file lớn nhất chỉ khoảng 63 MB |
| Khắc phục | Revert `clean-base-p1`. Cài lại, đồng thời chạy một lệnh chờ `ossec.conf` xuất hiện rồi đổi `<enabled>yes</enabled>` thành `no` trong khối `<vulnerability-detection>` |
| Xác minh | `grep -A3 "<vulnerability-detection>"` hiện `<enabled>no</enabled>` |
| Đánh đổi | Lab không có cảnh báo lỗ hổng CVE |

### 5.2. Kết quả sau cài lại (đã đo)

| Chỉ số | Giá trị |
|---|---|
| 4 dịch vụ (`wazuh-manager`, `wazuh-indexer`, `wazuh-dashboard`, `filebeat`) | `active` |
| `/` | 12 GB dùng, 15 GB trống, 44% |
| `/var/ossec` | 1,5 GB |
| RAM (`free -h`) | 3,8 GiB tổng; `available` 1,3 GiB; swap dùng 6 MB |
| Dashboard | Đăng nhập được tại `https://10.10.10.10` |
| Tìm kiếm | Sau 3 lần SSH sai mật khẩu (`sai_user`), alert xuất hiện trong Discover |

Chưa đo: RAM khi có agent gửi log liên tục trong thời gian dài; tăng trưởng đĩa theo ngày.

Tiêu chí: dashboard truy cập được, dịch vụ active, tìm kiếm được, không thiếu RAM kéo dài: **Đạt** (theo số đo trên, chưa theo dõi dài hạn).

Ghi chú: hostname VM vẫn hiển thị `ubuntu` (trường `manager.name`); việc đổi sang `wazuh` chưa được xác nhận.

## 6. Phase 3: Windows Endpoint

| Việc | Kết quả |
|---|---|
| IP | `10.10.10.20/24` trên NIC VMnet10 |
| Kết nối | `Test-NetConnection` tới 10.10.10.10 cổng 1514 và 1515: thành công |
| Agent | `win10-endpoint`, agent id 001, IP 10.10.10.20; dữ liệu hiện trong Discover |
| Sysmon | Cài với config SwiftOnSecurity; `Sysmon64` chạy (ảnh console cho thấy `SysmonDrv` và `Sysmon64` started) |
| Cấu hình Wazuh | Thêm `<localfile>` kênh `Microsoft-Windows-Sysmon/Operational` kiểu `eventchannel` |
| Sysmon ID 11 | Thấy trên dashboard: `msedge.exe` ghi file tạm; `powershell.exe` ghi `__PSScriptPolicyTest_*.ps1` |
| Sysmon ID 1 | Kiểm thử `whoami`, `hostname` (người dùng xác nhận kết quả đạt; ảnh dashboard ID 1 chưa được đính kèm) |
| Security 4625 | Thấy trên dashboard: `targetUserName: labuser`, `subStatus 0xc000006a`, `logonType 2`, `ipAddress 127.0.0.1` |

Sự cố nhỏ:

| Sự cố | Nguyên nhân | Khắc phục |
|---|---|---|
| `Invoke-WebRequest` báo `BlobNotFound` khi tải Sysmon | Lệnh dài bị tách dòng khi dán; PowerShell hỏi `Uri:` và nhận cả đoạn lệnh làm địa chỉ | Viết lệnh đủ tham số `-Uri` và `-OutFile` trên một dòng |
| Không sửa được `ossec.conf` bằng chuột phải | File `.conf` không có mục chạy bằng quyền Admin | Mở Notepad bằng `Start-Process notepad.exe -Verb RunAs` |
| SSH báo `REMOTE HOST IDENTIFICATION HAS CHANGED` | VM Wazuh được cài lại, khóa host mới | `ssh-keygen -R 10.10.10.10` |

Quan sát: giờ trong VM Windows (khoảng 8:5x AM) lệch múi giờ so với dashboard (22:5x); đã khuyến nghị `Set-TimeZone`. Việc đã thực hiện hay chưa chưa được xác nhận.

Tiêu chí: agent Active, log về SIEM, tìm theo trường, Sysmon, Security Events, sự kiện thử vô hại: **Đạt**.

## 7. Phase 4: Detection Engineering (đang làm)

### 7.1. Đọc rule mặc định

Rule `60122` ("Logon Failure - Unknown user or bad password", level 5) là rule con của `60105`, cha của nó là `60104`. Chi tiết: `docs/detection-engineering.md`.

### 7.2. Rule tùy chỉnh S01

`detection-rules/local_rules.xml`: rule `100100`, level 10, đếm `60122` 5 lần trong 60 giây, `same_field` theo `win.eventdata.targetUserName`, gắn T1110.

### 7.3. Kiểm thử S01

| Test | Hành vi | Kết quả (từ ảnh console) |
|---|---|---|
| A | 4 lần `net use \\127.0.0.1\IPC$ /user:fakeuser ...` | 4 dòng `Rule: 60122 (level 5)`, không có `100100` |
| B | Tiếp tục sai liên tiếp | Xuất hiện `Rule: 100100 (level 10)` |

| Chưa có | Tình trạng |
|---|---|
| Số lần sai khi `100100` bắn | Chưa ghi nhận |
| Thời gian từ lần sai cuối đến alert | Chưa đo |
| Ảnh dashboard `rule.id: 100100` | Chưa được xác nhận |

Kết luận tạm thời: rule **đã nạp và có bắn**; chưa đủ dữ liệu để ghi Pass cho S01 theo tiêu chí đã đặt.

Ghi chú: người dùng đã sửa `description` của rule trên VM (console hiện chuỗi khác với file trong repo). File `local_rules.xml` trong repo là bản gốc, cần đồng bộ lại với bản trên VM khi bổ sung bằng chứng.

## 8. Chỉ số đo hiệu quả

| Chỉ số | Giá trị hiện tại |
|---|---|
| Nguồn log tích hợp thành công | 3 (Sysmon, Windows Security, log hệ thống Wazuh server) |
| Kịch bản đã chạy | 1 trong 6 (S01), chưa kết luận Pass |
| Tỷ lệ phát hiện, false positive rate | Chưa đo (cần ≥ 2 kịch bản và tập dữ liệu nền) |
| Thời gian từ sự kiện đến alert | Chưa đo |

## 9. Giới hạn và rủi ro

| Mục | Ghi chú |
|---|---|
| RAM tổng ~11,7 GB | Wazuh 4 GB + Windows 3 GB đã chạy cùng host; chưa thử thêm VM thứ ba |
| Đĩa D | Còn ít sau khi tạo VM; cần kiểm tra trước khi thêm pfSense và Kali |
| Windows chưa kích hoạt | Không ảnh hưởng lab, chỉ ảnh hưởng giao diện |
| Windows host bật VBS | Có thể làm VMware chậm hơn; chưa đo |
| Vulnerability Detection tắt | Không phát hiện lỗ hổng CVE |
| Rule S01 đếm theo tài khoản | Không bắt được password spraying |
| Giá trị `MS_FREQ` | Thấy giá trị 8 trong `0602-win-wfirewall_rules.xml`; chưa xác minh là giá trị hiệu lực cho rule `60204` |

## 10. Việc tiếp theo

| # | Việc |
|---|---|
| 1 | Hoàn tất S01: đếm lại số lần sai khi bắn, đo thời gian, chụp ảnh Discover, kết luận Pass hoặc Fail |
| 2 | S02 (process creation, parent-child), S03 (PowerShell vô hại), S04 (FIM) |
| 3 | Ghi sổ false positive; tuning |
| 4 | Kiểm tra ổ D, quyết định chỗ đặt pfSense và Kali trước Phase 5 |
| 5 | Thay `<TEN-CUA-BAN>` trong `LICENSE`; che thông tin cá nhân trong ảnh |
