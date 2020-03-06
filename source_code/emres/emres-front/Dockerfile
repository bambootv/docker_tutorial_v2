# Stage 1 to build the application
FROM node:8.11.3 as builder

WORKDIR /usr/src/app
ENV PATH /usr/src/app/node_modules/.bin:$PATH

COPY package.json yarn.lock ./
RUN yarn install
COPY . ./
COPY ./.env.docker.staging ./.env.local
RUN yarn build

# Stage 2 to make nginx run that build
FROM nginx:1.15.1-alpine
COPY nginx_emres-front.conf /etc/nginx/conf.d/default.conf
COPY --from=builder /usr/src/app/build /usr/share/nginx/html
EXPOSE 80
