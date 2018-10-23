# See https://github.com/docker-library/php/blob/master/7.2/fpm/Dockerfile
FROM php:7.2-fpm

ARG TIMEZONE
ARG XDBG_IDEKEY
ARG XDBG_REMOTE_PORT
ARG HOST_WWW_UID
ARG HOST_WWW_GID

# Make sure 'www-data' IDs map to a host user.
RUN usermod -u ${HOST_WWW_UID} www-data
RUN groupmod -g ${HOST_WWW_GID} www-data

# Install base utilities.
RUN apt-get update && apt-get install -y \
    libicu-dev vim \
    git \
    libpng-dev \
    unzip libjpeg62-turbo-dev libfreetype6-dev \
    libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# Install PHP Extensions.
RUN docker-php-ext-configure pgsql -with-pgsql=/usr/local/pgsql \
    && docker-php-ext-install pdo pdo_pgsql pgsql zip

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
RUN composer --version

# Install code sniffer
# for Drupal (with coder)
# RUN composer global require drupal/coder \
#    && export PATH="/root/.composer/vendor/bin" \
#    && phpcs --config-set installed_paths ~/.composer/vendor/drupal/coder/coder_sniffer \
#    && phpcs -i

# For Other projects
 RUN pear install PHP_CodeSniffer \\
    && phpcs -i

# Set timezone
RUN ln -snf /usr/share/zoneinfo/${TIMEZONE} /etc/localtime && echo ${TIMEZONE} > /etc/timezone
RUN printf '[PHP]\ndate.timezone = "%s"\n', ${TIMEZONE} > /usr/local/etc/php/conf.d/tzone.ini
RUN "date"

# Install PSQL client (necessary for running drush commands).
RUN  mkdir -p /usr/share/man/man1 \
    && mkdir -p /usr/share/man/man7

RUN apt-get update && apt-get install -y \
    postgresql-client \
 && rm -rf /var/lib/apt/lists/*

# Install and configure xdebug.
RUN pecl install xdebug
RUN docker-php-ext-enable xdebug
RUN echo "error_reporting = E_ALL" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "display_startup_errors = On" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "display_errors = On" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_enable=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_connect_back=1" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.idekey=\"${XDBG_IDEKEY}\"" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.remote_port=${XDBG_REMOTE_PORT}" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini
RUN echo "xdebug.max_nesting_level=10000" >> /usr/local/etc/php/conf.d/docker-php-ext-xdebug.ini

WORKDIR /var/www/code

# Aliases.
RUN echo 'alias ll="ls -alh"' >> ~/.bashrc
RUN echo 'alias sf="php bin/console"' >> ~/.bashrc

# PATH adjustemsnts.
# Add global composer binaries to the path (makes drush avaliable, among other things).
ENV PATH="/root/.composer/vendor/bin/:${PATH}"
