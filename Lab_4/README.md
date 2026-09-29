# LAB 4 - NMAP: NHẬN DIỆN MÁY CHỦ, CỔNG VÀ DỊCH VỤ

## 1. Thông tin sinh viên

- **Họ và tên:** Nguyễn Minh Hoàng
- **MSSV:** 1150080134
- **Tên lab:** Lab 4 - Nmap: Nhận diện máy chủ, cổng và dịch vụ

---

## 2. Phiên bản và môi trường thực hành

- **Máy scanner:** Ubuntu Linux VM
- **Máy đích:** Metasploitable 2
- **VMware:** VMware Workstation
- **Nmap:** 7.98
- **Mạng thực hành:** VMware Host-only
- **Subnet thực tế:** `192.168.186.0/24`
- **Ubuntu scanner:** `192.168.186.130/24`
- **Metasploitable 2:** `192.168.186.131/24`
- **VMnet1:** Host-only, `192.168.186.0/24`

> Lưu ý: Trong quá trình thực hành, mạng thực tế của VMware là `192.168.186.0/24`, khác với subnet được ghi trong tài liệu mẫu. Ubuntu được sử dụng làm máy scanner và Metasploitable 2 là máy đích. Các thao tác quét được thực hiện trong mạng Host-only của môi trường lab.

---

## 3. Kiểm tra môi trường

### 3.1. Kiểm tra địa chỉ IP

Ubuntu sử dụng:

```text
192.168.186.130/24
```

Metasploitable 2 sử dụng:

```text
192.168.186.131/24
```

Hai máy cùng nằm trong mạng:

```text
192.168.186.0/24
```

### 3.2. Kiểm tra kết nối giữa Ubuntu và Metasploitable 2

Hai máy đã được cấu hình Host-only và có thể liên lạc với nhau bằng ICMP.

**Kết quả:** PASS

---

## 4. 14.1 - Các bài thực hành Nmap

### TH1 - So sánh TCP Connect, TCP SYN và ACK Scan

#### 4.1.1. TCP Connect Scan (`-sT`)

Lệnh:

```bash
nmap -sT -Pn 192.168.186.131
```

Kết quả:

```text
Host is up
23 open
977 closed
```

Các cổng mở chính:

```text
21/tcp    open  ftp
22/tcp    open  ssh
23/tcp    open  telnet
25/tcp    open  smtp
53/tcp    open  domain
80/tcp    open  http
111/tcp   open  rpcbind
139/tcp   open  netbios-ssn
445/tcp   open  microsoft-ds
512/tcp   open  exec
513/tcp   open  login
514/tcp   open  shell
1099/tcp  open  rmiregistry
1524/tcp  open  ingreslock
2049/tcp  open  nfs
2121/tcp  open  ccproxy-ftp
3306/tcp  open  mysql
5432/tcp  open  postgresql
5900/tcp  open  vnc
6000/tcp  open  X11
6667/tcp  open  irc
8009/tcp  open  ajp13
8180/tcp  open  unknown
```

**Nhận xét:** TCP Connect Scan hoàn thành kết nối TCP đầy đủ để xác định cổng mở.

**📸 ẢNH CẦN BỎ VÀO WORD: CÓ**

**Chú thích:** Hình 1. Kết quả TCP Connect Scan (`-sT`) trên Metasploitable 2.

---

#### 4.1.2. TCP SYN Scan (`-sS`)

Lệnh:

```bash
sudo nmap -sS -Pn 192.168.186.131
```

Kết quả:

```text
23 open
977 closed
```

Kết quả các cổng mở tương tự TCP Connect Scan.

**Nhận xét:** SYN Scan sử dụng gói SYN và phân tích phản hồi mà không hoàn thành kết nối TCP đầy đủ.

**📸 ẢNH CẦN BỎ VÀO WORD: CÓ**

**Chú thích:** Hình 2. Kết quả TCP SYN Scan (`-sS`) trên Metasploitable 2.

---

#### 4.1.3. ACK Scan (`-sA`)

Lệnh:

```bash
sudo nmap -sA -Pn 192.168.186.131
```

Kết quả:

