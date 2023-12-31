FROM ubuntu:22.04
ENV REFRESHED_AT 2023-11-21

# based on dgraziotin/lamp
# MAINTAINER Miodrag Iakulov  <mio@imiodrag.com>
# updated for PHP 8.1 Miodrag Iakulov <mio@imiodrag.com>

ENV DOCKER_USER_ID 501
ENV DOCKER_USER_GID 20

ENV BOOT2DOCKER_ID 1000
ENV BOOT2DOCKER_GID 50

ENV PHPMYADMIN_VERSION=5.1.1
ENV SUPERVISOR_VERSION=4.2.2

ARG PHP_VERSION
ENV PHP_VERSION=$PHP_VERSION

# Tweaks to give Apache/PHP write permissions to the app
RUN usermod -u ${BOOT2DOCKER_ID} www-data && \
  usermod -G staff www-data && \
  useradd -r mysql && \
  usermod -G staff mysql && \
  groupmod -g $(($BOOT2DOCKER_GID + 10000)) $(getent group $BOOT2DOCKER_GID | cut -d: -f1) && \
  groupmod -g ${BOOT2DOCKER_GID} staff

# Install packages
ENV DEBIAN_FRONTEND noninteractive

RUN apt-get update && \
  apt-get -y upgrade && \
  apt-get -y install postfix python3-setuptools wget git pwgen apache2 cron php${PHP_VERSION} libapache2-mod-php${PHP_VERSION} mysql-server php${PHP_VERSION}-mysql php${PHP_VERSION}-apcu php${PHP_VERSION}-gd php${PHP_VERSION}-xml php${PHP_VERSION}-mbstring zip unzip php${PHP_VERSION}-zip curl php${PHP_VERSION}-curl && \
  apt-get -y autoremove && \
  apt-get -y clean && \
  echo "ServerName localhost" >> /etc/apache2/apache2.conf

# Install supervisor 4
RUN curl -L https://pypi.io/packages/source/s/supervisor/supervisor-${SUPERVISOR_VERSION}.tar.gz | tar xvz && \
  cd supervisor-${SUPERVISOR_VERSION}/ && \
  python3 setup.py install

# Add image configuration and scripts
ADD supporting_files/start-apache2.sh /start-apache2.sh
ADD supporting_files/start-mysqld.sh /start-mysqld.sh
ADD supporting_files/run.sh /run.sh
RUN chmod 755 /*.sh
ADD supporting_files/supervisord-apache2.conf /etc/supervisor/conf.d/supervisord-apache2.conf
ADD supporting_files/supervisord-mysqld.conf /etc/supervisor/conf.d/supervisord-mysqld.conf
ADD supporting_files/supervisord-laravel.conf /etc/supervisor/conf.d/supervisord-laravel.conf
ADD supporting_files/supervisord-cronjob.conf /etc/supervisor/conf.d/supervisord-cronjob.conf
ADD supporting_files/mysqld_innodb.cnf /etc/mysql/conf.d/mysqld_innodb.cnf
ADD supporting_files/supervisord.conf /etc/supervisor/supervisord.conf

# Add supervisor log file
RUN mkdir -p /var/log/supervisord && touch /var/log/supervisord/supervisord.log

# Remove pre-installed database
RUN rm -rf /var/lib/mysql

# Add MySQL utils
ADD supporting_files/create_mysql_users.sh /create_mysql_users.sh

# Add phpmyadmin
RUN wget -O /tmp/phpmyadmin.tar.gz https://files.phpmyadmin.net/phpMyAdmin/${PHPMYADMIN_VERSION}/phpMyAdmin-${PHPMYADMIN_VERSION}-all-languages.tar.gz
RUN tar xfvz /tmp/phpmyadmin.tar.gz -C /var/www
RUN ln -s /var/www/phpMyAdmin-${PHPMYADMIN_VERSION}-all-languages /var/www/phpmyadmin
RUN mv /var/www/phpmyadmin/config.sample.inc.php /var/www/phpmyadmin/config.inc.php

# Add composer
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
  php composer-setup.php && \
  php -r "unlink('composer-setup.php');" && \
  mv composer.phar /usr/local/bin/composer

ENV MYSQL_PASS:-$(pwgen -s 12 1)
# config to enable .htaccess
ADD supporting_files/apache_default /etc/apache2/sites-available/000-default.conf
RUN a2enmod rewrite

# Configure /app folder with sample app
RUN mkdir -p /app && rm -fr /var/www/html && ln -s /app /var/www/html
ADD app/ /app

#Environment variables to configure php
ENV PHP_UPLOAD_MAX_FILESIZE 10M
ENV PHP_POST_MAX_SIZE 10M

# Add volumes for the app and MySql
VOLUME  ["/var/lib/mysql", "/app" ]

# Create the log file to be able to run tail
RUN touch /var/log/cron.log

EXPOSE 80 3306
#CMD ["/run.sh"]
CMD ["/run.sh"]
