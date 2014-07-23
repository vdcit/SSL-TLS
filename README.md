# Hướng dẫn cấu hình SSL Certificate qua Apache

**MỤC LỤC** 
được tạo bằng [DocToc](http://doctoc.herokuapp.com/)

# 1.Giới thiệu về SSL
SSL (Secure Socket Layer) là một giao thức được phát triển bởi Netscape để truyền tải dữ liệu cá nhân qua Internet, nói rõ hơn thì giao thức này thiết lập
ra một kênh truyền bảo mật giữa 2 máy. SSL hoạt động dựa trên một cặp khóa bất đối xứng để mã hóa dữ liệu- một khóa công khai và một khóa bí mật. Hiện nay, SSL thường được sử dụng
khi một web browser muốn bảo mật phiên kết nối tới một web server qua Internet.

# 2.Mô hình hoạt động

<img src=http://i.imgur.com/dvTssKl.png width="60%" height="60%" border="1">

- Bước 1: Client gửi yêu cầu kết nối tới Web Server.
- Bước 2: SSL Server gửi lại Certificate cho Client.
- Bước 3: Client sẽ kiểm tra Certificate với Certificate Authority(CA), là một tổ chức có trách nhiệm xác thực cho Web Server.
- Bước 4: Nếu xác thực thành công, Client sẽ gửi Certificate cho Web Server.
- Bước 5: Web Server sẽ check Certificate với CA.
- Bước 6: Nếu xác thực thành công, toàn bộ thông tin truyền tải giữa 2 máy sẽ được mã hóa bằng cặp khóa public và private của Web Server.

# 3. Cấu hình SSL Certificate
Mô hình triển khai

<img src=http://i.imgur.com/TNkLbb2.png width="60%" height="60%" border="1">

<img src=http://i.imgur.com/0Urd8x1.png width="60%" height="60%" border="1">

## 3.1.Khởi tạo Certificate Authorize Server
Cài đặt OpenSSL

    [root@ca]#apt-get install openssl -y
	
Tạo thư mục thử nghiệm

	[root@ca]#mkdir -m 755 /etc/pki
	[root@ca]#mkdir -m 755 /etc/pki/myCA /etc/pki/myCA/private  /etc/pki/myCA/certs /etc/pki/myCA/newcerts  /etc/pki/myCA/crl
	
Tạo file cấu hình

	[root@ca]#cd /etc/pki/myCA
	[root@ca]# touch index.txt
	[root@ca]# echo '01' > serial
	[root@ca]#wget https://raw.githubusercontent.com/longsube/SSL-TLS/master/testssl.conf
	
Tạo Certificate cho bản thân mình

	[root@ca]#cd /etc/pki/myCA
	[root@ca]#openssl req -new -x509 -keyout private/ca.key -out certs/ca.crt -days 1825
	
Phân quyền để bảo mật khóa private

	[root@ca]#chmod 400 /etc/pki/myCA/private/ca.key
	
## 3.2.Khởi tạo Certificate Request từ Web Server
Cài đặt OpenSSL

    [root@web]#apt-get install openssl -y
	
Tạo thư mục thử nghiệm

	[root@web]#mkdir -m 755 /etc/pki

Tạo thư mục để lưu CA
	
	[root@web]#mkdir -m 755 /etc/pki/myCA /etc/pki/myCA/private  

Tạo một certificate request

	[root@web]#cd /etc/pki/myCA 
	[root@web]#openssl req -new -nodes -keyout private/server.key -out server.csr -days 365
	
Chú ý: Common Name (CN) là tên dịch vụ Web

Giới hạn quyền truy cập file private

	[root@web]#chown root /etc/pki/myCA/private/server.key
	[root@web]#chmod 440 /etc/pki/myCA/private/server.key
	
Gửi chứng chỉ tới CA server 

	[root@web]#scp server.csr root@192.168.1.3:/etc/pki/myCA/

## 3.3.Cấp phát chứng chỉ cho Web Server

Chấp nhận một chứng chỉ 

	[root@ca]#cd /etc/pki/myCA/
	[root@ca]#openssl ca –config testssl.conf -out certs/server.crt -infiles server.csr
	
Xóa certificate request

	[root@ca]#rm -f /etc/pki/myCA/server.csr
	
Kiểm tra Certificate

	[root@ca]#openssl x509 -subject -issuer -enddate -noout -in /etc/pki/myCA/certs/server.crt
	
Hoặc

	[root@ca]#openssl x509 -in certs/server.crt -noout -text

Kiểm tra chứng thực với chứng thực máy chủ CA

	[root@ca]#openssl verify -purpose sslserver -CAfile /etc/pki/myCA/certs/ca.crt   /etc/pki/myCA/certs/server.crt
	
Tạo mới một CRL (Certificate Revokation List):

	[root@ca]#openssl ca -config testssl.conf -gencrl -out crl/myca.crl
	
Gửi lại chứng chỉ cho máy chủ Web

	[root@ca]#scp /etc/pki/myCA/certs/server.crt root@web:/etc/pki/myCA
	
## 3.3. Cấu hình Web Server sử dụng Certificate
Cài đặt apache
	
	[root@web]#apt-get install apache2 -y
	
Kích hoạt module ssl

	[root@web]#a2enmod ssl
	
Tạo thư mục chứa Server Key và Server Certificate

	[root@web]#mkdir /etc/apache2/ssl
	[root@web]#cp /etc/pki/myCA/private/server.key /etc/pki/myCA/server.crt /etc/apache2/ssl
	
Xóa file cấu hình mặc định

	[root@web]#rm /etc/apache2/sites-enabled/default-ssl
	
Tạo file cấu hình apache

	[root@web]#cd /etc/apache2/sites-available/
	[root@web]#wget 
	
Tạo trang web thử nghiệm

	[root@web]#cd /var/www/
	[root@web]#wget https://raw.githubusercontent.com/longsube/SSL-TLS/master/index.html
	
Chạy máy chủ web

	[root@web]#chkconfig httpd on
	[root@web]#service httpd start
	
## 3.4. Thử nghiệm
Trên máy client copy file ca.crt trên CA về

Import file ca.crt vào trình duyệt

Truy cập tới https://VDC-IT



	



	


  


