
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
![pfSense](https://img.shields.io/badge/pfSense-Firewall-212121?style=flat&logo=pfsense&logoColor=white)
![Suricata](https://img.shields.io/badge/Suricata-IDS-EF3B2D?style=flat)
![pfBlockerNG](https://img.shields.io/badge/pfBlockerNG-Package-2C3E50?style=flat)
![Cron](https://img.shields.io/badge/Cron-Scheduler-555555?style=flat)

</p>
</div>



## 📁 Cấu trúc repo | Repository Structure

```
soc-home-lab/
├── configs/            
├── detection-rules/   
├── diagrams/           
├── docs/               
├── evidence/           
├── queries/            
├── reports/           
├── scenarios/          
├── LICENSE
└── README.md

```

## Cách dùng các thư mục template | How to Use Template Directories


- **Cài đặt và cấu hình / Deployment and Configuration:** `docs/deployment-guide.md`
- **Phát hiện / Detection Engineering:** `detection-rules/`, `scenarios/` và `docs/detection-engineering.md`
- **Tổng kết và báo cáo / Reports and Summaries:** `reports/`

| Nội dung  / Content | Vị trí file / File Location | Ghi chú / Notes |
|---|---|---|
| Cài đặt và cấu hình VM, Wazuh, Agent, Sysmon (Phase 1, 2, 3, 5) / VM, Wazuh, Agent, and Sysmon installation and configuration | `docs/deployment-guide.md` | Mỗi phase một mục / One section per phase |
| Sơ đồ kiến trúc / Architecture diagrams | `diagrams/*.mmd` và `docs/architecture.md` | Sửa file `.mmd` rồi cập nhật README nếu kiến trúc thay đổi / Edit the `.mmd` file and update the README if the architecture changes |
| Địa chỉ IP, VMnet và luồng mạng được phép / IP addresses, VMnets, and permitted network flows | `docs/network-topology.md` | Thêm vùng mạng mới khi sang Phase 5 / Add new network segments in Phase 5 |
| File cấu hình mẫu / Sample configuration files | `configs/sysmon/`, `configs/wazuh/` | Mỗi cấu hình ghi rõ lý do sử dụng và cách xác minh / Document the purpose and verification method for each configuration |
| Rule tùy chỉnh (Phase 4) / Custom rules (Phase 4) | `detection-rules/local_rules.xml` | |
| Giải thích rule, so sánh với rule mặc định và theo dõi false positive / Rule explanations, comparison with default rules, and false-positive tracking | `docs/detection-engineering.md` | Thêm một mục cho mỗi rule mới / Add one section for each new rule |
| Kịch bản kiểm thử: hành vi, kết quả và trạng thái Pass/Fail / Test scenario: behavior, results, and Pass/Fail status | `scenarios/<scenario-name>/<ID>-<name>.md` | Sao chép từ `scenarios/_TEMPLATE.md` / Copy from `scenarios/_TEMPLATE.md` |
| Truy vấn Discover / Discover queries | `queries/discover-queries.md` | Ghi rõ truy vấn nào đã được xác minh trên lab / Clearly identify queries verified in the lab |
| Báo cáo điều tra sự cố / Incident investigation report | `reports/incident-reports/INC-<NNN>.md` | Sao chép từ `_TEMPLATE.md` / Copy from `_TEMPLATE.md` |
| Báo cáo tổng kết lab / Lab summary report | `reports/lab-report-*.md` | Tạo báo cáo cho mỗi cột mốc lớn; số liệu chưa đo ghi “chưa đo” / Create a report for each major milestone; mark unmeasured metrics as “not measured” |
| Quy trình ứng phó sự cố / Incident response procedure | `docs/incident-response.md` | Hoàn thiện ở Phase 6 / Complete in Phase 6 |

## Quy trình thêm nội dung mới / Workflow for Adding New Content

| Cấu hình / Config | Các bước / Steps |
|---|---|
| **Phase hạ tầng mới / New infrastructure phase** | 1. Thêm mục vào `docs/deployment-guide.md`. / Add a section to `docs/deployment-guide.md`.<br/>2. Cập nhật `docs/network-topology.md` nếu có VM hoặc mạng mới. / Update `docs/network-topology.md` if new VMs or networks are introduced.<br/>3. Cập nhật sơ đồ và bảng trạng thái. / Update the diagrams and status table. |
| **Kịch bản hoặc rule mới / New scenario or rule** | 1. Tạo thư mục `scenarios/<name>/` và sử dụng `_TEMPLATE.md`. / Create `scenarios/<name>/` using `_TEMPLATE.md`.<br/>2. Thêm rule vào `detection-rules/local_rules.xml`. / Add the rule to `detection-rules/local_rules.xml`.<br/>3. Giải thích rule trong `docs/detection-engineering.md`. / Document the rule in `docs/detection-engineering.md`.<br/>4. Thêm truy vấn vào `queries/`. / Add the query to `queries/`.<br/>5. Cập nhật bảng kịch bản kiểm thử. / Update the test scenario table. |
| **Điều tra / Incident investigation** | 1. Sao chép `reports/incident-reports/_TEMPLATE.md`. / Copy `reports/incident-reports/_TEMPLATE.md`.<br/>2. Điền timeline, bằng chứng và thông tin MITRE ATT&CK. / Document the timeline, evidence, and MITRE ATT&CK mapping.<br/>3. Phân loại rõ: Alert, Suspicious Activity hoặc Confirmed Incident. / Clearly classify the case as an Alert, Suspicious Activity, or Confirmed Incident. |
| **Kết thúc một cột mốc / Completing a milestone** | Cập nhật hoặc tạo báo cáo trong `reports/`. / Update or create a report in `reports/`. |

**Lưu ý / Note:** Chỉ tạo thư mục khi đã có nội dung thực tế. Ví dụ: tạo `scenarios/port-scanning/` khi bắt đầu Phase 5, không tạo sẵn các thư mục chưa sử dụng.

**Only create directories when they contain actual content.** For example, create `scenarios/port-scanning/` when starting Phase 5 rather than creating unused directories in advance.




## 🧑‍💻 Tech Stack đã sử dụng / Technology Stack


# Tech Stack đã sử dụng / Technology Stack

- Virtualization: VMware Workstation (VMnet10 host-only)
- Operating Systems: Ubuntu Server 22.04.5, Windows 10 Enterprise Evaluation
- SIEM: Wazuh 4.14 All-in-one (Manager, Indexer, Dashboard)
- Endpoint Monitoring: Wazuh Agent, Sysmon (config SwiftOnSecurity), Windows Security Event Log
- Detection Engineering: Wazuh custom rules (XML), MITRE ATT&CK
- Firewall & Routing: pfSense 
- Security (IDS): Suricata trên pfSense
- Attack Simulation: Kali Linux (CLI), Atomic Red Team 
- Traffic Analysis: Wireshark, tcpdump 



## Tác giả / Author
Mạnh Hà Nguyễn


🎓 Sinh viên chuyên ngành An toàn thông tin - Học viện Kỹ thuật Mật mã (KMA).

🎓 Information Security Student - Academy of Cryptography Techniques (KMA).






