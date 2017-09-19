FROM php:rc-alpine




RUN apk update && apk add bash nano git nodejs nodejs-npm yarn openssh augeas \
    && mkdir -p ~root/.ssh /etc/authorized_keys && chmod 700 ~root/.ssh/ \
    && augtool 'set /files/etc/ssh/sshd_config/AuthorizedKeysFile ".ssh/authorized_keys /etc/authorized_keys/%u"' \
    && echo -e "Port 22\n" >> /etc/ssh/sshd_config \
    && echo "PermitUserEnvironment yes" >> /etc/ssh/sshd_config \
    && echo "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin" >> ~root/.ssh/environment \
    && cp -a /etc/ssh /etc/ssh.cache \
    && rm -rf /var/cache/apk/* \
    && sed -i s/#PermitRootLogin.*/PermitRootLogin\ yes/ /etc/ssh/sshd_config \
  	&& echo "LogLevel VERBOSE" >> /etc/ssh/sshd_config \
  	&& sed -i s/#PasswordAuthentication\ yes/PasswordAuthentication\ no/ /etc/ssh/sshd_config \
  	&& passwd -d root \
  	&& php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \
    && php composer-setup.php \
    && php -r "unlink('composer-setup.php');" \
    && mv composer.phar /usr/local/bin/composer \
    && docker-php-ext-install pdo_mysql

COPY ./docker/services/builder/entrypoint.sh /entrypoint.sh
RUN ["chmod", "+x", "/entrypoint.sh"]

ENTRYPOINT ["/entrypoint.sh"]
CMD ["/usr/sbin/sshd", "-D", "-f", "/etc/ssh/sshd_config"]

EXPOSE 22

WORKDIR /var/www/applications/

