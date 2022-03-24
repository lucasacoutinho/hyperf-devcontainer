# Update the ALPINE_VERSION arg in docker-compose.yml
ARG ALPINE_VERSION="3.15"
FROM hyperf/hyperf:8.1-alpine-v${ALPINE_VERSION}-base
LABEL maintainer="Hyperf Developers <group@hyperf.io>" version="1.0" license="MIT"

# Options for common package install script
ARG USERNAME="none"
ARG INSTALL_ZSH="true"
ARG INSTALL_OH_MYS="true"
ARG COMMON_SCRIPT_SOURCE="https://raw.githubusercontent.com/microsoft/vscode-dev-containers/main/script-library/common-alpine.sh"

RUN curl -sSL ${COMMON_SCRIPT_SOURCE} -o /tmp/common-setup.sh \
  && ash /tmp/common-setup.sh "${INSTALL_ZSH}" "${USERNAME}" "${USER_UID}" "${USER_GID}" "${INSTALL_OH_MYS}" \
  && rm -rf /tmp/common-setup.sh

# Composer install
ARG COMPOSER_VERSION="2.2.9"
RUN wget -nv -O /usr/local/bin/composer https://github.com/composer/composer/releases/download/${COMPOSER_VERSION}/composer.phar \
    && chmod +x /usr/local/bin/composer

# Swoole dependencies
ENV PHPIZE_DEPS="autoconf dpkg-dev dpkg file g++ gcc libc-dev make php8-dev php8-pear pkgconf re2c pcre-dev pcre2-dev zlib-dev libtool automake"
RUN set -ex \
    && apk update \
    && apk add --no-cache postgresql-dev libstdc++ openssl git bash \
    && apk add --no-cache --virtual .build-deps $PHPIZE_DEPS libaio-dev openssl-dev curl-dev libpq-dev

# Install Swoole extension
ARG SW_VERSION="v4.8.8"
ENV SW_VERSION=${SW_VERSION}
RUN set -ex \
    && apk update \
    && cd /tmp \
    && curl -SL "https://github.com/swoole/swoole-src/archive/${SW_VERSION}.tar.gz" -o swoole.tar.gz \
    && cd /tmp \
    && mkdir -p swoole \
    && tar -xf swoole.tar.gz -C swoole --strip-components=1 \
    && ln -s /usr/bin/phpize8 /usr/local/bin/phpize \
    && ln -s /usr/bin/php-config8 /usr/local/bin/php-config \
    && ( \
        cd swoole \
        && phpize \
        && ./configure --enable-openssl --enable-http2 --enable-swoole-curl --enable-swoole-json \
        && make -s -j$(nproc) && make install \
    ) \
    && echo "memory_limit=1G" > /etc/php8/conf.d/00_default.ini \
    && echo "opcache.enable_cli = 'On'" >> /etc/php8/conf.d/00_opcache.ini \
    && echo "extension=swoole.so" > /etc/php8/conf.d/50_swoole.ini \
    && echo "swoole.use_shortname = 'Off'" >> /etc/php8/conf.d/50_swoole.ini \
    && php -v \
    && php -m \
    && php --ri swoole \
    && php --ri Zend\ OPcache

# Install Postgres extension
RUN set -ex \
    && apk update \
    && cd /tmp \
    && git clone "https://github.com/swoole/ext-postgresql.git" \
    && ( \
        cd ext-postgresql \
        && phpize \
        && ./configure \
        && make -s -j$(nproc) && make install \
    )\
    && echo "extension=swoole_postgresql.so" > /etc/php8/conf.d/51_swoole_postgresql.ini \
    && php -v \
    && php -m \
    && php --ri swoole_postgresql

# Cleanup
RUN apk del .build-deps \
    && rm -rf /var/cache/apk/* /tmp/* /usr/share/man /usr/local/bin/php* \
    && echo -e "\033[42;37m Build Completed :).\033[0m\n"
