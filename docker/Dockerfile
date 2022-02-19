FROM ubuntu:latest

ENV TZ=Asia/Tokyo

## apt
RUN apt update
RUN apt install -y unzip zip tzdata
RUN apt install -y apache2 apache2-utils
RUN apt install -y php7.4 libapache2-mod-php7.4 php7.4-mysql php-common php7.4-cli php7.4-common php7.4-json php7.4-dom
RUN apt install -y supervisor

# composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
ENV COMPOSER_ALLOW_SUPERUSER 1
ENV COMPOSER_HOME /composer
ENV PATH $PATH:/composer/vendor/bin
ENV APACHE_LOG_DIR /var/log/httpd

# supervisor
#RUN touch /var/run/supervisor.sock && chmod 0700 /var/run/supervisor.sock
RUN touch /dev/shm/supervisor.sock && chmod 0700 /dev/shm/supervisor.sock
COPY ./supervisor/supervisord.conf /etc/supervisor/supervisord.conf

# apache
RUN a2enmod rewrite
EXPOSE 80
CMD ["apachectl", "-D", "FOREGROUND"]

#RUN apt-get update && apt-get install -y zlib1g-dev libpq-dev mariadb-client unzip zip vim libzip-dev
#
## apache
#COPY ./apache/000-default.conf /etc/apache2/sites-available/000-default.conf
#COPY ./apache/*.conf /etc/apache2/sites-enabled/
#RUN a2enmod rewrite
#
## php
#COPY ./php/php.ini /usr/local/etc/php/
#RUN docker-php-ext-install pdo_mysql
#
## composer
#COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
#ENV COMPOSER_ALLOW_SUPERUSER 1
#ENV COMPOSER_HOME /composer
#ENV PATH $PATH:/composer/vendor/bin
#ENV APACHE_LOG_DIR /var/log/httpd
#
#WORKDIR /var/www/html/laravel
#
## laravel
#RUN composer global require "laravel/installer"
