#### Documentaion
  + [Home page](https://docs.docker.com/compose/rails/)


#### Command
  + `docker-compose run web rails new . --force --no-deps --database=postgresql`
  + `sudo chown -R $USER:$USER .`
  + `docker-compose build`
  + Config `database.yml`
    ```
    default: &default
      adapter: postgresql
      encoding: unicode
    ```
    ->
    ```
    default: &default
      adapter: postgresql
      encoding: unicode
      host: db
      username: postgres
      password:
      pool: 5
    ```
  + `docker-compose run --rm web rails db:migrate:reset`
  + `docker-compose up`