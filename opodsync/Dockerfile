# Minimal PHP runner (no s6 needed)
# FROM php:8.2-cli-alpine3.20
FROM php:8.2-cli-alpine

# Tools for download/extract and TLS validation + privilege dropper
RUN apk add --no-cache curl tar ca-certificates tzdata su-exec && update-ca-certificates

# Narrow webroot
WORKDIR /var/www/html

# Build args (optional integrity pinning)
ARG OPODSYNC_URL="https://fossil.kd2.org/opodsync/tarball/opodsync.tar.gz"
ARG OPODSYNC_SHA256=""

# Fetch opodsync (robust), locate 'server' folder, copy (incl. dotfiles),
# and present ROOT/data via symlink to /config/opodsync
RUN set -eux; \
    curl -fsSL "$OPODSYNC_URL" -o /tmp/opodsync.tar.gz; \
    if [ -n "$OPODSYNC_SHA256" ]; then \
      echo "$OPODSYNC_SHA256  /tmp/opodsync.tar.gz" | sha256sum -c -; \
    fi; \
    mkdir -p /tmp/opod; \
    tar -xzf /tmp/opodsync.tar.gz -C /tmp/opod; \
    SRVDIR="$(find /tmp/opod -type d -name server -print -quit)"; \
    if [ -z "$SRVDIR" ]; then echo "ERROR: 'server' directory not found in tarball"; exit 1; fi; \
    cp -a "$SRVDIR"/. /var/www/html/; \
    rm -rf /tmp/opod /tmp/opodsync.tar.gz; \
    \
    # Use HA's addon_config mapping as real storage, but keep ROOT/data expected by oPodSync
    mkdir -p /config/opodsync; \
    rm -rf /var/www/html/data; \
    ln -s /config/opodsync /var/www/html/data

# Minimal PHP hardening (adjust as needed)
# If oPodSync requires remote URL streams, set allow_url_fopen=On.
RUN { \
    echo "expose_php=Off"; \
    echo "display_errors=Off"; \
    echo "log_errors=On"; \
    echo "memory_limit=128M"; \
    echo "post_max_size=16M"; \
    echo "upload_max_filesize=16M"; \
    echo "file_uploads=On"; \
    # If oPodSync needs remote URL streams -> set this to On
    echo "allow_url_fopen=Off"; \
    # Allow webroot, symlinked ROOT/data, and the addon_config paths
    echo "open_basedir=/var/www/html:/var/www/html/data:/tmp:/config:/config/opodsync"; \
    echo "disable_functions=exec,passthru,shell_exec,system,proc_open,popen,expect_popen"; \
    echo "session.save_path=/config/opodsync/sessions"; \
  } > /usr/local/etc/php/conf.d/50-hardening.ini

# Unprivileged user + initial ownership (image layer)
RUN addgroup -S app && adduser -S -G app -h /var/www/html app && \
    chown -R app:app /var/www/html /config/opodsync

# HTTP port (TLS termination handled by Cloudflare/your proxy)
EXPOSE 8080

# First-start directory/ownership fix (Supervisor maps /config for addon_config)
# If config.local.php is missing in the runtime-mounted folder, create it once,
# then drop privileges and run the built-in PHP server with router.
CMD ["sh","-c","\
  set -e; \
  mkdir -p /config/opodsync/sessions /config/opodsync/cache; \
  if [ ! -f /config/opodsync/config.local.php ]; then \
    printf '%s\n' \
'<?php' \
'namespace OPodSync;' \
\"const BASE_URL = 'http://127.0.0.1:8080/';\" \
\"const DATA_ROOT = __DIR__;\" \
\"const CACHE_ROOT = DATA_ROOT . '/cache';\" \
\"const DB_FILE = DATA_ROOT . '/data.sqlite';\" \
    > /config/opodsync/config.local.php; \
  fi; \
  chown -R app:app /config/opodsync /var/www/html 2>/dev/null || true; \
  exec su-exec app php -S 0.0.0.0:8080 -t /var/www/html /var/www/html/index.php \
"]
