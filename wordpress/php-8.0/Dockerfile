FROM php:8.0-fpm-alpine

LABEL MAINTAINER="lewisay <liuyuan16@baidu.com>"

RUN set -eux; \
    apk add --no-cache \
        bash \
        sed \
        ghostscript \
    ;

# install redis
ENV REDIS_VERSION 5.3.4
RUN mkdir -p /usr/src/php/ext/redis; \
    curl -fsSL https://github.com/phpredis/phpredis/archive/$REDIS_VERSION.tar.gz | tar xvz -C "/usr/src/php/ext/redis" --strip 1;

# install the PHP extensions we need (https://make.wordpress.org/hosting/handbook/handbook/server-environment/#php-extensions)
RUN set -ex; \
    \
    apk add --no-cache --virtual .build-deps \
        $PHPIZE_DEPS \
        freetype-dev \
        libjpeg-turbo-dev \
        libpng-dev \
        libzip-dev \
    ; \
    \
    docker-php-ext-configure gd \
        --with-freetype \
        --with-jpeg \
    ; \
    docker-php-ext-install -j "$(nproc)" \
        bcmath \
        exif \
        gd \
        mysqli \
        zip \
        redis \
    ; \
    \
    runDeps="$( \
        scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
            | tr ',' '\n' \
            | sort -u \
            | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
    )"; \
    apk add --no-network --virtual .wordpress-phpexts-rundeps $runDeps; \
    apk del --no-network .build-deps;

# set recommended PHP.ini settings
# see https://secure.php.net/manual/en/opcache.installation.php
RUN set -eux; \
    docker-php-ext-enable opcache; \
    { \
        echo 'opcache.memory_consumption=128'; \
        echo 'opcache.interned_strings_buffer=8'; \
        echo 'opcache.max_accelerated_files=4000'; \
        echo 'opcache.revalidate_freq=2'; \
        echo 'opcache.fast_shutdown=1'; \
    } > /usr/local/etc/php/conf.d/opcache-recommended.ini

RUN rm -rf /usr/src;