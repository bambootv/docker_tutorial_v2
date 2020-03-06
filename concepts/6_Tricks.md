![](https://user-images.githubusercontent.com/49421807/59478352-34404d00-8e83-11e9-8a86-1fa835819c5a.png)
## 1. Tối ưu image

+ Người ta luôn cố gắng giới hạn kích thước của image xuống thấp nhất có thể. Để làm gì thì chắc các bạn cũng nắm được rồi.

+ Mỗi step của Dockerfile sẽ tạo ra 1 layer, từ đó tăng kích thước image => ta phải giảm số lượng layer đi, và cũng phải đảm bảo kích thước của nó không quá lớn. Để làm được điều này, hãy chú ý:

  + (1) Gộp những câu lệnh như COPY, RUN vào nếu được.
  + (2) Bỏ đi những dependencies không cần thiết khi chạy container
  + (3) Giảm dung lượng image -> image alpine

### 1.1 Gộp câu lệnh

+ Mỗi lệnh trong dockerfile sẽ tạo ra 1 layer mới, nó sẽ làm tăng kích thước image sau khi build.

+ Một số lệnh có khả năng cache

  + `yarn install` sử dụng `yarn.lock` và `package.json`
  + `bundle install` sử dụng `Gemfile.lock` và `Gemfile`

+ Ta nên copy những file này vào image trước khi những lệnh này, nó sẽ sử dụng cache thay vi cài đặt lại từ đầu, build 1 layer mới (và những layer sau cũng tạch nốt)

  ```Dockerfile
  COPY Gemfile* ./
  RUN bundle
  ```

+ Nên gộp những lệnh RUN lại với nhau:

  + VD khi cài nodejs + yarn:

    ```Dockerfile
    RUN curl -sL https://deb.nodesource.com/setup_10.x | bash - && \
      curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
      echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
      apt-get update && \
      apt-get install -y build-essential mysql-client libv8-dev nodejs yarn
    ```

+ Gộp lệnh COPY nếu có thể

  ```Dockerfile
  COPY Gemfile* ./
  ```

  thay vì

  ```Dockerfile
  COPY Gemfile .
  COPY Gemfile.lock .
  ```

### 1.2 Bỏ dependencies không cần thiết

###### Vấn đề

+ Trong Rails project, ta cần cài yarn để chạy `rake assets:precompile`
  + Ta buộc phải cài thêm yarn, rồi chạy yarn install => sinh ra node_modules với dung lượng khổng lồ.
  + Tuy nhiên sau đó khi up server thì ta chả cần đống đời thừa này làm gì nữa => lãng phí.

+ Hay những ngôn ngữ biên dịch như C/C++, Java, Golang, ta cần build rồi mới chạy được.

-> Tuy lúc dev thì phải cài một đống lib/môi trường, nhưng sau khi build ta chạy 1 file cái là được hay sao?

###### Giải quyết

+ Builder pattern

    + Có 1 pattern được cộng đồng Docker dùng khá lâu, đó là `builder pattern`.

    + Ta sẽ phải tạo 2 Dockerfile, 1 Dockerfile sẽ thực hiện nhiệm vụ build project, sau đó ta chạy container đó, và copy build output của nó ra ngoài host, và dùng output đó để build với Dockerfile còn lại.

      `Dockerfile.build`

      ```Dockerfile
      FROM golang:1.7.3
      WORKDIR /go/src/github.com/alexellis/href-counter/
      COPY app.go .
      RUN go get -d -v golang.org/x/net/html \
        && CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .
      ```

      `Dockerfile`

      ```Dockerfile
      FROM alpine:latest
      RUN apk --no-cache add ca-certificates
      WORKDIR /root/
      COPY app .
      CMD ["./app"]
      ```

      `build.sh`

      ```bash
      #!/bin/sh
      echo Building alexellis2/href-counter:build

      docker build --build-arg https_proxy=$https_proxy --build-arg http_proxy=$http_proxy \
          -t alexellis2/href-counter:build . -f Dockerfile.build

      docker container create --name extract alexellis2/href-counter:build
      docker container cp extract:/go/src/github.com/alexellis/href-counter/app ./app
      docker container rm -f extract

      echo Building alexellis2/href-counter:latest

      docker build --no-cache -t alexellis2/href-counter:latest .
      rm ./app
      ```

      Tuy nhiên khá vất vả khi phải tạo ra 2 Dockerfile, lại còn thêm quả `build.sh` kia nữa. Maintain cũng chết (dead).

+ Multi-stage builds

  + Từ Docker 17.05 trở đi, Docker đã cho ra mắt **Multi-stage builds** để cứu rỗi linh hồn của những con chiên lạc lối.

  + Không còn cần nhiều Dockerfile nữa, và cũng tạm biệt luôn cái `build.sh` kia đi.

    `Dockerfile`

    ```Dockerfile
    # builder
    FROM golang:1.7.3 as builder
    WORKDIR /go/src/github.com/alexellis/href-counter/
    RUN go get -d -v golang.org/x/net/html
    COPY app.go    .
    RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o app .

    # main image
    FROM alpine:latest
    RUN apk --no-cache add ca-certificates
    WORKDIR /root/
    COPY --from=builder /go/src/github.com/alexellis/href-counter/app .
    CMD ["./app"]
    ```

  + Khi build thì ta chỉ cần

    ```shell
    docker build -t alexellis2/href-counter:latest .
    ```

  + Magic ở đâu?

    ```Dockerfile
    FROM golang:1.7.3 as builder
    ...
    COPY --from=builder /go/src/github.com/alexellis/href-counter/app .
    ```

  + Chính nó, **FROM xyz as builder** và **COPY --from=builder** (builder chỉ là example, bạn ghi tên mình vào đó cũng được)

  + Trong khi build image chính của ta Docker đã build thêm một image nữa, chính là phần builder ta đã khai báo.

  + Sau khi build, ta có thể dùng COPY với tham số `--from=stage_name` để copy bất cứ thứ gì trong image trên. Không dừng lại ở đó, cậu lệnh này còn cho phép ta copy file từ image (đã build) ở ngoài, thậm chí là remote image.

    ```Dockerfile
    COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
    ```

  + Do ta chỉ copy một phần nhỏ của image builder, nên tất nhiên là chỉ những thứ ta copy mới tính vào kích cỡ image của ta, những thứ đời thừa khác không liên quan. Từ đó, ta đã giảm được một lượng kích thước đáng kể.

  + Khi Dockerfile của ta có nhiều stage, nếu chỉ muốn build 1 stage nào đó, hãy dùng thêm tham số `--target` khi build:

    ```
    docker build --target builder -t alexellis2/href-counter:latest .
    ```

  + Khi chạy câu lệnh trên, Docker sẽ chỉ build builder stage mà thôi, còn mặc định nó sẽ chạy hết tất cả stage.

-> Những ví dụ trên đều có ở trang chủ Docker, có thể xem thêm tại [đây](<https://docs.docker.com/develop/develop-images/multistage-build/>)

###### Ứng dụng khác của multi-stage builds

  + Khi ta phải làm việc trong một ứng dụng mà có repo FE/BE riêng rẽ, cả 2 đều có Dockerfile, docker-compose. Nhưng chúng quá nát, outdated, hay là nó là file của môi trường production, ... khiến ta không thể dùng được. Nếu xóa đi thì ta lại phải checkout lại mỗi lần gửi pull request (facepalm)

    => **Hãy tạo 1 wrapper folder, và viết Dockerfile của riêng mình**

    ```txt
    project
    |-- project-front
        |-- Dockerfile
    |-- project-server
        |-- Dockerfile
        |-- docker-compose.yml
    |-- .env
    |-- Dockerfile (new)
    |-- docker-compose.yml (new)
    ```

  + Nhưng mà ta có tận 2 project, vậy phải viết 2 Dockerfile sao? Nếu tinh ý, ta có thể thấy có thể tận dụng `--target` của  Docker Multi-stage builds

    `Dockerfile`

    ```Dockerfile
    # Build BE
    FROM ruby:latest as server
    RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs vim-tiny
    RUN mkdir -p /usr/src/app
    WORKDIR /usr/src/app
    COPY project-server/Gemfile .
    COPY project-server/Gemfile.lock .
    RUN bundle install
    EXPOSE 3000

    # Build FE
    FROM node:latest as front
    RUN mkdir -p /usr/src/app
    WORKDIR /usr/src/app
    COPY project-front .
    EXPOSE 3000
    ```

    `docker-compose.yml`

    ```yml
    version: '3.7'
    services:
      front:
        build:
          context: .
          target: front
        command: ["yarn", "start"]
        ports:
          - 3001:3000
        volumes:
          - ./project-front:/usr/src/app
        restart: unless-stopped

      server:
        build:
          context: .
          target: server
        command: ["rails", "s"]
        tty: true
        stdin_open: true
        ports:
          - 3000:3000
        volumes:
          - ./project-server:/usr/src/app
        restart: unless-stopped
        depends_on:
          - db

      db:
        image: postgres
        env_file:
          - ./.env
        volumes:
          - ~/.tmp/app/project/db:/var/lib/postgresql/data
        restart: unless-stopped
    ```

## 2. Tăng tính bảo mật

+ Chỉ nên viết những lệnh cần thiết để khởi tạo môi trường, một số lệnh không liên quan hoặc có thể bị replace bởi volumes thì không nên viết trong Dockerfile

  * Nếu chạy `yarn install` trong quá trình build, mà sau đó ta lại mount thư mục hiện tại vào trong container, thì node_modules sau khi chạy yarn install sẽ bị replace ngay và luôn => mất => vô dụng

  * Nên tránh cách viết như sau vì nó chả liên qua gì đến quá trình build cả.

    Dưới đây là Dockerfile trong 1 dự án của 1 công ty có doanh thu nghìn tỷ yên.

    ```shell
    RUN if [ "$LOCAL_MACHINE" != "true" ]; then bundle exec rake db:create && bundle exec rake db:migrate ; fi
    ADD package.json /package.json
    RUN yarn install
    ```

    Và can lộ lộ hết cả bí mật quốc gia

    ```Dockerfile
    ARG SECRET_KEY_BASE
    ARG MYSQL_DB_NAME
    ARG MYSQL_ROOT_PASSWORD

    ENV SECRET_KEY_BASE=$SECRET_KEY_BASE
    ENV MYSQL_DB_NAME=$MYSQL_DB_NAME
    ENV MYSQL_ROOT_PASSWORD=$MYSQL_ROOT_PASSWORD
    ```

  * Thay vì vậy hãy viết vào file shell ở ngoài, rồi sử dụng CMD, và tách riêng dockerfile cho dev/production ra.

    ```
    # Dockerfile
    CMD ["start.sh"]
    ```

    ```shell
    # start.sh
    yarn install
    rm -rf tmp/pids
    rails s
    ```

    Nếu viết như trên, khi container start, nó sẽ chạy `yarn install` sinh ra thư mục `node_modules`, sau đó sẽ được mount ra host, chứ không bị replace nữa.

* Nhớ sử dụng EXPOSE, mặc dù không viết thì nó vẫn chạy như thường.

## 3. CMD vs ENTRYPOINT

#### Mở đầu

+ Mỗi khi container được bật, nó đều cần có 1 lệnh nào đó để thực thi, làm nhiệm vụ của mình. `CMD` và `ENTRYPOINT` chính là những gì ta cần.

+ Cơ mà khi chúng ta viết Dockerfile hay docker-compose, ta chỉ thường chú ý đến command hay CMD, vậy ENTRYPOINT là cái khỉ mốc gì vậy?

#### Magic

+ Lấy 1 ví dụ đơn giản với image `ubuntu`

  `Dockerfile`

  ```Dockerfile
  FROM ubuntu
  CMD ["echo", "Hello world"]
  ```

+ Khi ta chạy image, nó sẽ in ra màn hình "Hello world". Đơn giản nó là chạy `echo "Hello world"`?

+ Thực ra là nó đã chạy `/bin/sh -c echo "Hello world"`. WTF Magic gì vậy?

+ Vì Docker mặc định cho ta `ENTRYPOINT ["/bin/sh", "-c"]`. Và câu lệnh cuối cùng mà container chạy là `command = ENTRYPOINT + CMD`, chứ không đơn thuần chỉ là `CMD`.

#### ENTRYPOINT VS CMD

* `ENTRYPOINT`:
  * Là command chạy mỗi khi container start
  * Required, mặc định là `/bin/sh -c`
* `CMD`:
  * Mảnh ghép còn lại của cuộc đời `ENTRYPOINT`, tuy nhiên có hay không cũng chả quan trọng, vì nếu chỉ 1 mình `ENTRYPOINT` là chạy được rồi thì không cần thêm `CMD` nữa.
  * Không có giá trị mặc định

Khi ta dùng `docker start` hay `docker run`, Docker cũng đều dùng `CMD` của ta để append vào `ENTRYPOINT`, sau đó mới chạy.

VD

```Dockerfile
FROM ubuntu
ENTRYPOINT ["/bin/cat"]
```

Khi ta chạy `docker run x /etc/passwd`, thực chất là ta đang chạy `/bin/cat /etc/passwd`, như vậy file `/etc/passwd` sẽ được in ra màn hình.

#### Style khi viết ENTRYPOINT và CMD

Có 2 phong cách viết `ENTRYPOINT`/`CMD`

* Shell form: `CMD rails s -b 0.0.0.0"`

* Exec form: `CMD ["rails", "s", "-b", "0.0.0.0"]`

Ta có thể dùng kiểu nào cũng được, tuy nhiên Docker khuyên ta nên dùng Exec form hơn.

Shell form invoke 1 câu lệnh shell, với đầy đủ tính năng của shell, ta có thể truy cập biến ENV, dùng &&, ...

Còn với exec form, Docker parse chuỗi của ta theo format JSON, vì vậy ta phải dùng mảng của các string (nhớ double quote), sau đó làm gì đó ma giáo, nhưng không chạy như 1 câu lệnh shell, vì vậy những tính năng của shell đều cuốn theo chiều gió hết. Muốn sử dụng, ta phải thêm lệnh shell vào `CMD ["sh", "-c", "echo $HOME"]` (tuy nhiên ta đã có default ENTRYPOINT là `/bin/sh -c` rồi nên chỉ khi bạn override nó mới cần lưu ý.)

Lưu ý nữa là khi dùng `ENTRYPOINT` theo shell form, `CMD` và command của `docker run` sẽ bị ignore hết.

Ngoài ra, nó còn có liên quan đến shell processing, signal processing, khá là hại não, mọi người có thể dựa vào những keyword này để tìm hiểu thêm.

## 4. So sánh LINK và DEPENDS_ON

Hiện tại `LINK` đã bị deprecated, không nên dùng nó nữa.

Trước đây, `LINK` dùng để liên kết các container, từ đó, container này có thể gọi đến container kia thông qua alias của nó (`NETWORK`). Ngoài ra nó còn 1 side-effect nữa là quy định trình tự start các container (`DEPENTS_ON`).

Dễ dàng thấy nó là `NETWORK + DEPENTS_ON`

Một note nhỏ là `depends_on` không chờ cho container khác *ready to work* mà nó chỉ chờ bật lên mà thôi, `depends_on` bị ignore khi dùng swarm mode.

## 5. Makefile

+ Phải công nhận là gõ những dòng docker-compose up -d ... nhiều như vậy thật là mệt, vậy thì hãy dùng Makefile

+ Thay vì phải gõ từng câu lệnh một

```
docker-compose up -d mysql redis worker
```

thì hãy viết vào trong [Makefile](https://github.com/longnv-0623/Div1_Docker_Course/blob/master/source_code/Makefile)

```
up:
   docker-compose up -d mysql redis worker

dev:
	docker-compose run --rm -p 3000:3000 app rails s
```

+ Khi đó, trên Terminal gõ `make up` để start các background containers, sau đó gõ `make dev` để start main container


+ Hơn nữa, khi có một member mới vào dự án và chưa rõ về Docker (có thể là member của team front end chẳng hạn), khi support setup project, bạn chỉ cần hướng dẫn.

  + Chạy make up để bật các tiến trình nền.

  + Chạy make dev để start project.

  + Chạy make test để test code trước khi gửi pull request.

+ Mà bản thân họ không cần phải có quá nhiều kiến thức về Docker, khá là tiện lợi phải không ^_^


## 6. Xóa dangling images

+ Dòng ---> Using cache xuất hiện mỗi khi bạn build image chính là tái sử dụng các layers. Những layers mà không được tái sử dụng nữa được gọi là dangling images, tạm dịch là những image lơ lửng -> nó không trỏ tới images nào cả.

+ Xóa dangling image

  ```
  sudo docker rmi -f $(docker images -f "dangling=true" -q)
  ```


## 7. For rails project

### 7.1 Ứng dụng trong Rails project (production)

  `Dockerfile`

  ```Dockerfile
  FROM ruby:2.5.3-alpine as base
  RUN apk --update add nodejs build-base tzdata postgresql-dev imagemagick
  ENV RAILS_ENV=production RACK_ENV=production RAILS_ROOT=/var/www/clvoz
  RUN mkdir -p $RAILS_ROOT
  WORKDIR $RAILS_ROOT
  COPY Gemfile* ./
  RUN bundle install --without development test
  COPY . .

  FROM base as builder
  ARG RAILS_MASTER_KEY
  RUN apk --update add yarn
  RUN rake assets:precompile RAILS_MASTER_KEY=${RAILS_MASTER_KEY}

  FROM base as appserver
  COPY config/puma.docker.production.rb config/puma.rb
  COPY --from=builder /var/www/clvoz/public/assets/* ./public/assets/
  EXPOSE 3000

  FROM nginx:1.15.6-alpine as webserver
  ENV RAILS_ROOT /var/www/clvoz
  RUN mkdir -p $RAILS_ROOT
  WORKDIR $RAILS_ROOT
  COPY deploy/clvoz.docker.conf /etc/nginx/conf.d
  COPY --from=builder /var/www/clvoz/public ./public/
  EXPOSE 80
  ```

  `docker-compose.yml`

  ```yml
  version: '3.7'
  services:
    portainer:
      image: portainer/portainer
      command: -H unix:///var/run/docker.sock --no-auth
      volumes:
        - /var/run/docker.sock:/var/run/docker.sock
        - /data/clvoz/production/portainer:/data
      restart: always

    nginx-production:
      image: moonlight8978/clvoz:nginx-production
      build:
        context: .
        target: webserver
        args:
          RAILS_MASTER_KEY: ${RAILS_MASTER_KEY}
      env_file:
        - .env
      restart: always
      ports:
        - 80:80
        - 443:443
      command: ['nginx', '-g', 'daemon off;']
      volumes:
        - /data/clvoz/production/nginx/log:/var/log/nginx
        - ./tmp/.htpasswd:/etc/auth/.htpasswd
        - ./deploy/clvoz.docker.conf:/etc/nginx/conf.d/clvoz.docker.conf
      depends_on:
        - server-production
        - portainer

    server-production:
      image: moonlight8978/clvoz:server-production
      build:
        context: .
        target: appserver
        args:
          RAILS_MASTER_KEY: ${RAILS_MASTER_KEY}
      env_file:
        - .env
      restart: always
      volumes:
        - /data/clvoz/production/rails/log:/var/www/clvoz/log
        - /data/clvoz/production/rails/storage:/var/www/clvoz/storage
      command: puma -C config/puma.rb
      depends_on:
        - db

    db:
      image: postgres:10.6-alpine
      env_file:
        - .env
      volumes:
        - /data/clvoz/production/postgresql/data:/var/lib/postgresql/data
      restart: always
  ```

+ Có thể nhận thấy, để build được assets, hay chạy 1 Rails app thì ta cần những step khá giống nhau (cài gem, system packages, ...), ta còn có thể sử dụng multi-stage builds để định nghĩa ra 1 base image nữa.

+ Builder của ta thực hiện nhiệm vụ chạy `rake assets:precompile`, nó sẽ cần cài yarn, đồng thời khi chạy câu lệnh trên còn sinh ra thêm 1 đống `node_modules` nữa, ta đã tiết kiệm được kha khá dung lượng của image production.


## Khi nào cần build lại image?

+ Khi hệ thống - nội tại image cần có sự thay đổi
  + cài gem mới (khi chạy bundle install)
  + cài thêm system dependencies
    + apt-get install

+ Khi mà hệ thống không phụ thuộc vào những thay đổi thì không cần rebuild
  + cài gem bằng bundle install --path vendor/bundle
  + yarn install

+ Ở môi trường dev:
  + Ta thường có config volumes: .:/path/to/app, vì vậy khi chạy những lệnh trên, nó sẽ trực tiếp mount luôn ra ngoài host, những lần chạy service sau đó, nó cứ lấy thư mục gem kia mount vào container và chạy ầm ầm thôi.

##### Note
  + Chúng ta có thể bỏ option `networks` trong file `docker-compose.yml` nếu chúng ta không cần config gì khác như `ip`, `subnet`,... (dùng luôn default network khi start container).
  + Nếu không có nhu cầu access vào database từ host machine thì cũng không cần bind `port` ra :smile: Cụ thể là ở trong service `mysql` chúng ta có thể bỏ options `ports`.
  + Một số file `docker-compose.yml` dùng `links` thay vì `depends_on` thì thay vì dùng short-hand syntax như:
    ```yaml
    web:
      links:
      - db
    ```
  + Chúng ta nên dùng `<tên service>:<service alias>` để dễ dàng cho việc maintain
    ```yaml
    web:
      links:
      - mysql:db
    ```
  + Khi làm việc với `cron jobs` chúng ta cần chú ý tới việc setting timezone cho container. Chúng ta có thể setting cho chúng thông qua `Dockerfile` hoặc biến env khi chạy container.

+ Để  speed up app thì chúng ta sẽ

  + Khởi động container `mysql`, `redis`, `worker` ngay từ đầu bằng việc chạy daemon:
    ``` bash
    docker-compose up -d mysql redis worker
    ```

  + Chạy rails app bằng lệnh sau:
    ```bash
    docker-compose run --rm --service-ports app
    ```

