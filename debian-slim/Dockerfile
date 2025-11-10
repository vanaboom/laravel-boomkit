# Debian (Bookworm) SLIM: no MSSQL (and no sqlsrv/pdo_sqlsrv), no Node/NPM, no ionCube
FROM php:8.4-cli-bookworm

# ---------- OCI Labels ----------
LABEL org.opencontainers.image.title="laravel-boomkit (debian-slim)"
LABEL org.opencontainers.image.description="Slim Laravel base on PHP 8.4 (Debian Bookworm) with common PHP/PECL runtime extensions (no MSSQL, no Node/NPM, no ionCube)."
LABEL org.opencontainers.image.source="https://github.com/vanaboom/laravel-boomkit"
LABEL org.opencontainers.image.licenses="MIT"
LABEL maintainer="Majid Hadavand"

# ---------- Build args / envs ----------
ARG CUSTOM_APT_MIRROR="http://deb.debian.org/debian"
ARG CUSTOM_APT_SECURITY_MIRROR="http://deb.debian.org/debian-security"
ARG TIMEZONE="UTC"

ENV TZ=${TIMEZONE} \
    DEBIAN_FRONTEND=noninteractive

# ---------- Single RUN: build + enable + cleanup build-deps (keep runtimes) ----------
RUN set -eux; \
    # (0) Optional: switch APT mirrors
    if [ -f /etc/apt/sources.list.d/debian.sources ]; then \
      sed -i "s|^URIs: http://deb.debian.org/debian$|URIs: ${CUSTOM_APT_MIRROR}|g" /etc/apt/sources.list.d/debian.sources; \
      sed -i "s|^URIs: http://deb.debian.org/debian-security$|URIs: ${CUSTOM_APT_SECURITY_MIRROR}|g" /etc/apt/sources.list.d/debian.sources; \
    fi; \
    \
    # (1) Install build deps + runtime deps (no Node, no MSSQL)
    apt-get update; \
    apt-get install -y --no-install-recommends \
      $PHPIZE_DEPS \
      libfreetype6-dev libgmp-dev libicu-dev libjpeg-dev libonig-dev \
      libldap2-dev libldb-dev libpng-dev libpq-dev libsqlite3-dev \
      libyaml-dev libzip-dev zlib1g-dev; \
    apt-get install -y --no-install-recommends \
      ca-certificates curl git gnupg iputils-ping \
      libonig5 libgd3 libgmp10 libicu72 libldap-2.5-0 libpq5 \
      libyaml-0-2 libzip4 openssh-client openssl sqlite3 supervisor \
      tzdata unzip zip dnsutils; \
    \
    # (2) Build PHP core extensions
    docker-php-ext-configure gd --with-freetype --with-jpeg; \
    docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu; \
    docker-php-ext-install -j"$(nproc)" \
      bcmath gd gmp intl ldap mbstring opcache pcntl pdo pdo_pgsql \
      pdo_sqlite pgsql sockets zip; \
    \
    # (3) PECL extensions (NO sqlsrv/pdo_sqlsrv here)
    pecl channel-update pecl.php.net; \
    pecl install -o -f igbinary redis apcu yaml || true; \
    docker-php-ext-enable igbinary redis apcu yaml || true; \
    rm -rf /tmp/pear ~/.pearrc || true; \
    \
    # (4) php.ini & timezone
    cp /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini; \
    sed -i "s@;date.timezone =@date.timezone = ${TZ}@" /usr/local/etc/php/php.ini; \
    echo "${TZ}" > /etc/timezone; \
    ln -snf /usr/share/zoneinfo/${TZ} /etc/localtime; \
    \
    # (5) Cleanup build deps in SAME layer + apt caches
    apt-get purge -y --auto-remove \
      $PHPIZE_DEPS \
      libfreetype6-dev libgmp-dev libicu-dev libjpeg-dev libonig-dev \
      libldap2-dev libldb-dev libpng-dev libpq-dev libsqlite3-dev \
      libyaml-dev libzip-dev zlib1g-dev; \
    apt-get clean; rm -rf /var/lib/apt/lists/*; \
    \
    # (6) SSH known_hosts
    mkdir -p /root/.ssh && chmod 700 /root/.ssh; \
    ssh-keyscan -t rsa github.com gitlab.com >> /root/.ssh/known_hosts

WORKDIR /app
