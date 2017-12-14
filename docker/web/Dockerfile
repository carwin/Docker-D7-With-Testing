FROM php:7.1-fpm

# RUN uname -r

# RUN echo "deb http://deb.debian.org/debian jessie main contrib non-free" > /etc/apt/sources.list
# RUN echo "deb-src http://deb.debian.org/debian jessie main contrib non-free" >> /etc/apt/sources.list
# RUN echo "deb http://deb.debian.org/debian jessie-updates main contrib non-free" >> /etc/apt/sources.list
# RUN echo "deb-src http://deb.debian.org/debian jessie-updates main non-free" >> /etc/apt/sources.list
# RUN echo "deb http://security.debian.org/debian jessie/updates main contrib non-free" >> /etc/apt/sources.list
# RUN echo "deb-src http://security.debian.org/debian jessie/updates main contrib non-free" >> /etc/apt/sources.list

# RUN cat /etc/apt/sources.list

RUN apt-get -qq update

RUN apt-get install -y \
        git \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng12-dev \
        # apache2 \
        nginx \
        # libapache2-mod-fastcgi \
        php5-fpm \
        mysql-client \
        supervisor \
        ocaml \
        vim

RUN pecl install xdebug \
    && docker-php-ext-enable --ini-name xdebug.ini xdebug \
    && docker-php-ext-install -j$(nproc) iconv mcrypt opcache pdo_mysql pdo zip \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd

# Move the supervisor configuration to the container
RUN mkdir -p /var/log/supervisor
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Create the directory for nginx's Document Root before we mount volumes
# Add an nginx user then move the nginx configuration to the container
# Also create stuff we need for Unison
RUN mkdir -p /var/www/html/docroot
RUN useradd nginx
COPY nginx.conf /etc/nginx/nginx.conf

# Create the directory for Apache's document root before we mount volumes
# Add an apache user then move the apache configuration to the container
# RUN mkdir -p /var/www/html/docroot
# RUN useradd apache
# COPY httpd.conf /etc/apache2/httpd.conf

# Move the xdebug configuration to the container
COPY xdebug.ini /usr/local/etc/php/conf.d/xdebug-host-stub.ini
RUN { \
    cat /usr/local/etc/php/conf.d/xdebug-host-stub.ini; \
    } >> /usr/local/etc/php/conf.d/xdebug.ini

# Remove the stub xdebug file
RUN rm /usr/local/etc/php/conf.d/xdebug-host-stub.ini

# Create the php.d directory and echo opcache settings into a file
RUN mkdir -p /etc/php.d
RUN { \
    echo 'opcache.memory_consumption=256'; \
    echo 'opcache.interned_strings_buffer=8'; \
    echo 'opcache.max_accelerated_files=5000'; \
    echo 'opcache.fast_shutdown=1'; \
    echo 'opcache.enable_cli=1'; \
  } > /etc/php.d/opcache-recommended.ini

# Echo PHP settings into the configuration file on the container
RUN { \
    echo "date.timezone = America/New_York"; \
    echo "memory_limit = 512M"; \
    echo "upload_max_filesize = 10M"; \
    echo "post_max_size = 10M"; \
    echo "realpath_cache_size = 256k"; \
    echo "realpath_cache_ttl = 300"; \
  } > /usr/local/etc/php/php.ini

# Composer
RUN curl -sS https://getcomposer.org/installer | php \
  && mv composer.phar /usr/local/bin/composer
ENV PATH /root/.composer/vendor/bin:$PATH
RUN mkdir -p /bin
COPY composer.json /bin
WORKDIR /bin
RUN composer install

# Make life simpler for running Drush and Drupal Console
ENV PATH /var/www/html/vendor/bin:$PATH

# Set the working directory to the docroot for ease of use when attaching a
# shell.
WORKDIR /var/www/html/docroot

# Run the supervisor process to spawn apache and php-fpm
CMD supervisord
