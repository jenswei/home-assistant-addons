# Basisimage: Leichtgewichtiger PHP-Webserver mit curl und tar
FROM php:8.2-cli-alpine

# Installiere benötigte Pakete
RUN apk add --no-cache curl tar

# Setze Arbeitsverzeichnis
WORKDIR /var/www/opodsync/

# Lade opodsync herunter und entpacke nur den server-Ordner
RUN curl -L https://fossil.kd2.org/opodsync/tarball/opodsync.tar.gz | \
    tar xz --strip-components=2 --wildcards '*/server/*'

# Port für den Webserver freigeben
EXPOSE 8080

VOLUME ["/var/www/opodsync/server/data"]

# Starte den PHP-Webserver
CMD ["php", "-S", "0.0.0.0:8080", "-t", "/var/www/opodsync/server", "/var/www/opodsync/server/index.php"]
