#FROM phpunit/phpunit AS phpunit
# PHPUnit Docker Container.
FROM php:7.2-fpm-alpine AS phpunit

ENV PEAR_PACKAGES foo

WORKDIR /tmp

RUN apk --no-cache add \
        bash \
        ca-certificates \
        curl \
        git \
        php7 \
        php7-bcmath \
        php7-ctype \
        php7-curl \
        php7-dom \
        php7-exif \
        php7-fileinfo \
        php7-json \
        php7-mbstring \
        php7-mcrypt \
        php7-mysqli \
        php7-opcache \
        php7-openssl \
        php7-pcntl \
        php7-pdo \
        php7-pdo_mysql \
        php7-pdo_pgsql \
        php7-pdo_sqlite \
        php7-phar \
        php7-session \
        php7-simplexml \
        php7-soap \
        php7-tokenizer \
        php7-xdebug \
        php7-xml \
        php7-xmlreader \
        php7-xmlwriter \
        php7-zip \
        php7-zlib \
        unzip 
RUN docker-php-ext-install pcntl

RUN php -r "copy('https://pear.php.net/go-pear.phar', 'go-pear.phar');" \
    && php go-pear.phar \
    && php -r "unlink('go-pear.phar');" \
    && php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"

COPY --from=composer:1.5 /usr/bin/composer /usr/bin/composer

RUN composer require --dev "phpunit/phpunit:<7.0" --prefer-source --no-interaction
#RUN composer require "phpunit/phpunit:~6.5.3" --prefer-source --no-interaction
RUN composer require "phpunit/php-invoker" --prefer-source --no-interaction


RUN apk --no-cache --virtual .build-xdebug-deps add \
    g++ \
    make \
    autoconf
RUN pecl install xdebug    
RUN docker-php-ext-enable xdebug
RUN sed -i 's/\;z/z/g' /etc/php7/conf.d/xdebug.ini
RUN php -m | grep -i xdebug #|| php -m && cat /etc/php7/conf.d/xdebug.ini && false
RUN apk del .build-xdebug-deps

ONBUILD RUN \
    { \
        [ "${PEAR_PACKAGES}" != "foo" ] \
    } || exit 0 && pear install ${PEAR_PACKAGES}


FROM wordpress:cli AS wordpress-cli

FROM wordpress:php7.2-fpm-alpine AS wordpress

RUN apk --no-cache add less
COPY ./wordpress-setup.sh /usr/local/bin/wordpress-setup.sh
COPY ./install-wp-tests.sh /usr/local/bin/install-wp-tests.sh

RUN ln -s /tmp/vendor/bin/phpunit /usr/local/bin/phpunit
COPY wait-for /usr/local/bin/

COPY --from=phpunit /etc/php7 /etc/php7
COPY --from=phpunit /tmp/vendor /tmp/vendor
COPY --from=phpunit /usr/lib/php7 /usr/lib/php7
COPY --from=phpunit /usr/lib/libpcre* /usr/lib/
COPY --from=phpunit /usr/local/etc/php/ /usr/local/etc/php/
COPY --from=phpunit /usr/local/lib/php/ /usr/local/lib/php/
COPY --from=phpunit /usr/bin/git* /usr/bin/
COPY --from=phpunit /usr/libexec/git-core/ /usr/libexec/git-core
COPY --from=phpunit /usr/share/git-core/ /usr/share/git-core

COPY --from=wordpress-cli /usr/local/bin/wp /usr/local/bin/wp
COPY --from=wordpress-cli /etc/mysql* /etc/
COPY --from=wordpress-cli /usr/bin/mysql* /usr/bin/
COPY --from=wordpress-cli /usr/share/mysql* /user/share/


COPY --from=wordpress /usr/src/wordpress/ /usr/share/nginx/html/
COPY --from=wordpress /var/www/html /var/www/html


WORKDIR ${WORDPRESS_PATH:-/usr/share/nginx/html}
# RUN /usr/local/bin/wordpress-setup.sh php-fpm
# RUN wp --allow-root config create --dbname=${WORDPRESS_DB_NAME} \
#     --dbuser=${WORDPRESS_DB_USER} \
#     --dbhost=${WORDPRESS_DB_HOST} \
#     --dbpass=${WORDPRESS_DB_PASSWORD}
# RUN wp --allow-root core install --url="http://localhost:8000" --title="PhpUnitWordPress" --admin_user=admin --admin_password=test --admin_email="test_wp@dougbeal.com" --skip-email

ENTRYPOINT ["wordpress-setup.sh"]
