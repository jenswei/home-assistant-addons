FROM php:8.1-cli

RUN apt-get update && \
    apt-get install -y sqlite3 libsqlite3-dev unzip curl && \
    docker-php-ext-install pdo pdo_sqlite

WORKDIR /var/www/server
RUN curl -L https://fossil.kd2.org/opodsync/tarball/opodsync.tar.gz | tar xz --strip-components=1

CMD ["php", "-S", "0.0.0.0:8080", "-t", "."]
EXPOSE 8080
