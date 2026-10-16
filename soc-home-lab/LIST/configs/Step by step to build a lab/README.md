# Sysmon

Config dùng: `sysmonconfig-export.xml` của SwiftOnSecurity (https://github.com/SwiftOnSecurity/sysmon-config).
Cài: `Sysmon64.exe -accepteula -i sysmonconfig.xml`.

Không sao chép file config vào repo nếu chưa kiểm tra giấy phép; ghi lại commit hoặc ngày tải xuống ở đây khi dùng.
Nhiễu đã quan sát: Sysmon ID 11 từ `msedge.exe` và file `__PSScriptPolicyTest_*.ps1` do PowerShell tạo. Chưa lọc, xem `docs/detection-engineering.md`.
