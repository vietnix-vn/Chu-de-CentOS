# Hướng dẫn cách cài đặt và sử dụng PostgreSQL trên CentOS 7 chi tiết

PostgreSQL hoặc Postgres là một hệ thống relational database management cung cấp triển khai việc truy vấn bằng ngôn ngữ SQL. Nó được sử dụng ở nhiều dự án nhỏ lớn khác nhau và có lợi thế là tuân thủ các tiêu chuẩn, có nhiều tính năng nâng cao. Trong bài viết này, Vietnix sẽ hướng dẫn bạn cách cài đặt và sử dụng PostgreSQL trên CentOS 7.

## Mục lục

- [Yêu cầu để cài đặt và sử dụng PostgreSQL trên CentOS 7](#1)
- [Bước 1: Cài đặt PostgreSQL](#2)
- [Bước 2: Tạo PostgreSQL databases cluster](#3)
- [Bước 3: Sử dụng roles và databases ở PostgreSQL](#4)
    - [Chuyển sang tài khoản postgres](#4.1)
    - [Truy cập giao diện dòng lệnh Postgres mà không cần chuyển đổi tài khoản](#4.2)
- [Bước 4: Tạo role mới](#5)
- [Bước 5: Tạo database](#6)
- [Bước 6: Mở Postgres Prompt với role mới](#7)
- [Bước 7: Tạo và xóa bảng](#8)
- [Bước 8: Thêm, truy vấn và xóa dữ liệu trong bảng](#9)
- [Bước 9: Thêm và xóa cột khỏi bảng](#10)
- [Bước 10: Cập nhật dữ liệu trong bảng](#11)
- [Lời kết](#12)
----
<a name="1"></a>
## Yêu cầu để cài đặt và sử dụng PostgreSQL trên CentOS 7

Để làm theo hướng dẫn này, bạn sẽ cần:

- Một máy chủ CentOS 7 bao gồm người dùng non-root có quyền sudo và tường lửa được thiết lập với firewalld.

- Databases có thể bị ảnh hưởng bởi sự thay đổi thời gian của hệ thống nếu chúng có nhiều thay đổi, hoạt động và có lưu mốc thời gian trên bản ghi nội bộ. Để ngăn chặn một số hành vi không đồng bộ có thể phát sinh từ out-of-sync, hãy đảm bảo thiết lập Network Time Protocol (NTP).

Nếu bạn cần một máy chủ CentOS 7 để cài đặt PostgreSQL, hãy tham khảo đăng ký thuê VPS tại Vietnix. Vietnix hiện đang cung cấp các gói VPS tốc độ cao hỗ trợ hệ điều hành CentOS với nhiều cấu hình và chi phí khác nhau giúp bạn dễ dàng lựa chọn gói dịch vụ phù hợp với nhu cầu sử dụng. VPS của Vietnix giúp bạn yên tâm cài đặt, triển khai các ứng dụng của mình nhờ tốc độ cao, ổn định và bảo mật, an toàn dữ liệu. Liên hệ ngay để được tư vấn lựa chọn.

<a name="2"></a>
## Bước 1: Cài đặt PostgreSQL

Postgres có thể được cài đặt bằng cách sử dụng kho lưu trữ CentOS mặc định. Nhưng khi viết hướng dẫn này, phiên bản có sẵn trong kho lưu trữ CentOS 7 Base đã khá cũ. Do đó, hướng dẫn này sẽ sử dụng kho lưu trữ Postgres chính thức.

Trước khi bạn chuyển sang thiết lập kho lưu trữ mới, hãy loại trừ tìm kiếm các gói `postgresql` khỏi kho lưu trữ CentOS-Base. Nếu không, các phần phụ thuộc đi kèm có thể phân giải thành `postgresql` được cung cấp bởi kho lưu trữ cơ sở.

Mở file cấu hình kho lưu trữ bằng trình soạn thảo. Ở đây sử dụng `vim`:

    sudo vi /etc/yum.repos.d/CentOS-Base.repo

Tìm phần `[base]` và `[updates]`, vào chế độ chèn bằng cách nhấn `i` và chèn `exclude-postgresql*` trong cả hai phần. Kết quả là, tệp của bạn sẽ hiển thị như sau:

```
...
[base]
name=CentOS-$releasever - Base
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

exclude=postgresql*

#released updates
[updates]
name=CentOS-$releasever - Updates
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
exclude=postgresql*
...
```

Khi bạn hoàn tất, nhấn **ESC** để rời khỏi chế độ chèn, sau đó `:wq` và **ENTER** để lưu và thoát file.

Bây giờ, hãy cài đặt gói cấu hình kho lưu trữ bằng kho lưu trữ PostgreSQL chính thức cho CentOS:

    sudo yum install https://download.postgresql.org/pub/repos/yum/reporpms/EL-7-x86_64/pgdg-redhat-repo-latest.noarch.rpm

Khi nhận được thông báo, hãy xác nhận cài đặt với `y`.

Kho lưu trữ PostgreSQL bao gồm thông tin cho tất cả các bản phát hành PostgreSQL có sẵn. Bạn có thể xem tất cả các gói và phiên bản có sẵn bằng lệnh sau:

    yum list postgresql*

Chọn và cài đặt phiên bản PostgreSQL mong muốn. Trong bài viết này, bạn sẽ được hướng dẫn sử dụng bản phát hành PostgreSQL 11.

Để cài đặt máy chủ PostgreSQL, sử dụng lệnh sau:

    sudo yum install postgresql11-server

Trong quá trình cài đặt, bạn sẽ được hỏi về việc nhập GPG key với thông báo như sau:

```
...
Importing GPG key 0x442DF0F8:
 Userid     : "PostgreSQL RPM Building Project <pgsqlrpms-hackers@pgfoundry.org>"
 Fingerprint: 68c9 e2b9 1a37 d136 fe74 d176 1f16 d2e1 442d f0f8
 Package    : pgdg-redhat-repo-42.0-5.noarch (installed)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-PGDG
Is this ok [y/N]:
```

Xác nhận bằng việc nhập `y` để quá trình cài đặt có thể hoàn tất.

<a name="3"></a>
## Bước 2: Tạo PostgreSQL databases cluster

Bạn phải tạo một PostgreSQL database cluster mới trước khi có thể sử dụng Postgres database. Database cluster (cụm cơ sở dữ liệu) là tập hợp databases được quản lý bởi một phiên bản máy chủ đơn. Việc tạo database cluster bao gồm việc tạo các thư mục trong đó chứa dữ liệu của databases, tạo các bảng danh mục được chia sẻ và tạo `template1` và `postgres` database.

`template1` database là cần thiết để tạo cơ sở dữ liệu mới. Mọi thứ được lưu trữ trong đó sẽ được đặt trong database mới một khi nó được tạo. `postgres` database là cơ sở dữ liệu mặc định được thiết kế để sử dụng bởi người dùng, tiện ích và third-party applications.

Tiến hành tạo PostgreSQL database cluster mới với `initdb`:

    sudo /usr/pgsql-11/bin/postgresql-11-setup initdb

Bạn sẽ nhận được output như sau:

```
Output
Initializing database ... OK
```

Bây giờ hãy khởi động và bật PostgreSQL bằng `systemctl`:

```
sudo systemctl start postgresql-11
sudo systemctl enable postgresql-11
```

Việc này sẽ cho ra output như sau:

```
Output
Created symlink from /etc/systemd/system/multi-user.target.wants/postgresql-11.service to /usr/lib/systemd/system/postgresql-11.service.
```

Bây giờ PostgreSQL đã được thiết lập và vận hành, bạn sẽ tiếp tục sử dụng các role để tìm hiểu cách Postgres hoạt động và nó khác với các hệ thống quản lý cơ sở dữ liệu tương tự mà bạn có thể đã từng sử dụng như thế nào.

<a name="4"></a>
## Bước 3: Sử dụng roles và databases ở PostgreSQL

Theo mặc định, Postgres sẽ sử dụng một khái niệm gọi là roles để xử lý trong xác thực và phân quyền. Roles có một số điểm tương đồng với các tài khoản Unix thông thường, nhưng khác ở chỗ là Postgres không phân biệt giữa người dùng và nhóm, thay vào đó sử dụng thuật ngữ linh hoạt hơn là roles.

Sau khi hoàn tất quá trình cài đặt, Postgres được thiết lập để sử dụng xác thực ident, có nghĩa là nó liên kết các role Postgres với tài khoản hệ thống Unix/Linux phù hợp. Nếu một role tồn tại trong Postgres, và tên người dùng Unix/Linux có cùng tên có thể đăng nhập với role đó.

Quy trình cài đặt đã tạo một tài khoản người dùng có tên `postgres` được liên kết với role Postgres mặc định. Để sử dụng Postgres, bạn có thể đăng nhập bằng tài khoản này.

Có một số cách sử dụng tài khoản để truy cập Postgres.

<a name="4.1"></a>
### Chuyển sang tài khoản postgres

Chuyển sang tài khoản postgres trên máy chủ bằng lệnh:

    sudo -i -u postgres

Truy cập vào prompt Postgres ngay bằng cách gõ:

    psql

Thao tác này sẽ giúp bạn đăng nhập vào giao diện dòng lệnh PostgreSQL và bạn có thể tương tác với hệ thống quản lý cơ sở dữ liệu ngay.

Thoát khỏi giao diện dòng lệnh PostgreSQL bằng cách gõ:

    \q

Bạn sẽ trở lại giao diện dòng lệnh Linux postgres. Bạn quay lại tài khoản sudo ban đầu với lệnh:

    exit

<a name="4.2"></a>
### Truy cập giao diện dòng lệnh Postgres mà không cần chuyển đổi tài khoản

Bạn cũng có thể chạy lệnh bạn muốn với tài khoản postgres trực tiếp với `sudo`.

Như ở ví dụ trước, bạn đã biết cách truy cập giao diện dòng lệnh Postgres bằng cách chuyển sang người dùng postgres và sau đó chạy `psql` để mở giao diện dòng lệnh Postgres. Bạn có thể làm điều này trong một bước bằng cách chạy duy nhất lệnh `psql` với tư cách là người dùng postgres kèm `sudo`, thực hiện như sau:

    sudo -u postgres psql

Thao tác này sẽ đăng nhập trực tiếp vào Postgres mà không cần thông qua `bash` shell.

Thoát khỏi Postgres bằng cách gõ như sau:

    \q

Trong bước này, bạn đã sử dụng tài khoản postgres để truy cập `psql`. Nhưng trong nhiều trường hợp sẽ yêu cầu nhiều hơn một role Postgres. Vì vậy, hãy cùng sang bước tiếp theo để tìm hiểu cách cấu hình roles mới.

<a name="5"></a>
## Bước 4: Tạo role mới

Hiện tại, bạn chỉ cần cấu hình postgres role trong database. Bạn có thể tạo role mới từ dòng lệnh bằng lệnh `createrole`. `--interactive` sẽ nhắc bạn nhập tên của role mới đồng thời hỏi bạn có cấp cho role mới này quyền superuser hay không.

Nếu bạn đã đăng nhập bằng tài khoản postgres, bạn có thể tạo người dùng mới bằng cách nhập:

    createuser --interactive

Thay vào đó, nếu bạn muốn sử dụng `sudo` cho mỗi lệnh mà không cần chuyển từ tài khoản thông thường của mình, hãy nhập:

    sudo -u postgres createuser --interactive

Tập lệnh này sẽ thông báo cho bạn với một số lựa chọn và dựa trên phản hồi của bạn, sẽ thực hiện các lệnh Postgres chính xác để tạo người dùng theo thông số kỹ thuật thích hợp. Ở đây sẽ tạo một người dùng vietnix và cung cấp quyền superuser:

```
Output
Enter name of role to add: vietnix
Shall the new role be a superuser? (y/n) y
```

Bạn có thể tăng cường khả năng kiểm soát bằng cách cung cấp một số flag bổ sung. Kiểm tra các tùy chọn bằng cách xem trang `man`:

    man createuser

<a name="6"></a>
## Bước 5: Tạo database

Một giả định khác mà hệ thống xác thực Postgres đưa ra theo mặc định là đối với bất kỳ role nào được sử dụng để đăng nhập, role đó sẽ cùng tên với database mà nó có thể truy cập.

Điều này có nghĩa là, nếu người dùng bạn đã tạo trong phần cuối cùng được gọi là vietnix, vai trò đó sẽ cố gắng để kết nối với database còn được gọi là `vietnix` theo mặc định. Bạn có thể tạo database thích hợp bằng  lệnh `createdb`.

Nếu bạn đã đăng nhập bằng tài khoản postgres, bạn sẽ nhập như sau:

    createdb vietnix

Nếu bạn muốn sử dụng `sudo` cho mỗi lệnh mà không cần chuyển từ tài khoản thông thường thì nhập:

    sudo -u postgres createdb vietnix

Tính linh hoạt này cung cấp nhiều đường dẫn để tạo database khi cần.

<a name="7"></a>
## Bước 6: Mở Postgres Prompt với role mới

Để đăng nhập dựa trên xác thực `ident` cần một người dùng Linux có cùng tên với role và database ở Postgres.

Nếu không có sẵn người dùng Linux phù hợp, bạn có thể tạo một người dùng bằng lệnh `adduser`. Bạn sẽ phải thực hiện việc này từ tài khoản non-root của mình với các quyền `sudo` (nghĩa là không đăng nhập với tư cách là người dùng postgres):

    sudo adduser vietnix

Khi tài khoản mới này đã sẵn sàng để sử dụng, bạn có thể chuyển đổi và kết nối với database bằng cách nhập:

```
sudo -i -u vietnix
psql
```

Hoặc thực hiện như sau:

    sudo -u vietnix psql

Lệnh này sẽ tự động đăng nhập cho bạn.

Nếu muốn người dùng kết nối với một database khác, bạn có thể chỉ định database như sau:

    psql -d postgres

Sau khi đăng nhập, bạn có thể kiểm tra thông tin kết nối hiện tại của mình bằng cách nhập:

    \conninfo

Việc này sẽ hiển thị output như sau:

```
Output
You are connected to database "vietnix" as user "vietnix" via socket in "/var/run/postgresql" at port "5432".
```

Điều này rất hữu ích nếu bạn đang kết nối với database không mặc định hoặc với người dùng không mặc định.

<a name="8"></a>
## Bước 7: Tạo và xóa bảng

Tại đây sẽ hướng dẫn thêm một số tác vụ quản lý Postgres cơ bản.

Đầu tiên, tạo một bảng để lưu trữ dữ liệu. Ví dụ, bạn sẽ tạo một bảng lưu trữ dữ liệu về thiết bị sân chơi ngoài trời.

Cú pháp để thực thi như sau:

```
CREATE TABLE table_name (
    column_name1 col_type (field_length) column_constraints,
    column_name2 col_type (field_length),
    column_name3 col_type (field_length)
);
```

Lệnh này cho phép đặt tên cho bảng, sau đó là tên cột, kiểu dữ liệu cho cột đó và số lượng kí tự tối đa trong cột. Bạn cũng có thể tìm hiểu thêm về các ràng buộc cho mỗi cột của bảng.

Dưới đây là ví dụ minh họa cho cú pháp tạo bảng:

```
CREATE TABLE playground (
    equip_id serial PRIMARY KEY,
    type varchar (50) NOT NULL,
    color varchar (25) NOT NULL,
    location varchar(25) check (location in ('north', 'south', 'west', 'east', 'northeast', 'southeast', 'southwest', 'northwest')),
    install_date date
);
```

Các lệnh này sẽ tạo ra một bảng thống kê thiết bị sân chơi ngoài trời. Bắt đầu với mã ID của thiết bị, thuộc kiểu `serial`. Kiểu dữ liệu này là một số nguyên tự động tăng. Cung cấp `primary key` (khóa chính) cho `equip_id`, việc này có nghĩa là các giá trị phải là duy nhất và không được null.

Đối với hai trong số các cột (là `equip_id` và `install_date`), sẽ không xác định số lượng tối đa cho kí tự. Việc này là do có một số kiểu dữ liệu không yêu cầu phải thiết lập số lượng tối đa cho kí tự

Hai câu lệnh có chứa `type` và `color` biểu thị cho loại thiết bị và màu sắc tương ứng, mỗi lệnh không được null. Câu lệnh sau tiếp theo tạo ra một cột có tên là `location` và thêm ràng buộc yêu cầu các giá trị của cột này phải thuộc 1 trong 8 giá trị sau: north, south, west, east, northeast, southeast, southwest, northwest. Câu lệnh cuối tạo một cột có tên là install_date ghi lại ngày mà bạn cài đặt thiết bị ngoài trời.

Bạn có thể xem bảng mới vừa tạo của mình bằng cách nhập câu lệnh sau:

    \d

Output được hiển thị như sau:

```
Output
                  List of relations
 Schema |          Name                      |   Type       | Owner 
------------+-----------------------------------+--------------+----------
 public    | playground                       | table        | vietnix
 public    | playground_equip_id_seq | sequence | vietnix
(2 rows)
```

Có một dòng gọi là `playground_equip_id_seq` thuộc loại `sequence`. Đây là do việc bạn đã khai báo kiểu `serial` trước đó cho cột `equip_id` của mình . Việc này sẽ giúp bạn tạo tự động ID theo trình tự cho các cột thuộc loại này.

Nếu bạn chỉ muốn xem bảng mà không có sequence, bạn có thể nhập:

    \dt

Output sẽ hiển thị như sau:

```
Output
          List of relations
 Schema |    Name      | Type  | Owner 
------------+----------------+--------+-------
 public    | playground | table | vietnix
(1 row)
```
<a name="9"></a>
## Bước 8: Thêm, truy vấn và xóa dữ liệu trong bảng

Hiện tại bạn đã vừa tạo một bảng dữ liệu cho mình, bạn có thể thực hiện việc thêm dữ liệu vào bảng đó.

Ví dụ: để thêm slide và swing bằng cách gọi bảng mà bạn muốn thêm dữ liệu vào, chỉ định tên cột, sau đó cung cấp dữ liệu cho từng cột, như sau:

```
INSERT INTO playground (type, color, location, install_date) VALUES ('slide', 'blue', 'south', '2017-04-28');
INSERT INTO playground (type, color, location, install_date) VALUES ('swing', 'yellow', 'northwest', '2018-08-16');
```

Khi nhập dữ liệu cần cẩn thận để tránh bị treo máy. Một việc cần lưu ý là bạn không nên đặt tên cột trong dấu ngoặc kép, nhưng các giá trị của cột mà bạn nhập cần được đặt trong dấu ngoặc kép.

Một điều khác cần lưu ý là bạn không cần nhập giá trị cho cột `equip_id`. Điều này là do giá trị của cột này được tạo tự động bất cứ khi nào một hàng dữ liệu mới trong bảng được tạo.

Truy xuất thông tin bạn vừa thêm bằng cách nhập lệnh sau:

    SELECT * FROM playground;

Output sẽ hiển thị như sau:

```
Output
 equip_id | type   | color   | location    | install_date 
-------------+--------+---------+--------------+----------------
        1     | slide   | blue    | south        | 2017-04-28
        2     | swing | yellow | northwest | 2018-08-16
(2 rows)
```

Ở đây, bạn có thể thấy rằng `equip_id` đã được thêm thành công và tất cả các dữ liệu khác của bạn đã được sắp xếp một cách chính xác.

Nếu slide từ bảng playground không còn cần thiết hoặc bị sai sót, bạn có thể thực hiện việc xóa slide khỏi bảng của mình bằng cách nhập:

    DELETE FROM playground WHERE type = 'slide';

Truy vấn lại bảng như sau:

    SELECT * FROM playground;

Kết quả sau khi thực hiện việc câu truy vấn trên:

```
Output
 equip_id | type  | color  | location  | install_date 
----------+-------+--------+-----------+--------------
        2 | swing | yellow | northwest | 2018-08-16
(1 row)
```

Lưu ý rằng dữ liệu về slide không còn là một phần của bảng.

<a name="10"></a>
## Bước 9: Thêm và xóa cột khỏi bảng

Sau khi tạo bảng, bạn có thể thực hiện việc thêm hoặc xóa cột. Thêm cột để hiển thị lần bảo trì cuối cùng cho từng thiết bị bằng cách nhập:

    ALTER TABLE playground ADD last_maint date;

Khi xem lại thông tin bảng sẽ thấy cột mới đã được thêm vào (nhưng không có dữ liệu nào được nhập):

    SELECT * FROM playground;

Bạn sẽ nhận được kết quả như sau:

```
Output
 equip_id |  type  | color   | location    | install_date   | last_maint 
-------------+--------+---------+--------------+------------------+------------
        2     | swing | yellow | northwest | 2018-08-16   | 
(1 row)
```

Xóa một cột cũng đơn giản như vậy. Nếu sử dụng một công cụ riêng biệt để theo dõi lịch sử bảo trì, bạn có thể xóa cột bằng cách nhập:

    ALTER TABLE playground DROP last_maint;

Thao tác này sẽ xóa cột `last_maint` và bất kỳ giá trị nào được tìm thấy trong đó, nhưng vẫn giữ nguyên tất cả các dữ liệu khác.

<a name="11"></a>
## Bước 10: Cập nhật dữ liệu trong bảng

Bạn có thể cập nhật các giá trị của dữ liệu bằng cách truy vấn vào bảng bạn muốn, chọn cột chứa giá trị cần cập nhật và thay đổi thành giá trị bạn muốn sử dụng. Bạn có thể truy vấn bản vào `swing` (giá trị này sẽ khớp với dữ liệu có cùng giá trị swing trong bảng của bạn) và thay đổi màu của nó thành `red`:

    UPDATE playground SET color = 'red' WHERE type = 'swing';

Xác minh thao tác đã thành công bằng cách truy vấn lại dữ liệu:

    SELECT * FROM playground;

Bạn sẽ thấy output như sau:

```
Output
 equip_id | type   | color | location    | install_date 
------------+---------+-------+---------------+--------------
        2     | swing | red    | northwest | 2010-08-16
(1 row)
```

Lúc này ở color đã được cập nhật thành màu đỏ.

Ngoài ra, bạn cũng có thể tham khảo thêm cách để bảo mật Nginx bằng Let’s Encrypt trên CentOS 7, bởi vì bảo mật trang web của bạn với Let's Encrypt trên Nginx và CentOS 7 là rất quan trọng để đảm bảo an toàn cho người dùng giúp bạn bảo vệ dữ liệu của mình trước các mối đe dọa mạng.

<a name="`12"></a>
## Lời kết

Trong bài viết này, chúng ta đã tìm hiểu về cách cài đặt và sử dụng PostgreSQL trên CentOS 7. Qua đó, bạn có thể tự tin triển khai hệ quản trị cơ sở dữ liệu PostgreSQL trên môi trường CentOS 7 của mình. Cảm ơn các bạn đọc bài hướng dẫn, nếu có góp ý hay bất kì thắc mắc nào xin bạn hãy để lại bình luận bên dưới nhé.

Nguồn: [Vietnix.vn](https://vietnix.vn/cai-dat-va-su-dung-postgresql-tren-centos-7/)