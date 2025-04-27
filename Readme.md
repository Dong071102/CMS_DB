# 📥 Hướng Dẫn Import File `CMS-pgsql-db.sql` Vào PostgreSQL

## 1. Yêu Cầu

- Đã cài PostgreSQL (server đang chạy)
- Có tài khoản user (ví dụ: `postgres`)
- Đã tạo sẵn database cần import (ví dụ: `cmsdb`)
- Đã có file `CMS-pgsql-db.sql` # đính kèm trong github

---

## 2. Import Bằng Terminal (Command Line)

### Câu lệnh cơ bản:

```bash
psql -U tên_user -d tên_database -f đường_dẫn_tới_CMS-pgsql-db.sql
```

### Ví dụ cụ thể:

```bash
psql -U postgres -d cmsdb -f /path/to/CMS-pgsql-db.sql
```

(Trên Windows)

```bash
psql -U postgres -d cmsdb -f "D:\path\to\CMS-pgsql-db.sql"
```

> **Lưu ý:**  
> - Khi chạy lệnh có thể PostgreSQL sẽ yêu cầu nhập mật khẩu.  
> - Nếu database `cmsdb` chưa tồn tại, cần tạo trước.

---

## 3. Import Bằng pgAdmin (Giao Diện Đồ Họa)

### Các bước:

1. Mở **pgAdmin** và kết nối vào Server.
2. Chuột phải vào **Database** (`cmsdb`) → chọn **Query Tool**.
3. Nhấn biểu tượng **Open file** 📂 → chọn file `CMS-pgsql-db.sql`.
4. Nhấn **Execute/Run** ⚡️ để thực thi.

> **Lưu ý:**  
> - Đảm bảo đang chọn đúng database `cmsdb` trước khi mở Query Tool.

---

## 4. Xử Lý Một Số Lỗi Thường Gặp

| Lỗi | Nguyên nhân |
|:----|:------------|
| `psql: command not found` | PostgreSQL client chưa cài hoặc chưa thêm vào PATH |
| `FATAL: database "cmsdb" does not exist` | Chưa tạo database `cmsdb` |
| `Permission denied` | User hiện tại không có quyền import |
| Ký tự lỗi (encoding) | File `CMS-pgsql-db.sql` không lưu dưới dạng UTF-8 |

---

## 5. Ghi Chú Thêm

- Nếu file SQL rất lớn, dùng `psql` qua Terminal sẽ nhanh hơn so với pgAdmin.
- Nếu cần import file nén `.gz`, có thể dùng:

```bash
gunzip -c CMS-pgsql-db.sql.gz | psql -U postgres -d cmsdb
```

---

# 🚀 Hướng cài đặt và chạy Kong API Gate-way
## 1. Tải docker 

## 2. Tải kong API gateway trong docker
### Chuẩn Bị Cơ Sở Dữ Liệu
Tạo một mạng Docker tùy chỉnh để các container có thể tìm thấy và giao tiếp với nhau:
```sh
 docker network create kong-net
```
Bạn có thể đặt tên mạng này theo bất kỳ tên nào bạn muốn. Trong hướng dẫn này, chúng tôi sử dụng `kong-net` làm ví dụ.
### Khởi Động Container PostgreSQL::
Tạo volume cho 'kong-database':
```sh
docker volume create kong-database-data
```
Khởi động container PostgreSQL:
```sh
docker run -d --name kong-database \
 --network=kong-net \
 -p 5432:5432 \
 -v kong-database-data:/var/lib/postgresql/data \
 -e "POSTGRES_USER=kong" \
 -e "POSTGRES_DB=kong" \
 -e "POSTGRES_PASSWORD=kongpass" \
 postgres:13
```Kong API Gateway
* POSTGRES_USER và POSTGRES_DB: Đặt giá trị này thành kong. Đây là giá trị mặc định mà Kong Gateway yêu cầu.
* POSTGRES_PASSWORD: Đặt mật khẩu cơ sở dữ liệu thành bất kỳ chuỗi nào.

Trong ví dụ này, container Postgres tên `kong-database` có thể giao tiếp với bất kỳ container nào trên mạng `kong-net`.
### Chuẩn Bị Cơ Sở Dữ Liệu Kong:
```sh
docker run --rm --network=kong-net \
-e "KONG_DATABASE=postgres" \
-e "KONG_PG_HOST=kong-database" \
-e "KONG_PG_PASSWORD=kongpass" \
-e "KONG_PASSWORD=test" \
kong/kong-gateway:3.10.0.1 kong migrations bootstrap
```
Ở đây:

