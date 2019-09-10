```
version: '3'
services:

  ### Nginx container #########################################

  nginx:
    container_name: nginx
    image: nginx:alpine
    ports:
      - "${HTTP_PORT}:80"
      - "${HTTPS_PORT}:443"
      - "8080:8080"
    volumes:
      - ./project:/var/www/html/:rw
      - ./work/components/nginx/config/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./work/components/nginx/config/conf.d:/etc/nginx/conf.d:ro
    restart: always
    privileged: true
    networks:
      - net-php
  ### PHP container #########################################
  php:
    container_name: php
    build:
      context: ./build/php
      args:
        TIME_ZONE: ${GLOBAL_TIME_ZONE}
        CHANGE_SOURCE: ${GLOBAL_CHANGE_SOURCE}
    ports:
      - "9501:9501"
    volumes:
      - ./project:/var/www/html:rw
      - ./work/components/php/config/php.ini:/usr/local/etc/php/php.ini:ro
      - ./work/components/php/config/php-fpm.conf:/usr/local/etc/php-fpm.d/www.conf:rw
    restart: always
    privileged: true
    networks:
      - net-php
      - net-mysql
      - net-redis
      - net-mongo
      - net-rabbitmq
  mysql:
    container_name: mysql
    image: mysql:8.0.17
    command: mysqld --default-authentication-plugin=mysql_native_password  
    ports:
      - "${MYSQL_PORT}:3306"
    volumes:
      - ./work/components/mysql/config/mysql.cnf:/etc/mysql/conf.d/mysql.cnf:ro
    restart: always
    privileged: true
    environment:
      MYSQL_ROOT_HOST: '%' 
      MYSQL_ROOT_PASSWORD: ${MYSQL_PASSWORD}
    networks:
      - net-mysql
  redis:
    container_name: redis
    image: redis:latest
    command: redis-server /usr/local/etc/redis/redis.conf
    ports:
      - "${REDIS_PORT}:6379"  
    volumes:
      - ./work/components/redis/config/redis.conf:/usr/local/etc/redis/redis.conf:ro
    restart: always
    privileged: true
    networks:
      - net-redis
  mongo:
    container_name: mongo
    image: mongo
    ports:
      - "${MONGO_PORT}:27017"  
    volumes:
      - ./work/components/mongo/config/mongod.conf:/etc/mongo/mongod.conf:ro
      - ./work/components/mongo/config/mongo-init.js:/docker-entrypoint-initdb.d/mongo-init.js:ro
    restart: always
    privileged: true
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: zhangzhen
    networks:
      - net-mongo
  rabbitmq:
    container_name: rabbitmq
    image: rabbitmq:management-alpine
    ports:
      - "${RABBITMQ_PORT}:5672"
      - "${RABBITMQ_MANAGEMENT_PORT}:15672"  
    volumes:
      - ./work/components/rabbitmq/config/rabbitmq.conf:/etc/rabbitmq/rabbitmq.conf:ro  
    networks:
      - net-rabbitmq
networks:
  net-php:
  net-mysql:
  net-redis:
  net-mongo:
  net-rabbitmq:
```