FROM alpine:latest
ARG UID=33
ARG GID=33

# ensure www-data user exists with uid 48 for use on Fedora
RUN set -xe \
  delgroup xfs \
  && deluser xfs \
  && addgroup -g $GID -S www-data \
  && adduser -u $UID -D -S -G www-data www-data

RUN apk add --no-cache \
  git \
  php7 php7-fpm php7-curl php7-ctype php7-dom php7-exif php7-fileinfo \
  php7-gd php7-iconv php7-json php7-mbstring php7-mcrypt \
  php7-mysqli php7-pcntl php7-pdo php7-pdo_mysql php7-pdo_sqlite php7-phar \
  php7-posix php7-session php7-simplexml php7-sqlite3 php7-tokenizer \
  php7-xml php7-xmlwriter php7-zlib \
  ssmtp

# set desired PHP settings
RUN set -ex \
  && { \
    echo '[global]'; \
    echo 'error_log = /proc/self/fd/2'; \
    echo; \
    echo '[www]'; \
    echo '; if we send this to /proc/self/fd/1, it never appears'; \
    echo 'access.log = /proc/self/fd/2'; \
    echo; \
    echo 'clear_env = no'; \
    echo; \
    echo '; Ensure worker stdout and stderr are sent to the main error log.'; \
    echo 'catch_workers_output = yes'; \
  } > /etc/php7/php-fpm.d/docker.conf \
  && { \
    echo '[global]'; \
    echo 'daemonize = no'; \
    echo; \
    echo '[www]'; \
    echo 'listen = [::]:9000'; \
    echo 'user = www-data'; \
    echo 'group = www-data'; \
  } > /etc/php7/php-fpm.d/zz-docker.conf \
  && { \
    # see https://secure.php.net/manual/en/opcache.installation.php
    echo 'opcache.memory_consumption=128'; \
    echo 'opcache.interned_strings_buffer=8'; \
    echo 'opcache.max_accelerated_files=4000'; \
    echo 'opcache.revalidate_freq=2'; \
    echo 'opcache.fast_shutdown=1'; \
    echo 'opcache.enable_cli=1'; \
  } > /etc/php7/conf.d/01_opcache-recommended.ini \
  && { \
    echo 'sendmail_path = "/usr/sbin/ssmtp -t"'; \
  } > /etc/php7/conf.d/99_sendmail.ini \
  && { \
    # Add .conf file to set memory limits:
    echo 'post_max_size=12M'; \
    echo 'upload_max_filesize=10M'; \
  } > /etc/php7/conf.d/99_memory.ini \
  && { \
    # increase max execution time, to allow for slower uploads
    echo 'max_execution_time=240'; \
  } > /etc/php7/conf.d/99_max_execution.ini

# configure SSMTP and set it up for use with PHP
COPY ssmtp/* /etc/ssmtp/

#VOLUME /var/www/html
VOLUME /var/www

COPY docker-entrypoint.sh /usr/local/bin/

ENTRYPOINT ["docker-entrypoint.sh"]
EXPOSE 9000
CMD ["php-fpm7"]
