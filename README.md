# Lớp: 58KTPM - Môn: Phát triển ứng dụng với mã nguồn mở-TEE0421

**Bài tập 02:** 
# SỬ DỤNG DJANGO ĐỂ TẠO WEB QUẢN LÝ TIỆM CẦM ĐỒ
## Deadline : 23h59 ngày 09 tháng 5 năm 2026.

1. TỔ CHỨC CSDL CHO HỆ THỐNG QUẢN LÝ TIỆM CẦM ĐỒ: viết tay ra giấy, lấy điện thoại chụp lại, upload ảnh lên github (đã nói về các nghiệp vụ trên lớp, ghi bảng)
   <img width="1920" height="2560" alt="image" src="https://github.com/user-attachments/assets/5ad386d2-d6ee-4913-86ce-a3b9d3036090" />

2. SỬ DỤNG DOCKER TRÊN UBUNTU ĐỂ:

**Mục tiêu:**
- Mariadb  : chứa csdl của hệ thống này
- Phpmyadmin: để soi được csdl (chỉ để xem, ko cần tạo bảng từ đây, django sẽ làm hết)
- Django: build 1 docker container (dùng Dockerfile): trên nền python, sử dụng django, nhớ mount thư mục để dễ edit, edit dùng: sudo nano ten_file

sau khi có 3 service này trong file docker-compose.yml :
- run nó, cấu hình để Django nhận csdl mariadb (sửa file settings.py), cấu hình user login ban đầu, mô tả các bảng trong models.py, .... (đc phép sử dụng AI để làm) => KQ được trang admin, y/c đăng nhập, vào trang admin: cho phép thêm sửa xoá dữ liệu các bảng. các trường là khoá ngoại chỉ việc chọn text (mặc dù là csdl tại trường FK đó lưu ID của PK mà nó tham chiếu : sử dụng phpmyadmin để kiểm chứng)
- chú ý kết hợp ssh để chạy lệnh tác động vào django và sudo nano để edit file.
- sử dụng template (file html, sử dụng cú pháp jinja2), lấy context từ 1 view home_page, để tạo trang liệt kê các con nợ đến hạn mà chưa trả tiền!
- sử dụng cloudflare tunnel để public kết quả lên 1 sub-domain => chụp kết quả

**Thực hiện:**

Trước hết ta cần tạo cây thư mục dự án:
```
camdoshop-project/
│
├── docker-compose.yml
│
├── django/
│   ├── Dockerfile
│   ├── requirements.txt
│   └── app/
│
└── mysql/
    └── data/
```
Ban đầu ta vào django; sửa nội dung các file 

Dockerfile

```
# Dùng image python chính thức
FROM python:3.12

# Không tạo file pyc
ENV PYTHONDONTWRITEBYTECODE=1

# Hiển thị log realtime
ENV PYTHONUNBUFFERED=1

# Thư mục làm việc trong container
WORKDIR /app

# Copy requirements vào container
COPY requirements.txt .

# Cài thư viện python
RUN pip install --no-cache-dir -r requirements.txt

# Copy toàn bộ source code
COPY app/ .

# Mở cổng django
EXPOSE 8000
```

và requirements.txt

```
# Framework Django
Django==5.2

# Driver kết nối MariaDB/MySQL
mysqlclient==2.2.7
```

sau đó, ta tạo file docker-compose.yml với nội dung như sau:

```
version: '3.9'

services:

  mariadb:
    image: mariadb
    container_name: camdoshop_mariadb
    restart: always

    environment:
      MYSQL_ROOT_PASSWORD: 0402
      MYSQL_DATABASE: camdoshop
      MYSQL_USER: hanh
      MYSQL_PASSWORD: 0402

    ports:
      - "3306:3306"

    volumes:
      - ./mysql/data:/var/lib/mysql

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    container_name: camdoshop_phpmyadmin
    restart: always

    environment:
      PMA_HOST: mariadb
      PMA_PORT: 3306

    ports:
      - "8080:80"

    depends_on:
      - mariadb

  django:
    build: ./django
    container_name: camdoshop_django
    restart: always

    ports:
      - "8000:8000"

    volumes:
      - ./django/app:/app

    depends_on:
      - mariadb

    command: tail -f /dev/null  #để container chạy mãi mãi mà không tiêu tốn CPU
```

