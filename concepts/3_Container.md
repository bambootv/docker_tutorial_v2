![](https://camo.githubusercontent.com/957fbc8b45fc596089690cb9186100224b270e97/68747470733a2f2f696d616765732e7669626c6f2e617369612f37353164373531322d633965372d343461352d626535362d3662316666393039366164662e706e67)

![](https://user-images.githubusercontent.com/49421807/60763883-68381800-a0a7-11e9-8e96-7fde320029e9.png)


- Run image (read-only) -> container.

## 1. Câu lệnh

+ [Syntax](https://docs.docker.com/engine/reference/commandline/run/)

> The docker run command first creates a writeable container layer over the specified image, and then starts it using the specified command

  ```shell
  docker run
    -v <forder_in_host>:<forder_in_container>
    -p <port_in_host>:<port_in_container>
    -it <image_name> /bin/bash
  ```

  Trong đó:

    -v : mount volume giữa máy thật và máy ảo.

    -p: Port từ máy thật trỏ tới port của máy ảo.

    -it: Chạy container và mở terminal bằng /bin/bash


  + Example:

    + Build hoặc pull image
      ```
      docker pull hoanky/ubuntu_nginx
      ```
    + Run
      ```
      docker run -p 9000:80 -it hoanky/ubuntu_nginx /bin/bash
      ```
      ![](https://user-images.githubusercontent.com/49421807/60284914-eb829c80-9936-11e9-8c04-cdc70708aab2.png)
      ![](https://camo.githubusercontent.com/53ccfe2c05911d26d0093800cea1b74eee7333ce/68747470733a2f2f696d616765732e7669626c6f2e617369612f38333531633333362d393130312d343334342d623632362d3937313933666434386563382e706e67)

    + Run with mount
      ```
      docker run -v ~/Div1_Docker_Course/source_code/dockerfile/web_root:/var/www/html -p 9000:80 -it hoanky/ubuntu_nginx /bin/bash
      ```

      Thay thế ```~/Div1_Docker_Course/source_code/dockerfile/web_root``` cho đúng với trên máy bạn nhé !

      ![](https://camo.githubusercontent.com/77244822b9ada2bc0b66b552c65a085075e00972/68747470733a2f2f696d616765732e7669626c6f2e617369612f30386435386566652d386434352d346335662d616138372d6135396237653936666638302e706e67)


- Kiểm tra

  ```shell
  docker ps
  docker ps -a
  ```

- Điều kiển

  ```shell
  docker stop container 		# dừng container lại
  docker start container 		# Hôm sau đến bật lại container đó
  docker kill container 		# Ngứa mắt kill luôn
  docker restart container 	# Khi sửa file config, cần restart lại container
  ```

- Docker commit

  + Image read-only, không thể thay đổi được
  + Tuy nhiên ta lại muốn lưu trạng thái của 1 container đang chạy lại, để từ đó tạo ra nhiều instance mới.
  + `docker-commit` giúp ta làm điều đó, nó sẽ tạo ra 1 image mới chứa toàn bộ dữ liệu của container lại thời điểm đó

  **Image là những snapshot, immuatable (bất biến, read-only) của container. Container là những running (hoặc stopped) instance của image**

  + [FYI](https://stackoverflow.com/a/23667302)
