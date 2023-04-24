# Hướng dẫn cài đặt MongoDB trên CentOS 7 chi tiết

MongoDB là một database hướng tài liệu (document) mã nguồn mở miễn phí thuộc dạng NoSQL database. Mọi dữ liệu sẽ được lưu trữ trong document theo kiểu JSON thay vì lưu theo dạng bảng, giúp truy vấn tài liệu nhanh hơn. Khác relational database, với MongoDB bạn có thể thay đổi schema bất cứ lúc nào mà không cần thiết lập một database mới với schema đã được cập nhật. Xem hướng dẫn cách cài đặt phiên bản MongoDB Community trên CentOS 7 sau đây.

## Mục luc 

- [Chuẩn bị](#1)
- [Bước 1: Thêm kho lưu trữ MongoDB](#2)
- [Bước 2: Cài đặt MongoDB](#3)
- [Bước 3: Xác minh việc khởi động](#4)
- [Bước 4: Nhập một tập dữ liệu ví dụ (Tùy chọn)](#5)
- [Lời kết](#6)
----
<a name="1"></a>
## Chuẩn bị

Để thực hiện các bước trong bài viết này bạn cần chuẩn bị một máy chủ CentOS 7 với người dùng non-root và có quyền `sudo`.

Nếu bạn chưa có máy chủ để thực hiện cài đặt MongoDB trên CentOS 7, hãy tham khảo đăng ký dịch vụ VPS của Vietnix. Vietnix hiện đang cung cấp các gói VPS tốc độ cao hỗ trợ hệ điều hành CentOS 7 với nhiều cấu hình và giá cả khác nhau. Liên hệ ngay với Vietnix để được tư vấn lựa chọn gói dịch vụ phù hợp với nhu cầu của bạn.

Sau khi đăng ký VPS CentOS 7 tại Vietnix, bạn có thể thực hiện cài đặt MongoDB trên CentOS 7 theo các bước sau đây.

<a name="2"></a>
## Bước 1: Thêm kho lưu trữ MongoDB

Gói `mongodb-org` không có sẵn trong kho lưu trữ mặc định của CentOS. Tuy nhiên, MongoDB có một kho lưu trữ riêng và bạn cần thêm kho đó vào máy chủ của bạn.

Đầu tiên, sử dụng soạn thảo `vi` để tạo một file `.repo` cho `yum` - một tiện ích quản lý gói cho CentOS:

    sudo vi /etc/yum.repos.d/mongodb-org.repo

Tiếp theo, truy cập vào phần Install on Red Hat của MongoDB’s documentation và thêm thông tin kho lưu trữ cho phiên bản mới nhất vào file:

```
[mongodb-org-6.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/6.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-6.0.asc
```

Lưu các thay đổi vào file bằng cách nhấn phím **ESC**. Sau đó gõ `:wq` và nhấn **ENTER**.

Trước khi tiếp tục, bạn nên kiểm tra kho lưu trữ MongoDB có đang tồn tại trong tiện ích `yum` hay không. Lệnh `repolist` sẽ  hiển thị danh sách các kho lưu trữ đã được kích hoạt:

    yum repolist

```
Output
. . .
repo id                          repo name
base/7/x86_64                    CentOS-7 - Base
extras/7/x86_64                  CentOS-7 - Extras
mongodb-org-6.0/7/x86_64         MongoDB Repository
updates/7/x86_64                 CentOS-7 - Updates
. . .
```

Sau khi thiết lập `kho lưu trữ MongoDB`, bạn có thể tiếp tục cài đặt.

<a name="3"></a>
## Bước 2: Cài đặt MongoDB

Bạn có thể cài đặt gói `mongodb-org` từ kho lưu trữ bên thứ ba bằng cách sử dụng tiện ích `yum`.

    sudo yum install mongodb-org

Sau đó, hai thông báo `Is this ok [y/N]:` sẽ hiển thị. Thông báo đầu tiên cho phép cài đặt các gói MongoDB và thông báo thứ hai yêu cầu nhập một khóa GPG. Nhà phát triển MongoDB cho phần mềm của họ và `yum` sử dụng một khóa để xác minh tính toàn vẹn của các gói đã tải xuống. Ở mỗi thông báo, bạn gõ Y và sau đó nhấn phím **ENTER**.

Bắt đầu dịch vụ MongoDB bằng tiện ích `systemctl`:

    sudo systemctl start mongod

Bạn có thể thay đổi trạng thái dịch vụ của MongoDB với các lệnh `reload` và `stop`.

Lệnh `reload` yêu cầu tiến trình `mongod` đọc file cấu hình `/etc/mongod.conf` và áp dụng thay đổi mà không cần khởi động lại.

    sudo systemctl reload mongod

Lệnh `stop` dừng tất cả các tiến trình `mongod` đang chạy.

    sudo systemctl stop mongod

Tiện ích `systemctl` không cung cấp kết quả sau khi thực hiện lệnh `start`. Nhưng bạn có thể kiểm tra dịch vụ được bắt đầu chưa bằng cách xem cuối file `mongod.log` bằng lệnh `tail`:

    sudo tail /var/log/mongodb/mongod.log

```
Output
. . .
[initandlisten] waiting for connections on port 27017
```

Output cho ra kết quả waiting for a connection xác nhận MongoDB đã khởi động thành công. Bạn có thể truy cập vào database của máy chủ với MongoDB Shell:

    mongo

**Lưu ý:** Khi khởi động MongoDB Shell, bạn có thể thấy một cảnh báo như sau:
** WARNING:** soft rlimits too low. rlimits set to 4096 processes, 64000 files. Number of processes should be at least 32000 : 0.5 times number of files.

MongoDB là một ứng dụng luồng có thể khởi chạy quy trình bổ sung để xử lý các công việc. Lời cảnh báo cho biết: Để MongoDB hoạt động hiệu quả nhất, số lượng quy trình ứng dụng được ủy quyền để khởi chạy phải bằng một nửa số lượng file mà có thể mở tại bất kỳ thời điểm nào. Để giải quyết cảnh báo, hãy thay đổi giá trị soft rlimit cho `mongod` bằng cách chỉnh sửa file `20-nproc.conf`:

    sudo vi /etc/security/limits.d/20-nproc.conf

Sau đó, thêm dòng sau vào cuối file:

```
. . .
mongod soft nproc 32000
```

Lưu các thay đổi bằng cách nhấn phím **ESC**, sau đó gõ `:wq` và nhấn **ENTER**.

Đối với giới hạn mới có sẵn cho MongoDB, hãy khởi động lại nó bằng tiện ích `systemctl`:

    sudo systemctl restart mongod

Khi bạn kết nối với MongoDB Shell, cảnh báo sẽ không còn tồn tại nữa.

Để tìm hiểu cách tương tác với MongoDB từ shell, bạn có thể xem output của phương thức `db.help ()`. Output sẽ cung cấp danh sách các phương thức cho đối tượng db.

    db.help()

```
Output
DB methods:
    db.adminCommand(nameOrDocument) - switches to 'admin' db, and runs command [ just calls db.runCommand(...) ]
    db.auth(username, password)
    db.cloneDatabase(fromhost)
    db.commandHelp(name) returns the help for the command
    db.copyDatabase(fromdb, todb, fromhost)
    db.createCollection(name, { size : ..., capped : ..., max : ... } )
    db.createUser(userDocument)
    db.currentOp() displays currently executing operations in the db
    db.dropDatabase()
. . .
```

Để lại `mongod` chạy ở nền và thoát khỏi shell bằng lệnh `exit`:

    exit

```
Output
Bye
```
<a name="4"></a>
## Bước 3: Xác minh việc khởi động

Vì một ứng dụng database-driven không thể hoạt động mà không có database nên bạn cần đảm bảo MongoDB daemon `- mongod` khởi động cùng hệ thống.

Sử dụng tiện ích `systemctl` để kiểm tra trạng thái khởi động:

    systemctl is-enabled mongod; echo $?

Kết quả trả về là số 0 xác nhận daemon đã được bật. Nếu kết quả trả về là số 1 thì có nghĩa là daemon đã bị tắt và sẽ không khởi động.

```
Output
. . .
enabled
0
```

Nếu daemon bị tắt, bạn hãy sử dụng tiện ích `systemctl` để bật:

    sudo systemctl enable mongod

Bây giờ bạn có một phiên bản MongoDB đang chạy và tự động khởi động sau khi hệ thống khởi động lại.

<a name="5"></a>
## Bước 4: Nhập một tập dữ liệu ví dụ (Tùy chọn)

Không giống như các database server khác, MongoDB không kèm theo dữ liệu trong `test` database. Bạn cần tải xuống tập dữ liệu mẫu từ ví dụ MongoDB. Tài liệu JSON này chứa collection các nhà hàng. Bạn sẽ sử dụng dữ liệu này để tương tác với MongoDB. Tránh gây ảnh hưởng đến những dữ liệu nhạy cảm.

Bắt đầu bằng cách di chuyển vào một thư mục có thể ghi:

    cd /tmp

Sử dụng lệnh `curl` và liên kết từ MongoDB để tải xuống file JSON:

    curl -LO https://raw.githubusercontent.com/mongodb/docs-assets/primer-dataset/primer-dataset.json

Lệnh `mongoimport` sẽ chèn dữ liệu vào test database. Cờ `--db` xác định database nào được sử dụng. Cờ `--collection` chỉ định nơi lưu trữ thông tin trong database. Cờ `--file` cho biết lệnh nào sẽ thực hiện hoạt động nhập:

    mongoimport --db test --collection restaurants --file /tmp/primer-dataset.json

Output xác nhận việc nhập dữ liệu từ file `primer-dataset.json`:

```
Output
connected to: localhost
imported 25359 documents
```

Tập dữ liệu mẫu đã sẵn sàng. Bạn có thể thực hiện truy vấn đối với dữ liệu mẫu.

Khởi động lại MongoDB Shell:

    mongo

Mặc định, shell sẽ chọn `test` database. Đó là nơi bạn đã nhập dữ liệu.

Truy vấn collection restaurants với phương thức `find()` để hiển thị danh sách tất cả các nhà hàng trong tập dữ liệu. Collection chứa hơn 25.000 mục. Do đó, bạn hãy sử dụng phương thức `limit()` để output của câu truy vấn trong một con số xác định. Phương thức `pretty()` làm thông tin dễ đọc hơn.

    db.restaurants.find().limit(1).pretty()

```
Output
{
    "_id" : ObjectId("57e0443b46af7966d1c8fa68"),
    "address" : {
        "building" : "1007",
        "coord" : [
            -73.856077,
            40.848447
        ],
        "street" : "Morris Park Ave",
        "zipcode" : "10462"
    },
    "borough" : "Bronx",
    "cuisine" : "Bakery",
    "grades" : [
        {
            "date" : ISODate("2014-03-03T00:00:00Z"),
            "grade" : "A",
            "score" : 2
        },
        {
            "date" : ISODate("2013-09-11T00:00:00Z"),
            "grade" : "A",
            "score" : 6
        },
        {
            "date" : ISODate("2013-01-24T00:00:00Z"),
            "grade" : "A",
            "score" : 10
        },
        {
            "date" : ISODate("2011-11-23T00:00:00Z"),
            "grade" : "A",
            "score" : 9
        },
        {
            "date" : ISODate("2011-03-10T00:00:00Z"),
            "grade" : "B",
            "score" : 14
        }
    ],
    "name" : "Morris Park Bake Shop",
    "restaurant_id" : "30075445"
}
```

Bạn có thể tiếp tục sử dụng tập dữ liệu mẫu để làm quen với MongoDB hoặc xóa tập này bằng phương thức `db.restaurants.drop()`:

    db.restaurants.drop()

Cuối cùng, sử dụng lệnh `exit` để thoát khỏi Shell:

    exit

```
Output
Bye
```
<a name="6"></a>
## Lời kết

Như vậy, bạn đã tìm hiểu về cách cài đặt MongoDB trên CentOS 7 qua hướng dẫn của chúng tôi, bạn đã có thể thao tác thêm kho lưu trữ bên thứ ba vào `yum`, cài đặt database server của MongoDB, nhập dữ liệu mẫu và thực hiện những truy vấn đơn giản. Bạn còn có thể làm rất nhiều thứ với MongoDB như tạo database với các collection, điền vào database nhiều document,... Hãy theo dõi những bài viết tiếp theo của Vietnix để tìm hiểu thêm nhiều kiến thức về MongoDB nhé.

Nguồn: [Vietnix.vn](https://vietnix.vn/cai-dat-mongodb-tren-centos-7/)