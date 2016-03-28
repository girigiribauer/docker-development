# ---------------------------------------------------------------------------- #
# container: centos6-httpd-basic
# ---------------------------------------------------------------------------- #

FROM centos:centos6
MAINTAINER girigiribauer

VOLUME www/ /var/www/

# time, timezone
# ref: http://qiita.com/yuki2006/items/6cea8c352e38f047b52a#comment-8e863c71962008035d0d
ENV LANG=ja_JP.UTF-8
RUN ln -sf /usr/share/zoneinfo/Japan /etc/localtime; \
    localedef -v -c -i ja_JP -f UTF-8 ja_JP.UTF-8 > /dev/null 2>&1; \
    sed -i -e 's/en_US/ja_JP/g' /etc/sysconfig/i18n; \
    date

# epel, remi repository with gpgkey
RUN rpm --import http://ftp.iij.ad.jp/pub/linux/fedora/epel/RPM-GPG-KEY-EPEL-6; \
    rpm -Uvh http://ftp.iij.ad.jp/pub/linux/fedora/epel/epel-release-latest-6.noarch.rpm; \
    rpm --import http://rpms.remirepo.net/RPM-GPG-KEY-remi; \
    rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-6.rpm; \
    yum -y update; \
    yum clean all

# supervisor (multi process control)
RUN yum install -y supervisor --enablerepo=epel; \
    sed -i \
        -e 's/^;\?nodaemon=false/nodaemon=true/g' \
        /etc/supervisord.conf

# httpd
RUN yum install -y httpd; \
    sed -i \
        -e 's/^ServerTokens OS$/ServerTokens Prod/g' \
        -e 's/^KeepAlive Off$/KeepAlive On/g' \
        -e 's/^ServerSignature On$/ServerSignature Off/g' \
        -e 's/AllowOverride None/AllowOverride All/g' \
        /etc/httpd/conf/httpd.conf; \
    { \
        echo '[program:httpd]'; \
        echo 'command=/usr/sbin/httpd -DFOREGROUND'; \
        echo 'startsecs=0'; \
        echo 'autorestart=true'; \
    } >> /etc/supervisord.conf

EXPOSE 80
CMD ["supervisord", "-c", "/etc/supervisord.conf"]
