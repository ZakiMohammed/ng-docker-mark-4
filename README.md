# Brotli Enabled Dockerized Angular App | NgDocker - Mark 4

Check out the CodeOmelet blog post for this project.

Link: https://codeomelet.com/posts/brotli-dockerized-angular-app-with-nginx-ngdocker
___

Gzip project for Angular and Docker powered by Cirrus UI.

## Build Docker Image and Run Docker Container

```
# build image
docker build -t ng-docker:mark-4 .

# run container
docker run -p 3300:80 --name ng-docker-mark-4-container ng-docker:mark-4

# list images
docker image ls

# stop container
docker stop ng-docker-mark-4-container

# remove container
docker rm ng-docker-mark-4-container
```

## nginx\nginx.conf

```
server {
    brotli on;
    brotli_static on;

    listen 80;

    root /usr/share/nginx/html;

    location / {
        index  index.html index.htm;
        try_files $uri $uri/ /index.html;
    }
    
    error_page   500 502 503 504  /50x.html;
    
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

## Dockerfile

```
# Build container
FROM node:18-alpine AS builder
WORKDIR /app

# Make sure we got brotli
RUN apk update
RUN apk add --upgrade brotli

# NPM install and build
ADD package.json .
RUN npm install
ADD . .
RUN npm run build:prod

RUN cd /app/dist && find . -type f -exec brotli {} \;

# Actual runtime container
FROM alpine
RUN apk add brotli nginx nginx-mod-http-brotli

# Minimal config
COPY nginx/nginx.conf /etc/nginx/http.d/default.conf

# Actual data
COPY --from=builder /app/dist/ng-docker-mark-4 /usr/share/nginx/html
CMD ["nginx", "-g", "daemon off;"]
EXPOSE 80
```
