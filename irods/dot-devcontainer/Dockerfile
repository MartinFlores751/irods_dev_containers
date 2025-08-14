# syntax=docker/dockerfile:1

ARG CPPCHECK_VERSION=2.16.1
ARG GNU_STOW_VERSION=2.4.1
ARG DIFFTASTIC_VERSION=0.62.0
ARG MERGIRAF_VERSION=0.4.0
ARG DELTA_VERSION=0.18.2



FROM ubuntu:22.04 AS cppcheck
ENV DEBIAN_FRONTEND=noninteractive

# Re-enable apt caching for RUN --mount
RUN rm -f /etc/apt/apt.conf.d/docker-clean && \
    echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' > /etc/apt/apt.conf.d/keep-cache

# Make sure we're starting with an up-to-date image
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get upgrade -y && \
    apt-get autoremove -y --purge

# Install system deps
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get install -y \
      binutils \
      gnupg \
      lsb-release \
      ninja-build \
      wget

# Get the iRODS package repos
RUN wget -qO - https://packages.irods.org/irods-signing-key.asc | apt-key add - && \
    echo "deb [arch=amd64] https://packages.irods.org/apt/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/renci-irods.list && \
    wget -qO - https://core-dev.irods.org/irods-core-dev-signing-key.asc | apt-key add - && \
    echo "deb [arch=amd64] https://core-dev.irods.org/apt/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/renci-irods-core-dev.list

# Install the iRDOS packages
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update -y && \
    apt-get install -y \
      irods-externals-cmake3.21.4-0 \
      irods-externals-clang16.0.6-0

# Specifiy desired version
ARG CPPCHECK_VERSION
RUN wget -qO - https://github.com/danmar/cppcheck/archive/refs/tags/${CPPCHECK_VERSION}.tar.gz | tar xz

# Run the cppcheck build
ENV PATH=${PATH}:/opt/irods-externals/cmake3.21.4-0/bin:/opt/irods-externals/clang16.0.6-0/bin
RUN cmake -DCMAKE_BUILD_TYPE=Release -DUSE_MATCHCOMPILER=ON -Wno-dev -G Ninja -B /build/cppcheck -S /cppcheck-${CPPCHECK_VERSION} && \
    cmake --build /build/cppcheck






FROM ubuntu:22.04 AS mergiraf
ENV DEBIAN_FRONTEND=noninteractive

# Re-enable apt caching for RUN --mount
RUN rm -f /etc/apt/apt.conf.d/docker-clean && \
    echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' > /etc/apt/apt.conf.d/keep-cache

# Make sure we're starting with an up-to-date image
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get upgrade -y && \
    apt-get autoremove -y --purge

# Install system deps
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get install -y \
      build-essential \
      wget

# install rustup; build and install mergiraf
RUN wget -q -O - https://sh.rustup.rs | sh -s -- -y && \
    . /root/.cargo/env && \
    cargo install --locked mergiraf








# TODO: Simplify gnustow build
FROM ubuntu:22.04 AS gnustow
ENV DEBIAN_FRONTEND=noninteractive

# Re-enable apt caching for RUN --mount
RUN rm -f /etc/apt/apt.conf.d/docker-clean && \
    echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' > /etc/apt/apt.conf.d/keep-cache

# Make sure we're starting with an up-to-date image
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get upgrade -y && \
    apt-get autoremove -y --purge

# Install system deps
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get install -y \
      build-essential \
      binutils \
      gnupg \
      lsb-release \
      wget

# Get the iRODS package repos
RUN wget -qO - https://packages.irods.org/irods-signing-key.asc | apt-key add - && \
    echo "deb [arch=amd64] https://packages.irods.org/apt/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/renci-irods.list && \
    wget -qO - https://core-dev.irods.org/irods-core-dev-signing-key.asc | apt-key add - && \
    echo "deb [arch=amd64] https://core-dev.irods.org/apt/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/renci-irods-core-dev.list

# Install the iRDOS packages
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update -y && \
    apt-get install -y \
      irods-externals-clang16.0.6-0

