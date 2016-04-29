FROM debian:jessie
# based on https://github.com/docker-library/php/blob/master/7.0/fpm/Dockerfile

#
RUN export DEBIAN_FRONTEND=noninteractive && \
    apt-get update && apt-get install -y autoconf file g++ gcc libc6-dev make pkg-config re2c --no-install-recommends && \\
    echo "Installing  persistent / runtime deps " && \
    apt-get install -y ca-certificates curl libedit2 libsqlite3-0 libxml2 --no-install-recommends && \
    export PHP_INI_DIR="/usr/local/etc/php" && \
    export PHP_EXTRA_CONFIGURE_ARGS=" --enable-fpm --with-fpm-user=www-data --with-fpm-group=www-data" && \
    export GPG_KEYS="1A4E8B7277C42E53DBA9C7B9BCAA30EA9C0D5763"
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
	rm "$PHP_FILENAME" && \
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
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false $buildDeps && \
    set -ex && \
    cd /usr/local/etc && \
    if [ -d php-fpm.d ]; then echo "for some reason, upstream's php-fpm.conf.default has \"include=NONE/etc/php-fpm.d/*.conf\" && \
        sed 's!=NONE/!=!g' php-fpm.conf.default | tee php-fpm.conf > /dev/null; \
        cp php-fpm.d/www.conf.default php-fpm.d/www.conf; \
	else \
		echo "PHP 5.x don't use \"include=\" by default, so we'll create our own simple config that mimics PHP 7+ for consistency"; \
		mkdir php-fpm.d; \
		cp php-fpm.conf.default php-fpm.d/www.conf; \
		{ \
			echo '[global]'; \
			echo 'include=etc/php-fpm.d/*.conf'; \
		} | tee php-fpm.conf; \
	fi \
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
	} | tee php-fpm.d/docker.conf \
	&& { \
		echo '[global]'; \
		echo 'daemonize = no'; \
		echo; \
		echo '[www]'; \
		echo 'listen = [::]:9000'; \
	} | tee php-fpm.d/zz-docker.conf


COPY docker-php-ext-* /usr/local/bin/
WORKDIR /var/www/html

EXPOSE 9000
CMD ["php-fpm"]
