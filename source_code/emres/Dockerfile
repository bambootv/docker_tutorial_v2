# Build BE
FROM ruby:latest as server
RUN apt-get update -qq && apt-get install -y build-essential libpq-dev nodejs vim-tiny
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
COPY emres-server/Gemfile .
COPY emres-server/Gemfile.lock .
RUN bundle install
EXPOSE 3000

# Build FE
FROM node:latest as front
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
COPY emres-front .
EXPOSE 3000