* [KONG_DATABASE](https://docs.konghq.com/gateway/3.10.x/reference/configuration/#database):Xác định loại cơ sở dữ liệu mà Kong đang sử dụng.
* [KONG_PG_HOST](https://docs.konghq.com/gateway/3.10.x/reference/configuration/#postgres-settings): Tên của container Postgres Docker mà giao tiếp qua mạng kong-net, từ bước trước.
* [KONG_PG_PASSWORD](https://docs.konghq.com/gateway/3.10.x/reference/configuration/#postgres-settings): Mật khẩu mà bạn đã đặt khi khởi động container Postgres trong bước trước.
* `KONG_PASSWORD`(Chỉ dành cho phiên bản Enterprise): Mật khẩu mặc định cho người dùng siêu quản trị của Kong Gateway.
* `{IMAGE-NAME:TAG}`  Đây là tên container Kong Gateway và tag, theo sau là lệnh để Kong chuẩn bị cơ sở dữ liệu Postgres.
## 3. Chạy Kong API Gateway
### Khởi động Kong API Gateway
Chạy lệnh sau để khởi động container với Kong Gateway:
```sh
docker run -d --name kong-gateway \
--network=kong-net \
-e "KONG_DATABASE=postgres" \
-e "KONG_PG_HOST=kong-database" \
-e "KONG_PG_USER=kong" \
-e "KONG_PG_PASSWORD=kongpass" \
-e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
-e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
-e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
-e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
-e "KONG_ADMIN_LISTEN=0.0.0.0:8001" \
-e "KONG_ADMIN_GUI_URL=http://localhost:8002" \
-e KONG_LICENSE_DATA \
-p 8000:8000 \
-p 8443:8443 \
-p 8001:8001 \
-p 8444:8444 \
-p 8002:8002 \
-p 8445:8445 \
-p 8003:8003 \
-p 8004:8004 \
kong/kong-gateway:3.10.0.1
```

Where:
* --name and --network:  Tên của container cần tạo và mạng Docker mà nó giao tiếp.
* [KONG_DATABASE](https://docs.konghq.com/gateway/3.10.x/reference/configuration/#database): Xác định loại cơ sở dữ liệu mà Kong đang sử dụng.
* [KONG_PG_HOST](https://docs.konghq.com/gateway/3.10.x/reference/configuration/#postgres-settings): Tên của container Postgres Docker mà giao tiếp qua mạng `kong-net`.
* [KONG_PG_USER and KONG_PG_PASSWORD](https://docs.konghq.com/gateway/3.10.x/reference/configuration/#postgres-settings): Tên người dùng và mật khẩu của Postgres. Kong Gateway cần thông tin đăng nhập này để lưu trữ dữ liệu cấu hình trong cơ sở dữ liệu `KONG_PG_HOST`.
* Các tham số [_LOG](https://docs.konghq.com/gateway/3.10.x/reference/configuration/#general-section) Đặt đường dẫn cho các file log để xuất ra, hoặc sử dụng các giá trị trong ví dụ để in thông điệp và lỗi ra stdout và stderr.
* [KONG_ADMIN_LISTEN](https://docs.konghq.com/gateway/3.10.x/reference/configuration/#admin_listen):  Cổng mà Kong Admin API lắng nghe yêu cầu.
* [KONG_ADMIN_GUI_URL](https://docs.konghq.com/gateway/3.10.x/reference/configuration/#admin_gui_url): URL để truy cập Kong Manager, có tiền tố giao thức (ví dụ, `http://`).
* `KONG_LICENSE_DATA`:  (Chỉ dành cho Enterprise) Nếu bạn có file license và đã lưu nó dưới dạng biến môi trường, tham số này sẽ lấy license từ môi trường của bạn.

### Xác Nhận Cài Đặt:
Truy cập endpoint `/services` bằng Admin API:
```sh
 curl -i -X GET --url http://localhost:8001/services
```
Bạn sẽ nhận được mã trạng thái `200`.
### Kiểm Tra Kong Manager đang chạy bằng cách truy cập vào URL đã chỉ định trong `KONG_ADMIN_GUI_URL`:


 ```sh
 http://localhost:8002
```
## Import dữ liệu vào kong api gateway
### kiểm tra đã có volume kong-database-data
```sh
docker inspect kong-databaseKong API Gateway
```
Lệnh này sẽ trả về một JSON đầy đủ thông tin của container kong-database, bao gồm các volume mà nó đang sử dụng.
Trong kết quả JSON, bạn sẽ tìm thấy phần Mounts chứa thông tin về các volume:
```sh
"Mounts": [
    {
        "Type": "volume",
        "Name": "kong-database-data",
        "Source": "/var/lib/docker/volumes/kong-database-data/_data",
        "Destination": "/var/lib/postgresql/data",
        "Driver": "local",
        "Mode": "z",
        "RW": true,
        "Propagation": ""
    }
]

```
### Tạo một container tạm để xử lý import
Ta sẽ sử dụng một container tạm (ví dụ `busybox` hoặc `alpine`) để mount volume `kong-database-data` và restore dữ liệu từ file backup vào volume này.
```sh
docker run --rm -v kong-database-data:/data -v $(pwd):/backup busybox sh -c "cd /data && tar xzvf /backup/kong_data_backup.tar.gz --strip 1"
```
Giải thích:

* `-v kong-database-data:/data`: Mount volume `kong-database-data` vào thư mục `/data` trong container.

* `-v $(pwd):/backup`: Mount thư mục hiện tại (nơi chứa file `kong_data_backup.tar.gz`) vào thư mục `/backup` trong container.
* `tar xzvf /backup/kong_data_backup.tar.gz --strip 1`: Giải nén file `kong_data_backup.tar.gz` vào thư mục `/data` của volume, bỏ qua thư mục gốc trong file backup (do `--strip 1`).
### Xác minh dữ liệu đã được import
Sau khi lệnh trên hoàn tất, bạn có thể kiểm tra lại việc import dữ liệu vào volume `kong-database-data` bằng cách:
* Kiểm tra container `kong-database`:
Bạn có thể truy cập vào container kong-database để kiểm tra dữ liệu trong cơ sở dữ liệu kong. Chạy lệnh sau để vào container:
```sh
    docker exec -it kong-database psql -U kong -d kong
```
* Sau đó, kiểm tra các bảng dữ liệu trong cơ sở dữ liệu kong.
Kiểm tra API của Kong:
Dùng curl hoặc trình duyệt để truy cập vào `http://localhost:8001/services` để kiểm tra xem các dịch vụ trong Kong đã được import chưa.
# Khởi chạy các container vào lần sau
```sh
docker start kong-gateway kong-database
```