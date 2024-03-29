FROM alpine:3.19

ENV PHPIZE_DEPS \
        autoconf \
        dpkg dpkg-dev \
        file \
        clang \
        llvm \
        libc-dev \
        make \
        pkgconf \
        re2c

ARG GPG_KEYS="1198C0117593497A5EC5C199286AF1F9897469DC AFD8691FDAEDF03BDF6E460563F15A9B715376CA C28D937575603EB4ABB725861C0779DC5C0A9DE4"

ARG PHP_VERSION=8.3.4
ARG PHP_URL="https://www.php.net/distributions/php-8.3.4.tar.xz"
ARG PHP_ASC_URL="https://www.php.net/distributions/php-8.3.4.tar.xz.asc"
ARG PHP_SHA256="39a337036a546e5c28aea76cf424ac172db5156bd8a8fd85252e389409a5ba63"

ENV PHP_INI_DIR="/usr/local/etc/php"
ENV PHP_SCAN_DIR="$PHP_INI_DIR/conf.d"

ENV PHP_CFLAGS="-fstack-protector-strong -fpic -fpie -O3 \
    -D_LARGEFILE_SOURCE -D_LARGEFILE64_SOURCE -D_FILE_OFFSET_BITS=64 \
    -march=native \
    -funroll-loops \
    -ffast-math \
    -finline-functions \
"

ENV PHP_CPPFLAGS="$PHP_CFLAGS"
ENV PHP_LDFLAGS="-Wl,-O3 -pie"

COPY docker-entrypoint docker-php-* /usr/local/bin/

RUN set -eux; \
      # Create dirs & www-data user
        adduser -u 82 -D -S -G www-data www-data; \
        [ ! -d "$PHP_SCAN_DIR" ]; mkdir -p "$PHP_SCAN_DIR"; \
        [ ! -d /var/www/html ]; mkdir -p /var/www/html; \
      # Set the correct owner and permissions.
        chown www-data:www-data /var/www/html; \
        chmod 1777 /var/www/html; \
	\
	  # Install build tools
        apk add --no-cache --virtual .build-deps \
            $PHPIZE_DEPS \
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
            curl \
            make; \
    \
       # Export required vars.
         export \
           CFLAGS="$PHP_CFLAGS" \
           CPPFLAGS="$PHP_CPPFLAGS" \
           LDFLAGS="$PHP_LDFLAGS" \
           PHP_BUILD_PROVIDER='https://github.com/nuldarkk/php-fpm' \
           PHP_UNAME='Linux - Docker' \
         ; \
    \
       # Gets PHP sources \
         mkdir -p /usr/src; \
         cd /usr/src; \
         \
         curl -fsSL -o php.tar.xz "$PHP_URL"; \
         docker-php-source extract; \
         \
         cd /usr/src/php; \
    \
      # Configure the PHP build.
        ./configure CC=clang CXX=clang++ \
            --build="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
            --prefix="/usr/local/" \
            --sbin="/usr/local/sbin" \
            --sysconfdir="$PHP_INI_DIR" \
            --localstatedir=/var \
            --mandir=/usr/share/man \
            --with-layout=GNU \
            --with-config-file-path="$PHP_INI_DIR" \
            --with-config-file-scan-dir="$PHP_SCAN_DIR" \
            --config-cache \
            --enable-option-checking=fatal \
            --disable-gcc-global-regs \
            --disable-rpath \
            --without-sqlite3 \
            --without-cdb \
            --with-pear \
            --disable-cgi \
            --enable-fpm \
            --with-fpm-user=www-data \
            --with-fpm-group=www-data \
            --disable-phpdbg; \
    \
      # Build & install PHP
        make -j $(nproc); \
        make install; \
    \
        find /usr/local \
          -type f \
          -perm '/0111' \
          -exec sh -euxc ' \
            strip --strip-all "$@" || : \
          ' -- '{}' + \
        ; \
    \
      # Cleaning up a the build.
        make clean; \
        make distclean; \
    \
      # Copy php.ini to PHP_INI_DIR
        cp php.ini-production "$PHP_INI_DIR/php.ini"; \
    \
      # Remove PHP sources
        cd /; \
        rm -rf /usr/src/php; \
    \
      # Install required deps, remove build tools.
        runDeps="$( \
          scanelf --needed --nobanner --format '%n#p' --recursive /usr/local \
            | tr ',' '\n' \
            | sort -u \
            | awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
        )";  \
        apk add --no-cache $runDeps; \
        apk del --no-network .build-deps; \
    \
      # Update pecl
        pecl update-channels; \
        rm -rf /tmp/pear ~/.pearrc; \
    \
      # Smoke test
        php --version; \
    \
      # Prepare PHP configs to properly run
        cd "$PHP_INI_DIR"; \
        if [ -d php-fpm.d ]; then \
          sed 's!=NONE/!=!g' php-fpm.conf.default | tee php-fpm.conf > /dev/null; \
          cp php-fpm.d/www.conf.default php-fpm.d/www.conf; \
        fi; \
      # delete all *.default conf
        find "$PHP_INI_DIR" \
          -type f \
          -name '*.default' \
          -exec rm {} + ; \
      # clear apk cache
        rm -rf /var/cache/apk/*

ENTRYPOINT [ "docker-entrypoint" ]
WORKDIR /var/www/html

STOPSIGNAL SIGQUIT

EXPOSE 9000
CMD [ "php-fpm", "-F" ]
