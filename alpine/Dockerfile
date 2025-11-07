FROM php:8.4-cli-alpine3.20

# ---------- OCI Labels ----------
LABEL org.opencontainers.image.title="laravel-boomkit (alpine)"
LABEL org.opencontainers.image.description="Production-ready Laravel base image on PHP 8.4 (Alpine) with common PHP/PECL extensions, MSSQL tools, Node.js, Ioncube, and Laravel Echo Server."
LABEL org.opencontainers.image.source="https://github.com/vanaboom/laravel-boomkit"
LABEL org.opencontainers.image.licenses="MIT"
LABEL maintainer="Mohsen Jadidi"

# ---------- Build args / envs ----------
ARG CUSTOM_APK_MIRROR="https://dl-cdn.alpinelinux.org/alpine"
ARG NPM_REGISTRY="https://registry.npmjs.org"
ARG TIMEZONE="UTC"

ENV TZ="${TIMEZONE}" \
    COMPOSER_ALLOW_SUPERUSER=1 \
    PATH="/opt/mssql-tools18/bin:${PATH}"

# ---------- (Optional) set custom apk mirror ----------
RUN if [ -f /etc/apk/repositories ]; then \
        sed -i "s|https://dl-cdn.alpinelinux.org/alpine|${CUSTOM_APK_MIRROR}|g" /etc/apk/repositories; \
    fi

# ---------- Build deps ----------
ENV BUILD_DEPS=" \
    freetype-dev \
    gmp-dev \
    icu-dev \
    libjpeg-turbo-dev \
    oniguruma-dev \
    openldap-dev \
    ldb-dev \
    libpng-dev \
    postgresql-dev \
    sqlite-dev \
    yaml-dev \
    libzip-dev \
    unixodbc-dev \
    zlib-dev \
    linux-headers \
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
    iputils \
    oniguruma \
    gd \
    gmp \
    icu-libs \
    openldap \
    postgresql-libs \
    yaml \
    libzip \
    netcat-openbsd \
    nodejs \
    npm \
    openssh-client \
    openssl \
    sqlite \
    supervisor \
    tzdata \
    unixodbc \
    unzip \
    zip \
    bind-tools \
    bash \
"

# ---------- Install & build ----------
RUN apk update; \
    apk add --no-cache ${BUILD_DEPS} ${PHPIZE_DEPS}; \
    \
    docker-php-ext-configure gd --with-freetype --with-jpeg; \
    docker-php-ext-configure ldap; \
    \
    docker-php-ext-install -j"$(nproc)" ${PHP_EXTENSIONS}; \
    \
    pecl channel-update pecl.php.net; \
    pecl install -o -f igbinary; \
    pecl install -o -f redis; \
    pecl install -o -f apcu yaml; \
    pecl install -o -f sqlsrv pdo_sqlsrv || echo "Skipping sqlsrv/pdo_sqlsrv on this PHP/Alpine build"; \
    \
    docker-php-ext-enable igbinary redis apcu yaml || true; \
    docker-php-ext-enable sqlsrv pdo_sqlsrv || true; \
    \
    apk del --no-cache ${BUILD_DEPS} ${PHPIZE_DEPS}; \
    rm -rf /tmp/pear ~/.pearrc

# ---------- Runtime packages ----------
RUN set -eux; \
    apk update; \
    apk add --no-cache ${RUN_DEPS}

# ---------- MSSQL tools (Alpine) ----------
ENV ACCEPT_EULA=Y

RUN mkdir -p /opt/microsoft/msodbcsql18; \
    echo "${ACCEPT_EULA}" > /opt/microsoft/msodbcsql18/ACCEPT_EULA; \
    \
    cd /tmp; \
    curl -fsSL -O "https://download.microsoft.com/download/fae28b9a-d880-42fd-9b98-d779f0fdd77f/msodbcsql18_18.5.1.1-1_amd64.apk"; \
    curl -fsSL -O "https://download.microsoft.com/download/7/6/d/76de322a-d860-4894-9945-f0cc5d6a45f8/mssql-tools18_18.4.1.1-1_amd64.apk"; \
    \
    apk add --no-cache --allow-untrusted \
        ./msodbcsql18_18.5.1.1-1_amd64.apk \
        ./mssql-tools18_18.4.1.1-1_amd64.apk; \
    \
    rm -f msodbcsql18_18.5.1.1-1_amd64.apk mssql-tools18_18.4.1.1-1_amd64.apk; \
    \
    mkdir -p /etc/profile.d; \
    echo 'export PATH="$PATH:/opt/mssql-tools18/bin"' > /etc/profile.d/mssql-tools.sh

# Ensure PATH is updated for all processes in the container
ENV PATH="${PATH}:/opt/mssql-tools18/bin"

# ---------- Ioncube (Alpine / musl) ----------
# ---------- Ioncube (Alpine / musl) ----------
# ---------- Ioncube (Alpine / musl) ----------
RUN curl -fsSL "https://downloads.ioncube.com/loader_downloads/ioncube_loaders_lin-musl_x86-64.tar.gz" -o /tmp/ioncube.tar.gz; \
    \
    mkdir -p /tmp/ioncube; \
    tar -xzf /tmp/ioncube.tar.gz -C /tmp/ioncube --strip-components=1; \
    \
    PHP_EXT_DIR="$(php -r "echo ini_get('extension_dir');")"; \
    PHP_MAJ_MIN="$(php -r "echo PHP_MAJOR_VERSION.'.'.PHP_MINOR_VERSION;")"; \
    LOADER_SO="/tmp/ioncube/ioncube_loader_lin-musl_${PHP_MAJ_MIN}.so"; \
    \
    echo "Using PHP extension dir: ${PHP_EXT_DIR}"; \
    echo "Expecting ionCube loader: ${LOADER_SO}"; \
    \
    if [ ! -f "${LOADER_SO}" ]; then \
        echo 'Available loaders in /tmp/ioncube:'; \
        ls -1 /tmp/ioncube; \
        echo "ERROR: ionCube loader for PHP ${PHP_MAJ_MIN} not found in musl archive."; \
        exit 1; \
    fi; \
    \
    cp "${LOADER_SO}" "${PHP_EXT_DIR}/"; \
    \
    echo "zend_extension=${PHP_EXT_DIR}/$(basename "${LOADER_SO}")" > /usr/local/etc/php/conf.d/00-ioncube.ini; \
    \
    rm -rf /tmp/ioncube /tmp/ioncube.tar.gz

# ---------- PHP config & SSH known_hosts ----------
RUN mkdir -p /root/.ssh; \
    \
    cp /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini; \
    \
    sed -i "s@;date.timezone =@date.timezone = ${TZ}@" /usr/local/etc/php/php.ini; \
    \
    echo "${TZ}" > /etc/timezone; \
    ln -snf "/usr/share/zoneinfo/${TZ}" /etc/localtime

# ---------- Composer ----------
ENV COMPOSER_ALLOW_SUPERUSER=1

RUN curl -fsSL https://getcomposer.org/installer -o /tmp/composer-setup.php; \
    \
    php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer; \
    \
    rm -f /tmp/composer-setup.php; \
    \
    composer --version

# ---------- Laravel Echo Server ----------
RUN npm config set registry "${NPM_REGISTRY}"; \
    \
    npm install -g laravel-echo-server; \
    \
    npm cache clean --force

# ---------- Optional: redis serializer igbinary ----------
# (Uncomment if you want it baked in the image)
# RUN printf "redis.serializer=igbinary\n" > /usr/local/etc/php/conf.d/zz-redis.ini

# ---------- Defaults ----------
WORKDIR /app
