FROM ubuntu:xenial

WORKDIR /root/

RUN apt-get update \
    && apt-get -y install curl wget git build-essential bsdtar \
        pkg-config  libglib2.0-dev libfdt-dev libpixman-1-dev zlib1g-dev \
        software-properties-common \
    && add-apt-repository ppa:jonathonf/python-3.6 \
    && apt-get update \
    && apt-get -y install python3.6 \
    && apt-get clean

# install rust environments
RUN bash -c " \
        curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh -s -- -y --default-toolchain nightly \
        && source '$HOME/.cargo/env' \
        && rustup component add rust-src \
        && rustup component add clippy --toolchain=nightly \
        && rustup component add rustfmt --toolchain=nightly \
        && cargo install cargo-xbuild \
    "

# install git lfs
# https://github.com/git-lfs/git-lfs/wiki/Installation
RUN build_deps="curl" && \
    apt-get update && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends ${build_deps} ca-certificates && \
    curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | bash && \
    DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends git-lfs && \
    git lfs install && \
    DEBIAN_FRONTEND=noninteractive apt-get purge -y --auto-remove ${build_deps} && \
    rm -r /var/lib/apt/lists/*

# fetch ovmf files
RUN git clone --depth=1 -b ovmf https://github.com/rust-osdev/uefi-rs \
    && cd uefi-rs/ \
    && git lfs pull \
    && cp -v x86_64/*.fd ~/

# build qemu
COPY ci/travis-qemu.sh /root/travis-qemu.sh
RUN /root/travis-qemu.sh

ENV PATH $PATH:/root/.cargo/bin
ENV PATH $PATH:/root/qemu/bin

CMD ["/bin/bash", "--login"]