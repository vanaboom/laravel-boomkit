FROM php:8.4-cli-bookworm

# ---------- OCI Labels ----------
LABEL org.opencontainers.image.title="laravel-boomkit (base)"
LABEL org.opencontainers.image.description="Production-ready Laravel base image on PHP 8.4 (Debian Bookworm) with common PHP/PECL extensions, MSSQL tools, Node.js, Ioncube, and Laravel Echo Server."
LABEL org.opencontainers.image.source="https://github.com/vanaboom/laravel-boomkit"
LABEL org.opencontainers.image.licenses="MIT"
LABEL maintainer="Majid Hadavand"

# ---------- Build args / envs ----------
ARG USE_CUSTOM_APT_MIRROR=false
ARG CUSTOM_APT_MIRROR="http://deb.debian.org/debian"
ARG CUSTOM_APT_SECURITY_MIRROR="http://deb.debian.org/debian-security"
ARG NPM_REGISTRY="https://registry.npmjs.org"

ENV TZ=Asia/Tehran \
    DEBIAN_FRONTEND=noninteractive \
    PHP_VERSION_SHORT=8.4

# ---------- (Optional) set custom apt mirror ----------
RUN if [ "$USE_CUSTOM_APT_MIRROR" = "true" ]; then \
      sed -i "s|^URIs: http://deb.debian.org/debian$|URIs: ${CUSTOM_APT_MIRROR}|g" /etc/apt/sources.list.d/debian.sources && \
      sed -i "s|^URIs: http://deb.debian.org/debian-security$|URIs: ${CUSTOM_APT_SECURITY_MIRROR}|g" /etc/apt/sources.list.d/debian.sources ; \
    fi

# ---------- Build deps ----------
ENV BUILD_DEPS=" \
    libfreetype6-dev \
    libgmp-dev \
    libicu-dev \
    libjpeg-dev \
    libonig-dev \
    libldap2-dev \
    libldb-dev \
    libpng-dev \
    libpq-dev \
    libsqlite3-dev \
    libyaml-dev \
    libzip-dev \
    unixodbc-dev \
    zlib1g-dev \
"

# ---------- PHP core & PECL ext lists ----------
ENV PHP_EXTENSIONS=" \
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
"
ENV PECL_EXTENSIONS=" \
    apcu \
    redis \
    sqlsrv \
    pdo_sqlsrv \
    yaml \
"

# ---------- Runtime deps ----------
ENV RUN_DEPS=" \
    ca-certificates \
    curl \
    git \
    gnupg \
    iputils-ping \
    libonig5 \
    libgd3 \
    libgmp10 \
    libicu72 \
    libldap-2.5-0 \
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
"

# ---------- Install & build ----------
RUN set -eux; \
    apt-get update && apt-get upgrade -y; \
    apt-get install -y --no-install-recommends $BUILD_DEPS $PHPIZE_DEPS; \
    docker-php-ext-configure gd --with-freetype --with-jpeg; \
    docker-php-ext-configure ldap --with-libdir=lib/x86_64-linux-gnu; \
    docker-php-ext-install -j"$(nproc)" $PHP_EXTENSIONS; \
    pecl channel-update pecl.php.net; \
    pecl install -o -f igbinary; \
    pecl install -o -f redis; \
    pecl install -o -f apcu yaml; \
    pecl install -o -f sqlsrv pdo_sqlsrv || echo "Skipping sqlsrv/pdo_sqlsrv on PHP ${PHP_VERSION_SHORT}"; \
    docker-php-ext-enable igbinary redis apcu yaml || true; \
    docker-php-ext-enable sqlsrv pdo_sqlsrv || true; \
    apt-get purge -y --auto-remove $BUILD_DEPS; \
    apt-get clean; rm -rf /var/lib/apt/lists/*


# ---------- Runtime packages ----------
RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends $RUN_DEPS; \
    apt-get clean; rm -rf /var/lib/apt/lists/*

# ---------- MSSQL tools ----------
ENV ACCEPT_EULA=Y
RUN set -eux; \
    curl -sSL -O "https://packages.microsoft.com/config/debian/$(. /etc/os-release && echo ${VERSION_ID%%.*})/packages-microsoft-prod.deb"; \
    dpkg -i packages-microsoft-prod.deb; rm -f packages-microsoft-prod.deb; \
    apt-get update; \
    apt-get install -y msodbcsql18 mssql-tools18 unixodbc-dev libgssapi-krb5-2; \
    apt-get clean; rm -rf /var/lib/apt/lists/*; \
    echo 'export PATH="$PATH:/opt/mssql-tools18/bin"' >> /etc/bash.bashrc
ENV PATH="${PATH}:/opt/mssql-tools18/bin"

# ---------- Ioncube ----------
RUN set -eux; \
    curl -fsSL "https://downloads.ioncube.com/loader_downloads/ioncube_loaders_lin_x86-64.tar.gz" -o /tmp/ioncube.tar.gz; \
    tar -xzf /tmp/ioncube.tar.gz -C /tmp; \
    cp "/tmp/ioncube/ioncube_loader_lin_${PHP_VERSION_SHORT}.so" "$(php -r 'echo ini_get("extension_dir");')/"; \
    echo "zend_extension=$(php -r 'echo ini_get("extension_dir");')/ioncube_loader_lin_${PHP_VERSION_SHORT}.so" > /usr/local/etc/php/conf.d/00-ioncube.ini; \
    rm -rf /tmp/ioncube /tmp/ioncube.tar.gz

# ---------- PHP config & SSH known_hosts ----------
RUN set -eux; \
    mkdir -p /root/.ssh; \
    cp /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini; \
    sed -i 's@;date.timezone =@date.timezone = Asia/Tehran@' /usr/local/etc/php/php.ini;

# ---------- Composer ----------
ENV COMPOSER_ALLOW_SUPERUSER=1
RUN set -eux; \
    curl -fsSL https://getcomposer.org/installer -o /tmp/composer-setup.php; \
    php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer; \
    rm -f /tmp/composer-setup.php; \
    composer --version

# ---------- Laravel Echo Server ----------
RUN set -eux; \
    npm config set registry "${NPM_REGISTRY}"; \
    npm install -g laravel-echo-server; \
    npm cache clean --force

# ---------- Defaults ----------
WORKDIR /app
