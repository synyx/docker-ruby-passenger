FROM ruby:2.4.6-alpine3.10

LABEL org.label-schema.schema-version="1.0" \
      org.label-schema.name="ruby-passenger" \
      org.label-schema.description="An Alpine Linux based minimal Ruby/Passenger base image" \
      org.label-schema.vendor="synyx GmbH & Co. KG"

ARG PASSENGER_VERSION=6.0.2
ENV PASSENGER_VERSION ${PASSENGER_VERSION}
ENV PASSENGER_ROOT "/opt/passenger"
ENV PATH "${PASSENGER_ROOT}/bin:$PATH"

RUN apk update \
 && apk add curl libexecinfo libstdc++ pcre tini \
 && apk add --virtual .build-deps \
      build-base curl-dev libexecinfo-dev libressl-dev linux-headers pcre-dev wget \
 # Download & extract passenger
 && mkdir -p /opt \
 && curl -L https://s3.amazonaws.com/phusion-passenger/releases/passenger-$PASSENGER_VERSION.tar.gz | \
    tar -xzf - -C /opt \
 && mv /opt/passenger-$PASSENGER_VERSION /opt/passenger \
 && chown -R root:root /opt/passenger \
 # Build & validate passenger with nginx
 && export EXTRA_PRE_CFLAGS='-O' EXTRA_PRE_CXXFLAGS='-O' EXTRA_LDFLAGS='-lexecinfo' \
 && mkdir /tmp/passenger-compile \
 && passenger-config compile-agent --auto --optimize \
 && passenger-config install-standalone-runtime --auto --url-root=no-builds-for-alpine --connect-timeout=1 \
 && passenger-config build-native-support \
 # Cleanup passenger build products
 && mkdir /tmp/passenger-keep \
 && mv ${PASSENGER_ROOT}/src/ruby_supportlib /tmp/passenger-keep/ \
 && mv ${PASSENGER_ROOT}/src/helper-scripts /tmp/passenger-keep/ \
 ## Remove obvious directories
 && rm -rf ${PASSENGER_ROOT}/src/* \
           ${PASSENGER_ROOT}/build \
           ${PASSENGER_ROOT}/doc \
           ${PASSENGER_ROOT}/man \
 # Remove obvious binary by-products
 && find ${PASSENGER_ROOT}/buildout -type f -name '*.o' -delete \
 # Restore final binaries
 && mv /tmp/passenger-keep/* ${PASSENGER_ROOT}/src/ \
 && strip ${PASSENGER_ROOT}/buildout/support-binaries/* \
 && strip ${PASSENGER_ROOT}/buildout/ruby/ruby-*/passenger_native_support.so \
 # Remove all those build dependencies & caches
 && apk del .build-deps \
 && rm -rf /var/cache/apk/* /tmp/* \
 && passenger-config validate-install

WORKDIR /app
EXPOSE 3000
ENTRYPOINT ["/sbin/tini", "--"]
CMD ["passenger", "start", "--no-install-runtime", "--no-compile-runtime", "--log-file", "/dev/stderr", "--disable-security-update-check", "--disable-anonymous-telemetry"]
