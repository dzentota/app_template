FROM php:7.1-fpm

# Install modules
RUN apt-get update && apt-get install -y git zlibc zlib1g zlib1g-dev libicu-dev g++ libssl-dev libmemcached-dev libmcrypt-dev ssh libpng-dev libfreetype6-dev --no-install-recommends
RUN docker-php-ext-configure mbstring && docker-php-ext-install mbstring zip intl bcmath pdo_mysql mcrypt
RUN docker-php-ext-configure gd --with-freetype-dir=/usr/include/ && docker-php-ext-install gd
RUN pecl channel-update pecl.php.net
RUN pecl install memcached && echo extension=memcached.so > /usr/local/etc/php/conf.d/ext-memcached.ini

RUN apt-get install -y wget unzip libssh2-1-dev libssh2-1 libxml2-dev \
 && wget https://github.com/Sean-Der/pecl-networking-ssh2/archive/php7.zip \
 && unzip php7.zip \
 && cd pecl-networking-ssh2-php7 \
 && phpize \
 && ./configure \
 && make \
 && make install \
 && rm -Rf pecl-networking-ssh2-php7 php7.zip \
 && docker-php-ext-install soap \
 && docker-php-ext-enable ssh2

RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

RUN mkdir -p /opt/icu/icudt52l/ \
    && curl http://source.icu-project.org/repos/icu/data/trunk/tzdata/icunew/2015e/44/le/metaZones.res > /opt/icu/icudt52l/metaZones.res \
    && curl http://source.icu-project.org/repos/icu/data/trunk/tzdata/icunew/2015e/44/le/timezoneTypes.res > /opt/icu/icudt52l/timezoneTypes.res \
    && curl http://source.icu-project.org/repos/icu/data/trunk/tzdata/icunew/2015e/44/le/windowsZones.res > /opt/icu/icudt52l/windowsZones.res \
    && curl http://source.icu-project.org/repos/icu/data/trunk/tzdata/icunew/2015e/44/le/zoneinfo64.res > /opt/icu/icudt52l/zoneinfo64.res

# Cron
RUN apt-get update && apt-get install -y cron supervisor \
    # making logging pipe
    && mkfifo --mode 0666 /var/log/cron.log \
    # make pam_loginuid.so optional for cron
    # see https://github.com/docker/docker/issues/5663#issuecomment-42550548
    && sed --regexp-extended --in-place \
    's/^session\s+required\s+pam_loginuid.so$/session optional pam_loginuid.so/' \
    /etc/pam.d/cron
RUN mkdir -p /var/log/supervisor
COPY supervisord.conf /etc/supervisord.conf
COPY start-cron /usr/sbin
RUN chmod +x /usr/sbin/start-cron
# Cron

RUN mkdir -p /var/log/php-fpm
RUN ln -sf /dev/stdout /var/log/php-fpm/access.log
RUN ln -sf /dev/stderr /var/log/php-fpm/error.log
RUN rm -r /var/lib/apt/lists/*

# Enable and configure xdebug
RUN pecl install xdebug
RUN docker-php-ext-enable xdebug
RUN echo "xdebug.remote_enable=on" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_autostart=true" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_handler=dbgp" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_mode=req" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_port=10000" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_log=/var/log/xdebug_remote.log" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.idekey=PHPSTORM" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_connect_back=Off" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_host=172.18.0.1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

CMD ["/usr/bin/supervisord","-c/etc/supervisord.conf"]
