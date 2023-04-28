# Hướng dẫn cách thiết lập khóa SSH trên CentOS 7 chi tiết

SSH (hay còn gọi là secure shell) là một giao thức được mã hóa để quản trị và giao tiếp với các máy chủ. Khi làm việc với máy chủ CentOS, bạn sẽ dành phần lớn thời gian trong phiên terminal để kết nối với máy chủ thông qua SSH. Trong bài viết này của Vietnix, bạn sẽ được học cách thiết lập khóa SSH trên CentOS 7 để đăng nhập vào máy chủ đơn giản và an toàn. Cùng tìm hiểu ngay.

## Mục lục

- [Bước 1 – Tạo cặp khóa RSA để thiết lập khóa SSH trên CentOS 7](#1)
- [Bước 2 – Sao chép khóa công khai đến máy chủ CentOS](#2)
    - [Sao chép khóa công khai bằng cách sử dụng ssh-copy-id](#2.1)
    - [Sao chép khóa công khai bằng SSH](#2.2)
    - [Sao chép khóa công khai theo cách thủ công](#2.3)
- [Bước 3 – Xác thực máy chủ CentOS bằng cách sử dụng khóa SSH](#3)
- [Bước 4 – Vô hiệu hóa xác thực mật khẩu trên máy chủ](#4)
- [Lời kết](#5)
-------
<a name="1"></a>
## Bước 1 - Tạo cặp khóa RSA để thiết lập khóa SSH trên CentOS 7

Đầu tiên, bạn cần tạo một cặp khóa trên máy khách (thông thường là máy tính của bạn):

    ssh-keygen

`ssh-keygen` mặc định sẽ tạo một cặp khóa RSA 2048 bit đủ an toàn để sử dụng trong hầu hết trường hợp. Bạn cũng có thể chọn cờ `-b 4096` để tạo một khóa 4096 bit.

Sau khi nhập lệnh, bạn sẽ một thấy thông báo như sau:

```
Output
Generating public/private rsa key pair.
Enter file in which to save the key (/your_home/.ssh/id_rsa):
```

Nhấn **ENTER** để lưu cặp khóa vào thư mục con `.ssh/` trong thư mục home của bạn. Bạn có thể chỉ định một đường dẫn khác để lưu cặp khóa.

Nếu đã tạo một cặp khóa SSH trước đó thì bạn có thể nhận được thông báo sau:

```
Output
/home/your_home/.ssh/id_rsa already exists.
Overwrite (y/n)?
```

Nếu chọn ghi đè khóa trên đĩa thì bạn sẽ không thể xác thực bằng khóa trước đó nữa. Lưu ý rằng khi chọn **yes** thì khóa cũ sẽ bị vô hiệu hóa nên bạn cần cẩn thận. Sau đó, bạn sẽ thấy thông báo:

```
Output
Enter passphrase (empty for no passphrase):
```

Ở đây, bạn có thể tùy chọn nhập mật khẩu an toàn. Đây là lớp bảo mật bổ sung để ngăn chặn đăng nhập của người dùng trái phép.

Sau đó, bạn sẽ thấy output sau đây:

```
Output
Your identification has been saved in /your_home/.ssh/id_rsa.
Your public key has been saved in /your_home/.ssh/id_rsa.pub.
The key fingerprint is:
a9:49:2e:2a:5e:33:3e:a9:de:4e:77:11:58:b6:90:26 username@remote_host
The key's randomart image is:
+--[ RSA 2048]----+
|     ..o         |
|   E o= .        |
|    o. o         |
|        ..       |
|      ..S        |
|     o o.        |
|   =o.+.         |
|. =++..          |
|o=++.            |
+-----------------+
```

Như vậy, bạn đã có một cặp khóa công khai và bí mật sử dụng cho quá trình xác thực. Bước tiếp theo, bạn sẽ tiến hành đặt khóa công khai vào máy chủ để có thể xác thực đăng nhập dựa trên khóa SSH.

<a name="2"></a>
## Bước 2 - Sao chép khóa công khai đến máy chủ CentOS

Cách nhanh nhất để sao chép khóa công khai đến máy chủ CentOS là sử dụng tiện ích `ssh-copy-id`. Phương pháp này được khuyến khích vì cách dùng rất đơn giản. Nếu không có `ssh-copy-id` trên máy, bạn có thể sử dụng một trong hai phương pháp thay thế: Sao chép qua SSH dựa trên mật khẩu hoặc sao chép thủ công.

<a name="2.1"></a>
### Sao chép khóa công khai bằng cách sử dụng `ssh-copy-id`

Công cụ `ssh-copy-id` đã được cài mặc định trong nhiều hệ điều hành. Để thực hiện phương pháp này, bạn phải đã có quyền truy cập SSH dựa trên mật khẩu vào máy chủ.

Bạn chỉ cần chỉ định máy chủ muốn kết nối từ xa và tài khoản người dùng có quyền truy cập SSH dựa trên mật khẩu. Khóa SSH công khai của bạn sẽ được sao chép đến tài khoản này.

Bạn gõ cú pháp như sau:

    ssh-copy-id username@remote_host

Sau đó, bạn có thể nhìn thấy thông báo sau đây:

```
Output
The authenticity of host '203.0.113.1 (203.0.113.1)' can't be established.
ECDSA key fingerprint is fd:fd:d4:f9:77:fe:73:84:e1:55:00:ad:d6:6d:22:fe.
Are you sure you want to continue connecting (yes/no)? yes
```

Thông báo này có nghĩa là máy tính của bạn không nhận ra máy chủ từ xa. Điều này xảy ra khi bạn kết nối với máy chủ mới lần đầu tiên. Nhập `yes` và nhấn **ENTER** để tiếp tục.

Tiếp theo, tiện ích sẽ quét tài khoản của bạn để tìm file khóa công khai `id_rsa.pub` mà bạn đã tạo trước đó. Khi tìm thấy khóa, tiện ích sẽ yêu cầu bạn nhập mật khẩu của tài khoản người dùng từ xa:

```
Output
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
password của username@203.0.113.1:
```

Hãy nhập mật khẩu và nhấn **ENTER**. Mật khẩu sẽ không hiển thị khi nhập nhằm mục đích bảo mật. Tiện ích sẽ kết nối đến tài khoản trên máy chủ từ xa bằng mật khẩu mà bạn đã cung cấp. Sau đó, nội dung của khóa `~/.ssh/id_rsa.pub` sẽ được sao chép vào file `authorized_keys` trong thư mục `~/.ssh` của tài khoản từ xa.

Bạn sẽ thấy output sau đây:

```
Output
Number of key(s) added: 1
Now try logging into the machine, with:   "ssh 'username@203.0.113.1'"
and check to make sure that only the key(s) you wanted were added.
```

Khóa `id_rsa.pub` của bạn đã được tải lên tài khoản từ xa. Bạn có thể tiếp tục bước 3.

<a name="2.2"></a>
### Sao chép khóa công khai bằng SSH

Trường hợp bạn không có `ssh-copy-id` nhưng có quyền truy cập vào một tài khoản trên máy chủ qua SSH thì có thể tải lên khóa bằng phương pháp SSH thông thường. Cụ thể, bạn sử dụng lệnh `cat` để đọc nội dung của khóa SSH công khai trên máy tính, sau đó chuyển những nội dung này qua kết nối SSH đến máy chủ từ xa.

Ngoài ra, bạn cần đảm bảo rằng thư mục `~/.ssh` tồn tại và có quyền truy cập dưới tài khoản bạn đang sử dụng. Sau đó, bạn có thể xuất nội dung đã chuyển tới thành file `authorized_keys` trong thư mục này. Sử dụng ký hiệu chuyển hướng `>>` để thêm nội dung mới mà không ghi đè lên nội dung cũ. Cách này cho phép bạn thêm khóa mới mà không phá hủy các khóa đã được thêm trước đó.

Bạn hãy gõ câu lệnh như sau:

```
cat ~/.ssh/id_rsa.pub | ssh username@remote_host "mkdir -p ~/.ssh && touch ~/.ssh/authorized_keys && chmod -R go= ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

Sau đó, bạn sẽ thấy một thông báo:

```
Output
The authenticity of host '203.0.113.1 (203.0.113.1)' can't be established.
ECDSA key fingerprint is fd:fd:d4:f9:77:fe:73:84:e1:55:00:ad:d6:6d:22:fe.
Are you sure you want to continue connecting (yes/no)? yes
```

Thông báo này cho thấy máy tính của bạn không nhận ra máy chủ từ xa nếu là lần kết nối đầu tiên. Gõ `yes` và nhấn **ENTER** để tiếp tục.

Sau đó, bạn sẽ được nhắc nhập mật khẩu tài khoản người dùng từ xa:

```
Output
username@203.0.113.1's password:
```

Sau khi nhập mật khẩu, nội dung của khóa `id_rsa.pub` sẽ được sao chép vào cuối file `authorized_keys` của tài khoản người dùng từ xa. Nếu thành công, bạn có thể tiếp tục đến Bước 3.

<a name="2.3"></a>
### Sao chép khóa công khai theo cách thủ công

Nếu không có quyền truy cập vào máy chủ bằng SSH thì bạn sẽ phải hoàn thành bằng cách thủ công bằng cách thêm nội dung của file `id_rsa.pub` vào cuối file `~/.ssh/authorized_keys` trên máy từ xa.

Thực hiện lệnh để hiển thị nội dung của khóa `id_rsa.pub`:

    cat ~/.ssh/id_rsa.pub

Bạn sẽ thấy nội dung của khóa trông giống như sau:

```
Output
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCqql6MzstZYh1TmWWv11q5O3pISj2ZFl9HgH1JLknLLx44+tXfJ7mIrKNxOOwxIxvcBF8PXSYvobFYEZjGIVCEAjrUzLiIxbyCoxVyle7Q+bqgZ8SeeM8wzytsY+dVGcBxF6N4JS+zVk5eMcV385gG3Y6ON3EG112n6d+SMXY0OEBIcO6x+PnUSGHrSgpBgX7Ks1r7xqFa7heJLLt2wWwkARptX7udSq05paBhcpB0pHtA1Rfz3K2B+ZVIpSDfki9UVKzT8JUmwW6NNzSgxUfQHGwnW7kj4jp4AT0VZk3ADw497M2G/12N0PPB5CnhHf7ovgy6nL1ikrygTKRFmNZISvAcywB9GVqNAVE+ZHDSCuURNsAInVzgYo9xgJDW8wUw2o8U77+xiFxgI5QSZX3Iq7YLMgeksaO4rBJEa54k8m5wEiEE1nUhLuJ0X/vh2xPff6SQ1BL/zkOhvJCACK6Vb15mDOeCSq54Cr7kvS46itMosi/uS66+PujOO+xt/2FWYepz6ZlN70bRly57Q06J+ZJoc9FfBCbCyYH7U/ASsmY095ywPsBo1XQ9PqhnN1/YOorJ068foQDNVpm146mUpILVxmq41Cj55YKHEazXGsdBIbXWhcrRf4G2fJLRcGUr9q8/lERo9oxRm5JFX6TCmj6kmiFqv+Ow9gI0x8GvaQ== demo@test
```

Tiếp theo, bạn cần truy cập tài khoản trên máy chủ từ xa. Sau khi bạn truy cập tài khoản của mình trên máy chủ từ xa, hãy đảm bảo rằng thư mục `~/.ssh` đã tồn tại. Nếu thư mục chưa tồn tại, bạn hãy gõ lệnh dưới đây:

    mkdir -p ~/.ssh

Bây giờ, bạn có thể tạo hoặc chỉnh sửa file `authorized_keys`. Bạn có thể thêm nội dung của file `id_rsa.pub` vào cuối file `authorized_keys`. Tạo file bằng lệnh sau:

    echo public_key_string >> ~/.ssh/authorized_keys

Trong lệnh trên, thay thế `public_key_string` bằng output từ lệnh cat `~/.ssh/id_rsa.pub`. Output phải bắt đầu bằng `ssh-rsa AAAA...`.

Cuối cùng, đảm bảo rằng thư mục `~/.ssh` và file `authorized_keys` được thiết lập các quyền thích hợp:

    chmod -R go= ~/.ssh

Bước này sẽ gỡ bỏ tất cả quyền “group” và “other” cho thư mục `~/.ssh/` bằng cách đệ quy.

Nếu bạn sử dụng tài khoản root để cài đặt các khóa cho tài khoản người dùng thì cần đảm bảo thư mục `~/.ssh` thuộc về người dùng và không thuộc về root. Trong ví dụ dưới đây, người dùng có tên là sammy nhưng bạn nên thay thế bằng tên người dùng của bạn.

    chown -R sammy:sammy ~/.ssh

Bây giờ, bạn có thể thử xác thực không cần mật khẩu với máy chủ CentOS của mình.

<a name="3"></a>
## Bước 3 - Xác thực máy chủ CentOS bằng cách sử dụng khóa SSH

Nếu đã hoàn thành các bước trên, bạn nên có thể đăng nhập vào máy chủ từ xa mà không cần mật khẩu của tài khoản từ xa.

Quy trình cơ bản là như nhau, bắt đầu với lệnh: 

    ssh username@remote_host

Nếu đây là lần đầu tiên kết nối với máy chủ (trong trường hợp đã sử dụng phương pháp cuối cùng ở trên) thì bạn có thể thấy thông báo như sau:

```
Output
The authenticity of host '203.0.113.1 (203.0.113.1)' can't be established.
ECDSA key fingerprint is fd:fd:d4:f9:77:fe:73:84:e1:55:00:ad:d6:6d:22:fe.
Are you sure you want to continue connecting (yes/no)? yes
```

Nhập `yes` và nhấn **ENTER** để tiếp tục.

Nếu bạn không cung cấp passphrase cho khóa riêng tư của mình thì sẽ được ngay lập tức. Còn nếu đã cung cấp passphrase cho khóa riêng tư thì bạn sẽ được yêu cầu nhập passphrase ngay bây giờ. Sau khi xác thực, phiên bản shell mới sẽ được mở với tài khoản được cấu hình trên máy chủ CentOS.

Ở bước tiếp theo, hãy cùng tìm hiểu cách bảo mật hệ thống tốt hơn bằng cách vô hiệu hóa xác thực mật khẩu.

<a name="4"></a>
## Bước 4 - Vô hiệu hóa xác thực mật khẩu trên máy chủ 

Nếu có thể đăng nhập vào tài khoản của mình bằng SSH mà không cần mật khẩu thì bạn đã cấu hình xác thực dựa trên khóa SSH thành công. Tuy nhiên, cơ chế xác thực dựa trên mật khẩu của bạn vẫn hoạt động. Điều đó có nghĩa là máy chủ của bạn có thể bị tiếp cận bởi các cuộc tấn công bằng mật khẩu.

Trước khi hoàn tất các bước trong phần này, hãy đảm bảo đã cấu hình xác thực dựa trên khóa SSH cho tài khoản root trên máy chủ này. Hoặc bạn đã cấu hình xác thực dựa trên khóa SSH cho một tài khoản non-root trên máy chủ với quyền `sudo`. Lý do là bởi bước này sẽ khóa đăng nhập dựa trên mật khẩu. Vì vậy, việc đảm bảo rằng bạn vẫn có thể truy cập vào hệ thống quản trị là rất quan trọng.

Sau khi xác nhận rằng tài khoản từ xa có quyền quản trị, bạn hãy đăng nhập vào máy chủ từ xa với khóa SSH hoặc là với tài khoản có quyền `sudo`. Sau đó, mở file cấu hình của dịch vụ SSH:

    sudo vi /etc/ssh/sshd_config

Trong file, bạn hãy tìm chỉ thị `PasswordAuthentication`. Có thể chỉ thị đã bị chú thích. Nhấn `i` để chèn văn bản và sau đó hủy chú thích bằng cách xóa dấu `#` phía trước chỉ thị `PasswordAuthentication`. Khi bạn tìm thấy chỉ thị, hãy thiết lập giá trị là `no`. Khi làm vậy, khả năng đăng nhập qua SSH bằng mật khẩu sẽ bị vô hiệu hóa:

```
...

PasswordAuthentication no

...
```

Khi hoàn thành việc thay đổi, nhấn **ESC** và sau đó gõ `:wq` để lưu các thay đổi vào file và thoát. Để thực hiện các thay đổi này, bạn cần khởi động lại dịch vụ `sshd`:

    sudo systemctl restart sshd.service

Hãy mở một cửa sổ terminal mới và kiểm tra xem dịch vụ SSH đang hoạt động đúng trước khi đóng phiên này:

ssh username@remote_host

Sau khi xác minh dịch vụ SSH, bạn có thể đóng tất cả các phiên máy chủ hiện tại một cách an toàn.

Lúc này, dịch vụ SSH trên máy chủ CentOS của bạn chỉ phản hồi SSH keys. Khả năng đăng nhập bằng mật khẩu đã được vô hiệu hóa thành công. Ngoài ra, bạn cũng có thể tham khảo thêm về việc sử dụng và cài đặt Eclipse Theia Cloud IDE Platform trên CentOS 7, điều này sẽ giúp cho bạn có thể dễ dàng truy cập và làm việc trên các dự án phát triển đa nền tảng, từ đó tăng hiệu suất và tiết kiệm thời gian

Nếu đang tìm kiếm một giải pháp máy chủ đáng tin cậy cho doanh nghiệp của mình, thì dịch vụ VPS Vietnix có thể là lựa chọn phù hợp với nhu cầu của bạn. Vietnix hiện đang cung cấp các gói VPS tốc độ cao với nhiều tùy chọn hệ điều hành khác nhau, bao gồm cả CentOS 7, Ubuntu và Debian. Hệ thống được tối ưu hóa để đảm bảo tốc độ cao và sự ổn định, giúp bạn tập trung vào công việc kinh doanh của mình một cách hiệu quả hơn.

Với giao diện quản trị dễ dàng sử dụng, bạn có thể quản lý VPS của mình một cách tiện lợi và nhanh chóng. Bên cạnh đó, Vietnix luôn cam kết cung cấp giá cả hợp lý và dịch vụ chất lượng cao để đáp ứng nhu cầu của khách hàng. Nếu bạn cần hỗ trợ hoặc có bất kỳ câu hỏi nào về việc sử dụng VPS, hãy liên hệ với đội ngũ Vietnix để được tư vấn nhanh chóng nhất.

<a name="5"></a>
## Lời kết

Qua bài viết trên, Vietnix đã hướng dẫn bạn cách thiết lập khóa SSH trên Centos 7. Bạn đã có thể cấu hình xác thực bằng khóa SSH trên máy chủ mà không cần cung cấp mật khẩu. Hãy theo dõi những bài viết tiếp theo của Vietnix để tìm hiểu thêm về cách làm việc với SSH. Nếu bạn gặp phải khó khăn gì trong quá trình thực hiện thì hãy để lại bình luận bên dưới để được giải đáp nhé.

Nguồn: [Vietnix.vn](https://vietnix.vn/cach-thiet-lap-khoa-ssh-tren-centos-7/)

## Tham khảo bài viết mới

1. [Hướng dẫn cài đặt MongoDB trên CentOS 7 chi tiết](https://github.com/vietnix-vn/Chu-de-CentOS/blob/main/H%C6%B0%E1%BB%9Bng%20d%E1%BA%ABn%20c%C3%A0i%20%C4%91%E1%BA%B7t%20MongoDB%20tr%C3%AAn%20CentOS%207%20chi%20ti%E1%BA%BFt.md)

2. [Hướng dẫn cách cài đặt và sử dụng PostgreSQL trên CentOS 7 chi tiết](https://github.com/vietnix-vn/Chu-de-CentOS/blob/main/H%C6%B0%E1%BB%9Bng%20d%E1%BA%ABn%20c%C3%A1ch%20c%C3%A0i%20%C4%91%E1%BA%B7t%20v%C3%A0%20s%E1%BB%AD%20d%E1%BB%A5ng%20PostgreSQL%20tr%C3%AAn%20CentOS%207%20chi%20ti%E1%BA%BFt.md)