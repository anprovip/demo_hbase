## Khởi động

```
docker compose up -d
```
## Đợi khoảng 2 phút để các services khởi động hoàn toàn
```
sleep 120
```
## Kiểm tra logs
```
docker compose logs -f hbase-regionserver
```
## Vào HBase Master
```
docker exec -it hbase-master bash
```
## Kiểm tra status
```
hbase shell
```
## Trong HBase shell, chạy:
```
status 'detailed'
```
Hướng dẫn demo HBase

xoa di la lai
disable 'newtbl'
drop 'newtbl'
create 'newtbl', 'cf1'


create
Đề xem các lệnh có thể làm với create
put 'newtbl', 'r1', 'knowledge:sports', 'cricket'

disable 'newtbl'

alter 'newtbl', 'test_info'

enable 'newtbl'

describe 'newtbl'

put 'newtbl', 'r1', 'knowledge:economic', 'market  economics'

Moi han co cac famili khac nhau
tao bang
create 'newtbl', 'knowledge', 'info', 'stats'

# Hàng r1 – dùng family knowledge
put 'newtbl', 'r1', 'knowledge:music', 'jazz'
put 'newtbl', 'r1', 'knowledge:science', 'physics'

# Hàng r2 – dùng family info
put 'newtbl', 'r2', 'info:name', 'Alice'
put 'newtbl', 'r2', 'info:age', '25'

# Hàng r3 – dùng family stats
put 'newtbl', 'r3', 'stats:score', '88'
put 'newtbl', 'r3', 'stats:rank', 'top10'

# Hàng r4 – dùng knowledge + info cùng lúc
put 'newtbl', 'r4', 'knowledge:history', 'world war II'
put 'newtbl', 'r4', 'info:country', 'Vietnam'

# Hàng r5 – chỉ info
put 'newtbl', 'r5', 'info:job', 'Engineer'

# Hàng r6 – chỉ stats
put 'newtbl', 'r6', 'stats:points', '1200'

# Hàng r7 – chỉ knowledge
put 'newtbl', 'r7', 'knowledge:math', 'algebra'

# Hàng r8 – kết hợp cả 3 family
put 'newtbl', 'r8', 'info:name', 'Bob'
put 'newtbl', 'r8', 'knowledge:language', 'English'
put 'newtbl', 'r8', 'stats:score', '99'

# Hàng r9 – chỉ info
put 'newtbl', 'r9', 'info:city', 'Hanoi'

# Hàng r10 – chỉ knowledge
put 'newtbl', 'r10', 'knowledge:philosophy', 'stoicism'


Chen nhieu ban ghi 
Chạy lệnh:

docker cp insert.hbase hbase-master:/tmp/insert.hbase

docker exec -it hbase-master hbase shell

source '/tmp/insert.hbase'
