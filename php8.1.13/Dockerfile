FROM alpine:3.17

ENV PHPIZE_DEPS \
	autoconf \
	dpkg-dev \
	dpkg \
	file \
	g++ \
	gcc \
	libc-dev \
	make \
	pkgconf \
	re2c 

ENV PHP_INI_DIR /usr/local/etc/php

ENV PHP_VERSION 8.1.13

ENV PHP_CFLAGS="-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64"

ENV PHP_CPPFLAGS="$PHP_CFLAGS"

ENV PHP_LDFLAGS="-Wl,-O1 -pie"

ENV PHP_INSTALL \
	bcmath \
	calendar \
	dba \
	mysqli \
	pcntl  \
	pdo_mysql \
	shmop \
	sockets \
	sysvmsg \
	sysvsem \
	sysvshm \
	opcache \
	zip \
	gettext \
	exif

# persistent / runtime deps
RUN set -eux; \
	#sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories; \
	sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories; \
	#sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories; \
	adduser -u 82 -D -S -G www-data www-data; \
	mkdir -p "$PHP_INI_DIR/conf.d"; \
	[ ! -d /var/www ]; \
	mkdir -p /var/www; \
	chown www-data:www-data /var/www; \
	chmod 777 /var/www; \
	[ ! -d /usr/src ]; \
	mkdir -p /usr/src

COPY docker-php-source /usr/local/bin/

COPY php.tar.xz  /usr/src

COPY php-amqp.tar.gz /usr/src

COPY php-ext-encry-1.0.1.tar.gz /usr/src

COPY docker-php-ext-* docker-php-entrypoint /usr/local/bin/

RUN set -eux; \
	apk add --no-cache --virtual .build-deps \
	$PHPIZE_DEPS \
	ca-certificates \
	curl \
	tar \
	xz \
	openssl \
	argon2-dev \
	coreutils \
	curl-dev \
	gnu-libiconv-dev \
	libsodium-dev \
	libxml2-dev \
	linux-headers \
	oniguruma-dev \
	openssl-dev \
	readline-dev \
	sqlite-dev \
	gettext-dev \
	libmcrypt-dev \
	libzip-dev \
	cmake \
	rabbitmq-c-dev \
	zlib-dev \
	libpng-dev \
	tzdata; \
	cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime;\
	echo "Asia/Shanghai" > /etc/timezone;\
	rm -vf /usr/include/iconv.h; \
	export \
	CFLAGS="$PHP_CFLAGS" \
	CPPFLAGS="$PHP_CPPFLAGS" \
	LDFLAGS="$PHP_LDFLAGS" \
	; \
	docker-php-source extract; \
	cd /usr/src/php; \
	gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)"; \
	./configure \
	--build="$gnuArch" \
	--with-config-file-path="$PHP_INI_DIR" \
	--with-config-file-scan-dir="$PHP_INI_DIR/conf.d" \
	--enable-option-checking=fatal \
	--with-mhash \
	--with-pic \
	--enable-ftp \
	--enable-mbstring \
	--enable-mysqlnd \
	--with-password-argon2 \
	--with-sodium=shared \
	--with-pdo-sqlite=/usr \
	--with-sqlite3=/usr \
	--with-curl \
	--with-iconv=/usr \
	--with-openssl \
	--with-readline \
	--with-zlib \
	--disable-phpdbg \
	--with-pear \
	$(test "$gnuArch" = 's390x-linux-musl' && echo '--without-pcre-jit') \
	--disable-cgi \
	--enable-fpm \
	--with-fpm-user=www-data \
	--with-fpm-group=www-data \
	; \
	make -j "$(nproc)"; \
	make install; \
	cp -v php.ini-* "$PHP_INI_DIR/"; \
	cd /usr/src; \
	tar -zxvf /usr/src/php-amqp.tar.gz; \
	cd /usr/src/php-amqp-1.11.0; \
	/usr/local/bin/phpize; \
	./configure  --with-php-config=/usr/local/bin/php-config; \
	make -j "$(nproc)"; \
	make install; \
	mkdir -p /usr/src/php-ext-encry-1.0.1; \
	tar -zxvf /usr/src/php-ext-encry-1.0.1.tar.gz -C /usr/src; \
	cd  /usr/src/php-ext-encry-1.0.1; \
	/usr/local/bin/phpize; \
	./configure  --with-php-config=/usr/local/bin/php-config; \
	make -j "$(nproc)"; \
	make install; \
	docker-php-ext-install \
	${PHP_INSTALL}; \
	pecl update-channels; \
	pecl install mcrypt \
	protobuf \
	redis ;\
	docker-php-source delete; \
	docker-php-ext-enable sodium \
	amqp \
	mcrypt \
	protobuf \
	redis \
	yaoling_encrypt \
	exif; \
	runDeps="$( \
	scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
	| tr ',' '\n' \
	| sort -u \
	| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)"; \
	apk add --no-cache $runDeps; \
	apk del --no-network .build-deps; \
	rm -rf /usr/src/php-ext-encry*; \
	rm -rf /usr/src/php-amqp*; \
	rm -rf /tmp/pear ~/.pearrc; \
	php --version;\
	cd /usr/local/etc; \
	if [ -d php-fpm.d ]; then \
	sed 's!=NONE/!=!g' php-fpm.conf.default | tee php-fpm.conf > /dev/null; \
	cp php-fpm.d/www.conf.default php-fpm.d/www.conf; \
	else \
	mkdir php-fpm.d; \
	cp php-fpm.conf.default php-fpm.d/www.conf; \
	{ \
	echo '[global]'; \
	echo 'include=etc/php-fpm.d/*.conf'; \
	} | tee php-fpm.conf; \
	fi; \
	{ \
	echo '[global]'; \
	echo 'error_log = /proc/self/fd/2'; \
	echo; echo '; https://github.com/docker-library/php/pull/725#issuecomment-443540114'; echo 'log_limit = 8192'; \
	echo; \
	echo '[www]'; \
	echo '; if we send this to /proc/self/fd/1, it never appears'; \
	echo 'access.log = /proc/self/fd/2'; \
	echo; \
	echo 'clear_env = no'; \
	echo; \
	echo '; Ensure worker stdout and stderr are sent to the main error log.'; \
	echo 'catch_workers_output = yes'; \
	echo 'decorate_workers_output = no'; \
	} | tee php-fpm.d/docker.conf; \
	{ \
	echo '[global]'; \
	echo 'daemonize = no'; \
	echo; \
	echo '[www]'; \
	echo 'listen = 9000'; \
	} | tee php-fpm.d/zz-docker.conf

RUN  set -eux; \
	apk add  libpng-dev zlib-dev ; \
	docker-php-ext-install gd; \
	apk add --no-cache --virtual .build-deps \
	$PHPIZE_DEPS linux-headers; \
	pecl update-channels; \
	pecl install xdebug; \
	docker-php-ext-enable gd xdebug ;\
	apk del --no-network .build-deps ;\
	rm -rf /tmp/pear ~/.pearrc


ENTRYPOINT ["docker-php-entrypoint"]

WORKDIR /var/www

STOPSIGNAL SIGQUIT

EXPOSE 9000
CMD ["php-fpm"]