```text
All 1000 scanned ports on 192.168.186.131 are in ignored states.
Not shown: 1000 unfiltered tcp ports (reset)
```

**Nhận xét:** ACK Scan không dùng để xác định trực tiếp cổng open/closed. Nó chủ yếu giúp đánh giá trạng thái lọc của firewall. Kết quả `1000 unfiltered` cho thấy các gói ACK nhận được phản hồi reset và Nmap không thấy dấu hiệu lọc trên các cổng được kiểm tra.

**Bảng so sánh:**

| Scan | Open | Closed | Filtered/Unfiltered |
|---|---:|---:|---|
| `-sT` | 23 | 977 | Không ghi nhận filtered |
| `-sS` | 23 | 977 | Không ghi nhận filtered |
| `-sA` | Không xác định | Không xác định | 1000 unfiltered |

**📸 ẢNH CẦN BỎ VÀO WORD: CÓ**

**Chú thích:** Hình 3. Kết quả ACK Scan (`-sA`) trên Metasploitable 2.

---

### TH2 - Quét toàn bộ 65.535 cổng

Lệnh:

```bash
sudo nmap -p- -T4 -Pn 192.168.186.131
```

Kết quả:

```text
30 open
65505 closed
```

Bảy cổng được phát hiện thêm so với 1.000 cổng mặc định:

```text
3632/tcp   open  distccd
6697/tcp   open  ircs-u
8787/tcp   open  msgsrvr
35453/tcp  open  unknown
42497/tcp  open  unknown
50097/tcp  open  unknown
58842/tcp  open  unknown
```

**Nhận xét:** Quét toàn bộ 65.535 cổng phát hiện thêm các dịch vụ nằm ngoài 1.000 cổng mặc định. Điều này cho thấy quét mặc định có thể bỏ sót các dịch vụ chạy trên cổng không phổ biến.

**📸 ẢNH CẦN BỎ VÀO WORD: CÓ**

**Chú thích:** Hình 4. Kết quả quét toàn bộ 65.535 cổng TCP trên Metasploitable 2.

---

### TH3 - Service Version Detection (`-sV`)

Lệnh:

```bash
sudo nmap -sV -Pn 192.168.186.131
```

Một số dịch vụ và phiên bản được nhận diện:

```text
21/tcp    vsftpd 2.3.4
22/tcp    OpenSSH 4.7p1 Debian 8ubuntu1
23/tcp    Linux telnetd
25/tcp    Postfix smtpd
53/tcp    ISC BIND 9.4.2
80/tcp    Apache httpd 2.2.8
139/tcp   Samba smbd 3.X - 4.X
445/tcp   Samba smbd 3.X - 4.X
1524/tcp  Metasploitable root shell
2049/tcp  NFS 2-4
2121/tcp  ProFTPD 1.3.1
3306/tcp  MySQL 5.0.51a-3ubuntu5
5432/tcp  PostgreSQL DB 8.3.0 - 8.3.7
5900/tcp  VNC protocol 3.3
6000/tcp  X11
6667/tcp  UnrealIRCd
8009/tcp  Apache Jserv (Protocol v1.3)
8180/tcp  Apache Tomcat/Coyote JSP engine 1.1
```

Một ví dụ về phiên bản cần chú ý là `vsftpd 2.3.4`. Phiên bản này được NVD ghi nhận có liên quan đến CVE-2011-2523. Việc phát hiện phiên bản không tự động chứng minh mọi điều kiện khai thác của lỗ hổng đều tồn tại; cần xác minh riêng trong đánh giá bảo mật.

**📸 ẢNH CẦN BỎ VÀO WORD: CÓ**

**Chú thích:** Hình 5. Kết quả Service Version Detection (`-sV`) trên Metasploitable 2.

---

### TH4 - NSE `smb-vuln-ms17-010`

Lệnh:

```bash
sudo nmap --script smb-vuln-ms17-010 -p445 -Pn 192.168.186.131
```

Kết quả:

```text
445/tcp open microsoft-ds
```

Không xuất hiện dòng `VULNERABLE`.

