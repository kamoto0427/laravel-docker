FROM php:8.0-apache-bullseye

RUN cd /etc/apache2/mods-enabled \
    && ln -s ../mods-available/rewrite.load

RUN apt-get update \
    && apt-get install -y \
    git \
    zip \
    unzip \
    libzip-dev \
    vim \
    libpng-dev \
    libpq-dev \
    libfreetype6-dev \
    libjpeg-dev \
    libonig-dev \
    && docker-php-ext-install pdo pdo_mysql mysqli zip

ENV COMPOSER_ALLOW_SUPERUSER 1
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

ADD docker/php/php.ini /usr/local/etc/php/
ADD docker/php/apache/000-default.conf /etc/apache2/sites-enabled/

WORKDIR /var/www/html

COPY ../../../back /var/www/html

RUN chown www-data storage/ -R \
 && composer install