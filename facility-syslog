####Đặt vấn đề
Khi triển khai log cho Openstack mình có lên google để tìm kiếm thông tin và tìm thấy được các thông tin hữu ích tại đây

- http://www.sebastien-han.fr/blog/2012/12/05/openstack-and-rsyslog/
- http://docs.openstack.org/admin-guide-cloud/content/section_manage-logs.html
- http://serverfault.com/questions/396136/how-to-forward-specific-log-file-outside-of-var-log-with-rsyslog-to-remote-serv

Khi triển khai thì một số khái niệm mình thấy cần được làm rõ hơn đó là : ***Verbose trong các file cấu hình các dịch vụ có tác dụng thế nào? facility để làm gì ? Không dùng rsyslog thì các dịch vụ có thể ghi log được không? Tại sao khi dùng syslog để ghi log dịch vụ lại cần phải định nghĩa ra một facility? Tác dụng chính của facility là gì***?...

---
####I. Các khái niệm chính
####1. Verbose
- Verbose: để hỗ trợ việc xử lý sự cố , cho phép ghi lại log của các dịch vụ
<ul>
<li>Verbose = True --> Cho phép ghi log</li>
<li>Verbose = False --> Không ghi log</li>
</ul>


####2. Facility là gì?
- Khi sử dụng ***rsyslog*** để ghi log thì khái niệm ***facility là không thể thiếu***.
- Facility giúp kiểm soát log đến dựa vào ***nguồn gốc***.
- syslog sẽ sử dụng Facility để ***"quy hoạch"*** lại log (Giống như đánh nhãn nó để dễ quản lý)

**Lấy một ví dụ**: Bạn là người quản lý một tòa nhà. Một ngày bạn nhận được rất nhiều các video ghi lại từ các camera an ninh. Một câu hỏi được đưa ra là: có quá nhiều video đến, và làm thế nào để "quy hoạch" lại chúng ??

"Facility" sẽ giúp bạn làm việc này, trước khi các video được chuyển đến cho bạn nó đã được "dán nhãn" trước (Floor1_video1; Floor1_video2; Floor3_video1...) và bạn chỉ việc đưa chúng vào các Thư mục tương ứng để tiện việc sử dụng.


####3. Ghi log trên các dịch vụ thế nào?

Một khái niệm khác cần phải nhớ đó là, log trên các dịch vụ có thể ***tự nó sinh ra*** hoặc được ***ghi thông qua syslog***.

- [Program] --> [log_file]
- [Program] --> syslog --> [logfile]

***Ở đây các dịch vụ của openstack cũng vậy, hoặc tự sinh ra log hoặc cần phải sử dụng syslog để ghi log***.

---

####II. Triển khai 

**Từ các kiến thức trên** áp dụng khi **triển khai** một mô hình lab mình chia làm 2 trường hợp sau

**TRƯỜNG HỢP 1 :  Dịch vụ tự ghi log**

```
[DEFAULT]
Verbose = True
#debug = False
#use_syslog = True
#syslog_log_facility = LOG_LOCAL0
...
```

Cấu hình trên tức là dịch vụ sẽ ***không dùng rsyslog*** tuy nhên vẫn sinh ra log tại file ***mặc định*** (eg:/var/log/nova/nova-api.log). Ta không thể biết được các log mặc định này thuộc ***facility*** nào (vì facility là khái niệm không thể thiếu khi triển khai log tập trung sử dụng syslog). 

Do đó nếu muốn *"đẩy"* các log lên server thì phải *"định nghĩa"* lại cho các log này một ***facility***

**TRƯỜNG HỢP 2: Sử dụng syslog**

```
[DEFAULT]
Verbose = True
debug = False
use_syslog = True
syslog_log_facility = LOG_LOCAL0
...
```

Lúc này log được ghi lại với ***facility là local0*** --> Các log sinh ra sau này sẽ mặc định được gán với ***facility local0*** này --> sau đó sử dụng ***rsyslog*** đẩy log lên server

**HÌNH MINH HỌA**

<img src="http://i.imgur.com/Piqx0FX.jpg">

