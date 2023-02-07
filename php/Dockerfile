FROM php:8.1-fpm-alpine

#TODO: list of todos:
#1. make all path references for packages and package installs dynamic
#2. remove all unused packages after everything is installed e.g wget
#3. chmod all dir/files correctly and secure
#4. Clean up this docker file AND order everything correctly for caching optimization
#5. double check eveything happening in this file is actually required or redundant
#6. Make it so IN PRODUCTION it runs php artisan production:build (compiles caches and optimizes app)

RUN pear config-set php_ini /usr/local/etc/php/php.ini-development

RUN apt-get update  && apt-get -y install git zip libzip-dev wget \
zlib1g-dev unzip libpq-dev git-core libmcrypt-dev vim libfcgi0ldbl gnupg \
libfreetype6-dev libjpeg62-turbo-dev libpng-dev libicu-dev libonig-dev


RUN apt-get update
RUN pecl channel-update pecl.php.net
RUN pecl install apcu
RUN pecl install igbinary
RUN pecl install mongodb-1.15.0
RUN pecl install build-base linux-headers swoole

RUN echo "extension=swoole.so" > /etc/php7/conf.d/swoole.ini

#ENABLE DOCKER LIBS
RUN docker-php-ext-install pdo
RUN docker-php-ext-install mbstring
RUN docker-php-ext-install gd
RUN docker-php-ext-configure intl && docker-php-ext-install intl
RUN docker-php-ext-configure opcache --enable-opcache
RUN docker-php-ext-configure gd --with-freetype --with-jpeg

##NEED TO INSTALL PHP REDIS EXTENSIONS RATHER THAN USING COMPOSER AS IS FAR MORE PERFORMANT
#    # compile Redis with igbinary support
#    pecl bundle redis && cd redis && phpize && ./configure --enable-redis-igbinary && make && make install && \
# I ASSUME I WILL NEED TO MOVE THIS INTO THE PHP.INI EXTENSION DIRECTORY LIKE THE MONGODB EXTENSIONS


RUN docker-php-ext-enable apcu
RUN docker-php-ext-enable igbinary
RUN docker-php-ext-enable mongodb
RUN docker-php-ext-enable opcache

RUN apt-get install nano msmtp bash curl -y

# INSTALL COMPOSER
COPY --from=composer:latest /usr/bin/composer /usr/local/bin/composer

ENV EXT_APCU_VERSION=5.1.17

RUN docker-php-source extract
RUN docker-php-ext-enable opcache
RUN docker-php-ext-install bcmath sockets
RUN apt-get update

RUN echo '\
opcache.interned_strings_buffer=16\n\
opcache.load_comments=Off\n\
opcache.max_accelerated_files=16000\n\
opcache.save_comments=Off\n\
' >> /usr/local/etc/php/conf.d/docker-php-ext-opcache.ini

RUN echo umask 0011 >> /etc/profile && \
      echo umask 0011 >> /etc/bash.bashrc

RUN docker-php-source delete

##FOUND AT https://stackoverflow.com/questions/48849166/how-can-i-enable-libmongoc-ssl
RUN apt-get install -y cmake libssl-dev libsasl2-dev

#SETUP MONGODB DRIVER
WORKDIR /
RUN git clone https://github.com/mongodb/mongo-php-driver.git
WORKDIR /mongo-php-driver
RUN git submodule update --init
RUN phpize
RUN ./configure --with-mongodb-ssl=openssl
RUN make
RUN make install

WORKDIR /var/www/

#MUST ADD GIT COMMMAND TO CLONE REPO THROUGH SSL (SSL WILL BE REQUIRED OR AUTH TOKEN)
#MUST ADD COMPOSER AND ARTISAN COMMANDS TO BE RUN FOR BUILDING PRODUCTION PLATFORM

CMD ["php-fpm"]

EXPOSE 9000
