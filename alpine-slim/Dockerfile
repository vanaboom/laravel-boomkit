FROM php:8.4-cli-alpine3.20

# ---------- OCI Labels ----------
LABEL org.opencontainers.image.title="laravel-boomkit (alpine-slim)"
LABEL org.opencontainers.image.description="Production Slim Laravel base image on PHP 8.4 (Alpine) with common PHP/PECL runtime extensions (no MSSQL, no Node/NPM, no ionCube)."
LABEL org.opencontainers.image.source="https://github.com/vanaboom/laravel-boomkit"
LABEL org.opencontainers.image.licenses="MIT"
LABEL maintainer="Mohsen Jadidi"

# ---------- Build args / envs ----------
ARG APK_MIRROR_URL="https://dl-cdn.alpinelinux.org/alpine"
ARG TIMEZONE="UTC"

ENV TZ="${TIMEZONE}" \
    COMPOSER_ALLOW_SUPERUSER=1

RUN set -eux; \
    if [ -f /etc/apk/repositories ]; then \
      sed -i "s|https://dl-cdn.alpinelinux.org/alpine|${APK_MIRROR_URL}|g" /etc/apk/repositories; \
    fi; \
    apk update && apk add --upgrade --no-cache --virtual .build-deps \
      $PHPIZE_DEPS \
      freetype-dev \
      gmp-dev \
      icu-dev \
      ldb-dev \
      libjpeg-turbo-dev \
      libldap \
      libpng-dev \
      libpq-dev \
      libzip-dev \
      linux-headers \
      oniguruma-dev \
      openldap-dev \
      postgresql-dev \
      sqlite-dev \
      tar \
      unixodbc-dev \
      yaml-dev \
      zlib-dev \
    ; \
    apk update && apk add --no-cache \
      7zip \
      bash \
      bind-tools \
      busybox-extras \
      coreutils \
      gd  \
      git  \
      gmp  \
      gnupg  \
      icu-libs \
      libgd \
      iputils  \
      libldap \
      libzip \
      libpq \
      libzip \
      netcat-openbsd \
      nodejs \
      npm \
      oniguruma  \
      openldap \
      openssh-client \
      openssl \
      postgresql-libs \
      sqlite \
      supervisor \
      tzdata \
      unixodbc \
      unzip \
      yaml \
      zip \
    ; \
    docker-php-ext-configure gd --with-freetype --with-jpeg; \
    docker-php-ext-configure ldap; \
    docker-php-ext-install -j"$(nproc)" \
      bcmath \
      gd \
      gmp \
      intl \
      ldap \
      mbstring \
      opcache \
      pcntl \
      pdo \
      pdo_pgsql \
      pdo_sqlite \
      pgsql \
      sockets \
      zip \
    ; \
    pecl channel-update pecl.php.net; \
    pecl install -o -f igbinary; \
    pecl install -o -f redis; \
    pecl install -o -f apcu; \
    pecl install -o -f yaml; \
    docker-php-ext-enable igbinary redis apcu yaml || true; \
    rm -rf /tmp/pear ~/.pearrc || true; \
    apk del --no-cache .build-deps; \
    rm -rf /var/cache/apk/*; \
    cp /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini; \
    sed -i "s@;date.timezone =@date.timezone = ${TZ}@" /usr/local/etc/php/php.ini; \
    echo "${TZ}" > /etc/timezone; \
    ln -snf "/usr/share/zoneinfo/${TZ}" /etc/localtime; \
    curl -fsSL https://getcomposer.org/installer -o /tmp/composer-setup.php; \
    php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer; \
    rm -f /tmp/composer-setup.php; \
    composer --version; \
    composer clear-cache || true; \
    mkdir -p /root/.ssh && chmod 700 /root/.ssh; \
    ssh-keyscan -t rsa github.com gitlab.com >> /root/.ssh/known_hosts; \
    rm -rf /tmp/* /usr/share/man /usr/share/doc || true
