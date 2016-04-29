# What is PHP?

PHP is a server-side scripting language designed for web development, but which can also be used as a general-purpose programming language. PHP can be added to straight HTML or it can be used with a variety of templating engines and web frameworks. PHP code is usually processed by an interpreter, which is either implemented as a native module on the web-server or as a common gateway interface (CGI).

How to install more PHP extensions

We provide the helper scripts docker-php-ext-configure, docker-php-ext-install, and docker-php-ext-enable to more easily install PHP extensions.

# PHP Core Extensions

For example, if you want to have a PHP-FPM image with iconv, mcrypt and gd extensions, you can inherit the base image that you like, and write your own Dockerfile like this:

```
FROM weldpua2008:php-5.3
RUN apt-get update && apt-get install -y \
        libfreetype6-dev \
        libjpeg62-turbo-dev \
        libmcrypt-dev \
        libpng12-dev \
    && docker-php-ext-install -j$(nproc) iconv mcrypt \
    && docker-php-ext-configure gd --with-freetype-dir=/usr/include/ --with-jpeg-dir=/usr/include/ \
    && docker-php-ext-install -j$(nproc) gd
```

Remember, you must install dependencies for your extensions manually. If an extension needs custom configure arguments, you can use the docker-php-ext-configure script like this example.

# PECL extensions

Some extensions are not provided with the PHP source, but are instead available through PECL. To install a PECL extension, use pecl install to download and compile it, then use docker-php-ext-enable to enable it:

```
FROM weldpua2008:php-5.3
RUN apt-get update && apt-get install -y libmemcached-dev \
    && pecl install memcached \
    && docker-php-ext-enable memcached
```

# Other extensions

Some extensions are not provided via either Core or PECL; these can be installed too, although the process is less automated:

```
FROM weldpua2008:php-5.3
RUN curl -fsSL 'https://xcache.lighttpd.net/pub/Releases/3.2.0/xcache-3.2.0.tar.gz' -o xcache.tar.gz \
    && mkdir -p xcache \
    && tar -xf xcache.tar.gz -C xcache --strip-components=1 \
    && rm xcache.tar.gz \
    && ( \
        cd xcache \
        && phpize \
        && ./configure --enable-xcache \
        && make -j$(nproc) \
        && make install \
    ) \
    && rm -r xcache \
    && docker-php-ext-enable xcache    
```

# Without a Dockerfile
If you don't want to include a Dockerfile in your project, it is sufficient to do the following:
$ docker run -d -p 80:80 --name my-apache-php-app -v "$PWD":/var/www/html weldpua2008:php-5.3