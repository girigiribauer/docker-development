# ---------------------------------------------------------------------------- #
# container: centos6-httpd-php56-mariadb100
# ---------------------------------------------------------------------------- #

FROM girigiribauer/docker-development:centos6-httpd-php56
MAINTAINER girigiribauer

VOLUME www/ /var/www/

ENV MYSQL_ROOT_PASSWORD "admin"
ENV MYSQL_DATABASE      "sample"
ENV MYSQL_USER          "sample"
ENV MYSQL_PASSWORD      "sample"
ENV MYSQL_HOST          "localhost"

# mariadb
RUN rpm --import https://yum.mariadb.org/RPM-GPG-KEY-MariaDB; \
    { \
        echo '[mariadb]'; \
        echo 'name=MariaDB'; \
        echo 'baseurl=https://yum.mariadb.org/10.0/centos6-amd64'; \
        echo 'gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB'; \
        echo 'gpgcheck=1'; \
    } > /etc/yum.repos.d/MariaDB.repo; \
    yum install -y --enablerepo=mariadb MariaDB-server MariaDB-client; \
    yum install -y --enablerepo=remi-php56 php-mysql; \
    mysqld --version; \
    { \
        echo '[program:mariadb]'; \
        echo 'command=/usr/bin/mysqld_safe'; \
    } >> /etc/supervisord.conf

# mariadb settings
# ref: http://txt.fliglio.com/2013/11/creating-a-mysql-docker-container/
COPY settings/my.cnf.d/server.cnf        /etc/my.cnf.d/server.cnf
COPY settings/my.cnf.d/mysql-clients.cnf /etc/my.cnf.d/mysql-clients.cnf
RUN /usr/bin/mysql_install_db; \
    /usr/bin/mysqld_safe & \
    sleep 10s && \
    mysql \
        -e "GRANT ALL ON *.* TO root@'%' IDENTIFIED BY 'admin' WITH GRANT OPTION;" \
        -e "UPDATE mysql.user SET Password=PASSWORD('${MYSQL_ROOT_PASSWORD}') WHERE User='root';" \
        -e "DELETE FROM mysql.user WHERE User='';" \
        -e "DELETE FROM mysql.user WHERE User='root' AND Host NOT IN ('localhost', '127.0.0.1', '::1');" \
        -e "DROP DATABASE test;" \
        -e "DELETE FROM mysql.db WHERE Db='test' OR Db='test\\_%';" \
        -e "CREATE DATABASE IF NOT EXISTS ${MYSQL_DATABASE};" \
        -e "GRANT ALL ON ${MYSQL_DATABASE}.* TO '${MYSQL_USER}'@'${MYSQL_HOST}' IDENTIFIED BY '${MYSQL_PASSWORD}';" \
        -e "GRANT ALL ON ${MYSQL_DATABASE}.* TO '${MYSQL_USER}'@'localhost' IDENTIFIED BY '${MYSQL_PASSWORD}';" \
# ここから設定確認
        -e "SHOW DATABASES;" \
        -e "SELECT Host,User,Password FROM mysql.user;" \
        -e "SHOW VARIABLES LIKE 'chara%';" \
# 設定反映
        -e "FLUSH PRIVILEGES;"; \
    killall mysqld; \
    sleep 10s

VOLUME /var/lib/mysql

EXPOSE 80 3306
CMD ["supervisord", "-c", "/etc/supervisord.conf"]
