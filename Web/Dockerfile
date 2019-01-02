FROM php:7.0-apache

MAINTAINER Beat Suhner <mail@beat.sun@hotmail.com>

RUN a2enmod rewrite

# Prepare Debian environment
ENV DEBIAN_FRONTEND noninteractive

# Performance optimization - see https://gist.github.com/jpetazzo/6127116
# this forces dpkg not to call sync() after package extraction and speeds up install
RUN echo "force-unsafe-io" > /etc/dpkg/dpkg.cfg.d/02apt-speedup
# we don't need an apt cache in a container
RUN echo "Acquire::http {No-Cache=True;};" > /etc/apt/apt.conf.d/no-cache

# Packages
RUN apt-get update && \
  DEBIAN_FRONTEND=noninteractive apt-get install -y \
    libbz2-dev \
    libmcrypt-dev \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    zlib1g-dev \
    libpq-dev \
    freetds-dev \
    libpng-dev \
    git --no-install-recommends \
  && rm -r /var/lib/apt/lists/*  \
  && cp -s /usr/lib/x86_64-linux-gnu/libsybdb.so /usr/lib/

# PHP Extension
RUN docker-php-ext-install pdo_pgsql pdo_dblib pdo_mysql mcrypt zip bz2 mbstring bcmath pcntl gd
RUN docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
  && docker-php-ext-install -j$(nproc) gd

# Update the default apache site with the config we created.
ADD apache-config.conf /etc/apache2/sites-enabled/000-default.conf

###############################
# Fuer den Produktions Einsatz
###############################

## Clone Repo
RUN apt-get --assume-yes install curl
# RUN apt-get --assume-yes install git-core

## Arbeitsverzeichnis setzen
# WORKDIR /var/www/html

## Repo in www clonen
# RUN git clone https://[EDIT username]:[password]@bitbucket.org/[EDIT username]/[EDIT Repository].git

## wechseln in das repo
# WORKDIR /var/www/html/[EDIT name]


## Composer installieren
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer && \
/usr/local/bin/composer global require "fxp/composer-asset-plugin"

# RUN composer install --no-progress

## Berechtigungen setzen
# RUN chmod 777 -R runtime/
# RUN chmod 777 -r web/assets

RUN service apache2 restart

# RUN rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

CMD /usr/sbin/apache2ctl -D FOREGROUND
