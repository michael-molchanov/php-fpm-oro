FROM php:8.0.3-fpm-alpine3.13

LABEL maintainer "Michael Molchanov <mmolchanov@adyax.com>"

# Install system dependencies
RUN apk add --update --no-cache \
    autoconf \
    bash \
    curl \
    freetype-dev \
    g++ \
    gcc \
    git \
    icu-dev \
    imap-dev \
    libjpeg-turbo-dev \
    libmcrypt-dev \
    libpng-dev \
    libpq \
    libxml2-dev \
    libxslt-dev \
    libzip-dev \
    linux-headers \
    make \
    mysql-client \
    oniguruma-dev \
    openldap-dev \
    openssh-client \
    patch \
    postgresql-dev \
    python2 \
    python2-dev \
    tidyhtml-dev \
    && rm -rf /var/cache/apk/*

RUN apk add --update --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/v3.13/community/ --allow-untrusted gnu-libiconv
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so php

# Install php extensions
RUN set -xeu \
    && pecl install \
      xdebug \
      redis \
    && docker-php-ext-enable xdebug \
    && docker-php-ext-enable redis \
    && docker-php-ext-configure gd --with-freetype=/usr/include/ --with-jpeg=/usr/include/ \
    && docker-php-ext-install -j$(nproc)  \
      bcmath \
      gd \
      imap \
      intl \
      ldap \
      mbstring \
      opcache \
      pcntl \
      pdo \
      pdo_pgsql \
      soap \
      tidy \
      xml \
      xsl \
      zip \
    && runDeps="$( \
    scanelf --needed --nobanner --format '%n#p' --recursive /usr/local/lib/php/extensions \
      | tr ',' '\n' \
      | sort -u \
      | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
    )" \
    && apk add --no-cache --update --virtual .composer-phpext-rundeps $runDeps \
    && rm -rf /tmp/*

# Install composer
RUN wget https://getcomposer.org/installer \
    && php installer --install-dir=/usr/local/bin/ --filename=composer --version=1.10.20 \
    && rm installer \
    && composer global --no-progress --no-interaction require hirak/prestissimo

ENV NVM_DIR /usr/local/nvm

# Profile
RUN echo 'export NVM_DIR="/usr/local/nvm"' >> /etc/profile.d/nvm.sh \
    && echo '[ -s "${NVM_DIR}/nvm.sh" ] && . "${NVM_DIR}/nvm.sh"  # This loads nvm' >> /etc/profile.d/nvm.sh \
    && echo '[ -s "${NVM_DIR}/bash_completion" ] && . "${NVM_DIR}/bash_completion" # This loads nvm bash_completion' >> /etc/profile.d/nvm.sh

RUN mkdir -p ${NVM_DIR} \
    && curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.2/install.sh | bash

ENV NODE_VERSION v14.16.0
ENV PATH /usr/local/nvm/versions/node/${NODE_VERSION}/bin:${PATH}

RUN source ${NVM_DIR}/nvm.sh \
    && nvm install -s ${NODE_VERSION} \
    && nvm alias default ${NODE_VERSION} \
    && nvm use --delete-prefix default \
    && rm -rf ${NVM_DIR}/.cache

RUN source ${NVM_DIR}/nvm.sh \
    && npm install -g yarn gulp-cli grunt-cli bower \
    && node --version \
    && npm --version \
    && grunt --version \
    && gulp --version \
    && bower --version \
    && yarn versions

WORKDIR /app

COPY php.ini /usr/local/etc/php/conf.d/default.ini

USER www-data

# Install prestissimo
RUN composer global --no-progress --no-interaction require hirak/prestissimo
