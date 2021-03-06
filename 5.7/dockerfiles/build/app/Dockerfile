FROM php:7.2-fpm


# Install dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    mysql-client \
    libpng-dev \
    libjpeg62-turbo-dev \
    libfreetype6-dev \
    locales \
    vim \
    curl \
    supervisor \
    librabbitmq-dev \
    software-properties-common \
    wget

RUN sed -i -e 's/# en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen && \
    locale-gen
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

RUN pecl install php-amqplib amqp xdebug
RUN docker-php-ext-configure bcmath --enable-bcmath
RUN docker-php-ext-install -j$(nproc) bcmath
RUN docker-php-ext-enable amqp xdebug

# Install extensions
RUN docker-php-ext-install pdo_mysql mbstring zip exif pcntl
RUN docker-php-ext-configure gd --with-gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ --with-png-dir=/usr/include/
RUN docker-php-ext-install gd
RUN docker-php-ext-install sockets

# Install composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Download xdebug and install
RUN wget http://xdebug.org/files/xdebug-2.6.1.tgz
RUN tar -xvzf xdebug-2.6.1.tgz
RUN cd xdebug-2.6.1 \
    && phpize \
    && ./configure --enable-xdebug \
    && make \
    && make install \
    && cp modules/xdebug.so /usr/local/lib/php/extensions/no-debug-non-zts-20170718 \
    && echo 'zend_extension = /usr/local/lib/php/extensions/no-debug-non-zts-20170718/xdebug.so' >> /usr/local/etc/php.ini \
    && echo 'xdebug.remote_enable=on' >> /usr/local/etc/php.ini \
    && echo 'xdebug.remote_connect_back=1' >> /usr/local/etc/php.ini \
    && echo 'xdebug.remote_host=0.0.0.0' >> /usr/local/etc/php.ini \
    && echo 'xdebug.remote_port=9000' >> /usr/local/etc/php.ini \
    && echo 'xdebug.remote_handler=dbgp' >> /usr/local/etc/php.ini \
    && echo 'xdebug.max_nesting_level=512' >> /usr/local/etc/php.ini \
    && echo 'extension=xdebug.so' >> /usr/local/etc/php.ini

WORKDIR /var/www
COPY laravel_5.7.tar.gz /var/www/laravel_5.7.tar.gz