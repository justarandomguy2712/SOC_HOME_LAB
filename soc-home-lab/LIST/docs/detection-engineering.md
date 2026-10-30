# Detection engineering (Phase 4)

## S01: Đăng nhập thất bại liên tiếp

| Mục | Nội dung |
|---|---|
| Mục tiêu | Phát hiện nhiều lần đăng nhập Windows thất bại vào cùng một tài khoản trong thời gian ngắn |
| Giả thuyết điều tra | Có thể đang có brute-force vào một tài khoản cụ thể |
| Điều kiện | Agent `win10-endpoint` Active; kênh Security gửi được 4625; rule `60122` bắn cho sự kiện 4625 |
| Nguồn log | `win10-endpoint` (10.10.10.20); Windows Security log, event 4625 |
| MITRE ATT&CK | T1110 (Brute Force) |

### Hành vi kiểm thử

Trên Windows (cmd), tạo đăng nhập sai bằng tài khoản không tồn tại, cùng một tên cho mọi lần:

```
net use \\127.0.0.1\IPC$ /user:fakeuser SaiMatKhau1
```

Dùng `fakeuser` để tránh khóa tài khoản thật. Mỗi lần trả `System error 1326`.

### Rule

`100100` trong [../../detection-rules/local_rules.xml](../../detection-rules/local_rules.xml): đếm `60122`, `frequency="5"`, `timeframe="60"`, `same_field` theo `win.eventdata.targetUserName`, level 10.

### Truy vấn Discover

```
rule.id: 60122
rule.id: 100100
data.win.system.eventID: 4625
```

### Kết quả

| Phép đo | Giá trị |
|---|---|
| Test A: 4 lần sai | `60122` bắn 4 lần, `100100` không bắn (ảnh console) |
| Test B: sai liên tiếp | `100100` (level 10) xuất hiện trên console sau các dòng `60122` (ảnh console) |
| Bắn ở lần sai thứ mấy | **Chưa ghi nhận.** Cần đếm lại hoặc đếm từ Discover (`rule.id: 60122`) |
| Thời gian từ lần sai cuối đến alert | **Chưa đo** |
| Alert trên dashboard (Discover) | **Chưa xác nhận trong báo cáo này** |
| Pass hoặc Fail | **Chưa kết luận.** Điều kiện Pass: 4 lần không bắn (đạt), bắn ở lần 5 hoặc 6 (chưa biết lần nào) |

### Bằng chứng cần bổ sung

- Ảnh Discover `rule.id: 100100`, hiện `rule.level`, `rule.mitre.id`, `targetUserName`
- Ảnh Discover `rule.id: 60122` với giờ của từng lần sai trong Test B
- Ảnh console Test A và Test B (đã có)

## False positive có thể gặp

Người dùng thật quên mật khẩu và gõ sai nhiều lần liên tiếp. Tác vụ tự động dùng mật khẩu cũ (dịch vụ, ổ mạng ánh xạ).

## Phân tích và khuyến nghị
