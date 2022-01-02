FROM php:7.2-fpm

ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

# Configure non-root user.
RUN userdel -f www-data &&\
    if getent group www-data ; then groupdel www-data; fi &&\
    groupadd -g ${PGID} www-data &&\
    useradd -l -u ${PUID} -g www-data www-data &&\
    install -d -m 0755 -o www-data -g www-data /home/www-data &&\
    chown --changes --silent --no-dereference --recursive \
          --from=33:33 ${PUID}:${PGID} \
        /home/www-data

# Install base extensions
RUN apt-get update && apt-get install -yqq \
        apt-utils \
        cron \
        git \
        libbz2-dev \
        libc-client-dev \
        libfreetype6-dev \
        libicu-dev \
        libjpeg62-turbo-dev \
        libkrb5-dev \
        libmcrypt-dev \
        libpq-dev \
        libpng-dev \
        libpspell-dev \
        libxml2-dev \
        libzip-dev \
        nano \
        vim \
        zip \
        unzip \
    && pecl install mcrypt-1.0.4 xdebug-3.1.2 \
    && docker-php-ext-enable mcrypt xdebug \
    && docker-php-ext-configure intl \
    && docker-php-ext-configure zip \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-configure imap --with-kerberos --with-imap-ssl \
    && docker-php-ext-install -j$(nproc) bcmath bz2 calendar dba exif gd gettext imap intl mysqli pcntl pgsql pspell \
        pdo_mysql pdo_pgsql shmop soap sockets sysvmsg sysvsem sysvshm zip

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer --version=1.10.23

# Copy xdebug configuration for remote debugging
COPY ./xdebug.ini /usr/local/etc/php/conf.d/xdebug.ini

# Install Supervisor:
RUN apt-get update && apt-get install -y supervisor
COPY ./supervisor/ /etc/supervisor/conf.d/

# Crontab
COPY ./crontab /etc/cron.d
RUN chmod -R 644 /etc/cron.d

COPY ./crontab/cron.sh /usr/bin/cron.sh
RUN chmod +x /usr/bin/cron.sh

# Configuring ssh keys
COPY ./ssh/id_rsa /home/www-data/.ssh/
RUN chmod 600 /home/www-data/.ssh/id_rsa

COPY ./ssh/id_rsa.pub /home/www-data/.ssh/
RUN chmod 644 /home/www-data/.ssh/id_rsa.pub

COPY ./ssh/known_hosts /home/www-data/.ssh/known_hosts
RUN chmod 644 /home/www-data/.ssh/known_hosts
#RUN ssh-keyscan -H -t ecdsa gitlab.corp.loyalme.com, 10.88.1.12 >> /home/www-data/.ssh/known_hosts
RUN chown -R www-data:www-data /home/www-data/.ssh/

COPY ./laravel.pool.conf /usr/local/etc/php-fpm.d/

USER www-data

RUN composer global require "hirak/prestissimo"

COPY ./entry_point.sh /entry_point.sh
ENTRYPOINT ["/entry_point.sh"]

USER root

# Clean up
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* && \
    rm /var/log/lastlog /var/log/faillog

WORKDIR /var/www/project

CMD ["php-fpm"]

EXPOSE 9000
