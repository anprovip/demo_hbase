

# Hướng Dẫn Demo HBase Với Docker Compose

Dự án này cung cấp một môi trường HBase (Master và RegionServer) và ZooKeeper tối thiểu được thiết lập qua Docker Compose, cho phép bạn dễ dàng thực hành các lệnh HBase Shell.



## 1. Thiết Lập Môi Trường (Setup)

### 1.1. Khởi Động Các Services

Sử dụng `docker compose` để xây dựng và khởi động các container ở chế độ nền.

```bash
docker compose up -d
````

### 1.2. Đợi Khởi Động (Quan Trọng\!) ⏱️

HBase cần thời gian để khởi động RegionServer và kết nối với ZooKeeper. Hãy đợi ít nhất **2 phút** trước khi truy cập shell.

```bash
sleep 120
```

### 1.3. Kiểm Tra Log (Tùy Chọn)

Kiểm tra log của RegionServer để đảm bảo nó đã khởi động thành công và sẵn sàng phục vụ các region.

```bash
docker compose logs -f hbase-regionserver
```

> **Lưu ý:** Chờ đến khi bạn thấy các thông báo xác nhận RegionServer đã chạy và được gán region.

-----

## 2\. Truy Cập HBase Shell

Sau khi các services đã chạy, bạn có thể truy cập vào container HBase Master và mở shell.

```bash
# Bước 1: Vào container HBase Master
docker exec -it hbase-master bash

# Bước 2: Khởi động HBase Shell bên trong container
hbase shell
```

### 2.1. Kiểm Tra Trạng Thái

Trong HBase Shell, hãy chạy lệnh sau để xác nhận HBase Cluster đang hoạt động:

```hbase
status 'detailed'
```

> **Kết quả mong đợi:** Bạn sẽ thấy **1 active master** và ít nhất **1 live RegionServer**.

-----

## 3\. Hướng Dẫn Demo Cơ Bản Với HBase Shell

Phần này hướng dẫn các lệnh cơ bản để tạo, thao tác và truy vấn dữ liệu trong HBase.

### 3.1. Thao Tác Tạo/Xóa Bảng (DDL)

| Lệnh | Mô Tả |
| :--- | :--- |
| `list` | Hiển thị danh sách tất cả các bảng. |
| `create 'newtbl', 'cf1'` | Tạo bảng **newtbl** với một họ cột **cf1**. |
| `describe 'newtbl'` | Xem cấu hình (các họ cột) của bảng. |
| `disable 'newtbl'` | Vô hiệu hóa (disable) bảng (cần thiết trước khi sửa đổi hoặc xóa). |
| `enable 'newtbl'` | Kích hoạt lại bảng. |
| `drop 'newtbl'` | Xóa bảng **newtbl** vĩnh viễn (yêu cầu bảng phải **disable**). |
| `alter 'newtbl', 'new_cf'` | Thêm một họ cột mới **new\_cf** vào bảng đang tồn tại. |

```hbase
# Ví dụ tạo lại một bảng đơn giản
disable 'newtbl'
drop 'newtbl'
create 'newtbl', 'cf1'
describe 'newtbl'
```

### 3.2. Demo Thao Tác Dữ Liệu (DML) với Nhiều Họ Cột

Bảng trong HBase được thiết kế để lưu trữ dữ liệu đa dạng trong cùng một hàng, sử dụng các **Họ Cột** (Column Families) khác nhau.

```hbase
# 1. Tạo bảng với 3 họ cột: knowledge, info, stats
disable 'newtbl'
drop 'newtbl'
create 'newtbl', 'knowledge', 'info', 'stats'

# 2. CHÈN DỮ LIỆU (PUT)

# Hàng r1: Dùng 2 Họ Cột knowledge
put 'newtbl', 'r1', 'knowledge:music', 'jazz'
put 'newtbl', 'r1', 'knowledge:science', 'physics'

# Hàng r2: Dùng 1 Họ Cột info
put 'newtbl', 'r2', 'info:name', 'Alice'
put 'newtbl', 'r2', 'info:age', '25'

# Hàng r3: Dùng 1 Họ Cột stats
put 'newtbl', 'r3', 'stats:score', '88'
put 'newtbl', 'r3', 'stats:rank', 'top10'

# Hàng r4: Kết hợp 2 Họ Cột knowledge và info
put 'newtbl', 'r4', 'knowledge:history', 'world war II'
put 'newtbl', 'r4', 'info:country', 'Vietnam'

# Hàng r8: Kết hợp cả 3 Họ Cột
put 'newtbl', 'r8', 'info:name', 'Bob'
put 'newtbl', 'r8', 'knowledge:language', 'English'
put 'newtbl', 'r8', 'stats:score', '99'

# 3. TRUY VẤN DỮ LIỆU (READ)

# Lấy toàn bộ hàng r8
get 'newtbl', 'r8'

# Lấy dữ liệu từ hàng r1, chỉ họ cột 'knowledge'
get 'newtbl', 'r1', 'knowledge'

# Lấy dữ liệu từ hàng r4, chỉ cột 'info:country'
get 'newtbl', 'r4', 'info:country'

# Quét 5 hàng đầu tiên
scan 'newtbl', {'LIMIT' => 5}

# Quét tất cả các hàng, nhưng chỉ lấy dữ liệu từ Họ Cột 'info'
scan 'newtbl', {'COLUMNS' => 'info'}

# Quét các hàng có Row Key từ 'r4' đến 'r9' (không bao gồm r9)
scan 'newtbl', {'STARTROW' => 'r4', 'STOPROW' => 'r9'}

# 4. CẬP NHẬT VÀ XÓA DỮ LIỆU

# Cập nhật tuổi của Alice
put 'newtbl', 'r2', 'info:age', '26'

# Xóa một cột cụ thể (knowledge:science) của r1
delete 'newtbl', 'r1', 'knowledge:science'

# Xóa toàn bộ hàng r3
deleteall 'newtbl', 'r3'

# Kiểm tra lại r1 và r3
get 'newtbl', 'r1'
get 'newtbl', 'r3'
```

### 3.3. Chèn Nhiều Bản Ghi Từ File Script

Nếu bạn có một file script lớn (`insert.hbase`), hãy sử dụng lệnh `source` để thực thi hàng loạt.

```bash
# 1. Copy file script từ máy host vào container
docker cp insert.hbase hbase-master:/tmp/insert.hbase

# 2. Vào HBase Shell (nếu chưa vào)
docker exec -it hbase-master hbase shell

# 3. Chạy file script
source '/tmp/insert.hbase'
```

```
```