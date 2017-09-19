FROM php:rc-fpm

# Install PHP extensions

RUN apt-get update && apt-get install -y libmcrypt-dev mysql-client libicu-dev libpng-dev unzip && docker-php-ext-install pdo_mysql intl gd zip


# Config mailcatcher
# RUN apt-get install -y ssmtp && \
#     echo "sendmail_path = /usr/sbin/ssmtp -t" > /usr/local/etc/php/conf.d/sendmail.ini && \
#     echo "mailhub=mailcatcher:25\nUseTLS=NO\nFromLineOverride=YES" > /etc/ssmtp/ssmtp.conf

# Enable PHP error log to docker log
RUN echo "php_admin_value[error_log] = /dev/stderr" >> /usr/local/etc/php-fpm.conf

ENV TZ=Europe/Amsterdam
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

WORKDIR /var/www/applications
