
<div align="center">

<table>
  <tr>
    <td valign="bottom"><img width="729" height="381" alt="09301" src="https://github.com/user-attachments/assets/e245b6dd-8523-4e7c-851a-db324dd50fbb" /></td>
    <td><h1>SOC_HOME_LAB</h1></td>
  </tr>
</table>

Vietnamese Version: Đây là dự án cá nhân xây dựng SOC Home Lab sử dụng Wazuh SIEM để mô phỏng hoạt động giám sát an ninh mạng, thu thập và phân tích log, phát hiện các sự kiện bảo mật trong môi trường ảo hóa VMware.

English Version: This is a personal SOC Home Lab project using Wazuh SIEM to simulate security monitoring, log collection and analysis, and security event detection in a virtualized VMware environment.

<p align="center">

![Status](https://img.shields.io/badge/status-active-brightgreen?style=flat)
![Wazuh](https://img.shields.io/badge/Wazuh-SIEM-005571?style=flat&logo=wazuh&logoColor=white)
![Ubuntu Server](https://img.shields.io/badge/Ubuntu_Server-Linux-E95420?style=flat&logo=ubuntu&logoColor=white)
![VMware](https://img.shields.io/badge/VMware-Workstation-607078?style=flat&logo=vmware&logoColor=white)
![Bash](https://img.shields.io/badge/Bash-Scripting-4EAA25?style=flat&logo=gnubash&logoColor=white)

</p>
</div>



## 📁 Cấu trúc repo | Repository Structure

```
soc-home-lab/
├── configs/            # Wazuh, agent and endpoint configuration files
├── detection-rules/    # Custom Wazuh decoders and detection rules
├── diagrams/           # Network and architecture diagrams
├── docs/               # Setup guides and lab documentation
├── evidence/           # Screenshots and logs proving each test result
├── queries/            # Search and hunting queries
├── reports/            # Incident and lab reports
├── scenarios/          # Attack simulation scenarios
├── LICENSE
└── README.md

```

## Cách dùng các thư mục template | How to Use Template Directories

Nguyên tắc: **Cài đặt và cấu hình** nằm ở `docs/deployment-guide.md`; **phát hiện** (Rule, kịch bản, kết quả) nằm ở `detection-rules/`, `scenarios/` và `docs/detection-engineering.md`; **Tổng kết** nằm ở `reports/`.

| Nội dung cần viết | Đặt ở đâu | Ghi chú |
|---|---|---|
| Cài đặt, cấu hình VM, Wazuh, agent, Sysmon (Phase 1, 2, 3, 5) | `docs/deployment-guide.md` | Mỗi phase một mục, theo `docs/phase-template.md` |
| Sơ đồ kiến trúc | `diagrams/*.mmd` và `docs/architecture.md` | Sửa file `.mmd` rồi cập nhật README nếu kiến trúc đổi |
| Địa chỉ IP, VMnet, luồng mạng được phép | `docs/network-topology.md` | Thêm vùng mạng mới khi sang Phase 5 |
| File cấu hình mẫu (đã loại bí mật) | `configs/sysmon/`, `configs/wazuh/` | Mỗi cấu hình ghi lý do và cách xác minh |
| Rule tùy chỉnh (Phase 4) | `detection-rules/local_rules.xml` | Giữ đúng bản đang chạy trên VM; ID từ `100100` trở lên |
| Giải thích rule, so sánh với rule mặc định, sổ false positive | `docs/detection-engineering.md` | Mỗi rule mới thêm một mục |
| Một kịch bản kiểm thử (hành vi test, kết quả, Pass hoặc Fail) | `scenarios/<tên-kịch-bản>/<ID>-<tên>.md` | Sao chép từ `scenarios/_TEMPLATE.md` |
| Truy vấn Discover | `queries/discover-queries.md` | Ghi rõ truy vấn nào đã xác minh trên lab |
| Ảnh của một phase | `docs/images/phase-N/` | Nhúng bằng đường dẫn tương đối |
| Ảnh của một kịch bản | `scenarios/<tên-kịch-bản>/images/` | Nhúng bằng `![mô tả](images/ten-anh.png)` |
| Điều tra một sự cố | `reports/incident-reports/INC-<NNN>.md` | Sao chép từ `_TEMPLATE.md` |
| Báo cáo tổng kết | `reports/lab-report-*.md` | Mỗi mốc lớn một báo cáo; số liệu chưa đo ghi "chưa đo" |
| Quy trình xử lý sự cố | `docs/incident-response.md` | Hoàn thiện ở Phase 6 |

## Quy trình thêm nội dung mới

| Khi làm | Các bước |
|---|---|
| Phase hạ tầng mới | 1. Thêm mục vào `docs/deployment-guide.md`. 2. Cập nhật `docs/network-topology.md` nếu có VM hoặc mạng mới. 3. Cập nhật sơ đồ và bảng Trạng thái. 4. Đặt ảnh vào `docs/images/phase-N/` |
| Kịch bản hoặc rule mới | 1. Tạo thư mục `scenarios/<tên>/` từ `_TEMPLATE.md`. 2. Thêm rule vào `detection-rules/local_rules.xml`. 3. Ghi giải thích vào `docs/detection-engineering.md`. 4. Thêm truy vấn vào `queries/`. 5. Cập nhật bảng Kịch bản kiểm thử |
| Sự cố điều tra | 1. Sao chép `reports/incident-reports/_TEMPLATE.md`. 2. Điền timeline, bằng chứng, MITRE. 3. Ghi rõ Alert, Suspicious activity hay Confirmed incident |
| Kết thúc một mốc | Cập nhật hoặc tạo báo cáo trong `reports/` |

Chỉ tạo thư mục khi đã có nội dung. Ví dụ `scenarios/port-scanning/` tạo ở Phase 5.


