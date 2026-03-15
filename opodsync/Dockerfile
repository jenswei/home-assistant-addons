# Minimaler PHP-Runner (kein s6 nötig)
#FROM php:8.2-cli-alpine3.20
FROM php:8.2-cli-alpine


# Tools für Download/Entpacken + TLS-Validierung
RUN apk add --no-cache curl tar ca-certificates tzdata && update-ca-certificates

# Webroot
WORKDIR /var/www/html

# Build-Argumente (optional: Integritätsprüfung aktivieren)
ARG OPODSYNC_URL="https://fossil.kd2.org/opodsync/tarball/opodsync.tar.gz"
ARG OPODSYNC_SHA256=""

# Opodsync holen (nur server/), danach data -> /data symlinken
RUN set -eux; \
    curl -fsSL "$OPODSYNC_URL" -o /tmp/opodsync.tar.gz; \
    if [ -n "$OPODSYNC_SHA256" ]; then \
      echo "$OPODSYNC_SHA256  /tmp/opodsync.tar.gz" | sha256sum -c -; \
    fi; \
    mkdir -p /tmp/opod; \
    tar -xzf /tmp/opodsync.tar.gz --strip-components=2 --wildcards '*/server/*' -C /tmp/opod; \
    cp -a /tmp/opod/* /var/www/html/; \
    rm -rf /tmp/opod /tmp/opodsync.tar.gz; \
    # sicherstellen, dass /data existiert und als data/ im Webroot gesehen wird
    mkdir -p /data; \
    rm -rf /var/www/html/data; \
    ln -s /data /var/www/html/data

# Minimal-Hardening für PHP (bei Bedarf anpassen)
RUN { \
    echo "expose_php=Off"; \
    echo "display_errors=Off"; \
    echo "log_errors=On"; \
    echo "memory_limit=128M"; \
    echo "post_max_size=16M"; \
    echo "upload_max_filesize=16M"; \
    echo "file_uploads=On"; \
    echo "allow_url_fopen=Off"; \ 
    # Falls Opodsync Remote-URL-Streams wirklich braucht -> auf On setzen
    echo "open_basedir=/var/www/html:/tmp:/data"; \
    echo "disable_functions=exec,passthru,shell_exec,system,proc_open,popen,expect_popen"; \
  } > /usr/local/etc/php/conf.d/50-hardening.ini

# Unprivilegierter User + Rechte
RUN addgroup -S app && adduser -S -G app -h /var/www/html app && \
    chown -R app:app /var/www/html /data
USER app

# HTTP-Port
EXPOSE 8080

# Start: PHP Built-in Server mit Webroot
CMD ["php","-S","0.0.0.0:8080","-t","/var/www/html","/var/www/html/index.php"]
