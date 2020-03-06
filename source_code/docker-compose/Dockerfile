FROM ruby:2.6.3-alpine3.9

WORKDIR /usr/src/app

RUN apk add -u nodejs

COPY Gemfile* ./
RUN bundle

COPY . .

CMD ["ruby", "main.rb"]
