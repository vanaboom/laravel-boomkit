FROM 8.4.15-alpine3.23

# ---------- OCI Labels ----------
LABEL org.opencontainers.image.title="laravel-boomkit (alpine)"
LABEL org.opencontainers.image.description="Production-ready Laravel base image on PHP 8.4 (Alpine) with common PHP/PECL extensions, MSSQL tools, Node.js, Ioncube, and Laravel Echo Server."
LABEL org.opencontainers.image.source="https://github.com/vanaboom/laravel-boomkit"
LABEL org.opencontainers.image.licenses="MIT"
LABEL maintainer="Mohsen Jadidi"

# ---------- Build args / envs ----------
ARG APK_MIRROR_URL="https://dl-cdn.alpinelinux.org/alpine"
ARG NPM_MIRROR_URL="https://registry.npmjs.org"
ARG TIMEZONE="UTC"

ENV TZ="${TIMEZONE}" \
    COMPOSER_ALLOW_SUPERUSER=1 \
    PATH="/opt/mssql-tools18/bin:${PATH}"

# ---------- Single RUN: build + enable + cleanup build-deps (keep runtimes) ----------
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
    pecl install -o -f sqlsrv pdo_sqlsrv || echo "skip sqlsrv/pdo_sqlsrv"; \
    docker-php-ext-enable igbinary redis apcu yaml || true; \
    docker-php-ext-enable sqlsrv pdo_sqlsrv || true; \
    rm -rf /tmp/pear ~/.pearrc || true; \
    apk del --no-cache .build-deps; \
    rm -rf /var/cache/apk/*; \
    arch="$(uname -m)"; \
    if [ "$arch" = "x86_64" ]; then \
      mkdir -p /opt/microsoft/msodbcsql18; echo "Y" > /opt/microsoft/msodbcsql18/ACCEPT_EULA; \
      cd /tmp; \
      curl -fsSL -O "https://download.microsoft.com/download/fae28b9a-d880-42fd-9b98-d779f0fdd77f/msodbcsql18_18.5.1.1-1_amd64.apk"; \
      curl -fsSL -O "https://download.microsoft.com/download/7/6/d/76de322a-d860-4894-9945-f0cc5d6a45f8/mssql-tools18_18.4.1.1-1_amd64.apk"; \
      apk add --no-cache --allow-untrusted ./msodbcsql18_18.5.1.1-1_amd64.apk ./mssql-tools18_18.4.1.1-1_amd64.apk; \
      rm -f /tmp/msodbcsql18_18.5.1.1-1_amd64.apk /tmp/mssql-tools18_18.4.1.1-1_amd64.apk; \
      mkdir -p /etc/profile.d; echo 'export PATH="$PATH:/opt/mssql-tools18/bin"' > /etc/profile.d/mssql-tools.sh; \
    else \
      echo "skip msodbcsql/mssql-tools for arch=$arch"; \
    fi; \
    curl -fsSL "https://downloads.ioncube.com/loader_downloads/ioncube_loaders_lin-musl_x86-64.tar.gz" -o /tmp/ioncube.tar.gz; \
    mkdir -p /tmp/ioncube; \
    tar -xzf /tmp/ioncube.tar.gz -C /tmp/ioncube --strip-components=1; \
    PHP_EXT_DIR="$(php -r "echo ini_get('extension_dir');")"; \
    PHP_MAJ_MIN="$(php -r "echo PHP_MAJOR_VERSION.'.'.PHP_MINOR_VERSION;")"; \
    LOADER_SO="/tmp/ioncube/ioncube_loader_lin-musl_${PHP_MAJ_MIN}.so"; \
    cp "${LOADER_SO}" "${PHP_EXT_DIR}/"; \
    echo "zend_extension=${PHP_EXT_DIR}/$(basename "${LOADER_SO}")" > /usr/local/etc/php/conf.d/00-ioncube.ini; \
    rm -rf /tmp/ioncube /tmp/ioncube.tar.gz; \
    cp /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini; \
    sed -i "s@;date.timezone =@date.timezone = ${TZ}@" /usr/local/etc/php/php.ini; \
    echo "${TZ}" > /etc/timezone; \
    ln -snf "/usr/share/zoneinfo/${TZ}" /etc/localtime; \
    curl -fsSL https://getcomposer.org/installer -o /tmp/composer-setup.php; \
    php /tmp/composer-setup.php --install-dir=/usr/local/bin --filename=composer; \
    rm -f /tmp/composer-setup.php; \
    composer --version; \
    composer clear-cache || true; \
    npm config set registry "${NPM_MIRROR_URL}"; \
    npm i -g laravel-echo-server; \
    npm cache clean --force; \
    mkdir -p /root/.ssh && chmod 700 /root/.ssh; \
    ssh-keyscan -t rsa github.com gitlab.com >> /root/.ssh/known_hosts; \
    rm -rf /tmp/* /usr/share/man /usr/share/doc || true