# Specifiy desired version
# TODO: gpg --verify stow.tar.gz.sig
ARG GNU_STOW_VERSION
RUN wget -q -O stow.tar.gz https://ftp.gnu.org/gnu/stow/stow-${GNU_STOW_VERSION}.tar.gz && \
    wget -q -O stow.tar.gz.sig https://ftp.gnu.org/gnu/stow/stow-${GNU_STOW_VERSION}.tar.gz.sig && \
    tar xaf stow.tar.gz

# Run the gnu stow build
ENV PATH=${PATH}:/opt/irods-externals/cmake3.21.4-0/bin:/opt/irods-externals/clang16.0.6-0/bin
WORKDIR /stow-${GNU_STOW_VERSION}
RUN ./configure && \
    make






FROM ubuntu:22.04 AS delta
ENV DEBIAN_FRONTEND=noninteractive

# Re-enable apt caching for RUN --mount
RUN rm -f /etc/apt/apt.conf.d/docker-clean && \
    echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' > /etc/apt/apt.conf.d/keep-cache

# Make sure we're starting with an up-to-date image
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get upgrade -y && \
    apt-get autoremove -y --purge

# Install system deps
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get install -y \
    wget && \
    mkdir /delta

WORKDIR /delta
ARG DELTA_VERSION
RUN wget -q -O delta.deb https://github.com/dandavison/delta/releases/download/${DELTA_VERSION}/git-delta_${DELTA_VERSION}_amd64.deb










FROM ubuntu:22.04 AS CodeChecker
ENV DEBIAN_FRONTEND=noninteractive

# Re-enable apt caching for RUN --mount
RUN rm -f /etc/apt/apt.conf.d/docker-clean && \
    echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' > /etc/apt/apt.conf.d/keep-cache

# Make sure we're starting with an up-to-date image
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get upgrade -y && \
    apt-get autoremove -y --purge

# Install system deps
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get install -y \
      build-essential \
      curl \
      g++ \
      gnupg \
      lsb-release \
      ninja-build \
      wget

# Get the iRODS package repos
RUN wget -qO - https://packages.irods.org/irods-signing-key.asc | apt-key add - && \
    echo "deb [arch=amd64] https://packages.irods.org/apt/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/renci-irods.list && \
    wget -qO - https://core-dev.irods.org/irods-core-dev-signing-key.asc | apt-key add - && \
    echo "deb [arch=amd64] https://core-dev.irods.org/apt/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/renci-irods-core-dev.list

# Install the iRDOS packages
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update -y && \
    apt-get install -y \
      irods-externals-cmake3.21.4-0 \
      irods-externals-clang16.0.6-0

ARG CODECHECKER_VERSION=6.24.4
RUN wget -qO - https://github.com/Ericsson/codechecker/archive/refs/tags/v${CODECHECKER_VERSION}.tar.gz | tar xz







FROM ubuntu:22.04

SHELL [ "/bin/bash", "-c" ]
ENV DEBIAN_FRONTEND=noninteractive

# Re-enable apt caching for RUN --mount
RUN rm -f /etc/apt/apt.conf.d/docker-clean && \
    echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' > /etc/apt/apt.conf.d/keep-cache

