# Detection engineering (Phase 4)

## Cấu trúc một rule Wazuh (đọc từ file rule mặc định `0580-win-security_rules.xml`)

| Thành phần | Vai trò | Ví dụ |
|---|---|---|
| `id` | Mã rule. 100000 trở lên dành cho rule người dùng | `100100` |
| `level` | Mức nghiêm trọng 0-15 | `5` (đăng nhập sai), `10` (rule đếm) |
| `if_sid` | Rule cha | `60105` là cha của `60122` |
| `field` | Khớp trường đã được decoder tách | `win.system.eventID` bằng `4625` |
| `if_matched_sid` / `if_matched_group` | Rule đếm: đếm số lần rule hoặc nhóm khác đã bắn | `60204` đếm nhóm `authentication_failed` |
| `frequency`, `timeframe` | Số sự kiện cần có trong số giây | `5` trong `60` |
| `same_field` | Mỗi giá trị khác nhau của trường có bộ đếm riêng | `win.eventdata.targetUserName` |
| `description` | Chuỗi hiện ở `rule.description` | |
| `group` | Nhãn nhóm | `authentication_failed` |
| `mitre` | Gắn kỹ thuật ATT&CK | `T1110` |

## Chuỗi rule cho sự kiện 4625

| Cấp | Rule | Điều kiện |
|---|---|---|
| 1 | `60104` | Windows audit failure |
| 2 | `60105` | Cấp 1 và `eventID` 4625 (hoặc mã cũ 529-539) |
| 3 | `60122` | Cấp 2 và `eventID` 4625 hoặc 529: "Unknown user or bad password", level 5 |
| 4 | `100100` (tùy chỉnh) | Đếm `60122`, 5 lần trong 60 giây, cùng tài khoản, level 10, T1110 |

Rule mặc định `60204` đếm nhóm `authentication_failed` theo `win.eventdata.ipAddress` với `frequency="$MS_FREQ"`, `timeframe="240"`. Giá trị `MS_FREQ` quan sát được trong `0602-win-wfirewall_rules.xml` là 8; chưa xác minh đó có phải giá trị hiệu lực cho `60204` (lệnh `grep` đã bị cắt bằng `head`).

## Khác biệt giữa `60204` và S01

| | `60204` | `100100` (S01) |
|---|---|---|
| Đếm theo | Nguồn (`ipAddress`) | Tài khoản bị thử (`targetUserName`) |
| Cửa sổ | 240 giây | 60 giây |
| Ngưỡng | `$MS_FREQ` | 5 |

Hạn chế của S01: đếm theo tài khoản nên bỏ sót password spraying (mỗi tài khoản vài lần). Chưa biết `100100` có gộp sự kiện của nhiều agent vào một bộ đếm không; lab hiện chỉ có một agent.

## Nhiễu quan sát được (sổ false positive)

| Nhiễu | Nguồn | Quyết định |
|---|---|---|
| Sysmon ID 11 từ `msedge.exe` (tệp trong `AppData\Local\Temp`) | Trình duyệt hoạt động bình thường | Chưa lọc; đánh giá khi viết rule theo file |
| `__PSScriptPolicyTest_*.ps1` trong `AppData\Local\Temp` | PowerShell kiểm tra execution policy khi khởi động | Chưa lọc; cần loại khi viết rule S03 |

## Quy trình thử rule

| # | Việc | Lệnh |
|---|---|---|
| 1 | Sao lưu | `sudo cp /var/ossec/etc/rules/local_rules.xml ~/local_rules.xml.bak` |
| 2 | Sửa | `sudo nano /var/ossec/etc/rules/local_rules.xml` |
| 3 | Kiểm tra cú pháp | `sudo /var/ossec/bin/wazuh-analysisd -t` |
| 4 | Áp dụng | `sudo systemctl restart wazuh-manager` |
| 5 | Theo dõi | `sudo tail -f /var/ossec/logs/alerts/alerts.log \| grep --line-buffered -E "Rule: (60122\|100100)"` |
| 6 | Kích hoạt hành vi trong lab | Xem file kịch bản |
| 7 | Ghi kết quả (Pass hoặc Fail) cùng bằng chứng | |
