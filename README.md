#Sử dụng rsyslog để thu thập log trong openstack

<a name="ml"></a>
[Mục lục](#ml)

- [I. Mô hình triển khai](#1)
- [II. Cấu hình chính](#2)
<ul>
<li>[a. Server](#sv)</li>
<li>[b. Client](#cl)</li>
</ul>
- [III. Kiểm tra](#3)
- [IV. Sử dụng mô hình vừa triển khai để phân tích log](#4)
- [V.Tài liệu tham khảo](#5)

---
Bài viết này sẽ hướng dẫn sẽ sử dụng rsyslog để thu thập log trong openstack. Cụ thể ở đây sẽ là một rsyslog-server để thu thập log và mô hình openstack triển khai theo 3 node. Ngoài ra các bạn cũng có thể tham khảo thêm các bài viết trước của mình

- [cài đặt openstack](https://github.com/huytm/Oenstack_juno)
- [log tập trung và rsyslog](https://github.com/huytm/Mot-vai-hieu-biet-ve-log)
- [Facility - syslog](https://github.com/huytm/ryslog-for-openstack/blob/master/Facility-syslog-openstack.md)

<a name="1"></a>
##I. Mô hình triển khai 

Ở đây khi triển khai lab mình sử dụng tất cả các OS là ubuntu-14.04.1 và triển khai theo trường hợp thứ 1 của [bài viết](https://github.com/huytm/ryslog-for-openstack/blob/master/Facility-syslog-openstack.md) này

<img src="http://i.imgur.com/JoaxVhX.png">

<a name="2"></a>
##II. Cấu hình chính

<a name="sv"></a>
####a. Tại rsyslog server 

Cấu hình file `vi /etc/rsyslog.conf`

- Tại đây mình sử dụng UDP để nhận log với port 514 nên sẽ uncommen 2 dònng sau

```
$ModLoad imudp
$UDPServerRun 514
```
<img src="http://i.imgur.com/C34S0yV.png">

- Thêm các dòng cấu hình sau để server nhận log và tạo các folder tương ứng với mỗi client đẩy log lên

```
$template TmplAuth,"/var/log/%HOSTNAME%/%PROGRAMNAME%.log"
Auth.*  ?TmplAuth
*.*     ?TmplAuth
& ~
```
<img src="http://i.imgur.com/5sFAUlq.png">

`%HOSTNAME%` : thông số này sẽ quy ước tạo ra trên server 1 thư mục trùng với tên hostname của client. Nếu bạn muốn tên thư mục là IP của server thì thay `%HOSTNAME% = %fromhost-ip%`.

Và chuyển chủ sở hưu tập tin /log/var cho syslog để nó có thể tạo các file và thư mục trong /var/log

- `chown syslog.syslog /var/log`

- Khởi động lại rsyslog `service rsyslog restart`

- Sử dụng câu lệnh sau `netstat -an |grep 514` Nếu thấy kết quả như sau thì cấu hình log server nhận log qua giao thức udp và port 514 là thành công 

<img src="http://i.imgur.com/yTTZUPR.png">


<a name="cl"></a>
####b. Tại các client

Ở đây mình xác định trong khi ***triển khai openstack theo mô hình 3 node*** thì ***controller*** node là node quan trọng, và nó có ***gần như*** đầy đủ những log ***mình cần*** để sử dụng phân tích hệ thống. Nên mình sẽ cấu hình để đẩy một số log cần thiết tại node controller, các bạn cũng có thể đẩy những file log ***khác*** tại ***network node*** hoặc ***compute node*** bằng cách thực hiện tương tự như sau

- Cấu hình file `vi /etc/rsyslog.conf` 

Ở đây sẽ đẩy một số file log quan trọng là apache-error.log, apache-access.log, nova-api.log, keystone-all.log... với cấu trúc như sau

```
$ModLoad imfile #Dòng này chỉ thêm một lần
$InputFileName /data/mysql/error.log (File log muốn đẩy)
$InputFileTag mysql-error  (Tên file sẽ được ghi lại vào local3) 
$InputFileStateFile stat-mysql-error (Trạng thái file)
$InputFileSeverity info (Các log từ mức info trở lên được ghi lại)
$InputFileFacility local3 (Facility log)
$InputRunFileMonitor
local3.* @hostname:<portnumber> 
...
Các file log khác
...
```
<img src="http://i.imgur.com/fpdwC5C.png">

- Khởi động lại rsyslog `service rsyslog restart`

---

<a name="3"></a>
##III. Kiểm tra hoạt động của log server

- Tại log server vào thư mục `cd /var/log`. Tại đây log của mỗi client từ đã được lưu với folder tương ứng

<img src="http://i.imgur.com/cemSnYZ.png">

Với các file log mình đã cấu hình để đẩy lên từ controller node `cd 172.16.69.200 && ls`

<img src="http://i.imgur.com/cxBzYma.png">

---

<a name="4"></a>
##IV. Sử dụng mô hình vừa triển khai để theo dõi log

- Nova

Tại dashboard của openstack thực hiện xóa một 1 instance và kiểm tra log
`cd /var/log/172.16.69.200 && tailf nova.log`

<img src="http://i.imgur.com/jSPLRkj.png">

- Keystone

Tại con troller node thực hiện lệnh `keystone service-list`

<img src="http://i.imgur.com/Y9YZVZj.png">


Tại server file `/var/log/172.16.69.200/keystone` hiển thị kết quả như sau

<img src="http://i.imgur.com/u5BD5GN.png">

Tương tự ta thực hiện lệnh `keystone user-create --name test --pass osjuno --email testrsyslog@abc.com` để ***tạo 1 user mới***

<img src="http://i.imgur.com/ny9G039.png"> 

<img src="http://i.imgur.com/foyn9zH.png">

***Xóa một user***

<img src="http://i.imgur.com/GaEdL7l.png">

<img src="http://i.imgur.com/4m8Bz8u.png">


- Apache

Apache gồm 2 log chính, *access log* thông báo các truy cập đến từ đâu, thực hiện trên trình duyệt nào, thành không hay không với các http code tương ứng. *error log* thông báo quá trình đăng  và các lỗi khác của apache

***access.log***

<img src="http://i.imgur.com/VzHZsru.png">

Log này thông báo với http code là 200 tức là thông báo mình đã truy cập thành công openstack dashboard với địa chỉ 172.16.69.200 từ địa chỉ máy của mình là 172.16.69.1 sử dụng trình duyệt Coccoc

***error.log***

<img src="http://i.imgur.com/CWSUABh.png">

Log này giúp mình nhận được các thông báo về quá trình đăng nhập vào openstack dashboard

---

<a name="5"></a>
##V. Tài liệu tham khảo

- https://github.com/caongocuy/RSYSLOG
- http://xmodulo.com/configure-syslog-server-linux.html
- http://www.rsyslog.com/doc/v8-stable/configuration/modules/imfile.html


