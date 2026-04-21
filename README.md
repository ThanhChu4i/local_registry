# local_registry

Thiết lập nhanh private container registry bằng Docker Compose.

## Thông tin đăng nhập registry

- Username: `admin`
- Password: `XGame@123`

File xác thực: `auth/htpasswd`

## Tạo user/password mới bằng OpenSSL (để người khác dùng sau)

`auth/htpasswd` có format:

username:hashed_password

### 1) Tạo hash password bằng OpenSSL

Ví dụ tạo hash cho password `MyPass@123`:

openssl passwd -apr1 'MyPass@123'

Kết quả sẽ giống dạng:

$apr1$xxxx$yyyyyyyyyyyyyyyyyyyyy/

### 2) Thêm user vào file `auth/htpasswd`

Ví dụ thêm user `dev1`:

dev1:$apr1$xxxx$yyyyyyyyyyyyyyyyyyyyy/

> Mỗi user nằm trên **1 dòng**.

### 3) Đổi password cho user cũ

- Tạo hash mới bằng lệnh OpenSSL ở bước (1)
- Sửa đúng dòng của user trong `auth/htpasswd`

### 4) Áp dụng thay đổi

Sau khi sửa file `auth/htpasswd`, chạy lại:

docker compose -f docker-compose.registry.yml restart registry

### 5) Người dùng đăng nhập và push/pull

docker login localhost:5000 -u dev1 -p 'MyPass@123'

docker tag my-app:latest localhost:5000/my-app:latest
docker push localhost:5000/my-app:latest
docker pull localhost:5000/my-app:latest

## Chạy registry

Khởi động:

docker compose -f docker-compose.registry.yml up -d

Kiểm tra log:

docker compose -f docker-compose.registry.yml logs -f

Dừng:

docker compose -f docker-compose.registry.yml down

## Push/Pull image (sau khi login)

Đăng nhập:

docker login localhost:5000 -u admin -p 'XGame@123'

Gắn tag image local lên registry:

docker tag my-app:latest localhost:5000/my-app:latest

Push:

docker push localhost:5000/my-app:latest

Pull lại:

docker pull localhost:5000/my-app:latest

## Lưu ý quan trọng

Registry lưu **Docker image/artifact**, không lưu source code Git trực tiếp.
Muốn "đưa code lên" thì bạn cần build image từ code trước, rồi push image lên registry.

## Dữ liệu lưu trữ

Dữ liệu image được lưu trong volume `registry_data` để không bị mất khi restart container.