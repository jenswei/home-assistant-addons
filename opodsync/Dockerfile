# Minimal PHP runner (no s6 needed)
FROM php:8.2-cli-alpine
# Pin the base image to improve build reproducibility
#FROM php:8.2-cli-alpine3.20

# Tools for downloading, extracting and proper TLS validation
RUN apk add --no-cache curl tar ca-certificates tzdata su-exec && update-ca-certificates

# Narrow webroot (serving from "/" is risky)
WORKDIR /var/www/html

# Build args for source and optional integrity pinning
ARG OPODSYNC_URL="https://fossil.kd2.org/opodsync/tarball/opodsync.tar.gz"
ARG OPODSYNC_SHA256=""

# Fetch opodsync (robust), locate the "server" directory regardless of tarball layout,
# copy its content (including dotfiles), then symlink data -> /data
RUN set -eux; \
    curl -fsSL "$OPODSYNC_URL" -o /tmp/opodsync.tar.gz; \
    if [ -n "$OPODSYNC_SHA256" ]; then \
      echo "$OPODSYNC_SHA256  /tmp/opodsync.tar.gz" | sha256sum -c -; \
    fi; \
    mkdir -p /tmp/opod; \
    tar -xzf /tmp/opodsync.tar.gz -C /tmp/opod; \
    SRVDIR="$(find /tmp/opod -type d -name server -print -quit)"; \
    if [ -z "$SRVDIR" ]; then echo "ERROR: 'server' directory not found in tarball"; exit 1; fi; \
    # Use "/." so dotfiles (e.g., .htaccess) are copied as well
    cp -a "$SRVDIR"/. /var/www/html/; \
    rm -rf /tmp/opod /tmp/opodsync.tar.gz; \
    # Ensure /data exists (will be bind-mounted by the HA supervisor at runtime)
    mkdir -p /data; \
    # Replace potential in-tree "data" folder with a symlink to /data
    rm -rf /var/www/html/data; \
    ln -s /data /var/www/html/data

# Minimal PHP hardening (adjust to your app's needs)
# - If opodsync needs remote URL streams, change allow_url_fopen=On later.
RUN { \
    echo "expose_php=Off"; \
    echo "display_errors=Off"; \
    echo "log_errors=On"; \
    echo "memory_limit=128M"; \
    echo "post_max_size=16M"; \
    echo "upload_max_filesize=16M"; \
    echo "file_uploads=On"; \
    echo "allow_url_fopen=Off"; \
    echo "open_basedir=/var/www/html:/tmp:/data"; \
    echo "disable_functions=exec,passthru,shell_exec,system,proc_open,popen,expect_popen"; \
    # Optional: store PHP sessions under /data if sessions are used
    echo "session.save_path=/data/sessions"; \
  } > /usr/local/etc/php/conf.d/50-hardening.ini

# Tiny health endpoint (handy for local checks and general monitoring)
#RUN printf "%s\n" "<?php echo 'OK';" > /var/www/html/health.php

# Create unprivileged runtime user
RUN addgroup -S app && adduser -S -G app -h /var/www/html app && \
    mkdir -p /data/sessions && chown -R app:app /var/www/html /data

# HTTP port (TLS termination is handled by Cloudflare/your proxy)
EXPOSE 8080

# First-start ownership fix:
# - Home Assistant mounts /data at runtime, which can override build-time ownership.
# - We chown once on first start (using a sentinel) and then drop privileges with su-exec.

#HEALTHCHECK --interval=30s --timeout=3s --start-period=10s --retries=3 \
#  CMD wget -qO- http://127.0.0.1:8080/health 2>/dev/null | grep -q OK || exit 1

CMD ["sh","-c", "\
  if [ ! -f /data/.perm_ok ]; then \
    chown -R app:app /data /var/www/html 2>/dev/null || true; \
    touch /data/.perm_ok; \
  fi; \
  exec su-exec app php -S 0.0.0.0:8080 -t /var/www/html /var/www/html/index.php \
"]
