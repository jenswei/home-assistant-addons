FROM php:7.4-apache

RUN apt-get update && \
    apt-get install -y sqlite3 libsqlite3-dev unzip curl && \
    docker-php-ext-install pdo pdo_sqlite

WORKDIR /var/www/html
RUN curl -L https://fossil.kd2.org/opodsync/tarball/opodsync.tar.gz | tar xz --strip-components=1

EXPOSE 8080
