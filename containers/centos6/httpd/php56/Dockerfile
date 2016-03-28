# ---------------------------------------------------------------------------- #
# container: centos6-httpd-php56
# ---------------------------------------------------------------------------- #

FROM girigiribauer/docker-development:centos6-httpd-basic
MAINTAINER girigiribauer

VOLUME www/ /var/www/

# postfix (for mail sending)
RUN yum install -y postfix; \
    which postfix; \
    { \
        echo '[program:postfix]'; \
        echo 'process_name = master'; \
        echo 'command = /usr/sbin/postfix -c /etc/postfix start'; \
        echo 'startsecs=0'; \
        echo 'autorestart=false'; \
    } >> /etc/supervisord.conf

# php56
RUN yum install -y --enablerepo=remi-php56 \
    php \
    php-cli \
    php-devel \
    php-gd \
    php-mbstring \
    php-mcrypt \
    php-opcache \
    php-pear \
    php-pecl-memcached \
    php-pecl-imagick \
    php-xml \
    php-xmlrpc; \
    php --version; \
    sed -i \
        -e 's/expose_php = On/expose_php = Off/g' \
        -e 's/memory_limit = .*$/memory_limit = 512M/g' \
        -e 's/error_reporting = .*$/error_reporting = E_ALL/g' \
        -e 's/display_errors = Off/display_errors = On/g' \
        -e 's|;error_log = syslog|error_log = /var/log/php_errors.log|g' \
        -e 's/post_max_size = .*$/post_max_size = 128M/g' \
        -e 's/upload_max_filesize = .*$/upload_max_filesize = 128M/g' \
        -e 's|^;date.timezone =.*$|date.timezone = Asia/Tokyo|g' \
        -e 's/;mbstring.language = .*$/mbstring.language = Japanese/g' \
        -e 's/;mbstring.internal_encoding = .*$/mbstring.internal_encoding = UTF-8/g' \
        -e 's/;mbstring.http_output = .*$/mbstring.http_output = UTF-8/g' \
        -e 's|; *extension_dir = "./"|extension_dir = "/usr/lib64/php/modules/"|g' \
        /etc/php.ini

# php/composer
RUN php -r "readfile('https://getcomposer.org/installer');" > composer-setup.php; \
    php -r "if (hash('SHA384', file_get_contents('composer-setup.php')) === '41e71d86b40f28e771d4bb662b997f79625196afcca95a5abf44391188c695c6c1456e16154c75a211d238cc3bc5cb47') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"; \
    php composer-setup.php; \
    php -r "unlink('composer-setup.php');"; \
    mv composer.phar /usr/local/bin/composer; \
    composer --version

# httpd
RUN sed -i \
        -e 's/^DirectoryIndex \(.*\)$/DirectoryIndex index.php \1/g' \
        /etc/httpd/conf/httpd.conf

EXPOSE 80
CMD ["supervisord", "-c", "/etc/supervisord.conf"]
