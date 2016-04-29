FROM debian:jessie
# based on https://github.com/docker-library/php/blob/master/7.0/fpm/Dockerfile

MAINTAINER Valeriy Solovyov <weldpua2008@gmail.com>

COPY docker-php-ext-* /usr/local/bin/
#
RUN export DEBIAN_FRONTEND=noninteractive && \
    apt-get update && apt-get install -y autoconf file g++ gcc libc6-dev make pkg-config re2c --no-install-recommends && \
    echo "Installing  persistent / runtime deps " && \
    apt-get install -y ca-certificates curl libedit2 libsqlite3-0 libxml2 --no-install-recommends && \
    export PHP_INI_DIR="/usr/local/etc/php" && \
    export GPG_KEYS="1A4E8B7277C42E53DBA9C7B9BCAA30EA9C0D5763" && \
    export PHP_VERSION=7.0.5 && \
    export PHP_FILENAME=php-7.0.5.tar.xz && \
    export PHP_SHA256=c41f1a03c24119c0dd9b741cdb67880486e64349fc33527767f6dc28d3803abb && \
    mkdir -p $PHP_INI_DIR/conf.d && \
    set -xe && \
    export buildDeps="$PHP_EXTRA_BUILD_DEPS libcurl4-openssl-dev libedit-dev libsqlite3-dev libssl-dev libxml2-dev xz-utils " && \
    echo "Installing build deps: $buildDeps" && \
    apt-get install -y $buildDeps --no-install-recommends && \
    curl -fSL "http://php.net/get/$PHP_FILENAME/from/this/mirror" -o "$PHP_FILENAME" && \
    echo "$PHP_SHA256 *$PHP_FILENAME" | sha256sum -c -  && \
    curl -fSL "http://php.net/get/$PHP_FILENAME.asc/from/this/mirror" -o "$PHP_FILENAME.asc" && \
    export GNUPGHOME="$(mktemp -d)" && \
    for key in $GPG_KEYS; do gpg --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-keys "$key"; done && \
    gpg --batch --verify "$PHP_FILENAME.asc" "$PHP_FILENAME" && \
    rm -r "$GNUPGHOME" "$PHP_FILENAME.asc" && \
    mkdir -p /usr/src/php && \
    tar -xf "$PHP_FILENAME" -C /usr/src/php --strip-components=1 && \
    rm -f "$PHP_FILENAME" && \
    cd /usr/src/php && \
    ./configure --with-config-file-path="$PHP_INI_DIR" \
    --with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
    --disable-cgi --enable-mysqlnd \
    --enable-mbstring \
    --with-curl \
    --with-libedit \
    --with-openssl \
    --with-zlib $PHP_EXTRA_CONFIGURE_ARGS  && \
    make -j"$(nproc)" && \
    make install && \
    { find /usr/local/bin /usr/local/sbin -type f -executable -exec strip --strip-all '{}' + || true; }  && \
    make clean && \
    apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false $buildDeps

CMD ["php", "-a"]
