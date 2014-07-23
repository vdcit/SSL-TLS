# Hướng dẫn cấu hình SSL Certificate qua Apache

**MỤC LỤC** 
được tạo bằng [DocToc](http://doctoc.herokuapp.com/)

# 1.Giới thiệu về SSL
SSL (Secure Socket Layer) là một giao thức được phát triển bởi Netscape để truyền tải dữ liệu cá nhân qua Internet, nói rõ hơn thì giao thức này thiết lập
ra một kênh truyền bảo mật giữa 2 máy. SSL hoạt động dựa trên một cặp khóa bất đối xứng để mã hóa dữ liệu- một khóa công khai và một khóa bí mật. Hiện nay, SSL thường được sử dụng
khi một web browser muốn bảo mật phiên kết nối tới một web server qua Internet.

# 2.Mô hình hoạt động

<img src=http://i.imgur.com/dvTssKl.png width="60%" height="60%" border="1">

- Bước 1: Client gửi yêu cầu kết nối tới SSL Server.
- Bước 2: SSL Server gửi lại Certificate cho Client.
- Bước 3: Client sẽ kiểm tra Certificate với Certificate Authority(CA), là một tổ chức có trách nhiệm xác thực cho SSL Server.
- Bước 4: Nếu xác thực thành công, Client sẽ gửi Certificate cho SSL Server.
- Bước 5: SSL Server sẽ check Certificate với CA.
- Bước 6: Nếu xác thực thành công, toàn bộ thông tin truyền tải giữa 2 máy sẽ được mã hóa bằng cặp khóa public và private của SSL Server.

# 3. Cấu hình SSL Certificate
Mô hình triển khai

<img src=http://i.imgur.com/TNkLbb2.png width="60%" height="60%" border="1">

<img src=http://i.imgur.com/0Urd8x1.png width="60%" height="60%" border="1">

# 3.1 Khởi tạo Certificate Authorize Server
Cài đặt OpenSSL
    [root@ca]#apt-get install openssl
Tạo thư mục thử nghiệm
	[root@ca]# mkdir -m 0755 /etc/pki
	[root@ca]# mkdir -m 0755 /etc/pki/myCA /etc/pki/myCA/private  /etc/pki/myCA/certs /etc/pki/myCA/newcerts  /etc/pki/myCA/crl

  


