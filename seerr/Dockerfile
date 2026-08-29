ARG BUILD_FROM=ghcr.io/hassio-addons/debian-base:9.4.0
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Setup base
ARG SEERR_VERSION=3.4.1
ENV COMMIT_TAG=v${SEERR_VERSION}
ENV NEXT_TELEMETRY_DISABLED=1
# hadolint ignore=DL3003,DL3016
RUN \
    apt-get update \
    && apt-get install -y --no-install-recommends \
        gpg=2.4.7-21+deb13u1+b4 \
    \
    && mkdir -p /etc/apt/keyrings \
    && curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key \
        | gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg \
    \
    && echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_22.x nodistro main" \
        > /etc/apt/sources.list.d/nodesource.list \
    \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        nodejs=22.23.2-1nodesource1 \
    \
    && apt-get install -y --no-install-recommends \
        build-essential=12.12 \
    \
    && curl -J -L -o /tmp/seerr.tar.gz \
        "https://github.com/seerr-team/seerr/archive/v${SEERR_VERSION}.tar.gz" \
    && tar zxf \
        /tmp/seerr.tar.gz \
        --strip 1 -C /opt \
    \
    && cd /opt \
    && npm install --global "$(node -p 'require("/opt/package.json").packageManager')" \
    && CI=true CYPRESS_INSTALL_BINARY=0 pnpm install --frozen-lockfile \
    && pnpm build \
    && CI=true pnpm install --prod --frozen-lockfile \
    && pnpm store prune \
    \
    && echo "{\"commitTag\": \"${COMMIT_TAG}\"}" > "/opt/committag.json" \
    \
    && find /opt/node_modules/.pnpm -maxdepth 1 -type d -name '*musl*' \
        -exec rm -f -r {} + \
    && find /opt/node_modules/.pnpm -type d \( \
        -path '*ace-builds/src' -o \
        -path '*ace-builds/src-min' -o \
        -path '*ace-builds/src-noconflict' -o \
        -path '*country-flag-icons/1x1' -o \
        -path '*country-flag-icons/react' -o \
        -path '*country-flag-icons/string' -o \
        -path '*@heroicons/react/16' \
        \) -exec rm -f -r {} + \
    \
    && npm cache clean --force \
    \
    && apt-get purge -y --auto-remove \
        build-essential \
        gpg \
    \
    && rm -f -r \
        /opt/.next/cache \
        /opt/cypress \
        /opt/cypress.config.ts \
        /opt/devenv.lock \
        /opt/devenv.nix \
        /opt/devenv.yaml \
        /opt/docs \
        /opt/eslint.config.mts \
        /opt/gen-docs \
        /opt/server \
        /opt/src \
        /root/{.cache,.config,.gnupg,.local,.npm} \
        /tmp/* \
        /var/{cache,log}/* \
        /var/lib/apt/lists/*

# Copy root filesystem
COPY rootfs /

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
    maintainer="Franck Nijhof <opensource@frenck.dev>" \
    org.opencontainers.image.title="${BUILD_NAME}" \
    org.opencontainers.image.description="${BUILD_DESCRIPTION}" \
    org.opencontainers.image.vendor="Home Assistant Community Apps" \
    org.opencontainers.image.authors="Franck Nijhof <opensource@frenck.dev>" \
    org.opencontainers.image.licenses="MIT" \
    org.opencontainers.image.url="https://frenck.dev/home-assistant-apps" \
    org.opencontainers.image.source="https://github.com/${BUILD_REPOSITORY}" \
    org.opencontainers.image.documentation="https://github.com/${BUILD_REPOSITORY}/blob/main/README.md" \
    org.opencontainers.image.created=${BUILD_DATE} \
    org.opencontainers.image.revision=${BUILD_REF} \
    org.opencontainers.image.version=${BUILD_VERSION}