# Make sure we're starting with an up-to-date image
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get upgrade -y && \
    apt-get autoremove -y --purge && \
    rm -rf /tmp/*
# To mark all installed packages as manually installed:
#apt-mark showauto | xargs -r apt-mark manual

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    --mount=type=cache,target=/root/.cache/pip,sharing=locked \
    --mount=type=cache,target=/root/.cache/wheel,sharing=locked \
    apt-get update && \
    apt-get install -y \
        apt-transport-https \
        catch2 \
        ccache \
        cmake \
        g++ \
        gcc \
        git \
        gnupg \
        help2man \
        libarchive-dev \
        libbz2-dev \
        libcurl4-gnutls-dev \
        libfmt-dev \
        libfuse-dev \
        libjson-perl \
        libkrb5-dev \
        libpam0g-dev \
        libspdlog-dev \
        libssl-dev \
        libsystemd-dev \
        libxml2-dev \
        lsb-release \
        lsof \
        make \
        ninja-build \
        nlohmann-json3-dev \
        odbc-postgresql \
        postgresql \
        python3 \
        python3-dev \
        python3-pip \
        python3-distro \
        python3-jsonschema \
        python3-packaging \
        python3-psutil \
        python3-pyodbc \
        python3-requests \
        sudo \
        super \
        unixodbc-dev \
        wget \
        zlib1g-dev \
        flex \
        bison \
    && \
    python3 -m pip install lief && \
    rm -rf /tmp/*

RUN wget -qO - https://packages.irods.org/irods-signing-key.asc | apt-key add - && \
    echo "deb [arch=amd64] https://packages.irods.org/apt/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/renci-irods.list && \
    wget -qO - https://core-dev.irods.org/irods-core-dev-signing-key.asc | apt-key add - && \
    echo "deb [arch=amd64] https://core-dev.irods.org/apt/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/renci-irods-core-dev.list

RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y \
        'irods-externals*' \
    && \
    rm -rf /tmp/*

ARG cmake_path="/opt/irods-externals/cmake3.21.4-0/bin"
ENV PATH=${cmake_path}:$PATH

ARG clang_path="/opt/irods-externals/clang16.0.6-0/bin"
ENV PATH=${clang_path}:$PATH

ENV file_extension="deb"
ENV package_manager="apt-get"

ENV CCACHE_DIR="/irods_build_cache"
# Default to a reasonably large cache size
ENV CCACHE_MAXSIZE="64G"
# Allow for a lot of files (1.5M files, 300 per directory)
ENV CCACHE_NLEVELS="3"
# Allow any uid to use cache
ENV CCACHE_UMASK="000"

# -----------------
# Developer Tooling
# -----------------

ARG CPPCHECK_VERSION
ARG GNU_STOW_VERSION

# Install vim
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y vim

# Install cppcheck
RUN --mount=type=bind,target=/build/cppcheck,source=/build/cppcheck,from=cppcheck,rw \
    --mount=type=bind,target=/cppcheck-${CPPCHECK_VERSION},source=/cppcheck-${CPPCHECK_VERSION},from=cppcheck \
    cmake --install /build/cppcheck

# Install gnu stow
# TODO: Verify WORKDIR change is required
WORKDIR /stow-${GNU_STOW_VERSION}
RUN --mount=type=bind,target=/stow-${GNU_STOW_VERSION},source=/stow-${GNU_STOW_VERSION},from=gnustow \
    make install
WORKDIR /

# Install difftastic and mergiraf
WORKDIR /usr/bin

ARG DIFFTASTIC_VERSION
RUN wget -q -O - https://github.com/Wilfred/difftastic/releases/download/${DIFFTASTIC_VERSION}/difft-x86_64-unknown-linux-gnu.tar.gz | tar xz

# TODO: See if there's an easy fix for bad libc6 version
# ARG MERGIRAF_VERSION
# RUN wget -q -O - https://codeberg.org/mergiraf/mergiraf/releases/download/v${MERGIRAF_VERSION}/mergiraf_x86_64-unknown-linux-gnu.tar.gz | tar xz

# mergiraf
COPY --from=mergiraf /root/.cargo/bin/mergiraf /usr/bin/

WORKDIR /


# Install delta
RUN --mount=type=bind,target=/delta,source=/delta,from=delta \
    dpkg -i /delta/delta.deb

# Install git
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y software-properties-common && \
    add-apt-repository ppa:git-core/ppa
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y git

# Install CodeChecker and pre-commit
RUN --mount=type=cache,target=/root/.cache/pip,sharing=locked \
    pip3 install codechecker pre-commit

# Add non-root user, taken from the following link:
# https://code.visualstudio.com/remote/advancedcontainers/add-nonroot-user
ARG USERNAME=dev
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Create the user
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    groupadd --gid $USER_GID $USERNAME && \
    useradd --uid $USER_UID --gid $USER_GID --shell /bin/bash -m $USERNAME && \
    apt-get update && \
    apt-get install -y sudo && \
    echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME && \
    chmod 0440 /etc/sudoers.d/$USERNAME

# Get the latest clang-tidy-diff.py script
# Use the following command to run it:
# git diff -U0 main...HEAD | python3 ~/clang-tidy-diff.py -p 1 -config-file /workspaces/irods/.clang-tidy -use-color -path /workspaces/irods/compile_commands.json -quiet -j 4
RUN wget -q -O /home/dev/clang-tidy-diff.py https://raw.githubusercontent.com/llvm/llvm-project/refs/heads/main/clang-tools-extra/clang-tidy/tool/clang-tidy-diff.py

USER $USERNAME
