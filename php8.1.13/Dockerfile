#
# NOTE: THIS DOCKERFILE IS GENERATED VIA "apply-templates.sh"
#
# PLEASE DO NOT EDIT IT DIRECTLY.
#

FROM debian:buster-slim

# prevent Debian's PHP packages from being installed
# https://github.com/docker-library/php/pull/542
RUN set -eux; \
	{ \
	echo 'Package: php*'; \
	echo 'Pin: release *'; \
	echo 'Pin-Priority: -1'; \
	} > /etc/apt/preferences.d/no-debian-php

# dependencies required for running "phpize"
# (see persistent deps below)
ENV PHPIZE_DEPS \
	autoconf \
	dpkg-dev \
	file \
	g++ \
	gcc \
	libc-dev \
	make \
	pkg-config \
	re2c

# persistent / runtime deps
RUN set -eux; \
	# echo "# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释" > /etc/apt/sources.list;\
	# echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free" >> /etc/apt/sources.list;\
	# echo "deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ buster main contrib non-free" >> /etc/apt/sources.list;\
	# echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free" >> /etc/apt/sources.list;\
	# echo "deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free" >>  etc/apt/sources.list; \
	# echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free" >> /etc/apt/sources.list;\
	# echo "deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ buster-backports main contrib non-free" >> /etc/apt/sources.list;\
	# echo "deb https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free" >> /etc/apt/sources.list;\
	# echo "deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security buster/updates main contrib non-free" >>  /etc/apt/sources.list; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
	$PHPIZE_DEPS \
	ca-certificates \
	curl \
	xz-utils \
	openssl \
	libargon2-dev \
	coreutils \
	libticonv-dev \
	libsodium-dev \
	libxml2-dev \
	libmcrypt-dev \
	libzip-dev \
	cmake \
	librabbitmq-dev \
	libpng-dev \
	tzdata; \
	cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime;\
	echo "Asia/Shanghai" > /etc/timezone;\
	rm -rf /var/lib/apt/lists/*

ENV PHP_INI_DIR /usr/local/etc/php

ENV PHP_VERSION 8.1.13

ENV PHP_CFLAGS="-fstack-protector-strong -fpic -fpie -O2 -D_LARGEFILE_SOURCE -D_FILE_OFFSET_BITS=64"

ENV PHP_CPPFLAGS="$PHP_CFLAGS"

ENV PHP_LDFLAGS="-Wl,-O1 -pie"

ENV ENCRY_VERSION=3.1.1

ENV SKYWALKING_VERSION=1.0.0

ENV CPP_ENCRY_FILE="yaoling_encrypt_plus-${ENCRY_VERSION}"

ENV RUST_SKYWALKING_FILE="php-skywalking-agent-${SKYWALKING_VERSION}"


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


RUN set -eux; \
	mkdir -p "$PHP_INI_DIR/conf.d"; \
	# allow running as an arbitrary user (https://github.com/docker-library/php/issues/743)
	[ ! -d /var/www ]; \
	mkdir -p /var/www; \
	chown www-data:www-data /var/www; \
	chmod 777 /var/www; 


COPY docker-php-source /usr/local/bin/

COPY php.tar.xz  /usr/src

COPY php-amqp.tar.gz /usr/src

COPY "${CPP_ENCRY_FILE}.tar.gz" /usr/src

COPY "${RUST_SKYWALKING_FILE}.tar.gz" /usr/src

COPY docker-php-ext-* docker-php-entrypoint /usr/local/bin/

RUN set -eux; \
	\
	savedAptMark="$(apt-mark showmanual)"; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
	libargon2-dev \
	libcurl4-openssl-dev \
	libonig-dev \
	libreadline-dev \
	libsodium-dev \
	libsqlite3-dev \
	libssl-dev \
	libxml2-dev \
	zlib1g-dev \
	; \
	export \
	CFLAGS="$PHP_CFLAGS" \
	CPPFLAGS="$PHP_CPPFLAGS" \
	LDFLAGS="$PHP_LDFLAGS" \
	; \
	docker-php-source extract; \
	cd /usr/src/php; \
	gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)"; \
	debMultiarch="$(dpkg-architecture --query DEB_BUILD_MULTIARCH)"; \
	# https://bugs.php.net/bug.php?id=74125
	if [ ! -d /usr/include/curl ]; then \
	ln -sT "/usr/include/$debMultiarch/curl" /usr/local/include/curl; \
	fi; \
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
	--with-iconv \
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
	--enable-fd-setsize=3096 \
	; \
	make -j "$(nproc)"; \
	make install; \
	cp -v php.ini-* "$PHP_INI_DIR/"; \
	cp -v php.ini-production "$PHP_INI_DIR/php.ini"; \
	cd /usr/src; \
	tar -zxvf /usr/src/php-amqp.tar.gz; \
	cd /usr/src/php-amqp-1.11.0; \
	/usr/local/bin/phpize; \
	./configure  --with-php-config=/usr/local/bin/php-config; \
	make -j "$(nproc)"; \
	make install; \
	mkdir -p "/usr/src/${CPP_ENCRY_FILE}"; \
	tar -zxvf "/usr/src/${CPP_ENCRY_FILE}.tar.gz" -C /usr/src; \
	cd  "/usr/src/${CPP_ENCRY_FILE}"; \
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
	yaoling_encrypt_plus \
	exif; \
	runDeps="$( \
	scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
	| tr ',' '\n' \
	| sort -u \
	| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)"; \
	apt-get install -y --no-install-recommends  $runDeps; \
	rm -rf /var/lib/apt/lists/* \
	rm -rf /usr/src/${CPP_ENCRY_FILE}*; \
	rm -rf /usr/src/php-amqp*; \
	rm -rf /tmp/pear ~/.pearrc; \
	php --version;

COPY docker-php-ext-* docker-php-entrypoint /usr/local/bin/

# sodium was built as a shared module (so that it can be replaced later if so desired), so let's enable it too (https://github.com/docker-library/php/issues/598)
RUN docker-php-ext-enable sodium

ENTRYPOINT ["docker-php-entrypoint"]
WORKDIR /home/www-data

RUN set -eux; \
	apt-get update;\
	apt-get install nodejs npm -y;\
	npm install pm2 -g;\
	cd /usr/local/etc; \
	if [ -d php-fpm.d ]; then \
	# for some reason, upstream's php-fpm.conf.default has "include=NONE/etc/php-fpm.d/*.conf"
	sed 's!=NONE/!=!g' php-fpm.conf.default | tee php-fpm.conf > /dev/null; \
	cp php-fpm.d/www.conf.default php-fpm.d/www.conf; \
	else \
	# PHP 5.x doesn't use "include=" by default, so we'll create our own simple config that mimics PHP 7+ for consistency
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
	} | tee php-fpm.d/zz-docker.conf; \
	mkdir -p "$PHP_INI_DIR/conf.d"; \
	{ \
	echo '; https://github.com/docker-library/php/issues/878#issuecomment-938595965'; \
	echo 'fastcgi.logging = Off'; \
	} > "$PHP_INI_DIR/conf.d/docker-fpm.ini";


RUN set -eux; \
	export CPU_COUNT="$(nproc)"; \
    export FIVE_CPU=$(expr $CPU_COUNT \* 5); \
	sed -i  "s/pm.max_children = 5/pm.max_children = $FIVE_CPU/g" "/usr/local/etc/php-fpm.d/www.conf"; \
	sed -i  's/pm = dynamic/pm = static/g' "/usr/local/etc/php-fpm.d/www.conf"; \
	sed -i  's/expose_php = On/expose_php = Off /g' "$PHP_INI_DIR/php.ini"; \
	echo 'yaoling.encrypt_path=/home/www-data/pms/app/Services' >> /usr/local/etc/php/conf.d/docker-php-ext-yaoling_encrypt_plus.ini; 

ENV RUSTUP_HOME=/usr/local/rustup \
    CARGO_HOME=/usr/local/cargo \
    PATH=/usr/local/cargo/bin:$PATH \
    RUST_VERSION=1.65.0

#skywalking_agent
RUN set -eux; \
	apt-get update; \
	apt-get install -y wget gcc make llvm-13-dev libclang-13-dev protobuf-compiler; \
	wget https://static.rust-lang.org/rustup/archive/1.25.1/x86_64-unknown-linux-gnu/rustup-init; \
	chmod +x rustup-init; \
	./rustup-init -y --no-modify-path --profile minimal --default-toolchain $RUST_VERSION --default-host x86_64-unknown-linux-gnu; \
	rm rustup-init; \
	chmod -R a+w $RUSTUP_HOME $CARGO_HOME; \
	mkdir -p "/usr/src/${RUST_SKYWALKING_FILE}"; \
	tar -zxvf "/usr/src/${RUST_SKYWALKING_FILE}.tar.gz" -C /usr/src; \
	cd  "/usr/src/${RUST_SKYWALKING_FILE}"; \
	/usr/local/bin/phpize; \
	./configure  --with-php-config=/usr/local/bin/php-config; \
	make -j "$(nproc)"; \
	make install; \
	docker-php-ext-enable skywalking_agent; \
	apt-get remove -y --auto-remove wget protobuf-compiler; \
	rm -rf /var/lib/apt/lists/*; \
	rm -rf /usr/src/${RUST_SKYWALKING_FILE}* ;

#composer
RUN set -eux; \
	php -r "copy('https://install.phpcomposer.com/installer', 'composer-setup.php');" ;\
	php composer-setup.php; \
	php -r "unlink('composer-setup.php');"; \
	mv composer.phar /usr/local/bin/composer; 

# Override stop signal to stop process gracefully
# https://github.com/php/php-src/blob/17baa87faddc2550def3ae7314236826bc1b1398/sapi/fpm/php-fpm.8.in#L163
STOPSIGNAL SIGQUIT

EXPOSE 9000
CMD ["php-fpm"]