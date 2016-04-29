FROM debian:wheezy

# based on https://raw.githubusercontent.com/docker-library/php/4677ca134fe48d20c820a19becb99198824d78e3/5.5/Dockerfile

MAINTAINER Valeriy Solovyov <weldpua2008@gmail.com>

# --enable-mysqlnd is included here because it's harder to compile after the fact than extensions are (since it's a plugin for several extensions, not an extension in itself)
# --enable-mbstring is included here because otherwise there's no way to get pecl to use it properly (see https://github.com/docker-library/php/issues/195)

RUN export DEBIAN_FRONTEND=noninteractive && \
	apt-get update && \
	apt-get install -y apt-utils autoconf file g++ 	gcc libc6-dev make pkg-config re2c --no-install-recommends	&& \
    apt-get install -y ca-certificates curl libedit2 libsqlite3-0 libxml2 --no-install-recommends && \
    export PHP_INI_DIR=/usr/local/etc/php && \
    mkdir -p $PHP_INI_DIR/conf.d && \
    export GPG_KEYS="0B96609E270F565C13292B24C13C70B87267B52D 0BD78B5F97500D450838F95DFE857D9A90D90EC1 F38252826ACD957EF380D39F2F7956BC5DA04B5D" && \
    export PHP_VERSION=5.3.27 && \
    export PHP_FILENAME=php-5.3.27.tar.bz2 && \
    export PHP_MD5=25ae23a5b9615fe8d33de5b63e1bb788 && \
  set -xe && \
	export buildDeps="$PHP_EXTRA_BUILD_DEPS libcurl4-openssl-dev libedit-dev libsqlite3-dev libssl-dev libxml2-dev xz-utils bzip2" && \
	apt-get install -y $buildDeps --no-install-recommends && \
	rm -rf /var/lib/apt/lists/* && \
	curl -fSL "http://php.net/get/$PHP_FILENAME/from/this/mirror" -o "$PHP_FILENAME" && \
	echo "$PHP_MD5 *$PHP_FILENAME" | md5sum -c - && \
	curl -fSL "http://php.net/get/$PHP_FILENAME.asc/from/this/mirror" -o "$PHP_FILENAME.asc" && \
	export GNUPGHOME="$(mktemp -d)" && \
	for key in $GPG_KEYS; do gpg --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-keys "$key"; done && \
	gpg --batch --verify "$PHP_FILENAME.asc" "$PHP_FILENAME" && \
	rm -r "$GNUPGHOME" "$PHP_FILENAME.asc" && \
	mkdir -p /usr/src/php && \
	tar jxf "$PHP_FILENAME" -C /usr/src/php --strip-components=1 && \
	rm "$PHP_FILENAME" && \
	cd /usr/src/php && \
	./configure --with-config-file-path="$PHP_INI_DIR" \
	--with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
	--disable-cgi \
	--enable-mysqlnd \
	--enable-mbstring \
	--with-curl \
	--with-libedit \
	--with-openssl \
	--with-zlib $PHP_EXTRA_CONFIGURE_ARGS && \
	make -j"$(nproc)" && \
	make install && \
	{ find /usr/local/bin /usr/local/sbin -type f -executable -exec strip --strip-all '{}' + || true; } && \
	make clean && \
	apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false -o APT::AutoRemove::SuggestsImportant=false $buildDeps && \
	rm -r /var/lib/apt/lists/*
COPY docker-php-ext-* /usr/local/bin/

CMD ["php", "-a"]

