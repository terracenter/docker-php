FROM php:8.1-fpm
RUN pecl channel-update pecl.php.net

RUN pecl install xdebug  \
    pecl install mbstring \
    docker-php-ext-install bcmath \
    && docker-php-ext-enable xdebug 
    