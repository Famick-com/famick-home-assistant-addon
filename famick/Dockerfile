# Famick Home Assistant Add-on
#
# FROMs the canonical self-hosted Famick image (Debian + .NET 10 ASP.NET
# runtime + Famick app + supercronic) and adds the bits an HA add-on needs:
# s6-overlay for process supervision, Postgres 16 for the bundled DB, and
# the rootfs/ tree that wires up cont-init scripts and per-service `run`
# files. The Famick app stays put at /app from the base image.

FROM mtherienfamick/homemanagement:latest

# TARGETARCH is set automatically by buildx per --platform. Do NOT give it a
# default value: a default SHADOWS buildx's per-platform value, so the aarch64
# build would still see amd64 and bake an x86_64 s6-overlay into the arm64
# image (=> "s6-overlay-suexec: Exec format error" on arm64 hosts). For a plain
# `docker build` (no buildx) TARGETARCH is empty; we fall back to the build
# host's arch via `dpkg --print-architecture` in the RUN below.
ARG TARGETARCH

# s6-overlay v3 (sysroot variant). Pin a known-good release; update
# deliberately when needed since the rootfs layout is sensitive to the
# major version.
ARG S6_OVERLAY_VERSION=3.2.3.0
ARG S6_OVERLAY_NOARCH_SHA256=b720f9d9340efc8bb07528b9743813c836e4b02f8693d90241f047998b4c53cf
ARG S6_OVERLAY_AMD64_SHA256=a93f02882c6ed46b21e7adb5c0add86154f01236c93cd82c7d682722e8840563
ARG S6_OVERLAY_ARM64_SHA256=0952056ff913482163cc30e35b2e944b507ba1025d78f5becbb89367bf344581

RUN apt-get update && apt-get install -y --no-install-recommends \
        postgresql-16 \
        postgresql-client-16 \
        sudo \
        xz-utils \
        ca-certificates \
    && rm -rf /var/lib/apt/lists/* \
    && S6_TARGETARCH="${TARGETARCH:-$(dpkg --print-architecture)}" \
    && case "${S6_TARGETARCH}" in \
        amd64) S6_ARCH=x86_64 ; S6_SHA="${S6_OVERLAY_AMD64_SHA256}" ;; \
        arm64) S6_ARCH=aarch64 ; S6_SHA="${S6_OVERLAY_ARM64_SHA256}" ;; \
        *) echo "Unsupported target arch '${S6_TARGETARCH}'"; exit 1 ;; \
    esac \
    && curl -fsSLO "https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-noarch.tar.xz" \
    && echo "${S6_OVERLAY_NOARCH_SHA256}  s6-overlay-noarch.tar.xz" | sha256sum -c - \
    && tar -C / -Jxpf s6-overlay-noarch.tar.xz \
    && rm s6-overlay-noarch.tar.xz \
    && curl -fsSLO "https://github.com/just-containers/s6-overlay/releases/download/v${S6_OVERLAY_VERSION}/s6-overlay-${S6_ARCH}.tar.xz" \
    && echo "${S6_SHA}  s6-overlay-${S6_ARCH}.tar.xz" | sha256sum -c - \
    && tar -C / -Jxpf "s6-overlay-${S6_ARCH}.tar.xz" \
    && rm "s6-overlay-${S6_ARCH}.tar.xz"

# Drop the rootfs (cont-init.d + services.d) on top of s6-overlay's layout.
COPY rootfs/ /

# Make sure shell scripts are executable (git preserves +x, but COPY can
# strip the bit in some build contexts — being explicit is cheap insurance).
RUN find /etc/cont-init.d /etc/services.d -type f -exec chmod +x {} +

# If any cont-init script fails, abort the container start instead of
# silently continuing into s6's stage 2 with half-initialized state.
ENV S6_BEHAVIOUR_IF_STAGE2_FAILS=2 \
    POSTGRES_DATA=/data/postgres \
    POSTGRES_BIN=/usr/lib/postgresql/16/bin \
    POSTGRES_RUN=/var/run/postgresql

# The web app listens on 8088 internally — Supervisor's Ingress wraps it.
EXPOSE 8088

HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD curl -fsS http://localhost:8088/health || exit 1

ENTRYPOINT ["/init"]
