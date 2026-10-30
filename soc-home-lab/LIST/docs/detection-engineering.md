# Detection engineering (Phase 4)

## S01: Đăng nhập thất bại liên tiếp

| Mục | Nội dung |
|---|---|
| Mục tiêu | Phát hiện nhiều lần đăng nhập Windows thất bại vào cùng một tài khoản trong thời gian ngắn |
| Kịch bản | Brute-force vào một tài khoản cụ thể |
| Điều kiện | Agent `win10-endpoint` Active; kênh Security gửi được 4625; rule `60122` bắn cho sự kiện 4625 |
| Nguồn log | `win10-endpoint` (10.10.10.20); Windows Security log, event 4625 |
| MITRE ATT&CK | T1110 (Brute Force) |

### Hành vi kiểm thử

1.1. Khảo sát và đánh giá các Rule mặc định đối với hành vi đăng nhập thất bại


Đầu tiên, tiến hành truy xuất các cảnh báo được ghi nhận trong file ```alerts.json``` nhằm xác định Rule ID, mức độ cảnh báo (Level) và mô tả tương ứng với sự kiện Event ID 4625. Sau đó, thực hiện tìm kiếm trong thư mục Ruleset mặc định của Wazuh để xác định file cấu hình và nội dung Rule liên quan.

Quá trình này giúp làm rõ cơ chế phát hiện đăng nhập thất bại hiện có của Wazuh, đồng thời cung cấp cơ sở để phân tích, điều chỉnh và xây dựng các Custom Rule nhằm nâng cao khả năng giám sát và phát hiện các hành vi đăng nhập bất thường trong hệ thống.

*Ta dùng hai câu lệnh như sau:*

**Câu lệnh thứ nhất**:


```sudo grep '"eventID":"4625"' /var/ossec/logs/alerts/alerts.json | tail -3 | grep -o '"rule":{"level":[0-9]*,"description":"[^"]*","id":"[0-9]*"'```




 **Mục đích: Truy xuất 3 cảnh báo gần nhất liên quan đến Event ID 4625 trong Wazuh, sau đó trích xuất thông tin Rule ID, Level và Description tương ứng.**




| Command | Meaning |
|---|---|
| ```sudo grep '"eventID":"4625"'``` | Thực thi với quyền root để lọc ra EventID 4625. |
| ```/var/ossec/logs/alerts/alerts.json``` | File JSON lưu trữ các cảnh báo do Wazuh tạo ra. |
| ```tail -3``` | Lấy 3 dòng cuối cùng trong kết quả lọc. |
| ``` grep -o``` | 	Chỉ hiển thị phần nội dung khớp với biểu thức tìm kiếm. |
| ``` rule":{"level":[0-9]*,"description":"[^"]*","id":"[0-9]*``` | Lấy thông tin Rule bao gồm Level, Description và ID.|


**Câu lệnh thứ hai**:

```sudo grep -rn -i "logon failure" /var/ossec/ruleset/rules/ | head -5```

**Mục đích: Lấy 5 kết quả đầu tiên liên quan đến cụm từ "Logon Failure" trong toàn bộ thư mục Ruleset mặc định của Wazuh, nhằm xác định file cấu hình và vị trí khai báo các Rule phục vụ phát hiện hành vi đăng nhập thất bại.**



| Command | Meaning |
|---|---|
| ```-r``` | Tìm kiếm đệ quy trong toàn bộ thư mục và thư mục con.. |
| ```-i``` | Không phân biệt chữ hoa và chữ thường. |
| ``` logon failure``` | 	Chuỗi từ khóa cần tìm kiếm. |
| ```  head -5 ``` | Chỉ hiển thị 5 kết quả đầu tiên.|
| ```  -n ``` | Hiển thị số dòng.|

Và **Output** của 2 câu lệnh này cho ta kết quả như sau

```
/var/ossec/ruleset/rules/0580-win-security_rules.xml:66:    <description>Windows Logon Failure</description>
/var/ossec/ruleset/rules/0580-win-security_rules.xml:270:    <description>Logon Failure - Unknown user or bad password</description>
/var/ossec/ruleset/rules/0580-win-security_rules.xml:281:    <description>Logon Failure - Account logon time restriction violation</description>
/var/ossec/ruleset/rules/0580-win-security_rules.xml:292:    <description>Logon Failure - Account currently disabled</description>
/var/ossec/ruleset/rules/0580-win-security_rules.xml:303:    <description>Logon Failure - Specified account expired</description>
```

|Dòng |	Description | Ý nghĩa |
|---|---|---|
|66|Windows Logon Failure	|Mô tả chung cho hành vi đăng nhập Windows thất bại.|
|270|	Unknown user or bad password|	Tài khoản không tồn tại hoặc mật khẩu không chính xác.|
|281|	Account logon time restriction violation|	Vi phạm giới hạn thời gian đăng nhập của tài khoản.|
|292|	Account currently disabled|	Tài khoản hiện đang bị vô hiệu hóa|
|303|	Specified account expired	|Tài khoản đã hết hạn sử dụng.|

Thông qua 2 câu lệnh này ta đã xác định được file chứa ruleset mặc định (0580-win-security_rules.xml), từ đó có thêm dữ liệu để viết custom rule cho chính xác hơn

+ Sau khi xác định được file chứa Ruleset mặc định, bước tiếp theo là phân tích các điều kiện và trường dữ liệu mà Rule sử dụng để kích hoạt cảnh báo.




*Ta dùng 2 câu lệnh như sau:*



**Câu lệnh thứ nhất**:

```sudo sed -n '58,75p;262,278p' /var/ossec/ruleset/rules/0580-win-security_rules.xml``` 

**Mục đích: Trích xuất nội dung tại hai khoảng dòng cụ thể trong file Ruleset để khảo sát cấu trúc các Rule mặc định.**


| Command | Meaning |
|---|---|
| ```sed``` | Tìm kiếm đệ quy trong toàn bộ thư mục và thư mục con. |
| ```-n``` | Không tự động in toàn bộ nội dung file. |
| ```58,75p``` | 	In các dòng từ 58 đến 75. |
| ``` 262,278p``` | 	In các dòng từ 262 đến 278. |
| ```  0580-win-security_rules.xml ``` | File chứa các Rule mặc định liên quan đến Windows Security.|





**Câu lệnh thứ hai**: 

```sudo grep -n "frequency\|if_matched_sid" /var/ossec/ruleset/rules/0580-win-security_rules.xml | head```

**Mục đích: Tìm kiếm các thành phần frequency và if_matched_sid trong file Ruleset nhằm kiểm tra sự tồn tại của các cơ chế phát hiện dựa trên events.**


| Command | Meaning |
|---|---|
| ```grep``` | Tìm kiếm chuỗi ký tự trong file. |
| ```-n``` | Hiển thị số dòng trong file. |
| ```frequency``` | 	Tần suất cần đạt ngưỡng đã cấu hình để kích hoạt các event alert. |
| ``` \|``` | 	Toán tử OR trong biểu thức tìm kiếm của grep. |
| ```  if_matched_sid ``` |Tham chiếu đến SID của một Rule đã được kích hoạt trước đó.|
| ```  head ``` |Hiển thị 10 dòng đầu tiên (Mặc định)|











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
