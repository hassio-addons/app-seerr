ARG BUILD_FROM=ghcr.io/hassio-addons/base:21.0.3

###############################################################################
# Get prebuilt containers from Seerr
###############################################################################
FROM ghcr.io/seerr-team/seerr:v3.4.1 AS seerr

###############################################################################
# Build the actual app.
###############################################################################
# hadolint ignore=DL3006
FROM ${BUILD_FROM}

# Set shell
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# Get Seerr, and the Node.js runtime it is built against, from the official
# images. Seerr ships native modules that are bound to that exact Node.js
# version, so the two cannot be separated.
COPY --from=seerr /usr/local/bin/node /usr/local/bin/node
COPY --from=seerr /app /opt

# Setup base
RUN \
  apk add --no-cache \
    libstdc++=15.2.0-r5 \
  \
  && rm -f -r \
    /opt/compose.postgres.yaml \
    /opt/cypress.config.ts \
    /opt/devenv.lock \
    /opt/devenv.nix \
    /opt/devenv.yaml \
    /opt/eslint.config.mts \
    /opt/server \
    /opt/src

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
