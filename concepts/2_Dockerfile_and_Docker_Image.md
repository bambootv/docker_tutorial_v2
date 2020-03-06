![](https://user-images.githubusercontent.com/49421807/60164694-3c579f80-9828-11e9-83df-a1947b97ecdc.png)

## 1. Dockerfile là gì ?

![](https://camo.githubusercontent.com/957fbc8b45fc596089690cb9186100224b270e97/68747470733a2f2f696d616765732e7669626c6f2e617369612f37353164373531322d633965372d343461352d626535362d3662316666393039366164662e706e67)

  + [Dockerfile:](https://docs.docker.com/engine/reference/builder/)
    + Một file dạng text.
    + Không có đuôi.
    + Tập hợp các lệnh.
    + Hướng dẫn Docker tạo image.

## 2. Tư tưởng của Dockerfile

  + **Thiết lập image gốc**
  + **Cài đặt phần mềm**
  + **Cấu hình**

  [<img src="https://user-images.githubusercontent.com/49421807/59746865-f2a80b80-92a1-11e9-8faf-c3d56ef4feb4.png">](https://github.com/longnv-0623/Div1_Docker_Course/blob/master/source_code/dockerfile/Dockerfile)

  -> Liên tưởng tới cài lại OS

  -> `Layers on Docker`

## 3. Viết Dockerfile

[Source code](https://github.com/longnv-0623/Div1_Docker_Course/tree/master/source_code/dockerfile)

![](https://user-images.githubusercontent.com/49421807/60113123-4a131380-979b-11e9-9b83-d06065439113.png)

+ `Dockerfile` -> `Docker image` -> `Container`

+ `web_root folder:` project code

+ `start.sh` Khởi động service đi kèm.

**Thiết lập image gốc**
+ [FROM](https://docs.docker.com/engine/reference/builder/#from)
  + Sử dụng 1 image có sẵn làm parent image
  + Hoạt động:
    + Tìm image `ubuntu:16.04` ở trong máy
    + Tự động `pull image` này về từ [DockerHub](https://hub.docker.com/_/ubuntu)
  + Note:
    + ubuntu: tên image, 16:04 là tên tag.
        ![](https://user-images.githubusercontent.com/49421807/59745699-8b895780-929f-11e9-817f-2558d84e1b20.png)
    + Chọn [Official image](https://hub.docker.com/search?q=ubuntu&type=image)

+ [LABEL](https://docs.docker.com/engine/reference/builder/#label)
  + Thông tin đi kèm (Optional)
  + Ví dụ:
    ```
    LABEL author.name="..."
    LABEL author.email="..."
    LABEL description="This docker tutorial"
    ```

**Cài đặt ứng dụng**

+ [ENV](https://docs.docker.com/engine/reference/builder/#env)
  + Khai báo biến môi trường
  + Vẫn được lưu ngay cả sau khi build xong image.

  -> Tránh khai báo thông tin nhạy cảm.

+ [WORKDIR](https://docs.docker.com/engine/reference/builder/#workdir)
  + Thiết lập thư mục làm việc cho RUN, CMD, ENTRYPOINT, COPY và ADD
  + Hoặc khi exec vào container

+ [RUN](https://docs.docker.com/engine/reference/builder/#run)
  + Thực thi câu lệnh.

+ [COPY](https://docs.docker.com/engine/reference/builder/#copy)
  + Copy file từ host sang image
  + VD:
    + Docker file: `/home/sun/app/Dockerfile`
    + WORKDIR `/usr/src/app`
    + `COPY library.json ./`
    -> `/home/sun/app/library.json` (máy thật) -> `/usr/src/app/library.json` (máy ảo).

+ [ADD](https://docs.docker.com/engine/reference/builder/#add)
  Tương tự COPY
  + Hỗ trợ cả url
  + Hỗ trợ giải nén ...

> Note
  + `-y`: Option yes trong yes/no

    ![](https://user-images.githubusercontent.com/49421807/59990178-62384500-966c-11e9-84ce-d90ed95dcafd.png)
  + `set -x`: Hiển thị câu lệnh chuẩn bị chạy tới.
    ![](https://user-images.githubusercontent.com/49421807/59752762-04db7700-92ad-11e9-9c04-7126468822bb.png)

**Cấu hình**

+ [EXPOSE](https://docs.docker.com/engine/reference/builder/#expose):
  + Thông báo cho Docker rằng container sẽ lắng nghe trên các cổng được chỉ định khi chạy

+ [ARG](https://docs.docker.com/engine/reference/builder/#arg):
  + Tham số truyền để build khi chạy `docker run --build-arg`
  + VD khi ta truyền RAILS_MASTER_KEY vào để compile assets file, ta nên dùng lệnh này, vì nó sẽ không lưu lại trong image.

+ [VOLUME](https://docs.docker.com/engine/reference/builder/#volume):
  + Khi dữ liệu container lớn, ta nên dùng volume để lưu dữ liệu thay vì writable layer
  + Nhanh hơn writable layer

+ [CMD](https://docs.docker.com/engine/reference/builder/#cmd):
  + Lệnh mặc định khi start container

+ [ENTRYPOINT](https://docs.docker.com/engine/reference/builder/#entrypoint):
  + Khi chạy docker run, docker exec ... container sẽ lấy ENTRYPOINT + CMD rồi chạy.
  + Ví dụ:
    + image ubuntu có ENTRYPOINT là /bin/bash -c
    + Nếu ta sử dụng CMD là bash => lệnh chạy thực sự sẽ là `/bin/bash -c bash`
  + Những lệnh này thường hay dùng trong docker-compose thay vì viết trong dockerfile


## 4. Build Docker images

+ [Syntax](https://docs.docker.com/engine/reference/commandline/build/):

    ```
    docker build -t <image_name> .
    ```

   Ví dụ:

    ```
    docker build -t ubuntu_nginx .
    ```
    ![](https://user-images.githubusercontent.com/49421807/60146675-ab1b0580-97f4-11e9-96b7-23616cc3ce4d.png)
+ Test:

    + List image:
        ```
        docker images
        ```
    + Detail
        ```
        docker inspect ubuntu_nginx
        ```
        ![](https://user-images.githubusercontent.com/18675907/59087541-23498600-892f-11e9-9caf-9b56da755f51.png)

## 5. Cơ chế build image qua layers

+ Layers `xếp chồng`
  + Mô hình cha con
  + Sinh sau đẻ muộn hơn thì là layer con
  + Kế thừa từ layer cha
    ![](https://user-images.githubusercontent.com/49421807/60145335-eb2bb980-97ef-11e9-9cd0-6edcf5b838df.png)
  + Áp dụng cho cả pull và build image.
    ![](https://user-images.githubusercontent.com/49421807/60146505-0c8ea480-97f4-11e9-993a-e887f04aa499.png)

+ Cơ chế `layers caching`
  + Tái sử dụng
  + Giảm không gian đĩa
  + Giảm thời gian build image
    ![](https://user-images.githubusercontent.com/49421807/60155332-04466180-9814-11e9-9adf-dd60e300904e.png)

+ Dangling image.
  + Layers mà không được layer cha trỏ tới nữa do Dockerfile đã thay đổi.
  + [List images](https://docs.docker.com/engine/reference/commandline/images/)
    ```
    docker images
    ```
    ![](https://user-images.githubusercontent.com/49421807/60166419-5a72cf00-982b-11e9-9be0-3f8c10f2f007.png)

    > The default docker images will show all top level images, their repository and tags, and their size.
  + List dangling images
    ```
    docker images --filter "dangling=true"
    ```
    ![](https://user-images.githubusercontent.com/49421807/60166469-74acad00-982b-11e9-9423-2f2a9eee5e69.png)
  + Delete dangling images
    ```
    docker rmi $(docker images -f "dangling=true" -q)
    ```
    ![](https://user-images.githubusercontent.com/49421807/60166547-95750280-982b-11e9-803a-a7303f2da3dc.png)
  + Kết quả:
    ![](https://user-images.githubusercontent.com/49421807/60166598-a887d280-982b-11e9-8e08-86ce91917b5c.png)

  + Remove all image
    ```
    sudo docker rmi -f $(sudo docker images)
    ```
## 6. Docker Hub

  + [Registration](https://hub.docker.com/signup)
  + Create repository
  ![](https://user-images.githubusercontent.com/49421807/60173439-e4756480-9838-11e9-8a8e-22a74aa25152.png)
  + Config
  ![](https://user-images.githubusercontent.com/49421807/60174079-729e1a80-983a-11e9-9241-de380a3343d7.png)
  + Result
  ![](https://user-images.githubusercontent.com/49421807/60174168-ab3df400-983a-11e9-9a43-b75246cb6bb1.png)

## 7. Practice

+ [Best practices for writing Dockerfiles
](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)
