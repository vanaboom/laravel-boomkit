# Debian (Bookworm) with MSSQL, Node/NPM, Echo Server, ionCube
FROM php:8.4-cli-bookworm

# ---------- OCI Labels ----------
LABEL org.opencontainers.image.title="laravel-boomkit (debian-tools)"
LABEL org.opencontainers.image.description="Laravel base on PHP 8.4 (Debian Bookworm) with common PHP/PECL extensions, MSSQL tools, Node.js, ionCube, and Laravel Echo Server."
LABEL org.opencontainers.image.source="https://github.com/vanaboom/laravel-boomkit"
LABEL org.opencontainers.image.licenses="MIT"
LABEL maintainer="Majid Hadavand"

# ---------- Build args / envs ----------
ARG APT_MIRROR_URL="http://deb.debian.org/debian"
ARG APT_SECURITY_MIRROR_URL="http://deb.debian.org/debian-security"
ARG NPM_MIRROR_URL="https://registry.npmjs.org/"
ARG TIMEZONE="UTC"

ENV TZ=${TIMEZONE} \
    DEBIAN_FRONTEND=noninteractive \
    PHP_VERSION_SHORT=8.4

# ---------- Single RUN: build + enable + cleanup build-deps (keep runtimes) ----------
RUN set -eux; \
    if [ -f /etc/apt/sources.list.d/debian.sources ]; then \
      sed -i "s|^URIs: http://deb.debian.org/debian$|URIs: ${APT_MIRROR_URL}|g" /etc/apt/sources.list.d/debian.sources; \
    fi; \
    apt-get update && apt-get upgrade -y; \
    apt-get install -y --no-install-recommends \
      $PHPIZE_DEPS  \
      libfreetype6-dev \
      libgmp-dev \
      libicu-dev \
      libjpeg-dev \
      libldap2-dev \
      libldb-dev \
      libonig-dev \
      libpng-dev \
      libpq-dev \
      libsqlite3-dev \
      libyaml-dev \
      libzip-dev \
      unixodbc-dev \
      zlib1g-dev \
    ; \
    apt-get install -y --no-install-recommends \
      curl \
      dnsutils \
      git \
      gnupg \
      iputils-ping \
      libgd3 \
      libgmp10 \
      libicu72 \
      libldap-2.5-0 \
      libonig5 \
      libpq5 \
      libyaml-0-2 \
      libzip4 \
      netcat-openbsd \
      nodejs \
      npm \
      openssh-client \
      openssl \
      sqlite3 \
      supervisor \
      tzdata \
      unixodbc \
      unzip \
      zip \
    ; \
    docker-php-ext-configure gd --with-freetype --with-jpeg; \
    docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu; \
    docker-php-ext-install -j"$(nproc)" \
      bcmath gd gmp intl ldap mbstring opcache pcntl pdo pdo_pgsql \
      pdo_sqlite pgsql sockets zip; \
    pecl channel-update pecl.php.net; \
    pecl install -o -f igbinary redis apcu yaml || true; \
    pecl install -o -f sqlsrv pdo_sqlsrv || true; \
    docker-php-ext-enable igbinary redis apcu yaml || true; \
    docker-php-ext-enable sqlsrv pdo_sqlsrv || true; \
    rm -rf /tmp/pear ~/.pearrc || true; \
    curl -sSL -O "https://packages.microsoft.com/config/debian/$(. /etc/os-release && echo ${VERSION_ID%%.*})/packages-microsoft-prod.deb"; \
    dpkg -i packages-microsoft-prod.deb; rm -f packages-microsoft-prod.deb; \
    apt-get update; \
    echo "msodbcsql18 msodbcsql/ACCEPT_EULA boolean true" | debconf-set-selections; \
    echo "mssql-tools18 mssql-tools/ACCEPT_EULA boolean true" | debconf-set-selections; \
    ACCEPT_EULA=Y DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
      msodbcsql18 mssql-tools18 libgssapi-krb5-2; \
    curl -fsSL "https://downloads.ioncube.com/loader_downloads/ioncube_loaders_lin_x86-64.tar.gz" -o /tmp/ioncube.tar.gz || true; \
    if [ -s /tmp/ioncube.tar.gz ]; then \
      tar -xzf /tmp/ioncube.tar.gz -C /tmp; \
      PHP_EXT_DIR=$(php -r 'echo ini_get("extension_dir");'); \
      cp "/tmp/ioncube/ioncube_loader_lin_${PHP_VERSION_SHORT}.so" "$PHP_EXT_DIR/" 2>/dev/null || true; \
      echo "zend_extension=$PHP_EXT_DIR/ioncube_loader_lin_${PHP_VERSION_SHORT}.so" > /usr/local/etc/php/conf.d/00-ioncube.ini || true; \
    fi; \
    rm -rf /tmp/ioncube /tmp/ioncube.tar.gz; \
    cp /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini; \
    sed -i "s@;date.timezone =@date.timezone = ${TZ}@" /usr/local/etc/php/php.ini; \
    echo "${TZ}" > /etc/timezone; \
    ln -snf /usr/share/zoneinfo/${TZ} /etc/localtime; \
    curl -fsSL https://getcomposer.org/installer -o /tmp/composer-setup.php; \
    php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer; \
    rm -f /tmp/composer-setup.php; \
    composer --version; \
    npm config set registry "${NPM_MIRROR_URL}"; \
    npm i -g laravel-echo-server; \
    npm cache clean --force; \
    apt-get purge -y --auto-remove \
      $PHPIZE_DEPS \
      libfreetype6-dev libgmp-dev libicu-dev libjpeg-dev libonig-dev \
      libldap2-dev libldb-dev libpng-dev libpq-dev libsqlite3-dev \
      libyaml-dev libzip-dev unixodbc-dev zlib1g-dev; \
    apt-get clean; rm -rf /var/lib/apt/lists/*; \
    mkdir -p /root/.ssh && chmod 700 /root/.ssh; \
    ssh-keyscan -t rsa github.com gitlab.com >> /root/.ssh/known_hosts
