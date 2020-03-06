![](https://www.threatstack.com/wp-content/uploads/2017/06/docker-cloud-twitter-card.png)


## 1. Bài toán

**Bài toán 1:**

  + project_1 của bạn `đã được cài` và đang hoạt động ổn định trên máy.

  + project_2 `chuẩn bị` được cài đặt.

  -> Việc `cài đặt thêm` các software, package dễ ảnh hưởng tới những chương trình đang chạy.

**Bài toán 2:**

  + Cài đặt project vào máy, yêu cầu:
    + Ruby,
    + Raisl,
    + Mysql,
    + MongoDB,
    + Redis,
    + Sidekiq ...
  cùng với version, package đi kèm ...

-> Việc cài đặt, cấu hình `phức tạp`.

## 2. Phương pháp ảo hóa

  + Tạo ra `máy ảo` && đặt dự án `chạy bên trong` máy ảo đó.
  ![](https://user-images.githubusercontent.com/49421807/59422554-c6504300-8dfa-11e9-8969-712e254d9952.png)
  + Các công nghệ máy ảo ra đời:
    + Hypervisor (Virtualbox, Vmware, BlueStack..v..v)
    + Containerization (**Docker**, incognito_1, incognito_2, ...)

## 3. Công nghệ Hypervisor

+ `Hypervisor` là công nghệ ảo hóa ở `tầng Hardware` (phần cứng)
+ Tư tưởng:
  + Cấp 4 GB RAM, 10 GB disk, % CPU ...
  + Dùng tài nguyên trên để cài đặt OS (Operation System )
  ![](https://user-images.githubusercontent.com/49421807/59420465-23e29080-8df7-11e9-9188-390eda275f98.png)
+ Ưu điểm:
  + Giải quyết được bài toán 1.
  + Dễ học, dễ sử dụng.
+ Nhược điểm:
  + Lãng phí tài nguyên nếu không sử dụng hết.
  + Thời gian khởi động lớn.
  + Vẫn chưa giải quyết được bài toán 2.

## 4. Công nghệ Containerization

  + `Containerization` là công nghệ ảo hóa ở tầng OS(hệ điều hành).
  + Tư tưởng:
    + Cài đặt `máy ảo` trên `máy thật`.
    + `Dùng chung` tài nguyên với máy thật (share kernel với nhau).
    + Cần bao nhiêu, cấp bấy nhiêu, dùng xong thì giải phóng.
    ![](https://user-images.githubusercontent.com/49421807/59420970-05c96000-8df8-11e9-829f-58bda1ca642c.png)
  + Nền tảng:
    + Container:
      + Một container (chiếc thùng) là một máy ảo.
      + `Tính đóng gói:`
        + Chứa các phần mềm cần thiết để dự án có thể chạy được.
      + `Tính nhất quán`:
        + Không sợ sự sai khác về mặt môi trường.
      + `Tính cô lập`:
        + Không bị ảnh hưởng từ bên ngoài,
        + Không làm ảnh hưởng ra bên ngoài.
      + `Tính mở rộng`:
        + Dễ dàng cài cắm thêm những môi trường cần thiết.
      + `Hiệu năng cao`:
        + Có thể  khởi động rất nhanh, chứa các môi trường cần thiết chỉ bằng 1 câu lệnh.
        + Vì sao ? (ảo hóa ở tầng OS)
      + `Dễ quản lý`:
        + Tính đóng gói.
        + Có các trạng thái run, started, stopped, moved và deleted.

  + Ưu điểm:
    + Nhanh.
    + Giải quyết cả 2 bài toán.
    + Giảm chi phí cơ sở hạ tầng.
  + Khuyết điểm:
    + Khó học, khó áp dụng hơn.
    + Chỉ tạo ra được máy ảo mà máy host hỗ trợ.
      + Máy thật cài Linux thì chỉ có thể dựng lên được máy ảo thuộc họ Linux (Ubuntu, CentOS,... )
      + Máy thật cài Windows thì không thể dựng lên máy ảo Linux.
      + Vì sao ?
    + Bảo mật:
      + Nếu tầng Kernel bị hack hoặc bị hư hại thì sẽ ảnh hưởng đến tất cả các container.
      + [Mối lo về bảo mật](http://genk.vn/xu-huong-su-dung-container-trong-lap-trinh-dang-bien-no-thanh-mo-vang-cho-cac-hacker-20170803121608714.chn)
  + Ông lớn:
    + Google sử dụng rộng rãi.
  ![](https://user-images.githubusercontent.com/49421807/59021343-d3f34f00-8875-11e9-92e4-4d86fbb3116a.png)

## 5. Giới thiệu về Docker

### 5.1 Docker Inc

  + Các `ông lớn` dùng phần mềm gì để ứng dụng công nghệ `containerization`:
    + `Google`, ...
      + Private source code.
    + [rkt](https://github.com/rkt)
    + [Docker Inc](https://www.docker.com/company) [(Wikipedia)](https://en.wikipedia.org/wiki/Docker,_Inc.)
      + [2013, Go language, Linux OS, open source](https://github.com/docker)
      + 2015: > 25,600 star (top 20 project with star), 1100 contributors
      + 2016: nhóm Docker, Cisco, Google, Huawei, IBM, Microsoft, và Red Hat.
      + Docker is the leader in the containerization market,...
    [<img src="https://user-images.githubusercontent.com/49421807/59482172-fa2a7780-8e91-11e9-9324-216e85b2c878.png">](https://www.docker.com/why-docker)
      + [Products](https://www.docker.com/products)
        + Offerings
          + [Docker Community Edition](https://docs.docker.com/install/) (We are using)
          + [Docker Enterprise](https://docs.docker.com/ee/supported-platforms/)
          + [Docker Hub](https://hub.docker.com)
        + Technologies
          ![](https://user-images.githubusercontent.com/49421807/59489623-e3454e80-8eac-11e9-80fe-7a3ed9102f27.png)
      + [Customers](https://www.docker.com/customers)
        + [Visa:](https://www.docker.com/customers/visa) Visa Achieves a 10x Increase in Scalability with Docker Enterprise
        + [PayPal:](https://www.docker.com/customers/paypal) PayPal Manages 200,000 Containers in the Cloud to Speed Transactions
        + [Circle CI](https://circleci.com)
        ![](https://camo.githubusercontent.com/e93591edba8c651f251d3b24717097d77d705c34/68747470733a2f2f7669626c6f2e617369612f75706c6f6164732f35623162623362342d663362382d343138332d383533612d6431316630663461343939372e706e67)

  + `Docker` là:
    + Nền tảng phần mềm,
    + Xây dựng môi trường ảo hóa,
    + Trên công nghệ containerization.

  + `Docker` cung cấp:
    + cách để triển khai ứng dụng,
    + được cách ly an toàn trong container,
    + đóng gói với tất cả `dependencies` and `libraries`.

  **Docker mà trước nay, mọi người đang sử dụng cho dự án nhỏ của mình là Docker Community Edition - một phần trong hệ sinh thái Docker.**

### 5.2 Cài đặt

Tham khảo tại [trang chủ](https://docs.docker.com/install/linux/docker-ce/ubuntu)

### 5.3 Các khái niệm

[Docker Documentation](https://docs.docker.com)

![](https://camo.githubusercontent.com/957fbc8b45fc596089690cb9186100224b270e97/68747470733a2f2f696d616765732e7669626c6f2e617369612f37353164373531322d633965372d343461352d626535362d3662316666393039366164662e706e67)

+ **Basic:**
  + [Docker Image:](https://docs.docker.com/engine/reference/commandline/images/)
    + Khung xương định hình cho container.
    + Snapshot tại một thời điểm của container.
    + Liên tưởng tới file ghost windows.
    + Phong cách OOP:
      + Docker image -> class.
      + Docker container -> instance của class đó.

  + [Dockerfile:](https://docs.docker.com/engine/reference/builder/)
    + Một file dạng text.
    + Không có đuôi. Tại sao ?
    + Tập hợp các lệnh.
    + Hướng dẫn Docker tạo image.

  + [Docker Hub:](https://docs.docker.com/docker-hub/)
    + Dùng để lưu trữ các docker image.
    + Có thể  pull hoặc push image.

  + [Docker Compose:](https://docs.docker.com/compose/overview/)
    + Chuyên biệt hóa, mỗi container làm một nhiệm vụ riêng.
      + Container nginx,
      + Container mysql,
      + Container redis ...
    + Khai báo và điều phối hoạt động các container.

+ **Advanced:**

  + [Docker engine:](https://docs.docker.com/engine) Khi người ta nói đến Docker thì có thể hiểu đơn giản là họ đang nhắc đến Docker Engine. Docker Engine cung cấp đầy đủ các function để cho người sử dụng có thể làm việc được với Docker image và  Docker container. Để gọi đến các function này, người sử dụng có thể dùng gọi đến REST API của Docker, hoặc thực hiện thông qua giao diện conmand line (CLI – cấp cao hơn REST API). Ví dụ: `docker run <image>` ... và nhiều câu lệnh khác.

  + [Docker Toolbox:](https://docs.docker.com/toolbox/overview/) đây là tool của Docker, được sử dụng  trên MacOS và Window

  + [Docker Trusted Registry (DTR):](https://docs.docker.com/ee/dtr/) Nếu công ti của bạn cần bảo mật và chỉ muốn chia sẻ image ở nội bộ, hoặc chỉ cho những ai bạn muốn chia sẻ. Thì DTR như một Docker Hub phiên bản private. (đây là phiên bản mất phí, bạn sẽ trả tiền, và sẽ có người của công ti support cho bạn trong việc cài đặt và bảo trì hệ thống)

  + [Docker Machine:](https://docs.docker.com/machine/overview/) là công cụ giúp bạn cài đặt Docker Engine lên môi trường máy ảo một cách tự động, đồng thời cũng quản lí các host này với các câu lệnh của docker-machine (cái này hỗ trợ cả trên host thật, cloud…).

  + [Docker Swarm:](https://docs.docker.com/engine/swarm/) is native clustering for Docker (theo docs.docker.com). Hiểu nôm na nó như một thằng trung gian giữa bạn và các Docker Host, nó tập trung các Docker Host về thành một mối. Và khi bạn làm việc thì bạn chỉ cần làm việc với cái “virtual host” mà Swarm tạo ra là ổn.

  + [Docker Registry:](https://docs.docker.com/registry/) Chức năng tương tự như Docker Trusted Registry. Kho chứa images phiên bản open-sources (không mất phí).

  + [Docker Cloud:](https://docs.docker.com/v17.12/docker-cloud/getting-started/intro_cloud/) là hệ thống Paas cho phép bạn dễ dàng triển khai các app của mình lên môi trường cloud.

  + [Docker Daemon:](https://docs.docker.com/config/daemon/) Docker daemon chạy trên các máy host. Người dùng sẽ không tương tác trực tiếp với các daemon, mà thông qua Docker Client.
  ![](https://user-images.githubusercontent.com/49421807/59673053-d4cb9f80-91ea-11e9-8aca-4d27e7461c92.png)

    Docker sử dụng kiến trúc client-server. Docker client sẽ liên lạc với các Docker daemon, các Docker daemon sẽ thực hiện các tác vụ build, run và distribuing các Docker container.  Cả Docker client và Docker daemon có thể chạy trên cùng 1 máy, hoặc có thể kết nối theo kiểu Docker client điều khiển các docker daemon giao tiếp với nhau thông qua socket hoặc RESTful API.

  + [Docker Client:](https://docs.docker.com/engine/reference/commandline/cli/) Là giao diện người dùng của Docker, nó cung cấp cho người dùng giao diện dòng lệnh và thực hiện phản hồi với các Docker Daemon.
