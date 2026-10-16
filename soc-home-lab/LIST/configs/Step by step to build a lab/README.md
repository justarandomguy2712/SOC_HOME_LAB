# Các bước chi tiết dựng SOC HOME LAB (Security Operation Center) với Wazuh

## Phase 1 🛠️


### I. Thiết lập các card mạng ảo để phân tách và phân vùng các khu vực mạng trong môi trường lab.


+ Bỏ chọn “Use local DHCP service” và tick vào ô “Connect a host virtual adapter
+ Vào Virtual Network Editor: Add Network -> Chọn Host-only -> Tạo thêm 3 vmnet 10,11,12 với subnet IP như bảng dưới: 

| Card mạng | Subnet IP |
|---|---|
| VMnet 10 | 10.10.10.0 |
| VMnet 11 | 10.10.20.0 |
| VMnet 12 | 10.10.30.0 |


<p align="center">
<img width="582" height="48" alt="Screenshot 2026-10-14 104623" src="https://github.com/user-attachments/assets/7081986f-a4ce-4119-b171-7d706f502ce5" />
  <br>
  <em>Hình 1: Cấu hình các card mạng trên VMware.</em>
</p>

### II. Cài đặt Ubuntu Server trên VMware 
#### 2.1
+ Import file ISO của version Ubuntu 22.04.LTS (Jammy Jellyfish)

Lấy file iso tại web sau: https://releases.ubuntu.com/jammy 

+ Thực hiện cấu hình Ubuntu server theo bảng sau: 

| Devices | Configuration |
|---|---|
| Processors | 2vCPU |
| RAM | 4 GB |
| Disk | 30 GB (Note: Không tick vào “Allocate all disk space now” |
| Network Adapter 1 | Custom > VMnet10 |
| Network Adapter 2 | NAT |
| CD/DVD | Import file ISO Ubuntu |
 #### 2.2
 + Bật VM và chọn bản Ubuntu Server (Không chọn bản Ubuntu Server minimized)
 + NIC trên VMnet10 (chọn card ens34) : IPv4 Manual, subnet 10.10.10.0/24, address 10.10.10.10, để trống gateway và DNS và NIC còn lại để nguyên
+ Cài OpenSSH server

Sau khi xong xuôi các bước cấu hình thì ta vào được với giao diện sau:


<p align="center">
<img width="663" height="550" alt="Screenshot 2026-10-14 142202" src="https://github.com/user-attachments/assets/4ecbac72-2d10-4cf7-89ab-7377dfc0cbb5" />
  <br>
  <em>Hình 2: Giao diện Ubuntu Server CLI</em>
</p>

### III. Kiểm tra thông mạng và check dung lượng đã đạt đúng yêu cầu chưa

#### 3.1 
+ Sử dụng 4 câu lệnh sau để check thông mạng và xem ổ đĩa đúng 30GB như đã cấu hình chưa

```cmd
ip -br a; df -h /; lsblk; ping -c3 10.10.10.1
```
**Giải thích nhanh ý nghĩa 4 câu lệnh**

| Command | Meaning |
|---|---|
| ```ip -br a``` | Hiển thị nhanh các card mạng, địa chỉ IP và trạng thái kết nối |
| ```df -h``` | Kiểm tra dung lượng ổ đĩa của phân vùng gốc /, gồm tổng dung lượng, đã sử dụng và còn trống |
| ```lsblk``` | Liệt kê các thiết bị lưu trữ, phân vùng và cấu trúc ổ đĩa. |
| ``` ping -c3 10.10.10.1``` | Gửi 3 gói tin ICMP đến địa chỉ 10.10.10.1 để kiểm tra khả năng kết nối mạng. |


<p align="center">
<img width="789" height="419" alt="Screenshot 2026-10-14 142400" src="https://github.com/user-attachments/assets/00cbe1d0-a3ca-4624-b1eb-878a0775d27f" />
  <br>
  <em>Hình 3: Thông tin về mạng và ổ đĩa sau khi kiểm tra</em>
</p>


Note: Mặc dù đã cấu hình dung lượng ổ đĩa lên 30 GB, kết quả thực thi lệnh ```df -h``` vẫn cho thấy dung lượng khả dụng chỉ khoảng 14 GB, như minh họa trong hình bên dưới.

<p align="center">
<img width="580" height="38" alt="Screenshot 2026-10-14 142644" src="https://github.com/user-attachments/assets/c9842b62-82b2-4b96-9760-e2f8670cfafe" />
  <br>
  <em>Hình 4: Dung lượng hiển thị sai là 14 GB thay vì 30GB </em>
</em>
</p>


Do đó, cần thực hiện các lệnh mở rộng phân vùng và hệ thống tệp để dung lượng sử dụng trong Ubuntu được cập nhật tương ứng với dung lượng thực tế đã cấp phát cho ổ đĩa.

```bash
sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
```
**Giải thích nhanh câu lệnh**: 

- Chức năng: Mở rộng hệ thống tệp ext2/ext3/ext4 để sử dụng dung lượng khả dụng của phân vùng Logical Volume.
- sudo: Thực thi lệnh với quyền quản trị.
- resize2fs: Công cụ thay đổi kích thước hệ thống tệp.
- /dev/ubuntu-vg/ubuntu-lv: Đường dẫn đến Logical Volume cần mở rộng.
Lưu ý: Lệnh này không tự mở rộng dung lượng Logical Volume. Nếu Logical Volume chưa được mở rộng lên 30 GB, cần tăng kích thước Logical Volume trước, sau đó mới chạy resize2fs.

<p align="center">
<img width="813" height="95" alt="Screenshot 2026-10-14 142605" src="https://github.com/user-attachments/assets/ff005246-4f8b-4ed9-a479-992d386f59d4" />
  <br>
  <em>Hình 5: Dung lượng ổ đĩa sau khi mở rộng hệ thống tệp  </em>
</em>
</p>

### Phase 2











