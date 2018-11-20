FROM ruby:2.3.8-alpine3.8

ENV PASSENGER_VERSION 5.3.6

RUN apk update \
 && apk add curl libexecinfo pcre \
 && apk add --virtual .build-deps \
      build-base curl-dev libexecinfo-dev libressl-dev linux-headers pcre-dev wget \
 # Build & validate passenger with nginx
 && gem install passenger -v ${PASSENGER_VERSION} --no-ri --no-rdoc \
 && EXTRA_CFLAGS="-s" EXTRA_CXXFLAGS="-s" passenger-install-nginx-module --auto-download --auto \
 && passenger-config validate-install \
 # Cleanup passenger build products
 && PASSENGER_ROOT=`passenger-config about root` \
 && mv ${PASSENGER_ROOT}/src/ruby_supportlib /tmp/passenger.ruby_supportlib \
 && mv ${PASSENGER_ROOT}/src/helper-scripts /tmp/passenger.helper-scripts \
 ## Remove obvious directories
 && rm -rf ${PASSENGER_ROOT}/src/* \
           ${PASSENGER_ROOT}/build \
           ${PASSENGER_ROOT}/dev \
           ${PASSENGER_ROOT}/doc \
           ${PASSENGER_ROOT}/download_cache \
           ${PASSENGER_ROOT}/man \
 # Remove obvious binary by-products
 && find ${PASSENGER_ROOT}/buildout -type f -name '*.o' -delete \
 # Remove unneeded src & build directories
 && rm -rf ${PASSENGER_ROOT}/buildout/common \
           ${PASSENGER_ROOT}/buildout/libev \
           ${PASSENGER_ROOT}/buildout/libuv \
 # Restore final binaries
 && mv /tmp/passenger.ruby_supportlib ${PASSENGER_ROOT}/src/ruby_supportlib \
 && mv /tmp/passenger.helper-scripts ${PASSENGER_ROOT}/src/helper-scripts \
 && strip ${PASSENGER_ROOT}/buildout/support-binaries/PassengerAgent \
 && strip /opt/nginx/sbin/nginx \
 # Remove all those build dependencies & caches
 && apk del .build-deps \
 && rm -rf /var/cache/apk/* /usr/local/bundle/cache/*

WORKDIR /app
CMD [ "/usr/local/bundle/bin/passenger", "start", ".", "--nginx-bin","/opt/nginx/sbin/nginx", "--log-file", "/dev/stderr" ]
