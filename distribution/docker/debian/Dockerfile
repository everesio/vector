FROM docker.io/debian:trixie-slim@sha256:b6e2a152f22a40ff69d92cb397223c906017e1391a73c952b588e51af8883bf8 AS builder

WORKDIR /vector

# Install only what we need for extraction
RUN apt-get update && apt-get install -y --no-install-recommends \
    tar \
    && rm -rf /var/lib/apt/lists/*

# Copy the prebuilt GNU artifact
COPY vector-*-x86_64-unknown-linux-gnu.tar.gz /tmp/vector.tar.gz

# Extract Vector into a clean rootfs-like structure
RUN mkdir -p /vector-root/usr/bin \
    /vector-root/etc/vector \
    /vector-root/var/lib/vector && \
    tar -xzf /tmp/vector.tar.gz -C /tmp && \
    VECTOR_DIR="$(find /tmp -maxdepth 1 -type d -name 'vector*' | head -n1)" && \
    cp "${VECTOR_DIR}/bin/vector" /vector-root/usr/bin/vector && \
    if [ -d "${VECTOR_DIR}/config" ]; then \
      cp -r "${VECTOR_DIR}/config/"* /vector-root/etc/vector/; \
    fi && \
    rm -rf /tmp/vector* /tmp/vector.tar.gz


FROM docker.io/debian:trixie-slim@sha256:b6e2a152f22a40ff69d92cb397223c906017e1391a73c952b588e51af8883bf8

# https://github.com/opencontainers/image-spec/blob/main/annotations.md
LABEL org.opencontainers.image.url="https://vector.dev"
LABEL org.opencontainers.image.source="https://github.com/vectordotdev/vector"
LABEL org.opencontainers.image.documentation="https://vector.dev/docs"

# hadolint ignore=DL3008
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    tzdata \
    libsasl2-2 \
    && rm -rf /var/lib/apt/lists/*

COPY --from=builder /vector-root/usr/bin/vector /usr/bin/vector
COPY --from=builder /vector-root/etc/vector /etc/vector
COPY --from=builder /vector-root/var/lib/vector /var/lib/vector

# Smoke test
RUN ["/usr/bin/vector", "--version"]

ENTRYPOINT ["/usr/bin/vector"]