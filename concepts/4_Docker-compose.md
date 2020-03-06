![](https://user-images.githubusercontent.com/49421807/60306722-8d23e100-996b-11e9-9f0a-80682ff61d3b.png)

## 1. Docker compose

  [Khai báo và vận hành nhiều container](https://docs.docker.com/compose/overview/)

## 2. Viết docker-compose

### 2.1 Khai báo service

+ Cấu trúc

  ```yaml
  version:
  services:
    service1:
      ...
    service2:
      ...
    ...
  networks:
  volumes:
  ```

+ `version`
  + Khai báo phiên bản
  + Ví dụ:
    ```yaml
    version: "3.7"
    ```
+ `service`
  + `service1` và `service2` giúp định nghĩa những service mà dự án sử dụng.
  + Ví dụ:
    ```yaml
    services:
      mysql:
        image: mysql:5.7
        container_name: mysql
        restart: always
        environment:
          MYSQL_ROOT_PASSWORD: root
        volumes:
          - ./docker/database:/var/lib/mysql
      web:
        container_name: web
        build: .
        volumes:
          - .:/my_app
        ports:
          - "3000:3000"
        environment:
          DATABASE_HOST: mysql
          DATABASE_USER_NAME: root
          DATABASE_PASSWORD: root
    ```

  + `build`: chỉ định context để build image, từ đó docker sẽ lấy image này để tạo container

      nếu không có build thì ta phải dùng `image`

  + `env_file`: khi chạy container sẽ có những biến env được khai báo trong đó

  + `volumes, command`: tập trung volumes vào docker-compose để tiện maintain

  + `ports`: nếu cần publish port đó ra host

  + `tty/stdin_open`: cho phép attach vào container, chỉ dùng khi dev

  + `restart`: always, để tránh down-time (lúc init project thì nên tránh vì nếu config sai => tạch => restart => tạch => ....)

  + `depends_on`: sẽ đợi cho những services khác chạy xong mới chạy

      Lưu ý là nó chỉ chờ service chạy ok thôi chứ không chờ đến lúc service *ready to work*

      Trong swarm mode depends_on bị ignore nên `restart: always` rất cần thiết, vì trong trường hợp 1 service bị phụ thuộc vào service khác, mà service đó lại chưa up => crash => restart

  * Tất cả các path của host định nghĩa trong file docker-compose đều lấy context là thư mục chứa file docker-compose

  * Khi thay đổi settings trong file docker-compose thì không cần rebuild lại image (trừ những thứ liên quan đến việc build)

* `network`:

  * Nếu ta không muốn dùng network mặc định của docker thì có thể tự định nghĩa.
  * Chung network, service có thể gọi tới nhau thông qua tên service

* `volumes`:

  * Khi ta tự tạo volume ở bên ngoài (external) thì khai báo ở đây để có thể dùng được trong service.


## 3. Cách sử dụng

#### Lý thuyết

  + docker-compose build

    + Build image đã được định nghĩa trong docker-compose.yml

    + Khi khai báo service tất cả những key ngoài `build` ra thì những thứ còn tại đều vô dụng trong Dockerfile

    + Tương tự `docker build` tuy nhiên tiết kiệm thời gian rất nhiều.

  + docker-compose up

    + Tiến hành run service

    + Có thể chạy background với tham số -d

    + Khi chạy ngầm, phải dùng `docker-compose logs` để xem log của service.

    + Ngay cả khi chưa chạy lệnh build, khi up, docker vẫn sẽ build cho ta hoặc pull image về, tạo volume cần thiết, rồi mới up service

    + Mặc định docker sẽ tạo cho ta 1 network nếu ta không khai báo network, tất cả các service đều có thể nhìn thấy nhau.

  + docker-compose down

    + Stop và remove container

    + Volume không bị xóa, có thể remove đi bằng cách thêm tham số -v

  + docker-compose logs

    + Xem log service

#### Thực hành

+ [Get started with Docker Compose
](https://docs.docker.com/compose/gettingstarted/)

+ [Sample apps with Compose: Rails, Django, WordPress](https://docs.docker.com/compose/samples-for-compose/)

###### Note

  + Mặc định khi chạy lệnh docker-compose, những biến trong file .env sẽ có thể truy cập trong file docker-compose, nhưng với Dockerfile thì không.
  + Để có thể dùng trong Dockerfile của service, ta cần khai báo thêm args.
  + Bản chất là wrapper của docker CLI
