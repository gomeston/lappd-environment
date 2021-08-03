ARG IMG_DB_VERSION=8.0-apache-buster
FROM php:${IMG_DB_VERSION}

LABEL MAINTAINER="Ton Gomes <neilgomesdev@gmail.com>"

## Disable interactive mode for installation via APT
ENV DEBIAN_FRONTEND=noninteractive

## Update package information
RUN apt-get update \
  && apt-get upgrade -f -y

RUN apt install -f -y \
  apt-utils \
  apt-transport-https \
  software-properties-common \
  nano \
  iputils-ping \
  wget \
  gnupg \
  rsync \
  locales \
  locales-all

## Locale
RUN sed -i -e 's/# pt_BR.UTF-8 UTF-8/pt_BR.UTF-8 UTF-8/' /etc/locale.gen \
  && dpkg-reconfigure --frontend=noninteractive locales \
  && update-locale LANG=pt_BR.UTF-8

ENV LC_ALL pt_BR.UTF-8
ENV LANG pt_BR.UTF-8
ENV LANGUAGE pt_BR.UTF-8
ENV TZ=America/Sao_Paulo
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

## Install MySQL Client
### Get key from pgp.mit.edu or keyserver.ubuntu.com
RUN echo 'deb http://repo.mysql.com/apt/debian/ buster mysql-8.0' >> /etc/apt/sources.list.d/mysql.list
RUN apt-key adv --keyserver pgp.mit.edu --recv-keys 5072E1F5 \
  && apt-get update \
  && apt-get install -f -y mysql-client

## Install PostgreSQL Client
RUN sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
RUN wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | apt-key add - \
  && apt-get update \
  && apt-get install -f -y postgresql-client


## Install Composer
### To get a specific version use: --version=1.10.16
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
ENV COMPOSER_ALLOW_SUPERUSER=1
ENV COMPOSER_NO_INTERACTION=0
ENV COMPOSER_MEMORY_LIMIT=2G

##  Install libraries
RUN apt-get install -f -y \
  git \
  zlib1g-dev \
  libzip-dev \
  zip \
  gzip \
  libfreetype6-dev \
  libjpeg62-turbo-dev \
  libpng-dev \
  libicu-dev \
  libxml2-dev libxslt-dev \
  libbz2-dev \
  libcurl4 \
  curl \
  ca-certificates \
  openssl \
  libcurl4-openssl-dev \
  librabbitmq-dev \
  libssh-dev \
  libsqlite3-dev \
  libsqlite3-0 \
  sqlite3 \
  libpq-dev

##  Install extensions
RUN docker-php-ext-install sockets \
  && docker-php-ext-install opcache \
  && docker-php-ext-install pdo_mysql \
  && docker-php-ext-install pgsql \
  && docker-php-ext-install pdo_pgsql \
  && docker-php-ext-install zip \
  && docker-php-ext-configure gd --with-jpeg=/usr/include/ --with-freetype=/usr/include/ \
  && docker-php-ext-install -j$(nproc) gd \
  && docker-php-ext-install -j$(nproc) intl \
  && docker-php-ext-install xsl \
  && docker-php-ext-install soap \
  && docker-php-ext-install bz2 \
  && docker-php-ext-install bcmath \
  && docker-php-ext-install calendar \
  && docker-php-ext-configure hash --with-mhash \
  && docker-php-ext-install pcntl \
  && docker-php-ext-install posix

## Xdebug
ARG LOG_DIR=/var/log
RUN mkdir -p "$PHP_INI_DIR/conf.d"
RUN mkdir -p "/var/log/xdebug"
RUN touch "$LOG_DIR/xdebug/xdebug.log"

RUN pecl install xdebug-3.0.4

RUN echo "zend_extension=xdebug.so" >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini \
  && echo "error_reporting = E_ALL" >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini \
  && echo "\n[xdebug]" >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini \
  && echo "xdebug.mode = develop,coverage,debug,gcstats,profile,trace" >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini \
  && echo "xdebug.start_with_request = yes" >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini \
  && echo "xdebug.log_level = 0" >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini \
  && echo 'xdebug.remote_handler = dbgp' >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini \
  && echo 'xdebug.discover_client_host = true' >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini \
  && echo 'xdebug.client_port = 9003' >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini \
  && echo 'xdebug.start_with_request = yes' >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini \
  && echo 'xdebug.start_upon_error = yes' >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini \
  && echo 'xdebug.var_display_max_depth = -1' >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini \
  && echo 'xdebug.var_display_max_children = -1' >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini \
  && echo 'xdebug.var_display_max_data = -1' >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini \
  && echo "xdebug.cli_color = 1" >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini \
  && echo "xdebug.log = $LOG_DIR/xdebug.log" >> $PHP_INI_DIR/conf.d/docker-php-ext-xdebug.ini

## Configure Apache
RUN a2enmod ssl
RUN a2enmod rewrite
ADD ./.docker/apache_php/config/000-default.conf /etc/apache2/sites-available/000-default.conf
ADD ./.docker/apache_php/config/php.ini /usr/local/etc/php/php.ini

## Reload permissions script
ADD .docker/scripts/reload_permissions.sh /opt/reload_permissions.sh
RUN chmod +x /opt/reload_permissions.sh
RUN ln -s /opt/reload_permissions.sh /bin/reload_permissions

# Copy existing application directory permissions to container
ARG WORK_DIR=/var/www/html
WORKDIR ${WORK_DIR}
ARG NEWUID=1000
ARG NEWGID=1000
ARG WEB_USER=www-data
ARG WEB_GROUP=www-data
RUN usermod -u ${NEWUID} ${WEB_USER} \
  && groupmod -g ${NEWGID} ${WEB_GROUP} \
  && chown -R ${WEB_USER}:${WEB_GROUP} ${WORK_DIR} \
  && chmod u+rwx,g+rx,o+rx ${WORK_DIR}
