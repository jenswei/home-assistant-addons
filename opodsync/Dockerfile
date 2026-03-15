# Minimal PHP runner (no s6 needed)
# FROM php:8.2-cli-alpine3.20
FROM php:8.2-cli-alpine

# Tools for download/extract and proper TLS validation
RUN apk add --no-cache curl tar ca-certificates tzdata su-exec && update-ca-certificates

# Narrow webroot (serving from "/" is risky)
WORKDIR /var/www/html

# Build args (optional: enable integrity pinning with OPODSYNC_SHA256)
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
    # Ensure /data exists; HA supervisor will bind-mount this at runtime
    mkdir -p /data; \
    # Replace any in-tree "data" folder with a symlink to /data
    rm -rf /var/www/html/data; \
    ln -s /data /var/www/html/data

# Minimal PHP hardening (adjust to your app’s needs)
# If opodsync requires remote URL streams, set allow_url_fopen=On later.
RUN { \
    echo "expose_php=Off"; \
    echo "display_errors=Off"; \
    echo "log_errors=On"; \
    echo "memory_limit=128M"; \
    echo "post_max_size=16M"; \
    echo "upload_max_filesize=16M"; \
    echo "file_uploads=On"; \
    # If opodsync needs remote URL streams -> set this to On
    echo "allow_url_fopen=Off"; \
    echo "open_basedir=/var/www/html:/tmp:/data"; \
    echo "disable_functions=exec,passthru,shell_exec,system,proc_open,popen,expect_popen"; \
  } > /usr/local/etc/php/conf.d/50-hardening.ini

# Unprivileged user + initial ownership for image layer
RUN addgroup -S app && adduser -S -G app -h /var/www/html app && \
    chown -R app:app /var/www/html /data

# HTTP port (TLS termination handled by Cloudflare/your proxy)
EXPOSE 8080

# First-start ownership fix:
# - HA mounts /data at runtime, which can override build-time ownership.
# - Chown once at container start, then drop privileges with su-exec.
CMD ["sh","-c","chown -R app:app /data /var/www/html 2>/dev/null || true; exec su-exec app php -S 0.0.0.0:8080 -t /var/www/html /var/www/html/index.php"]
