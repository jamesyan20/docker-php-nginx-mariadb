
ARG PHP_VERSION
FROM php:${PHP_VERSION}

ARG APP_DIR=/var/www/app

ARG MARIADB_MYSQL_SOCKET_DIRECTORY='/var/run/mysqld'

### apt-utils and supervisor 
RUN apt-get update -y && apt-get install -y --no-install-recommends \
    apt-utils \ 
    supervisor

### Dependencies for linux
RUN apt-get update && apt-get install -y \
    zlib1g-dev \
    libzip-dev \
    unzip \
    libpng-dev \
    libpq-dev \
    libxml2-dev

### php dependencies ---  you can change it if you need more resources
RUN docker-php-ext-install mysqli pdo pdo_mysql pdo_pgsql pgsql session xml opcache zip iconv simplexml pcntl gd fileinfo

### Supervisor is need to controll daemons execution in background
COPY ./docker/supervisord/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
COPY ./docker/supervisord/supervisord.conf /etc/supervisord.d/

COPY ./docker/php/extra-php.ini "$PHP_INI_DIR/99_extra.ini"
COPY ./docker/php/extra-php-fpm.conf /etc/php8/php-fpm.d/www.conf

###changing application folder permissions
WORKDIR $APP_DIR
RUN cd $APP_DIR
RUN chown www-data:www-data $APP_DIR
RUN chmod -R 755 $APP_DIR

### if you wanna copy the aplication to container and use composer inside the container 
#RUN chmod -R 777 $APP_DIR/storage
#COPY --chown=www-data:www-data ./app .
#RUN rm -rf vendor
#RUN composer install --no-interaction


### nginx installation and configuration
RUN apt-get install nginx -y
RUN rm -rf /etc/nginx/sites-enabled/* && rm -rf /etc/nginx/sites-available/*
COPY ./docker/nginx/sites.conf /etc/nginx/sites-enabled/default.conf
COPY ./docker/nginx/error.html /var/www/html/error.html

### mariadb installation and some configurations
RUN apt install mariadb-server -y
RUN sed -i -e"s/^bind-address\s*=\s*127.0.0.1/bind-address = 0.0.0.0/" /etc/mysql/my.cnf
COPY ./docker/set-password.sh /tmp/set-password.sh
#RUN /bin/sh /tmp/set-password.sh

### give the right permissions for unix mysql socket
RUN mkdir -p $MARIADB_MYSQL_SOCKET_DIRECTORY && \
    chown root:mysql $MARIADB_MYSQL_SOCKET_DIRECTORY && \
    chmod 774 $MARIADB_MYSQL_SOCKET_DIRECTORY


RUN apt-get clean && rm -rf /var/lib/apt/lists/*
# RUN apt update -y && apt install nano git -y

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]
