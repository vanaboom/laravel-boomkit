FROM php:8.4-cli-alpine3.20

# ---------- OCI Labels ----------
LABEL org.opencontainers.image.title="laravel-boomkit (alpine-slim)"
LABEL org.opencontainers.image.description="Production Slim Laravel base image on PHP 8.4 (Alpine) with common PHP/PECL runtime extensions (no MSSQL, no Node/NPM, no ionCube)."
LABEL org.opencontainers.image.source="https://github.com/vanaboom/laravel-boomkit"
LABEL org.opencontainers.image.licenses="MIT"
LABEL maintainer="Mohsen Jadidi"

# ---------- Build args / envs ----------
ARG CUSTOM_APK_MIRROR="https://dl-cdn.alpinelinux.org/alpine"
ARG TIMEZONE="UTC"

ENV TZ="${TIMEZONE}" \
    COMPOSER_ALLOW_SUPERUSER=1

# ---------- Single RUN: build + enable + cleanup build-deps (keep runtimes) ----------
RUN set -eux; \
    # (0) Optional: switch mirror
    if [ -f /etc/apk/repositories ]; then \
      sed -i "s|https://dl-cdn.alpinelinux.org/alpine|${CUSTOM_APK_MIRROR}|g" /etc/apk/repositories; \
    fi; \
    \
    # (1) Install build deps as a virtual group, and install runtime deps separately
    apk add --no-cache --virtual .build-deps \
      $PHPIZE_DEPS \
      freetype-dev gmp-dev icu-dev libjpeg-turbo-dev oniguruma-dev \
      openldap-dev ldb-dev libpng-dev postgresql-dev sqlite-dev \
      yaml-dev libzip-dev zlib-dev linux-headers; \
    apk add --no-cache \
      ca-certificates curl git gnupg iputils oniguruma gd gmp icu-libs \
      openldap postgresql-libs yaml libzip openssh-client openssl sqlite \
      supervisor tzdata unzip zip bind-tools bash; \
    \
    # (2) Build PHP core extensions
    docker-php-ext-configure gd --with-freetype --with-jpeg; \
    docker-php-ext-configure ldap; \
    docker-php-ext-install -j"$(nproc)" \
      bcmath gd gmp intl ldap mbstring opcache pcntl pdo pdo_pgsql \
      pdo_sqlite pgsql sockets zip; \
    \
    # (3) Build PECL extensions
    pecl channel-update pecl.php.net; \
    pecl install -o -f igbinary; \
    pecl install -o -f redis; \
    pecl install -o -f apcu; \
    pecl install -o -f yaml; \
    docker-php-ext-enable igbinary redis apcu yaml || true; \
    rm -rf /tmp/pear ~/.pearrc || true; \
    \
    # (4) Remove ONLY build deps in the SAME layer (PHP .so remain intact)
    apk del --no-cache .build-deps; \
    rm -rf /var/cache/apk/*; \
    \
    # (5) php.ini & timezone
    cp /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini; \
    sed -i "s@;date.timezone =@date.timezone = ${TZ}@" /usr/local/etc/php/php.ini; \
    echo "${TZ}" > /etc/timezone; \
    ln -snf "/usr/share/zoneinfo/${TZ}" /etc/localtime; \
    \
    # (6) Composer (binary) and final cache cleanup
    curl -fsSL https://getcomposer.org/installer -o /tmp/composer-setup.php; \
    php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer; \
    rm -f /tmp/composer-setup.php; \
    composer --version; \
    composer clear-cache || true; \
    \
    # (7) SSH known_hosts and final cleanup
    mkdir -p /root/.ssh && chmod 700 /root/.ssh; \
    ssh-keyscan -t rsa github.com gitlab.com >> /root/.ssh/known_hosts; \
    rm -rf /tmp/* /usr/share/man /usr/share/doc || true

# ---------- Defaults ----------
WORKDIR /app