Ta cần chú ý command: tail -f /dev/null; Khi lần đầu chạy `sudo docker-compose up -d --build` thì ta chưa có file manage.py trong app/ nên ta phải để lệnh command đó để giữ cho container sống vẫn chạy bình thường mà không làm gì cả, mục đích là để ta thực hiện lệnh `sudo docker-compose exec django bash` để ta chui vào trong container django và thao tác thủ công với project trong đó. Nếu ngay từ đầu đã runserver thì django sẽ bị crash liên tục, khiến restarting service liên tục và không thể thực hiện gì tiếp như hình dưới đây.
<img width="857" height="391" alt="image" src="https://github.com/user-attachments/assets/5c1da139-9b8f-4650-bcd8-0cbea847dfbd" />

Sau khi service chạy ngon rồi thì vào trong container django. 
Ta tạo prj `python manage.py startapp camdoshop`

Tạo prj bằng `django-admin startproject core .`

Kiểm tra `ls` ta phải thấy có core/ và manage.py đã được tạo.

<img width="853" height="279" alt="image" src="https://github.com/user-attachments/assets/7343b389-c66e-486a-b84b-a526ca4931f7" />

Tiếp chạy test migrate SQLite trước `python manage.py migrate` nếu thấy log OK là chuẩn rồi;
<img width="832" height="307" alt="image" src="https://github.com/user-attachments/assets/033d5e53-a103-4e4e-ad39-0a5a030b3743" />

Tạo admin: `python manage.py createsuperuser`

Nhập:
- username
- email
- password

ok. Giờ thoát container `exit`

<img width="832" height="153" alt="image" src="https://github.com/user-attachments/assets/b1fbc7a3-0e7b-4a8c-bf92-072f2c459779" />

Sau đó, ta vào lại docker-compose.yml và sửa command trong django thành như này:
<img width="856" height="269" alt="image" src="https://github.com/user-attachments/assets/4e1df6a5-5044-4cc1-b97a-38cebaf5bd1f" />

```
command: >
  sh -c "
  python manage.py runserver 0.0.0.0:8000
  "
```

Sau khi đã thực hiện thủ công với prj trong django thì đã có đủ file để chạy server rồi;

```
sudo docker-compose down
sudo docker-compose up -d
```

Kiểm tra log `sudo docker logs -f camdoshop_django` ta thấy có dòng 'Starting development server at http://0.0.0.0:8000/' là chạy ngon rồi.
<img width="863" height="184" alt="image" src="https://github.com/user-attachments/assets/2db5411e-8cf1-422b-b07e-d756d6f33bd1" />

Tuy nhiên khi ta truy cập http://192.168.56.10:8000 thì lại thấy rằng không truy cập được host này;
<img width="959" height="392" alt="image" src="https://github.com/user-attachments/assets/1b8351d2-c65c-4596-84bd-4eef0bbdc1f9" />

Lý do là ta chưa cho phép server django chạy trên host đó. Cần vào ngay /django/app/core/settings.py; tìm tới dòng 'ALLOWED_HOSTS = []' sửa thành 'ALLOWED_HOSTS = [192.168.56.10]' rồi load lại trình duyệt là ok;

<img width="857" height="287" alt="image" src="https://github.com/user-attachments/assets/0b433f3d-df13-4a59-a329-a0bcc305a253" />

<img width="959" height="391" alt="image" src="https://github.com/user-attachments/assets/43eff890-5a8e-4c25-909f-250880fe7504" />

Sau khi django đã chạy ngon nghẻ thì ta vào trong http://192.168.56.10:8000/admin, đăng nhập bằng username và password đã đặt ở trên;
<img width="959" height="434" alt="image" src="https://github.com/user-attachments/assets/e35e5dbd-fe42-4e52-b573-624f017db0a9" />

Ra giao diện như này là ngon luôn; Giờ đã django đã chạy rồi; Chỉ là chưa có csdl;
<img width="959" height="362" alt="image" src="https://github.com/user-attachments/assets/fbe95366-3df8-48eb-85d3-facd7b835c25" />


Hướng dẫn:
- Tạo thư mục để chứa image tự buid cho django
- Vào thư mục đó tạo file tên: Dockerfile (nội dung hỏi AI xem file này cần có nội dung gì, full comment cho từng dòng lệnh trong file này => mục tiêu kép: để hiểu và để hệ thống chạy được)
- AI sẽ nói cần thêm file requirements.txt để cài các thư viện cho python (cài qua lệnh pip) => tạo file requirements.txt với nội dung tưng ứng, trong file này cũng comment được => comment xem thư viện nào dùng để làm gì
- Sau mỗi lần sửa đỏi có thể phải chạy lệnh dạng : **docker compose exec TÊN_SERVICE_DJANGO_CỦA_BẠN python manage.py migrate** để tác động vào django (còn nhiều lệnh khác chứ ko luôn như này), để django thay đổi csdl hoặc thay đổi cấu hình.

  
