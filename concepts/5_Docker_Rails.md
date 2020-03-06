## Ứng dụng Docker trong framework Ruby On Rails

![](https://user-images.githubusercontent.com/49421807/60308227-1ccc8e00-9972-11e9-8227-e9879d3e3a06.png)

### 1. Khởi tạo Rails project với Docker

+ Tài liệu tham khảo: 
  + [Quickstart: Compose and Rails](https://docs.docker.com/compose/rails)

+ Solution:
  + Dựng máy ảo,
  + Rails new trong máy ảo,
  + Mount các thư mục vừa được tạo ra máy host.

+ Các file cần tạo:

  **Note**: Bỏ dòng `COPY . /myapp` vì không cần thiết, mỗi lần build image sẽ rất mất thời gian.

  ![](https://user-images.githubusercontent.com/49421807/62334864-91f02d80-b4f3-11e9-8c16-5cb16d82bca7.png)

+ Source_code:

  + Tham khảo tại [đây](https://github.com/longnv-0623/Div1_Docker_Course/tree/master/source_code/2_init_rails_with_docker)

+ Note:
  + Mặc dù là hướng dẫn trên trang chủ nhưng chưa thực sự tối ưu.
  + Tham khảo phần 2 để tối ưu hơn.

### 2. Ứng dụng Docker cho Rails project

Tạo thêm các file sau (lưu ý tùy chỉnh phiên bản tương ứng)

+ docker/Dockerfile
    
    ```Dockerfile
    FROM ruby:2.5.3

    # Information about author
    LABEL author.name="HoanKy" \
      author.email="hoanki2212@gmail.com"

    # Install apt based dependencies required to run Rails as
    # well as RubyGems. As the Ruby image itself is based on a
    # Debian image, we use apt-get to install those.
    RUN apt-get update && \
      apt-get install -y nodejs nano vim

    # Set the timezone.
    ENV TZ=Asia/Ho_Chi_Minh
    RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

    # Configure the main working directory. This is the base
    # directory used in any further RUN, COPY, and ENTRYPOINT
    # commands.
    ENV APP_PATH /my_app
    WORKDIR $APP_PATH

    # Copy the Gemfile as well as the Gemfile.lock and install
    # the RubyGems. This is a separate step so the dependencies
    # will be cached unless changes to one of those two files
    # are made.
    COPY Gemfile Gemfile.lock $APP_PATH/
    RUN bundle install --without production --retry 2 \
      --jobs `expr $(cat /proc/cpuinfo | grep -c "cpu cores") - 1`

    # Add a script to be executed every time the container starts.
    COPY docker/entrypoint.sh /usr/bin/
    RUN chmod +x /usr/bin/entrypoint.sh
    ENTRYPOINT ["entrypoint.sh"]
    EXPOSE 3000

    # The main command to run when the container starts. Also
    # tell the Rails dev server to bind to all interfaces by
    # default.
    CMD ["rails", "server", "-b", "0.0.0.0"]
    ```

+ docker-compose.yml
  ```yml
  version: '3.5'
  services:
    mysql:
      image: mysql:5.7
      container_name: mysql
      restart: always
      environment:
        MYSQL_ROOT_PASSWORD: $MYSQL_ROOT_PASSWORD
      volumes:
        - ./docker/database:/var/lib/mysql
    app:
      container_name: app
      build:
        context: .
        dockerfile: docker/Dockerfile
      volumes:
        - .:/my_app
      ports:
        - "3000:3000"
      env_file: 
        - .env
      environment:
        DATABASE_HOST: $DATABASE_HOST
        DATABASE_USER_NAME: $DATABASE_USER_NAME
        DATABASE_PASSWORD: $DATABASE_PASSWORD
  ```

+ docker/entrypoint.sh

  ```sh
  #!/bin/bash
  set -e

  # Remove a potentially pre-existing server.pid for Rails.
  rm -f /my_app/tmp/pids/server.pid

  # Then exec the container's main process use CMD (what"s set as CMD in the Dockerfile).
  exec "$@"
  ```
+ .gitignore
  
  ```.gitignore
  # Ignore config env
  /.env

  # Ignore database backup folder
  /docker/database/
  ```

### 3. Tối ưu

[1. Add spring container](https://github.com/jonleighton/spring-docker-example)

+ Step 1: Add service to docker-compose.yml

  ```yml
  spring:
    container_name: spring
    build: .
    volumes:
      - .:/app
    restart: always
    command: bundle exec spring server
    # This ensures that the pid namespace is shared between the host
    # and the container. It's not necessary to be able to run spring
    # commands, but it is necessary for "spring status" and "spring stop"
    # to work properly.
    pid: host
  ```

+ Step 2: Add gem

  ```Gemfile
  group :development do
    gem 'rubocop', '~> 0.74.0', require: false
    gem 'spring-commands-rubocop'
  end
  
  group :development, :test do
    gem 'rspec-rails', '~> 3.8'
    gem 'spring-commands-rspec'
  end
  ```
+ Step 3: Config

  ```
  # config/spring_client.rb
  ENV["SPRING_SOCKET"] = "tmp/spring.sock"
  ```

+ Step 4: Create Makefile

  ```Makefile
  build: down
	# Current user can not access some files, use `ls -la` to check
	sudo docker-compose build

  up:
    docker-compose up -d mysql
    docker-compose up -d spring
    # docker-compose up -d redis
    # docker-compose up -d worker

  dev:
    docker-compose run --rm -p 3000:3000 app rails s -b 0.0.0.0

  migrate:
    docker exec -it spring /bin/bash -c "rails db:create && rails db:migrate"

  migrate_dev:
    docker-compose run --rm -p 3000:3000 app /bin/bash -c "rails db:create db:migrate && rails s -b 0.0.0.0"

  seed:
    docker exec -it spring /bin/bash -c "rails db:migrate:reset && rails db:seed"

  console:
    docker exec -it spring rails c

  routes:
    docker exec -it spring rails routes

  down:
    docker-compose down

  rspec:
    docker exec -it -e RAILS_ENV=test spring spring $(MAKECMDGOALS)

  rspecs:
    docker exec -it -e RAILS_ENV=test spring spring rspec

  rubocop:
    docker exec -it spring rubocop -a app spec lib config db

  tests:
    docker exec -it -e RAILS_ENV=test spring /bin/bash -c "spring rubocop -a app spec lib config db && spring rspec"

  spring:
    docker exec -it $(MAKECMDGOALS)
  ```

+ Step 5: Generate and test

  ```sh
  sudo make build
  make up
  bundle exec spring binstub --all # Run inside container
  rails generate rspec:install # Run inside container
  sudo make build
  make dev
  make rspecs
  make rubocop
  ```
