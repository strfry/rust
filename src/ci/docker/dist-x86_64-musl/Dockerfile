FROM ubuntu:16.04

RUN apt-get update && apt-get install -y --no-install-recommends \
  g++ \
  make \
  file \
  wget \
  curl \
  ca-certificates \
  python2.7 \
  git \
  cmake \
  xz-utils \
  sudo \
  gdb \
  patch \
  libssl-dev \
  pkg-config

WORKDIR /build/

COPY scripts/musl-toolchain.sh /build/
# We need to mitigate rust-lang/rust#34978 when compiling musl itself as well
# TODO: Check what this issue is and if we can ignore it

RUN bash musl-toolchain.sh x86_64-linux-musl && rm -rf build

COPY scripts/sccache.sh /scripts/
RUN sh /scripts/sccache.sh

ENV RUST_CONFIGURE_ARGS \
      --musl-root-x86_64=/usr/local/x86_64-linux-musl \
      --enable-extended \
      --disable-docs

# Newer binutils broke things on some vms/distros (i.e., linking against
# unknown relocs disabled by the following flag), so we need to go out of our
# way to produce "super compatible" binaries.
#
# See: https://github.com/rust-lang/rust/issues/34978
#ENV CFLAGS_x86_64_unknown_linux_musl=-Wa,-mrelax-relocations=no

ENV HOSTS=x86_64-unknown-linux-musl \
    CC_x86_64_unknown_linux_musl=x86_64-linux-musl-gcc \
    CXX_x86_64_unknown_linux_musl=x86_64-linux-musl-g++

#    CARGO_TARGET_ARM_UNKNOWN_LINUX_MUSLEABIHF_LINKER=musl-gcc \
#    CARGO_TARGET_ARM_UNKNOWN_LINUX_MUSLEABIHF_RUNNER="qemu-arm -L /musl-arm"

ENV SCRIPT \
      python2.7 ../x.py test --host $HOSTS --target $HOSTS && \
      python2.7 ../x.py dist --host $HOSTS --target $HOSTS
