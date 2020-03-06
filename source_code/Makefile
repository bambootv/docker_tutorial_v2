build: down
	# Current user can not access some files, use `ls -la` to check
	sudo docker-compose build

up:
	docker-compose up -d mysql
	docker-compose up -d spring
	# docker-compose up -d redis
	# docker-compose up -d worker

dev:
	docker-compose run --rm -p 3000:3000 app rails s

migrate:
	docker exec -it spring /bin/bash -c "rails db:create && rails db:migrate"

migrate_dev:
	docker-compose run --rm -p 3000:3000 app /bin/bash -c "rails db:create db:migrate && rails s"

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