**Nhận xét:** Metasploitable 2 đang chạy Linux/Samba. MS17-010 là lỗ hổng liên quan đến SMB của Windows, vì vậy việc script không báo `VULNERABLE` trên Metasploitable 2 là phù hợp. Kết quả cũng cho thấy việc cổng 445 mở không đồng nghĩa với việc máy tồn tại MS17-010.

**📸 ẢNH CẦN BỎ VÀO WORD: CÓ**

**Chú thích:** Hình 6. Kết quả NSE `smb-vuln-ms17-010` trên Metasploitable 2.

---

### TH5 - Decoy Scan

#### Quét bình thường

Lệnh:

```bash
sudo nmap -sS -Pn 192.168.186.131
```

Kết quả:

```text
23 open
977 closed
```

**📸 ẢNH CẦN BỎ VÀO WORD: CÓ**

**Chú thích:** Hình 7. Kết quả quét SYN bình thường trên Metasploitable 2 trước khi sử dụng Decoy.

#### Quét sử dụng Decoy

Lệnh:

```bash
sudo nmap -sS -Pn -D RND:5,ME 192.168.186.131
```

Kết quả:

```text
23 open
977 closed
```

**Nhận xét:** Kết quả dịch vụ của máy đích vẫn tương tự quét bình thường. Tùy chọn `-D RND:5,ME` tạo thêm các địa chỉ nguồn giả trong quá trình quét. Về mặt phòng thủ, kỹ thuật này có thể làm nhật ký mạng xuất hiện nhiều địa chỉ nguồn, gây khó khăn hơn cho việc xác định nguồn quét thực tế. Decoy không đồng nghĩa với ẩn danh hoàn toàn.

**📸 ẢNH CẦN BỎ VÀO WORD: CÓ**

**Chú thích:** Hình 8. Kết quả quét SYN sử dụng kỹ thuật Decoy trên Metasploitable 2.

---

### TH6 - Xuất kết quả bằng `-oA`

Lệnh:

```bash
sudo nmap -sV -Pn -oA lab4_meta 192.168.186.131
```

Nmap đã tạo đủ 3 file:

```text
lab4_meta.nmap
lab4_meta.xml
lab4_meta.gnmap
```

Kiểm tra:

```bash
ls -lh lab4_meta.*
```

Kết quả thực tế:

```text
lab4_meta.gnmap
lab4_meta.nmap
lab4_meta.xml
```

**Ý nghĩa:**

- `.nmap`: kết quả dạng văn bản dễ đọc.
- `.xml`: dữ liệu có cấu trúc, thuận tiện cho xử lý tự động và tích hợp công cụ.
- `.gnmap`: định dạng grepable, thuận tiện tìm kiếm/lọc bằng công cụ dòng lệnh.

**Nhận xét:** `-oA` giúp lưu kết quả ở nhiều định dạng, thuận tiện cho việc lưu bằng chứng, phân tích và lập báo cáo.

**📸 ẢNH CẦN BỎ VÀO WORD: CÓ**

**Chú thích:** Hình 9. Xuất kết quả quét Nmap bằng tùy chọn `-oA`.

---

## 5. 14.2 - Host Discovery và xác định dịch vụ

### 5.1. Host Discovery

Lệnh:

```bash
sudo nmap -sn 192.168.186.0/24
```

Kết quả phát hiện 4 host hoạt động:

```text
192.168.186.1
192.168.186.130
192.168.186.131
192.168.186.254
```

Trong đó:

- `192.168.186.130`: Ubuntu scanner.
- `192.168.186.131`: Metasploitable 2, máy đích lab.
- `192.168.186.1` và `192.168.186.254`: hạ tầng VMware Host-only/DHCP, không sử dụng làm máy mục tiêu trong bài.

**📸 ẢNH CẦN BỎ VÀO WORD: CÓ**

**Chú thích:** Hình 10. Kết quả Host Discovery trên mạng lab `192.168.186.0/24`.

---

### 5.2. Xác định dịch vụ trên Metasploitable 2

Lệnh:

```bash
sudo nmap -sV -Pn 192.168.186.131
```

Bảng tổng hợp:

| Cổng | Dịch vụ | Phiên bản |
|---:|---|---|
| 21 | FTP | vsftpd 2.3.4 |
| 22 | SSH | OpenSSH 4.7p1 |
| 23 | Telnet | Linux telnetd |
| 25 | SMTP | Postfix smtpd |
| 53 | DNS | ISC BIND 9.4.2 |
| 80 | HTTP | Apache httpd 2.2.8 |
| 139 | SMB | Samba 3.X - 4.X |
| 445 | SMB | Samba 3.X - 4.X |
| 1524 | bindshell | Metasploitable root shell |
| 2049 | NFS | NFS 2-4 |
| 2121 | FTP | ProFTPD 1.3.1 |
| 3306 | MySQL | 5.0.51a-3ubuntu5 |
| 5432 | PostgreSQL | 8.3.0 - 8.3.7 |
| 5900 | VNC | Protocol 3.3 |
| 6000 | X11 | Access denied |
| 6667 | IRC | UnrealIRCd |
| 8009 | AJP13 | Apache Jserv 1.3 |
| 8180 | HTTP | Apache Tomcat/Coyote JSP engine 1.1 |

**📸 ẢNH CẦN BỎ VÀO WORD: CÓ**

**Chú thích:** Hình 11. Kết quả xác định dịch vụ và phiên bản trên Metasploitable 2.

### 5.3. Ba dịch vụ cần chú ý

#### 1. FTP - port 21

- Phiên bản: `vsftpd 2.3.4`.
- Đây là phiên bản cũ và có lịch sử lỗ hổng bảo mật.
- Biện pháp: cập nhật phần mềm hoặc vô hiệu hóa FTP nếu không cần; ưu tiên giao thức an toàn như SFTP.

#### 2. Telnet - port 23

- Telnet không cung cấp cơ chế bảo vệ thông tin xác thực như SSH.
- Biện pháp: tắt Telnet và sử dụng SSH.

#### 3. Bindshell - port 1524

- Nmap nhận diện dịch vụ là `Metasploitable root shell`.
- Đây là dịch vụ đặc biệt nguy hiểm trong môi trường Metasploitable.
- Biện pháp: tắt/xóa dịch vụ không cần thiết và kiểm soát truy cập bằng firewall.

**Nhận xét tổng hợp:**

> Metasploitable 2 đang cung cấp nhiều dịch vụ và sử dụng nhiều phiên bản phần mềm cũ. FTP, Telnet và bindshell làm tăng bề mặt tấn công. Biện pháp giảm thiểu gồm vô hiệu hóa dịch vụ không cần thiết, thay thế giao thức không an toàn và cập nhật phần mềm.

---

## 6. 14.3 - Câu hỏi phân tích

### Câu 11. Vì sao quét toàn bộ 65.535 cổng chậm hơn?

Quét toàn bộ 65.535 cổng chậm hơn quét 1.000 cổng mặc định vì Nmap phải gửi và xử lý số lượng gói tin lớn hơn rất nhiều. Với mỗi cổng, Nmap cần xác định trạng thái như `open`, `closed` hoặc `filtered`, nên thời gian quét tăng theo số lượng cổng.

Trong bài thực hành, quét 1.000 cổng phát hiện 23 cổng mở, trong khi quét toàn bộ 65.535 cổng phát hiện 30 cổng mở, trong đó có 7 cổng nằm ngoài 1.000 cổng mặc định.

Quét toàn bộ 65.535 cổng phù hợp khi cần đánh giá đầy đủ bề mặt dịch vụ của một máy, đặc biệt khi nghi ngờ dịch vụ chạy trên cổng không phổ biến.

**📸 ẢNH CẦN BỎ VÀO WORD: KHÔNG**

### Câu 12. Decoy và Fragmentation

Decoy (`-D`) tạo thêm các địa chỉ nguồn giả trong quá trình quét, khiến hệ thống giám sát có thể nhìn thấy nhiều địa chỉ IP cùng xuất hiện trong hoạt động quét.

Fragmentation (`-f`) chia nhỏ gói tin IP thành các mảnh, có thể gây khó khăn cho một số cơ chế phát hiện dựa trên phân tích từng gói tin.

Về phòng thủ, IDS/IPS và firewall nên tái lắp ráp và phân tích lưu lượng, đồng thời tương quan nhiều sự kiện thay vì chỉ dựa vào một địa chỉ nguồn hoặc một gói tin riêng lẻ.

