FROM alpine:3.4
MAINTAINER Wonderfall <wonderfall@schrodinger.io>

ARG NEXTCLOUD_VERSION=10.0.0
ARG GNU_LIBICONV_VERSION=1.14
ARG GPG_nextcloud="2880 6A87 8AE4 23A2 8372  792E D758 99B9 A724 937A"

ENV UID=991 GID=991 \
    UPLOAD_MAX_SIZE=10G \
    APC_SHM_SIZE=128M \
    OPCACHE_MEM_SIZE=128 \
    CRON_PERIOD=15m

RUN echo "@commuedge https://nl.alpinelinux.org/alpine/edge/community" >> /etc/apk/repositories \
 && echo "@testing https://nl.alpinelinux.org/alpine/edge/testing" >> /etc/apk/repositories \
 && BUILD_DEPS=" \
    gnupg \
    tar \
    build-base \
    autoconf \
    automake \
    libtool" \
 && apk -U add \
    ${BUILD_DEPS} \
    nginx \
    s6 \
    openssl \
    ca-certificates \
    libsmbclient \
    samba-client \
    su-exec \
    php7@commuedge \
    php7-fpm@commuedge \
    php7-intl@commuedge \
    php7-mbstring@commuedge \
    php7-curl@commuedge \
    php7-gd@commuedge \
    php7-mcrypt@commuedge \
    php7-opcache@commuedge \
    php7-json@commuedge \
    php7-session@commuedge \
    php7-pdo@commuedge \
    php7-dom@commuedge \
    php7-ctype@commuedge \
    php7-pdo_mysql@commuedge \
    php7-pdo_pgsql@commuedge \
    php7-pgsql@commuedge \
    php7-pdo_sqlite@commuedge \
    php7-sqlite3@commuedge \
    php7-zlib@commuedge \
    php7-zip@commuedge \
    php7-xmlreader@commuedge \
    php7-posix@commuedge \
    php7-openssl@commuedge \
    php7-ldap@commuedge \
    php7-ftp@commuedge \
    php7-apcu@testing \
    php7-redis@testing \
    php7-pear@commuedge \
    php7-dev@commuedge \
    tzdata \
 && mkdir /nextcloud && cd /tmp \
 && wget -q http://ftp.gnu.org/pub/gnu/libiconv/libiconv-${GNU_LIBICONV_VERSION}.tar.gz \
 && tar xzf libiconv-${GNU_LIBICONV_VERSION}.tar.gz && cd libiconv-${GNU_LIBICONV_VERSION} \
 && ./configure --prefix=/usr/local \
 && wget -q https://raw.githubusercontent.com/mxe/mxe/7e231efd245996b886b501dad780761205ecf376/src/libiconv-1-fixes.patch \
 && patch -p1 -u < libiconv-1-fixes.patch \
 && make && make install && libtool --finish /usr/local/lib && cd /tmp \
 && wget -q http://is1.php.net/get/php-7.0.11.tar.gz/from/this/mirror -O php7.tar.gz \
 && tar xzf php7.tar.gz && cd /tmp/php-7.0.11/ext/iconv && phpize7 \
 && ./configure --with-iconv=/usr/local --with-php-config=/usr/bin/php-config7 \
 && make && cp modules/iconv.so /usr/lib/php7/modules && cd /tmp \
 && echo "extension=iconv.so" > /etc/php7/conf.d/00_iconv.ini \
 && NEXTCLOUD_TARBALL="nextcloud-${NEXTCLOUD_VERSION}.tar.bz2" \
 && wget -q https://download.nextcloud.com/server/releases/${NEXTCLOUD_TARBALL} \
 && wget -q https://download.nextcloud.com/server/releases/${NEXTCLOUD_TARBALL}.sha256 \
 && wget -q https://download.nextcloud.com/server/releases/${NEXTCLOUD_TARBALL}.asc \
 && wget -q https://nextcloud.com/nextcloud.asc \
 && echo "Verifying both integrity and authenticity of ${NEXTCLOUD_TARBALL}..." \
 && CHECKSUM_STATE=$(echo -n $(sha256sum -c ${NEXTCLOUD_TARBALL}.sha256) | tail -c 2) \
 && if [ "${CHECKSUM_STATE}" != "OK" ]; then echo "Warning! Checksum does not match!" && exit 1; fi \
 && gpg --import nextcloud.asc \
 && FINGERPRINT="$(LANG=C gpg --verify ${NEXTCLOUD_TARBALL}.asc ${NEXTCLOUD_TARBALL} 2>&1 \
  | sed -n "s#Primary key fingerprint: \(.*\)#\1#p")" \
 && if [ -z "${FINGERPRINT}" ]; then echo "Warning! Invalid GPG signature!" && exit 1; fi \
 && if [ "${FINGERPRINT}" != "${GPG_nextcloud}" ]; then echo "Warning! Wrong GPG fingerprint!" && exit 1; fi \
 && echo "All seems good, now unpacking ${NEXTCLOUD_TARBALL}..." \
 && tar xjf ${NEXTCLOUD_TARBALL} --strip 1 -C /nextcloud \
 && apk del ${BUILD_DEPS} php7-pear php7-dev \
 && rm -rf /var/cache/apk/* /tmp/* /root/.gnupg

COPY nginx.conf /etc/nginx/nginx.conf
COPY php-fpm.conf /etc/php7/php-fpm.conf
COPY opcache.ini /etc/php7/conf.d/00_opcache.ini
COPY apcu.ini /etc/php7/conf.d/apcu.ini
COPY run.sh /usr/local/bin/run.sh
COPY occ /usr/local/bin/occ
COPY s6.d /etc/s6.d

RUN chmod +x /usr/local/bin/* /etc/s6.d/*/* /etc/s6.d/.s6-svscan/*

VOLUME /data /config /apps2

EXPOSE 8888

LABEL description="A server software for creating file hosting services" \
      nextcloud="Nextcloud v${NEXTCLOUD_VERSION}"

CMD ["run.sh"]
