FROM php:7.4.16-fpm-alpine

LABEL maintainer "Michael Molchanov <mmolchanov@adyax.com>"

# Install system dependencies
RUN apk update \
    && apk add --no-cache --virtual dev-deps git autoconf gcc g++ make linux-headers \
    && apk add --no-cache \
    bash \
    curl \
    freetype-dev \
    icu-dev \
    imap-dev \
    libjpeg-turbo-dev \
    libmcrypt-dev \
    libpng-dev \
    libpq \
    libxml2-dev \
    libxslt-dev \
    libzip-dev \
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

RUN apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/edge/community/ --allow-untrusted gnu-libiconv
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

WORKDIR /app

COPY php.ini /usr/local/etc/php/conf.d/default.ini

USER www-data

ENV HOME /home/www-data
ENV NVM_DIR $HOME/.nvm
ENV PATH ${HOME}/.yarn/bin:${HOME}/.nvm/versions/node/${NODE_VERSION}/bin:${PATH}

# Profile
RUN echo 'export NVM_DIR="$HOME/.nvm"' >> "$HOME/.bashrc" \
    && echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> "$HOME/.bashrc" \
    && echo '[ -s "$NVM_DIR/bash_completion" ] && . "$NVM_DIR/bash_completion" # This loads nvm bash_completion' >> "$HOME/.bashrc"

RUN mkdir -p $NVM_DIR \
    && curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.2/install.sh | bash

ENV NODE_VERSION v10.24.0

RUN source ${NVM_DIR}/nvm.sh \
    && nvm install -s ${NODE_VERSION} \
    && nvm alias default ${NODE_VERSION} \
    && nvm use --delete-prefix default

RUN source ${NVM_DIR}/nvm.sh \
    && npm install -g gulp-cli grunt-cli bower \
    && curl -o- -L https://yarnpkg.com/install.sh | bash \
    && node --version \
    && npm --version \
    && grunt --version \
    && gulp --version \
    && bower --version \
    && yarn versions