Có thể sử dụng **Hình 8** làm minh chứng cho kỹ thuật Decoy.

**📸 ẢNH CẦN BỎ VÀO WORD: KHÔNG**

### Câu 13. Vì sao `-oA` hữu ích?

`-oA` cho phép lưu kết quả Nmap đồng thời ở `.nmap`, `.xml` và `.gnmap`. Điều này hữu ích trong quá trình đánh giá vì kết quả được lưu lại làm bằng chứng, phục vụ phân tích sau này và hỗ trợ lập báo cáo. XML còn thuận tiện cho xử lý tự động.

Có thể sử dụng **Hình 9** làm minh chứng.

**📸 ẢNH CẦN BỎ VÀO WORD: KHÔNG**

### Câu 14. Nếu tất cả các cổng đều `filtered` thì sao?

Nếu tất cả các cổng được quét đều ở trạng thái `filtered`, điều này thường cho thấy firewall hoặc cơ chế lọc mạng đang chặn các gói tin thăm dò hoặc ngăn Nmap nhận được phản hồi cần thiết để xác định trạng thái cổng.

Trạng thái `filtered` không chứng minh rằng các dịch vụ bên trong máy không tồn tại; nó chỉ cho thấy Nmap không thể xác định trạng thái do cơ chế lọc.

**📸 ẢNH CẦN BỎ VÀO WORD: KHÔNG**

---

## 7. Các câu hỏi phân tích 2-10

### Câu 2. Vì sao `-sS` thường cần quyền cao hơn `-sT`?

`-sS` (SYN Scan) gửi gói SYN và phân tích phản hồi mà không hoàn thành kết nối TCP đầy đủ, nên thường cần quyền cao để tạo và xử lý gói tin thô.

`-sT` (TCP Connect Scan) sử dụng cơ chế kết nối TCP thông thường của hệ điều hành và hoàn thành quá trình bắt tay TCP, vì vậy thường không cần quyền cao như `-sS`.

### Câu 3. Vì sao FIN/Xmas/NULL có thể khó diễn giải?

Các scan FIN/Xmas/NULL dựa vào cách hệ điều hành hoặc firewall phản hồi những gói TCP có cờ đặc biệt hoặc không có cờ thông thường. Cách xử lý các gói này khác nhau giữa hệ điều hành và thiết bị mạng. Vì vậy kết quả có thể xuất hiện dạng `open|filtered`, khó xác định chính xác port đang mở hay bị lọc.

### Câu 4. ACK scan trả lời câu hỏi gì khác với SYN scan?

SYN scan chủ yếu được dùng để xác định trạng thái `open/closed` của port.

ACK scan chủ yếu giúp xác định port có bị lọc hay không, tức là đánh giá khả năng firewall đang lọc gói tin. ACK scan không dùng để khẳng định một port đang `open`.

### Câu 5. Vì sao UDP scan thường chậm và dễ xuất hiện `open|filtered`?

UDP không có cơ chế bắt tay như TCP nên Nmap thường phải dựa vào phản hồi ICMP hoặc phản hồi từ chính dịch vụ để xác định trạng thái. Nhiều UDP port không trả lời khi nhận gói tin, khiến Nmap khó phân biệt giữa `open` và `filtered`. Vì vậy UDP scan thường chậm và có thể xuất hiện `open|filtered`.

### Câu 6. `-sV` đóng vai trò gì trong quản lý lỗ hổng?

`-sV` giúp xác định dịch vụ và phiên bản đang chạy trên port.

Chỉ biết `80/tcp open` cho biết có dịch vụ HTTP nhưng chưa biết đó là phần mềm nào và phiên bản nào. Khi dùng `-sV`, ví dụ trên Metasploitable 2 xác định được Apache httpd 2.2.8. Thông tin phiên bản giúp đối chiếu với các lỗ hổng và đánh giá rủi ro chính xác hơn.

### Câu 7. OS fingerprinting có những giới hạn nào?

OS fingerprinting dựa trên đặc điểm phản hồi của hệ thống mạng nên kết quả có thể bị ảnh hưởng bởi firewall, thiết bị trung gian, cấu hình mạng hoặc việc các hệ điều hành có đặc điểm tương tự nhau.

