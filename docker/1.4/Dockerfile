# syntax = docker/dockerfile:1.3

# ---
# 1. Use Alpine as base
FROM alpine:3.16 AS base

# ---
# 2. Upgrade system and installed dependencies for security patches
RUN --mount=type=cache,target=/var/cache/apk \
    set -eux; \
    apk upgrade

# ---
# 3. Setup non-root user (fixuid)
#
# TODO: detect arch other than x86_64
RUN --mount=type=cache,target=/var/cache/apk \
    --mount=type=tmpfs,target=/tmp \
    set -eux; \
    # create non-root user & give passwordless sudo
    { \
        apk add sudo; \
        addgroup -g 1000 user; \
        adduser -u 1000 -G user -h /home/user -s /bin/sh -D user; \
        mkdir -p /etc/sudoers.d; \
        echo "user ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/user; \
        # cleanup backup copies
        rm /etc/group- /etc/passwd- /etc/shadow-; \
    }; \
    # Install fixuid
    { \
        cd /tmp; \
        export FIXUID_VERSION=0.5.1 \
            FIXUID_SHA256=1077e7af13596e6e3902230d7260290fe21b2ee4fffcea1eb548e5c465a34800; \
        wget -q -O fixuid.tar.gz https://github.com/boxboat/fixuid/releases/download/v${FIXUID_VERSION}/fixuid-${FIXUID_VERSION}-linux-amd64.tar.gz; \
        echo "${FIXUID_SHA256} *fixuid.tar.gz" | sha256sum -c - >/dev/null 2>&1; \
        tar -xf fixuid.tar.gz; \
        mv fixuid /usr/local/bin/; \
        chmod u+s /usr/local/bin/fixuid; \
        rm fixuid.tar.gz; \
    }; \
    # Generate fixuid config
    mkdir -p /etc/fixuid; \
    { \
        echo "user: user"; \
        echo "group: user"; \
    } | tee /etc/fixuid/config.yml

# Adjust ENTRYPOINT
ENTRYPOINT [ "/usr/local/bin/fixuid", "-q" ]
CMD [ "/bin/sh" ]

# ---
# 4. Install Crystal dependencies & common packages
RUN --mount=type=cache,target=/var/cache/apk \
    set -eux; \
    apk add \
        curl \
        gc-dev \
        gcc \
        git \
        libevent-static \
        musl-dev \
        openssl-dev \
        openssl-libs-static \
        pcre-dev \
        sqlite-static \
        tzdata \
        yaml-static \
        zlib-dev \
        zlib-static \
    ;

# ---
# 5. Download and extract Crystal
#
# TODO: detect arch other than x86_64
RUN --mount=type=tmpfs,target=/tmp \
    set -eux; \
    export \
        CRYSTAL_VERSION=1.4.1 \
        CRYSTAL_SHA256=a5bdf1b78897b3cdc7d715b5f7adff79e84401d39b7ab546ab3249dc17fc770c; \
    cd /tmp; \
    { \
        curl --fail -Lo crystal.tar.gz https://github.com/crystal-lang/crystal/releases/download/${CRYSTAL_VERSION}/crystal-${CRYSTAL_VERSION}-1-linux-x86_64.tar.gz; \
        echo "${CRYSTAL_SHA256} *crystal.tar.gz" | sha256sum -c - >/dev/null 2>&1; \
        tar -xf crystal.tar.gz; \
        rm crystal-${CRYSTAL_VERSION}-1/lib/crystal/libgc.a; \
        mv crystal-${CRYSTAL_VERSION}-1 /usr/local/crystal; \
        rm crystal.tar.gz; \
    }; \
    # smoke test
    PATH=/usr/local/crystal/bin:$PATH; \
    [ "$(command -v crystal)" = '/usr/local/crystal/bin/crystal' ]; \
    [ "$(command -v shards)" = '/usr/local/crystal/bin/shards' ]; \
    crystal --version; \
    shards --version

ENV PATH /usr/local/crystal/bin:$PATH

# ---
# 6. Install development utilities
#
# TODO: Support arch other than x86_64
RUN --mount=type=cache,target=/var/cache/apk \
    --mount=type=tmpfs,target=/tmp \
    set -eux; \
    cd /tmp; \
    # Overmind (needs tmux)
    { \
        apk add \
            tmux \
        ; \
        export \
            OVERMIND_VERSION=2.3.0 \
            OVERMIND_SHA256=d6a715c0810ceb39c94bf61843befebe04a83a0469b53d6af0a52e2fea4e2ab3; \
        curl --fail -Lo overmind.gz https://github.com/DarthSim/overmind/releases/download/v${OVERMIND_VERSION}/overmind-v${OVERMIND_VERSION}-linux-amd64.gz; \
        echo "${OVERMIND_SHA256} *overmind.gz" | sha256sum -c - >/dev/null 2>&1; \
        gunzip overmind.gz; \
        chmod +x overmind; \
        mv overmind /usr/local/bin/; \
    }; \
    # Watchexec
    { \
        export \
            WATCHEXEC_VERSION=1.20.6 \
            WATCHEXEC_SHA256=6e746704b61d4a2a467546930a837ceef3a4003fc568e574cf6f43a798a4ab00; \
        curl --fail -Lo watchexec.tar.xz https://github.com/watchexec/watchexec/releases/download/v${WATCHEXEC_VERSION}/watchexec-${WATCHEXEC_VERSION}-x86_64-unknown-linux-musl.tar.xz; \
        echo "${WATCHEXEC_SHA256} *watchexec.tar.xz" | sha256sum -c - >/dev/null 2>&1; \
        tar -xf watchexec.tar.xz; \
        mv watchexec-${WATCHEXEC_VERSION}-x86_64-unknown-linux-musl/watchexec /usr/local/bin/; \
        rm -rf watchexec.tar.xz watchexec-${WATCHEXEC_VERSION}-x86_64-unknown-linux-musl; \
    }; \
    # smoke tests
    [ "$(command -v overmind)" = '/usr/local/bin/overmind' ]; \
    [ "$(command -v watchexec)" = '/usr/local/bin/watchexec' ]; \
    overmind --version; \
    watchexec --version
