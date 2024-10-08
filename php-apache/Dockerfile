FROM php:8.3.12-apache

LABEL maintainer="Sohel Rana <me.sohelrana@gmail.com>"

# Setup docker arguments.
ARG HTTP_PROXY=""
ARG HTTPS_PROXY=""
ARG NO_PROXY="localhost,127.0.0.1"

# Setup some enviornment variables.
ENV http_proxy="${HTTP_PROXY}" \
    https_proxy="${HTTPS_PROXY}" \
    no_proxy="${NO_PROXY}"

USER root

# Install & manage application dependencies
RUN echo "-- Install/Upgrade APT Dependencies --" \
        && apt-get update \
        && apt-get upgrade -y \
    && echo "-- Install PHP Extensions --" \
        && install-php-extensions \
            pcov \
            redis \
            gd \
    && echo "--- Setting Up Apache ---" \
        && a2enmod rewrite ssl headers \
    && echo "--- Clean Up ---" \
        && apt-get clean -y \
        && apt-get autoremove -y \
        && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/* \
        && rm /var/log/lastlog /var/log/faillog

WORKDIR /var/www/html

# Apache envs
ENV APACHE_ENABLE_HTTP_TRAFFIC="true" \
    APACHE_ENABLE_HTTPS_TRAFFIC="false" \
    APACHE_RUN_USER="www-data" \
    APACHE_RUN_GROUP="www-data" \
    APACHE_VHOST_SERVER_ADMIN="webmaster@example.com" \
    APACHE_VHOST_SERVER_NAME="_" \
    APACHE_VHOST_DOCUMENT_ROOT="/var/www/html/public" \
    APACHE_VHOST_HTTP_PORT="8080" \
    APACHE_VHOST_HTTPS_PORT="8443" \
    APACHE_VHOST_SSL_CERTIFICATE_CRT="/etc/ssl/certs/default.pem" \
    APACHE_VHOST_SSL_CERTIFICATE_KEY="/etc/ssl/certs/default.key"

# PHP INI envs
ENV PHP_INI_PCOV_ENABLED="0" \
    PHP_INI_PCOV_DIRECTORY="/var/www/html" \
    PHP_INI_PCOV_EXCLUDE="~(bootstrap|config|deploy|docker|node_modules|public|resources|routes|storage|tests|vendor|.php-cs-fixer.dist.php|server.php)~" \
    PHP_INI_PCOV_INITIAL_MEMORY="65536" \
    PHP_INI_PCOV_INITIAL_FILES="64"

COPY ./php/php-pcov.ini /usr/local/etc/php/conf.d
COPY --chown=www-data:www-data ./composer.json ./composer.lock* /var/www/html/

# Installing the packages here to cache them
# So further installs from child images will
# download less / no dependencies.
RUN export composerInstallArgs="--no-interaction --no-scripts --no-autoloader --prefer-dist" \
    && composer install $composerInstallArgs

COPY ./run-scheduler.sh /usr/local/bin/

RUN chmod ugo+x /usr/local/bin/run-scheduler.sh \
    && unset http_proxy \
    && unset https_proxy \
    && unset no_proxy

CMD ["apache2-foreground"]

USER www-data
