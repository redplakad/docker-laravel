FROM php:8.1-fpm-bullseye AS base

WORKDIR /workspace

# timezone environment
ENV TZ=Asia/Jakarta \
  # locale
  LANG=en_US.UTF-8 \
  LANGUAGE=en_US:en \
  LC_ALL=en_US.UTF-8 \
  # composer environment
  COMPOSER_HOME=/composer

ARG UID=1000
ARG GID=1000

# Salin Composer dari image resmi
COPY --from=composer:2.7 /usr/bin/composer /usr/bin/composer

# Tambahkan grup dan pengguna untuk phper
RUN groupadd -g ${GID} phper && useradd -u ${UID} -g phper -m phper

# Install dependencies dan ekstensi PHP
RUN apt-get update && apt-get install -y \
    git \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libpng-dev \
    libzip-dev \
    libicu-dev \
    libonig-dev \
    default-mysql-client \
    locales \
    git \
    unzip \
    && docker-php-ext-configure zip \
    && docker-php-ext-install -j$(nproc) zip \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j$(nproc) gd \
    && docker-php-ext-install -j$(nproc) intl pdo_mysql bcmath \
    && locale-gen en_US.UTF-8 \
    && localedef -f UTF-8 -i en_US en_US.UTF-8 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Buat direktori dan set permission
RUN mkdir /composer \
    && mkdir -p /home/phper/.config/psysh \
    && chown -R phper:phper /composer /workspace /home/phper/.config/psysh

# Tahap development
FROM base AS development

COPY ./infra/docker/php/php.development.ini /usr/local/etc/php/php.ini

USER phper

# Tahap development dengan xdebug
FROM base AS development-xdebug

RUN pecl install xdebug \
    && docker-php-ext-enable xdebug

COPY ./infra/docker/php/xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

USER phper

# Tahap deploy
FROM base AS deploy

COPY ./infra/docker/php/php.deploy.ini /usr/local/etc/php/php.ini
COPY --chown=phper:phper ./src /workspace

USER phper

RUN composer install --quiet --no-interaction --no-ansi --no-dev --no-scripts --no-progress --prefer-dist \
    && composer dump-autoload --optimize \
    && chmod -R 777 storage bootstrap/cache \
    && php artisan optimize:clear \
    && php artisan optimize
