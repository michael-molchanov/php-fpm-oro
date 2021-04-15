FROM php:7.4.16-fpm-buster

LABEL maintainer "Michael Molchanov <mmolchanov@adyax.com>"

ENV DEBIAN_FRONTEND=noninteractive
ENV TZ=Etc/UTC

# Install system dependencies
RUN apt-get update \
    && apt-get install -y \
      autoconf \
      bash \
      build-essential \
      curl \
      default-mysql-client \
      git \
      libc-client-dev \
      libfreetype6-dev \
      libicu-dev \
      libjpeg-dev \
      libkrb5-dev \
      libldap2-dev \
      libmcrypt-dev \
      libonig-dev \
      libpng-dev \
      libpq-dev \
      libtidy-dev \
      libxml2-dev \
      libxslt-dev \
      libzip-dev \
      linux-headers-amd64 \
      make \
      openssh-client \
      patch \
      postgresql-client \
      unzip \
      wget \
      zlib1g-dev \
    && rm -rf /var/lib/apt/lists/*

# Install php extensions
RUN set -xeu \
    && pecl install \
      mongodb \
      redis \
      xdebug \
    && docker-php-ext-enable mongodb \
    && docker-php-ext-enable redis \
    && docker-php-ext-enable xdebug \
    && docker-php-ext-configure gd --with-freetype=/usr/include/ --with-jpeg=/usr/include/ \
    && docker-php-ext-configure imap --with-kerberos=/usr/include/ --with-imap-ssl=/usr/include/ \
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
    && rm -rf /tmp/*

# Install composer
RUN wget https://getcomposer.org/installer \
    && php installer --install-dir=/usr/local/bin/ --filename=composer --version=2.0.12 \
    && rm installer

RUN composer global --no-progress --no-interaction require symfony/flex

# Install nodejs & grunt.
RUN curl -sL https://deb.nodesource.com/setup_15.x | bash - \
    && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
    && apt-get update \
    && apt-get install -y nodejs yarn \
    && rm -rf /var/lib/apt/lists/* \
    && npm install -g gulp-cli grunt-cli bower \
    && grunt --version \
    && gulp --version \
    && bower --version \
    && yarn versions

RUN mkdir -p /var/www/.composer \
    && chown -R www-data:www-data /var/www/.composer

WORKDIR /app

COPY php.ini /usr/local/etc/php/conf.d/default.ini

USER www-data

RUN composer global --no-progress --no-interaction require symfony/flex
