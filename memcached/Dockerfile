an debian:stretch-slim

# add our user and group first to make sure their IDs get assigned consistently, regardless of whatever dependencies get added
RUN groupadd -r memcache && useradd -r -g memcache memcache

ENV MEMCACHED_VERSION 1.5.2
ENV MEMCACHED_SHA1 5d6c85fe15b5e575a563ec1bbeb01020e0132958

ENV Codename=$( (lsb_release -a)|awk '{print $2}'|tail -n 1 )
RUN cp /etc/apt/sources.list /etc/apt/sources.list.bak \
    && echo "\
	deb http://mirrors.aliyun.com/ubuntu/ $Codename main multiverse restricted universe \
	deb http://mirrors.aliyun.com/ubuntu/ $Codename-backports main multiverse restricted universe \
	deb http://mirrors.aliyun.com/ubuntu/ $Codename-proposed main multiverse restricted universe \
	deb http://mirrors.aliyun.com/ubuntu/ $Codename-security main multiverse restricted universe \
	deb http://mirrors.aliyun.com/ubuntu/ $Codename-updates main multiverse restricted universe \
	deb-src http://mirrors.aliyun.com/ubuntu/ $Codename main multiverse restricted universe \
	deb-src http://mirrors.aliyun.com/ubuntu/ $Codename-backports main multiverse restricted universe \
	deb-src http://mirrors.aliyun.com/ubuntu/ $Codename-proposed main multiverse restricted universe \
	deb-src http://mirrors.aliyun.com/ubuntu/ $Codename-security main multiverse restricted universe \
	deb-src http://mirrors.aliyun.com/ubuntu/ $Codename-updates main multiverse restricted universe ">/etc/apt/sources.list

RUN set -x \
	\
	&& buildDeps=' \
		ca-certificates \
		dpkg-dev \
		gcc \
		libc6-dev \
		libevent-dev \
		libsasl2-dev \
		sasl2-bin \
                libsasl2-modules \
		make \
		perl \
		wget \
	' \
	&& apt-get update && apt-get install -y $buildDeps --no-install-recommends \
	&& rm -rf /var/lib/apt/lists/* \
	\
	&& wget -O memcached.tar.gz "https://memcached.org/files/memcached-$MEMCACHED_VERSION.tar.gz" \
	&& echo "$MEMCACHED_SHA1  memcached.tar.gz" | sha1sum -c - \
	&& mkdir -p /usr/src/memcached \
	&& tar -xzf memcached.tar.gz -C /usr/src/memcached --strip-components=1 \
	&& rm memcached.tar.gz \
	\
	&& cd /usr/src/memcached \
	\
	&& ./configure \
		--build="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
		--enable-sasl \
	&& make -j "$(nproc)" \
	\
	&& make test \
	&& make install \
	\
	&& cd / && rm -rf /usr/src/memcached \
	\
	&& apt-mark manual \
		libevent-2.0-5 \
		libsasl2-2 \
	&& apt-get purge -y --auto-remove $buildDeps \
	\
	&& memcached -V

COPY docker-entrypoint.sh /usr/local/bin/
RUN ln -s usr/local/bin/docker-entrypoint.sh /entrypoint.sh # backwards compat
ENTRYPOINT ["docker-entrypoint.sh"]

USER memcache
EXPOSE 11211
CMD ["memcached"]