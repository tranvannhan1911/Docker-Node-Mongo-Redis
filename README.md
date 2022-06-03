# Docker-Node-Mongo-Redis

1. Cài đặt
    1. Dockerfile
        
        ```bash
        FROM node:13-alpine
        
        WORKDIR /app
        
        COPY . .
        
        RUN npm install
        
        CMD ["npm", "run", "dev"]
        ```
        
    2. Vì sử dụng docker mongo để lưu trữ dữ liệu, dữ liệu sẽ biến mất sau khi container bị ngừng. Vì vậy cần mount volumes các thư mục chưa dữ liệu.
        - Tạo thư mục .docker/data/db để chứa dữ liệu mongodb
        - Tạo thư mục .docker/data/redis để chứa dữ liệu redis
    3. docker-compose.yml
        
        ```bash
        version: "3.4"
        
        services:
          app:
            image: learning-docker/docker-node-mongo-redis:v1
            volumes:
              - ./:/app
            environment:
              - DB_HOST=${DB_HOST}
              - DB_NAME=${DB_NAME}
              - REDIS_HOST=${REDIS_HOST}
              - REDIS_PORT=${REDIS_PORT}
              - PORT=${PORT}
            ports:
              - "${PORT}:${PORT}"
            restart: unless-stopped
            depends_on:
              - redis
              - db
        
          db:
            image: mongo
            volumes:
              - .docker/data/db:/data/db
            restart: unless-stopped
        
          redis:
            image: redis:5-alpine
            volumes:
              - .docker/data/redis:/data
            restart: unless-stopped
        ```
        
        - depends_on: khi chạy service app, sẽ chạy service db và redis trước
        - Khi chạy **docker-compose up app** thì cả service db và redis cũng sẽ chạy
        - Khi chạy **docker-compose stop** thì service app sẽ tắt trước service db và redis
        - Lưu ý: mặc dù các service db và redis chạy trước khi service app chạy thì cũng không đảm bảo mongo sẵn sàng cho việc kết nối.
    4. Chạy npm install để tạo thư mục node_modules
        - Nếu môi trường ngoài không code npm thì cần chạy một container tạm thời
        
        ```bash
        $ docker run --rm -v $(pwd):/app -w /app node:13-alpine npm install
        ```
        
    5. Khởi chạy
        
        ```bash
        $ docker-compose up
        ```
        
2. Tài liệu tham khảo
    1. [https://viblo.asia/p/dockerize-project-nodejs-mongodb-redis-passport-4P856NXW5Y3](https://viblo.asia/p/dockerize-project-nodejs-mongodb-redis-passport-4P856NXW5Y3)