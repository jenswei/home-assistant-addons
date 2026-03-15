# Minimaler PHP-Runner (kein s6 nötig)
#FROM php:8.2-cli-alpine3.20
FROM php:8.2-cli-alpine


# Tools für Download/Entpacken + TLS-Validierung
RUN apk add --no-cache curl tar ca-certificates tzdata su-exec && update-ca-certificates

# Webroot
WORKDIR /var/www/html

# Build-Argumente (optional: Integritätsprüfung aktivieren)
ARG OPODSYNC_URL="https://fossil.kd2.org/opodsync/tarball/opodsync.tar.gz"
ARG OPODSYNC_SHA256=""

# Opodsync holen (robust), server/ finden, data -> /data symlinken
RUN set -eux; \
    curl -fsSL "$OPODSYNC_URL" -o /tmp/opodsync.tar.gz; \
    if [ -n "$OPODSYNC_SHA256" ]; then \
      echo "$OPODSYNC_SHA256  /tmp/opodsync.tar.gz" | sha256sum -c -; \
    fi; \
    mkdir -p /tmp/opod; \
    tar -xzf /tmp/opodsync.tar.gz -C /tmp/opod; \
    SRVDIR="$(find /tmp/opod -type d -name server -print -quit)"; \
    if [ -z "$SRVDIR" ]; then echo "ERROR: 'server' directory not found in tarball"; exit 1; fi; \
    # Wichtig: '/.' kopiert auch .htaccess & andere Dotfiles
    cp -a "$SRVDIR"/. /var/www/html/; \
    rm -rf /tmp/opod /tmp/opodsync.tar.gz; \
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

# HTTP-Port
EXPOSE 8080

# Chown bei Start + PHP als 'app' laufen lassen
CMD ["sh","-c","chown -R app:app /data /var/www/html 2>/dev/null || true; exec su-exec app php -S 0.0.0.0:8080 -t /var/www/html /var/www/html/index.php"]
