# syntax=docker/dockerfile:1.9
ARG BUILD_FROM=ghcr.io/hassio-addons/base:16.2.0

# hadolint ignore=DL3006
FROM ${BUILD_FROM}
ARG BUN_RUNTIME_TRANSPILER_CACHE_PATH=0
ENV BUN_RUNTIME_TRANSPILER_CACHE_PATH=${BUN_RUNTIME_TRANSPILER_CACHE_PATH}

# Ensure `bun install -g` works
ARG BUN_INSTALL_BIN=/usr/local/bin
ENV BUN_INSTALL_BIN=${BUN_INSTALL_BIN}
# Copy Node-RED package.json
COPY package.json /opt/
COPY node-red-dashboard-show-dashboard.patch /tmp/

# Set workdir
WORKDIR /opt

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# hadolint ignore=DL3018
RUN \
    apk add --no-cache --virtual .build-dependencies \
        build-base=0.5-r3 \
        linux-headers=6.6-r0 \
        py3-pip=24.0-r2 \
        python3-dev=3.12.3-r1 \
    \
    && apk add --no-cache \
        git=2.45.2-r0 \
        icu-data-full=74.2-r0 \
        nginx=1.26.1-r0 \
        openssh-client=9.7_p1-r4 \
        patch=2.7.6-r10

# hadolint ignore=DL3022
COPY --from=oven/bun:1.1.22-alpine /usr/local/bin/bun /usr/local/bin/

RUN <<-EOT
if [[ "$(uname -m)" == "aarch64" ]] ;
    then
        wget -q "https://github.com/Rory-Z/alpine-pkg-glibc/releases/download/2.37-r1/glibc-2.37-r1-aarch64.apk" -O /tmp/glibc.apk
        wget -q "https://github.com/Rory-Z/alpine-pkg-glibc/releases/download/2.37-r1/glibc-bin-2.37-r1-aarch64.apk" -O /tmp/glibc-bin.apk
        wget -q "https://github.com/Rory-Z/alpine-pkg-glibc/releases/download/2.37-r1/glibc-dev-2.37-r1-aarch64.apk" -O /tmp/glibc-dev.apk
    else 
        wget -q https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.34-r0/glibc-2.34-r0.apk -O /tmp/glibc.apk
        wget -q https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.34-r0/glibc-bin-2.34-r0.apk -O /tmp/glibc-bin.apk
        wget -q https://github.com/sgerrand/alpine-pkg-glibc/releases/download/2.34-r0/glibc-dev-2.34-r0.apk -O /tmp/glibc-dev.apk
fi
EOT

# hadolint ignore=DL3018
RUN \
    addgroup -g 1000 bun \
    && adduser -u 1000 -G bun -s /bin/sh -D bun \
    && wget -q -O /etc/apk/keys/sgerrand.rsa.pub https://alpine-pkgs.sgerrand.com/sgerrand.rsa.pub \
    && apk --no-cache --force-overwrite --allow-untrusted add \
        /tmp/glibc.apk \
        /tmp/glibc-bin.apk \
        /tmp/glibc-dev.apk \
    && ln -s /usr/local/bin/bun /usr/local/bin/bunx \
    && which bun \
    && which bunx \
    && bun --version \
    \
    && bun install \
    \
    && bun pm cache rm \
    \
    && echo -e "StrictHostKeyChecking no" >> /etc/ssh/ssh_config \
    \
    && patch -d /opt/node_modules/node-red-dashboard -p1 \
             -i /tmp/node-red-dashboard-show-dashboard.patch \
    \
    && apk del --no-cache --purge .build-dependencies \
    && rm -fr \
        /etc/nginx \
        /root/.cache \
        /root/.bun \
        /tmp/*

# Copy root filesystem
COPY rootfs /

# Health check
HEALTHCHECK --start-period=10m \
    CMD curl --fail http://127.0.0.1:46836 || exit 1

# Build arguments
ARG BUILD_ARCH
ARG BUILD_DATE
ARG BUILD_DESCRIPTION
ARG BUILD_NAME
ARG BUILD_REF
ARG BUILD_REPOSITORY
ARG BUILD_VERSION

# Labels
LABEL \
    io.hass.name="${BUILD_NAME}" \
    io.hass.description="${BUILD_DESCRIPTION}" \
    io.hass.arch="${BUILD_ARCH}" \
    io.hass.type="addon" \
    io.hass.version=${BUILD_VERSION} \
    maintainer="Sergei Krashevich <svk@svk.su>" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="Sergei Krashevich <svk@svk.su>" \
    org.opencontainers.image.authors="Sergei Krashevich <svk@svk.su>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://addons.community" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}/blon/main/node-red/Dockerfile" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
