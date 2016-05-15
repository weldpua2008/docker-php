FROM weldpua2008:5.3.27-fpm

MAINTAINER Valeriy Solovyov <weldpua2008@gmail.com>

VOLUME /var/www/html
RUN export DEBIAN_FRONTEND=noninteractive && \
	apt-get update && \
	apt-get install -y libpng12-dev libjpeg-dev && \
	rm -rf /var/lib/apt/lists/* && \
	docker-php-ext-configure gd --with-png-dir=/usr --with-jpeg-dir=/usr && \
	docker-php-ext-install gd mysqli opcache && \
	{ \
		echo 'opcache.memory_consumption=128'; \
		echo 'opcache.interned_strings_buffer=8'; \
		echo 'opcache.max_accelerated_files=4000'; \
		echo 'opcache.revalidate_freq=60'; \
		echo 'opcache.fast_shutdown=1'; \
		echo 'opcache.enable_cli=1'; \
	} > /usr/local/etc/php/conf.d/opcache-recommended.ini && \
	curl -o wordpress.tar.gz -SL https://wordpress.org/wordpress-${WORDPRESS_VERSION}.tar.gz && \
	echo "$WORDPRESS_SHA1 *wordpress.tar.gz" | sha1sum -c - && \
	tar -xzf wordpress.tar.gz -C /usr/src/ && \
	rm wordpress.tar.gz && \
	chown -R www-data:www-data /usr/src/wordpress

COPY docker-entrypoint.sh /entrypoint.sh

# grr, ENTRYPOINT resets CMD now
ENTRYPOINT ["/entrypoint.sh"]
CMD ["php-fpm"]
