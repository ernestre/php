FROM php:7.2-fpm

RUN apt-get update -yqq && apt-get install -yqq \
    build-essential \
    curl \
    git \
    gzip \
    libglib2.0-dev \
    libicu-dev \
    libmcrypt-dev \
    libmcrypt4 \
    openssh-client \
    patchelf \
    python \
    tar \
    unzip \
    zlib1g-dev

RUN pecl install xdebug
RUN docker-php-ext-enable xdebug
RUN docker-php-ext-install zip
RUN docker-php-ext-install bcmath
RUN docker-php-ext-configure intl
RUN docker-php-ext-install intl

RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php composer-setup.php \
    && php -r "unlink('composer-setup.php');" \
    && mv composer.phar /usr/bin/composer

RUN mkdir /v8js-build \
    && cd /v8js-build \
    && git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git \
    && PATH=/v8js-build/depot_tools:"$PATH" \
    && fetch v8 \
    && cd /v8js-build/v8 \
    && tools/dev/v8gen.py -vv x64.release -- is_component_build=true \
    && ninja -C out.gn/x64.release/ \
    && mkdir -p /opt/v8/lib \
    && mkdir -p /opt/v8/include \
    && cp out.gn/x64.release/lib*.so out.gn/x64.release/*_blob.bin out.gn/x64.release/icudtl.dat /opt/v8/lib/ \
    && cp -R include/* /opt/v8/include/ \
    && for A in /opt/v8/lib/*.so; do patchelf --set-rpath '$ORIGIN' $A; done \
    && cd /v8js-build \
    && git clone https://github.com/phpv8/v8js.git \
    && cd /v8js-build/v8js \
    && phpize \
    && ./configure --with-v8js=/opt/v8 LDFLAGS="-lstdc++" \
    && make \
    && make test \
    && make install \
    && echo extension=v8js.so > /usr/local/etc/php/conf.d/docker-php-ext-v8js.ini \
    && cd \
    && rm -rf /v8js-build
