FROM alpine:3.8

LABEL maintainer="shaopenghk@qq.com"

ENV NGINX_VERSION 1.14.2 
ENV USER www
ENV GROUP www

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories \
	&& apk add --no-cache --virtual .build-deps \
		gcc \
		libc-dev \
		make \
		openssl-dev \
		pcre-dev \
		zlib-dev \
		linux-headers \
		curl \
		wget \
		gnupg1 \
		libxslt-dev \
		gd-dev \
		geoip-dev \
		tzdata \
		unzip \
	&& cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
	&& addgroup -S $GROUP \
	&& adduser -D -S -h /usr/local/nginx -s /sbin/nologin -G $GROUP $USER \
	\
	&& mkdir -p /usr/local/src \
	&& cd /usr/local/src \
	&& wget http://package.xwx.cn/nginx/nginx-$NGINX_VERSION.tar.gz \
	&& wget http://package.xwx.cn/nginx/nginx_module/echo-nginx-module-master.zip \
	&& wget http://package.xwx.cn/nginx/nginx_module/headers-more-nginx-module-master.zip \
	&& wget http://package.xwx.cn/nginx/nginx_module/nginx_upstream_check_module-master.zip \
	&& wget http://package.xwx.cn/nginx/nginx_module/nginx-upload-module-2.2.zip \
	&& wget http://package.xwx.cn/nginx/nginx_module/ngx_cache_purge-2.3.tar.gz \
	\
	&& unzip echo-nginx-module-master.zip \
	&& unzip headers-more-nginx-module-master.zip \
	&& unzip nginx_upstream_check_module-master.zip \
	&& unzip nginx-upload-module-2.2.zip \
	&& tar xf ngx_cache_purge-2.3.tar.gz \
	&& tar xf nginx-$NGINX_VERSION.tar.gz \
	\
	&& cd /usr/local/src/nginx-$NGINX_VERSION \
	&& sed -i -e 's#'$NGINX_VERSION'##g' -e 's#nginx/#xws#g' -e 's#"NGINX"#"XWS"#g' src/core/nginx.h \
	&& sed -i 's#nginx<#xws<#g' src/http/ngx_http_special_response.c \ 
	&& sed -i 's#Server: nginx#Server: xws#g' src/http/ngx_http_header_filter_module.c \
	&& patch -p1 < ../nginx_upstream_check_module-master/check_1.14.0+.patch \
	&& ./configure \
		--sbin-path=/usr/sbin/nginx \
		--conf-path=/etc/nginx/nginx.conf \
		--modules-path=/usr/lib/nginx/modules \
		--error-log-path=/var/log/nginx/error.log \
		--http-log-path=/var/log/nginx/access.log \
		--pid-path=/var/run/nginx.pid \
		--lock-path=/var/run/nginx.lock \
		--user=$USER \
		--group=$GROUP \
		--with-file-aio \
		--with-poll_module \
		--with-http_realip_module \
		--with-http_image_filter_module \
		--with-http_gunzip_module \
		--with-http_gzip_static_module \
		--with-http_addition_module \
		--with-http_sub_module \
		--with-http_dav_module \
		--with-http_flv_module \
		--with-http_mp4_module \
		--with-http_slice_module \
		--with-http_random_index_module \
		--with-http_secure_link_module \
		--with-http_degradation_module \
		--with-http_ssl_module \
		--with-http_v2_module \
		--with-http_stub_status_module \
		--with-http_slice_module \
		--with-http_geoip_module \
		--with-http_auth_request_module \
		--with-stream \
		--with-stream_ssl_module \
		--with-stream_ssl_preread_module \
		--with-stream_realip_module \
		--with-threads \
		--with-pcre \
		--add-module=./../ngx_cache_purge-2.3 \
		--add-module=./../nginx_upstream_check_module-master \
		--add-module=./../echo-nginx-module-master \
		--add-module=./../headers-more-nginx-module-master \
		--add-module=./../nginx-upload-module-2.2 \
	&& make -j$(getconf _NPROCESSORS_ONLN) \
	&& make install \
  && strip /usr/sbin/nginx* \
	&& mkdir /etc/nginx/conf.d/ \
	&& touch /usr/local/nginx/html/favicon.ico \
	&& rm -rf /usr/local/src/ \
	&& apk add --no-cache --virtual .gettext gettext \
	&& runDeps="$( \
		scanelf --needed --nobanner --format '%n#p' /usr/sbin/nginx /usr/lib/nginx/modules/*.so \
			| tr ',' '\n' \
			| sort -u \
			| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
	)" \
	&& apk add --no-cache --virtual .nginx-rundeps $runDeps \
	&& apk del .build-deps \
	&& apk del .gettext \
	&& ln -sf /dev/stdout /var/log/nginx/access.log \
	&& ln -sf /dev/stderr /var/log/nginx/error.log

COPY nginx.conf /etc/nginx/nginx.conf
COPY default.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
STOPSIGNAL SIGTERM
CMD ["nginx", "-g", "daemon off;"]
