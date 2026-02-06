# syntax=docker/dockerfile:1.7
ARG ZPAQ_REPOSITORY=fcorbelli/zpaqfranz

FROM ubuntu:24.04 AS build

ARG TARGETARCH
ARG ZPAQ_VERSION
WORKDIR /src

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
      build-essential \
      ca-certificates \
    && rm -rf /var/lib/apt/lists/*

COPY zpaqfranz.cpp /src/zpaqfranz.cpp

RUN set -eux; \
    resolved_arch="${TARGETARCH:-}"; \
    if [ -z "$resolved_arch" ]; then \
      case "$(uname -m)" in \
        x86_64) resolved_arch="amd64" ;; \
        aarch64 | arm64) resolved_arch="arm64" ;; \
      esac; \
    fi; \
    extra_defines=""; \
    case "$resolved_arch" in \
      amd64) extra_defines="-DHWSHA2" ;; \
      *) extra_defines="-DNOJIT" ;; \
    esac; \
    g++ -O3 -pthread -Dunix $extra_defines zpaqfranz.cpp -o zpaqfranz -lm \
    && strip zpaqfranz

FROM ubuntu:24.04

ARG ZPAQ_VERSION
ARG ZPAQ_REPOSITORY

RUN apt-get update \
    && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
      ca-certificates \
      libstdc++6 \
    && rm -rf /var/lib/apt/lists/*

RUN groupadd -r zpaq && useradd -r -g zpaq -d /var/lib/zpaq -s /usr/sbin/nologin zpaq \
    && mkdir -p /var/lib/zpaq /data \
    && chown zpaq:zpaq /var/lib/zpaq /data

COPY --from=build /src/zpaqfranz /usr/local/bin/zpaqfranz

RUN /usr/local/bin/zpaqfranz -version

LABEL org.opencontainers.image.title="zpaqfranz" \
      org.opencontainers.image.description="Minimal multi-arch zpaqfranz packaged on Ubuntu LTS" \
      org.opencontainers.image.source="https://github.com/${ZPAQ_REPOSITORY}" \
      org.opencontainers.image.version="${ZPAQ_VERSION}"

WORKDIR /data
USER zpaq

ENTRYPOINT ["/usr/local/bin/zpaqfranz"]
CMD ["-h"]
