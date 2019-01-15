## Các bước kiểm tra và xử lý một server Linux bị virut 

## 1. Tình trạng
- Khi hệ thống có hiện tượng băng thông tăng đột biến từ một VM theo chiều OUT cần truy cập vào VM đó để kiểm tra
- Ta có thể dùng lệnh bên dưới để kiểm tra các kết nối 
```sh
lsof -Pnl +M -i4
```
- Ngoài ra cũng có thể sử dụng lệnh top -c và kiểm tra xem process lạ nào đang chiếm tài nguyên hệ thống sau đó dùng lệnh "lsof -p PID" (với PID là ID của process đó) để kiểm tra file nguồn của process đó
- Trong cả 2 cách kiểm tra trên có một điểm chung là sẽ có 1 process hoạt động với tên lạ như fnhtdaooubx.Khi đó ta xác định server đã bị nhiễm virut và cần phải xử lý

## 2. Kiểm tra
- Cài đặt mlocate and updatedb
```sh
yum install mlocate -y
updatedb
```

- Kiểm tra các process đang hoạt động
```sh
ps -ej
```

- Kiểm tra các service tự start khi khởi động server
```sh
chkconfig --list
```

- Kiểm tra file crontab,Khi server bị nhiễm virut thì file crontab sẽ có nội dung như sau
```sh
*/3 * * * * root /etc/cron.hourly/gcc.sh
```

- Kiểm tra cron.hourly
```sh
ls -alt /etc/cron.hourly/
```

  - Nội dung có thể như sau
  ```sh
  drwxr-xr-x. 69 root root 4096 Jan 26 00:10 ..
  -rwxr-xr-x.  1 root root  228 Jan 25 10:42 gcc.sh
  -rwxr-xr-x.  1 root root  148 Jan 25 10:42 fnhtdaooubx.sh
  drwxr-xr-x.  2 root root 4096 Dec 25 10:59 .
  -rwxr-xr-x.  1 root root  409 Aug 23  2016 0anacron
  ```
  
## 3. Cách xử lý
- Xem nội dung file gcc.sh, nếu nội dung hiển thị như bên dưới thì đây là script virut 
```sh
cat /etc/cron.hourly/gcc.sh
#! / bin / sh
PATH = / bin: / sbin: / usr / bin: / usr / sbin: / usr / local / bin: / usr / local / sbin: / usr / X11R6 / bin
for i in `cat / proc / net / dev | grep: | awk -F: {'print $ 1'}`; do ifconfig $ i up & done
cp /lib/libudev.so /lib/libudev.so.6
/lib/libudev.so.6
++++++++++
```

- Ta cần làm
```sh
chmod 000 /lib/libudev.so
```

- Xóa dòng "*/3 * * * * root /etc/cron.hourly/gcc.sh" trong file crontab

- Xóa file gcc.sh 
```sh
rm -f /etc/cron.hourly/gcc.sh 
```

- Kiểm tra các file virut hiện đang chạy
```sh
chkconfig --list
```

  - Nội dung có thể hiển thị như bên dưới
  ```sh
  sdf3fslsdf13    0:off   1:on    2:on    3:on    4:on    5:on    6:off
  fnhtdaooubx     0:off   1:on    2:on    3:on    4:on    5:on    6:off
  ```
  
  - Tiếp tục
  ```sh
  ps -ej
  ```
  Lệnh trên sẽ hiển thị tên của một vài process lạ như
  ```sh
  fnhtdaooubx
  sdf3fslsdf13
  ```
  
  Lưu ý tìm đúng process nguồn của virut vì khi gõ lệnh ps -ej sẽ hiển thị rất nhiều process virut nhưng sé có 1 process nguồn, process nguồn là process tên sẽ không thay đổi liên tục khi ta gõ lệnh "ps -ej" liên tục
  
  - Kiểm tra PID và stop PID của các process lạ được tìm thấy 
  Lưu ý: Chỉ stop chứ không kill, STOP với lệnh bên dưới
  ```sh
  kill -STOP 2109
  ```
  Với 2109 là PID của process virut 
  
  - Sau khi STOP process thì tiến hành xóa các file.Ví dụ với file virut nguồn tên sdf3fslsdf13, ta sẽ dùng lệnh sau để tìm các đường dẫn chứa file
  ```sh
  locate sdf3fslsdf13
  ```
  
  Nội dung sẽ hiển thị như sau
  ```sh
  /bin/sdf3fslsdf13
  /etc/rc.d/init.d/sdf3fslsdf13
  /etc/rc.d/rc0.d/K90sdf3fslsdf13
  /etc/rc.d/rc1.d/S90sdf3fslsdf13
  /etc/rc.d/rc2.d/S90sdf3fslsdf13
  /etc/rc.d/rc3.d/S90sdf3fslsdf13
  /etc/rc.d/rc4.d/S90sdf3fslsdf13
  /etc/rc.d/rc5.d/S90sdf3fslsdf13
  /etc/rc.d/rc6.d/K90sdf3fslsdf13
  ```
  
  Tiến hành xóa các file trên
  ```sh
  rm -f /bin/sdf3fslsdf13 /etc/rc.d/init.d/sdf3fslsdf13 /etc/rc.d/rc0.d/K90sdf3fslsdf13 etc/rc.d/rc1.d/S90sdf3fslsdf13 /etc/rc.d/rc2.d/S90sdf3fslsdf13 /etc/rc.d/rc4.d/S90sdf3fslsdf13 /etc/rc.d/rc5.d/S90sdf3fslsdf13 /etc/rc.d/rc6.d/K90sdf3fslsdf13
  ```
  
  - Xóa các file trong đường dẫn 
  ```sh
  rm -f /usr/bin/sdf3fslsdf13
  ```
  
- Kiểm tra lại trong đường dẫn /usr/bin với các file thay đổi gần đây xem còn file nào có tên loằng ngoàng thì xóa hết đi, Lưu ý không xóa nhầm file của hệ thống, các file virut thường có tên dạng như *sdf3fslsdf13*
```sh
ls -lt /usr/bin | more
```

- Tiến hành kill các process nguồn ở bước trước đó STOP
```sh
pkill sdf3fslsdf13
```

- Xóa file body virut 
```sh
rm -f /lib/libudev.so
```

- secure fstab : Thêm vào file fstab với nội dung như sau 
```sh
tmpfs                   /dev/shm                tmpfs   defaults,nodev,nosuid,noexec        0 0
```

- Khi đã xử lý xong virut thì check bằng một số lệnh như top, "ps -ej" xem có tồn tại process lạ không, trong file crontab còn sinh ra các dòng crond lạ được chèn vào không

