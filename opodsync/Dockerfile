# Basisimage: from ghcr.io/hassio-addons/base as mentioned in https://community.home-assistant.io/t/issue-with-addon-persistent-data-storage/819322/3
RG BUILD_FROM
FROM $BUILD_FROM as builder

# Installiere benötigte Pakete
RUN apk add --no-cache curl tar

# Setze Arbeitsverzeichnis
WORKDIR /var/www/server/

# Lade opodsync herunter und entpacke nur den server-Ordner
RUN mkdir -p /var/www/server/data && \
    curl -L https://fossil.kd2.org/opodsync/tarball/opodsync.tar.gz | \
    tar xz --strip-components=2 --wildcards '*/server/*'

# Port für den Webserver freigeben
EXPOSE 8080

VOLUME ["/data"]

# Starte den PHP-Webserver
CMD ["php", "-S", "0.0.0.0:8080", "-t", "/var/www/server", "/var/www/server/index.php"]
