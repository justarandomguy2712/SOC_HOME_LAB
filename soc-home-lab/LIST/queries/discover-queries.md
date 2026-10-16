# Truy vấn Discover (index pattern `wazuh-alerts-*`)

| Mục đích | Truy vấn | Đã xác minh trên lab |
|---|---|---|
| Đăng nhập thất bại | `data.win.system.eventID: 4625` | Có |
| Rule S01 | `rule.id: 100100` | Bắn trên console; Discover chưa xác nhận trong tài liệu |
| Rule đăng nhập thất bại mặc định | `rule.id: 60122` | Có (console) |
| Process creation (Sysmon ID 1) | `data.win.system.eventID: 1` | Có |
| Tìm `whoami` | `data.win.system.eventID: 1 AND data.win.eventdata.image: *whoami*` | Chưa chạy lại sau khi viết |
| Tệp tạo mới (Sysmon ID 11) | `data.win.system.eventID: 11` | Có |
| Ẩn nhiễu Edge | `NOT data.win.eventdata.image: *msedge*` | Chưa kiểm tra |

Cột nên thêm: `agent.name`, `rule.id`, `rule.level`, `rule.description`, `data.win.eventdata.targetUserName`, `data.win.eventdata.image`, `data.win.eventdata.commandLine`.
Các trường `commandLine`, `parentImage` chưa được xác nhận trong ảnh chụp dashboard của lab.