Vì vậy kết quả `-O` là ước đoán dựa trên fingerprint, không phải bằng chứng tuyệt đối về hệ điều hành.

### Câu 8. NSE timeout có đồng nghĩa "không có lỗ hổng" không?

Không.

Timeout chỉ cho biết script không nhận được phản hồi cần thiết trong thời gian cho phép. Nguyên nhân có thể là firewall, dịch vụ không phản hồi, mạng hoặc script không tương thích.

Do đó không thể kết luận rằng không có lỗ hổng chỉ dựa trên việc NSE timeout.

### Câu 9. Before/after hardening: thay đổi nào chứng minh hiệu quả?

Biện pháp phòng thủ có hiệu lực khi sau hardening, những port/dịch vụ không cần thiết chuyển từ `open` sang `closed` hoặc `filtered`, đồng thời dịch vụ không cần thiết không còn được truy cập từ nguồn quét.

Cần chạy cùng một lệnh scan trước và sau khi hardening để so sánh.

### Câu 10. Ba cấu hình phòng thủ giúp giảm bề mặt tấn công

1. Tắt hoặc gỡ các dịch vụ không cần thiết, giảm số lượng port đang mở.
2. Cấu hình firewall chỉ cho phép các kết nối cần thiết.
3. Cập nhật và vá lỗi phần mềm/dịch vụ, đồng thời loại bỏ các dịch vụ hoặc phiên bản không còn cần thiết.

---

## 8. Kết luận

Qua LAB 4, em đã sử dụng Ubuntu Linux làm máy scanner và Metasploitable 2 làm máy đích trong mạng VMware Host-only.

Các nội dung đã thực hiện gồm:

- Kiểm tra kết nối giữa Ubuntu và Metasploitable 2.
- TCP Connect Scan (`-sT`).
- TCP SYN Scan (`-sS`).
- ACK Scan (`-sA`).
- Quét toàn bộ 65.535 cổng.
- Service Version Detection (`-sV`).
- NSE `smb-vuln-ms17-010`.
- Decoy Scan (`-D`).
- Xuất kết quả bằng `-oA`.
- Host Discovery bằng `-sn`.
- Tổng hợp dịch vụ và phiên bản trên Metasploitable 2.
- Phân tích các câu hỏi lý thuyết liên quan đến Nmap.

Kết quả cho thấy Metasploitable 2 có nhiều dịch vụ TCP đang hoạt động, trong đó có FTP, SSH, Telnet, HTTP, SMB/Samba, MySQL, PostgreSQL, VNC và các dịch vụ khác. Việc sử dụng `-sV`, quét toàn bộ cổng, NSE và Decoy giúp bổ sung thông tin cho quá trình đánh giá bề mặt dịch vụ.

Toàn bộ hoạt động quét được thực hiện trong mạng VMware Host-only:

```text
192.168.186.0/24
```

với:

```text
Ubuntu scanner:      192.168.186.130
Metasploitable 2:    192.168.186.131
```

Không thực hiện quét hệ thống bên ngoài phạm vi môi trường lab.

---

## 9. Danh sách ảnh cần chèn vào Word

| Hình | Nội dung | Chèn Word |
|---:|---|:---:|
| 1 | TCP Connect Scan (`-sT`) | CÓ |
| 2 | TCP SYN Scan (`-sS`) | CÓ |
| 3 | ACK Scan (`-sA`) | CÓ |
| 4 | Quét 65.535 cổng | CÓ |
| 5 | Service Version Detection (`-sV`) | CÓ |
| 6 | NSE `smb-vuln-ms17-010` | CÓ |
| 7 | SYN Scan bình thường trước Decoy | CÓ |
| 8 | SYN Scan với Decoy | CÓ |
| 9 | Xuất kết quả bằng `-oA` | CÓ |
| 10 | Host Discovery | CÓ |
| 11 | Service Version Detection trong 14.2 | CÓ |

> Bản này chỉ ghi các nội dung đã thực hiện trong quá trình làm lab hiện tại. **Không đưa bước copy 3 file sang máy thật vào nội dung thực hành.**
