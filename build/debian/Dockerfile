# ----------------------------------------
# Stage 1: Build / Download MediathekView
# ----------------------------------------
FROM debian:trixie-slim AS builder


# ----------------------------------------
# Build-Argument
# ----------------------------------------
ARG APP_VERSION

# Setze Arbeitsverzeichnis für temporäre Dateien
WORKDIR /tmp


# ----------------------------------------
# Installiere wget und ca-certificates für HTTPS-Download
# --no-install-recommends hält das Image klein
# ----------------------------------------
RUN apt-get update \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    git \
    ca-certificates \
 && rm -rf /var/lib/apt/lists/*


# ----------------------------------------
# Create /opt/testssl.sh directory and download the latest testssl.sh release
# ----------------------------------------
RUN set -eux; \
    test -n "${APP_VERSION}"; \
    git init /opt/testssl.sh; \
    cd /opt/testssl.sh; \
    git remote add origin https://github.com/testssl/testssl.sh.git; \
    git fetch --depth 1 origin \
        "refs/tags/${APP_VERSION}:refs/tags/${APP_VERSION}"; \
    git checkout "refs/tags/${APP_VERSION}"; \
    rm -rf .git


# ----------------------------------------
# Stage 2: Runtime
# ----------------------------------------
FROM debian:trixie-slim

# Build-Argument
ARG APP_VERSION
ARG DOCKER_IMAGE_VERSION


# ----------------------------------------
# Setze Umgebungsvariablen
# UWSGI_PROCESSES   Number of uWSGI processes: amount of max. parallel running SSL checks
# UWSGI_THREADS     Number of uWSGI threads: amount of max. parallel running SSL checks
# TEST_TIMEOUT      Set the timeout for the portal site (default 300 seconds)
# TESTSSLDEBUG      Enable debugging for testssl.sh by setting this variable to a higher value
# ----------------------------------------
ENV UWSGI_PROCESSES=4 \
    UWSGI_THREADS=2 \
    TEST_TIMEOUT=300 \
    TESTSSLDEBUG=0


# ----------------------------------------
# OCI image metadata
# ----------------------------------------
LABEL org.opencontainers.image.authors="Daniel Wydler" \
      org.opencontainers.image.description="Web frontend for testssl.sh." \
      org.opencontainers.image.documentation="https://github.com/wydler/testsslsh-gui/blob/master/README.md" \
      org.opencontainers.image.source="https://github.com/wydler/testsslsh-gui" \
      org.opencontainers.image.title="wydler//testsslsh-gui" \
      org.opencontainers.image.url="https://hub.docker.com/r/wydler/testsslsh-gui" \
      org.opencontainers.image.version="${DOCKER_IMAGE_VERSION}" \
      org.opencontainers.image.base.name="debian:trixie-slim" \
      org.opencontainers.image.testssl.version="${APP_VERSION}"


# ----------------------------------------
# Runtime dependencies
# ----------------------------------------
RUN apt-get update \
 && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
    openssl \
    net-tools \
    dnsutils \
    aha \
    xxd \
    python3-pkg-resources \
    python3-flask \
    bsdmainutils \
    procps \
    nginx-light \
    uwsgi \
    uwsgi-plugin-python3 \
    supervisor \
    socat \
 && rm -rf /var/lib/apt/lists/*


# ----------------------------------------
# Application files and configuration
# ----------------------------------------
COPY --from=builder /opt/testssl.sh /opt/testssl.sh

COPY rootfs /

RUN chmod 0755 /entrypoint.sh

WORKDIR /var/www/html/testssl

EXPOSE 5000

ENTRYPOINT ["/entrypoint.sh"]
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]