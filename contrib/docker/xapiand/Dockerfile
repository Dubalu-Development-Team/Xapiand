################################################################################
# __  __           _                 _
# \ \/ /__ _ _ __ (_) __ _ _ __   __| |
#  \  // _` | '_ \| |/ _` | '_ \ / _` |
#  /  \ (_| | |_) | | (_| | | | | (_| |
# /_/\_\__,_| .__/|_|\__,_|_| |_|\__,_|
#           |_|
#
# Build from the project root:
#   docker build -f contrib/docker/xapiand/Dockerfile -t dubalu/xapiand:latest .
#   docker tag dubalu/xapiand:latest dubalu/xapiand:$(docker run --rm dubalu/xapiand:latest --version)
#
# Run:
#   docker run -d -p 8880:8880 dubalu/xapiand:latest
#

################################################################################
# Stage 1: Builder
# Using Ubuntu 20.04 with Clang 12 (closest to Apple Clang 13, which is known
# to compile the codebase). GCC cannot be used because the source relies on
# Clang-specific preprocessor features (__has_builtin).
################################################################################
FROM ubuntu:20.04 AS builder

ENV DEBIAN_FRONTEND=noninteractive

# Install build dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    clang-12 \
    git \
    ca-certificates \
    make \
    autoconf \
    automake \
    libtool \
    perl \
    tcl \
    cmake \
    ninja-build \
    pkg-config \
    zlib1g-dev \
    uuid-dev \
  && rm -rf /var/lib/apt/lists/*

ENV CC=clang-12
ENV CXX=clang++-12

# Build Xapian from Kronuz fork (custom patches for Xapiand)
# Remove -Werror from Makefiles after configure to allow building with
# newer compilers that produce additional warnings on this older codebase.
RUN mkdir -p /usr/src \
  && git clone -b RELEASE/1.4 --single-branch --depth 1 \
    "https://github.com/Kronuz/xapian.git" /usr/src/xapian \
  && cd /usr/src/xapian/xapian-core \
  && ./preautoreconf \
  && autoreconf --force --install -Im4 \
  && ./configure \
    --disable-dependency-tracking \
    --disable-documentation \
    --enable-maintainer-mode \
    --prefix=/usr \
  && find . -name Makefile -exec sed -i 's/-Werror//g' {} + \
  && make -j"$(nproc)" \
  && make install \
  && cp bin/xapian-inspect /usr/bin/ \
  && rm -rf /usr/src/xapian

# Copy Xapiand source (uses local source instead of git clone)
COPY . /usr/src/Xapiand

# Build Xapiand
# Remove -Werror from build.ninja after cmake configure to allow building
# with newer compilers that produce warnings on bundled ChaiScript headers.
RUN mkdir -p /usr/src/Xapiand/build \
  && cd /usr/src/Xapiand/build \
  && cmake -GNinja \
    -DCMAKE_INSTALL_PREFIX:PATH=/usr \
    -DCMAKE_BUILD_TYPE=Release \
    .. \
  && sed -i 's/-Werror//g' build.ninja \
  && ninja \
  && ninja install \
  && rm -rf /usr/src/Xapiand


################################################################################
# Stage 2: Runtime
################################################################################
FROM ubuntu:20.04

LABEL maintainer="Dubalu LLC"

# Install only runtime dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    libuuid1 \
    zlib1g \
    libstdc++6 \
  && rm -rf /var/lib/apt/lists/*

# Copy installed artifacts from builder
COPY --from=builder /usr/bin/xapiand /usr/bin/xapiand
COPY --from=builder /usr/bin/xapian-inspect /usr/bin/xapian-inspect
COPY --from=builder /usr/lib/libxapian.so* /usr/lib/
COPY --from=builder /usr/share/xapiand/ /usr/share/xapiand/

# Refresh shared library cache
RUN ldconfig

# Create xapiand user and data directory
RUN mkdir -p /var/db \
  && addgroup --system xapiand \
  && adduser --disabled-password --system \
    --home /var/db/xapiand \
    --shell /sbin/nologin \
    --ingroup xapiand \
    xapiand

COPY contrib/docker/xapiand/entrypoint.sh /entrypoint.sh
RUN chmod +x /entrypoint.sh

EXPOSE 8880

ENTRYPOINT ["/entrypoint.sh"]
CMD ["-v"]
