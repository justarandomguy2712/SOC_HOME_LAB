# Bằng chứng

Chỉ đưa vào repo ảnh và log đã làm sạch: che IP mạng nhà (192.168.x.x), tên máy cá nhân, tên người dùng thật, mật khẩu.

| Thư mục | Nội dung |
|---|---|
| `screenshots/` | Ảnh dashboard, console |
| `sanitized-logs/` | Trích log đã che thông tin |

## Danh sách ảnh nên có

| Phase | Ảnh | Đã có? |
|---|---|---|
| 1 | `ip -br a`, `df -h /` sau mở rộng, ping `10.10.10.1` | Cập nhật khi lưu |
| 2 | 4 dịch vụ `active`, `grep` Vulnerability Detection `no`, Discover alert `sai_user` | |
| 3 | Agent Active, Discover Sysmon ID 11, Discover 4625 | |
| 4 | Console Test A (4 lần, không `100100`), console Test B (`100100`), Discover `rule.id: 100100` | |
