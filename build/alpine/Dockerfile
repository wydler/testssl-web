# ----------------------------------------
# Stage 1: Build / Download testssl.sh
# ----------------------------------------
FROM alpine:3.22 AS builder

ARG APP_VERSION

WORKDIR /tmp

RUN apk add --no-cache \
    ca-certificates \
    git

RUN set -eux; \
    test -n "${APP_VERSION}"; \
    git init /opt/testssl.sh; \
    cd /opt/testssl.sh; \
    git remote add origin https://github.com/testssl/testssl.sh.git; \
    git fetch --depth 5 origin \
        "refs/tags/${APP_VERSION}:refs/tags/${APP_VERSION}"; \
    git checkout "refs/tags/${APP_VERSION}"; \
    git log -n 5 > git-log.txt; \
    rm -rf .git bin/openssl.Darwin.x86_64 bin/openssl.FreeBSD.amd64


# ----------------------------------------
# Stage 2: Runtime
# ----------------------------------------
FROM alpine:3.22

ARG APP_VERSION
ARG DOCKER_IMAGE_VERSION

ENV UWSGI_PROCESSES=4 \
    UWSGI_THREADS=2 \
    TEST_TIMEOUT=300 \
    TESTSSLDEBUG=0

LABEL org.opencontainers.image.authors="Daniel Wydler" \
      org.opencontainers.image.description="Web frontend for testssl.sh." \
      org.opencontainers.image.documentation="https://github.com/wydler/testsslsh-gui/blob/master/README.md" \
      org.opencontainers.image.source="https://github.com/wydler/testsslsh-gui" \
      org.opencontainers.image.title="wydler//testsslsh-gui" \
      org.opencontainers.image.url="https://hub.docker.com/r/wydler/testsslsh-gui" \
      org.opencontainers.image.version="${DOCKER_IMAGE_VERSION}" \
      org.opencontainers.image.base.name="alpine:3.22" \
      org.opencontainers.image.testssl.version="${APP_VERSION}"

RUN apk add --no-cache \
    aha \
    bash \
    bind-tools \
    net-tools \
    nginx \
    openssl \
    procps-ng \
    py3-flask \
    py3-setuptools \
    python3 \
    socat \
    supervisor \
    uwsgi \
    uwsgi-python3 \
    util-linux-misc \
    xxd \
 # && addgroup -S www-data \
 && adduser -S -D -H -G www-data www-data \
 && mkdir -p /etc/nginx/sites-enabled

COPY --from=builder /opt/testssl.sh /opt/testssl.sh

COPY rootfs /

RUN chmod 0755 /entrypoint.sh \
 && cp /etc/supervisor/conf.d/supervisord.conf /etc/supervisord.conf \
 && sed -i 's#/usr/bin/uwsgi#/usr/sbin/uwsgi#' /etc/supervisord.conf

WORKDIR /var/www/html/testssl

EXPOSE 5000

ENTRYPOINT ["/entrypoint.sh"]
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
