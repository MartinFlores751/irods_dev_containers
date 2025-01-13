# syntax=docker/dockerfile:1

ARG CPPCHECK_VERSION=2.16.1

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
      irods-externals-clang13.0.1-0

# Specifiy desired version
ARG CPPCHECK_VERSION
RUN wget -qO - https://github.com/danmar/cppcheck/archive/refs/tags/${CPPCHECK_VERSION}.tar.gz | tar xz

# Run the cppcheck build
ENV PATH=${PATH}:/opt/irods-externals/cmake3.21.4-0/bin:/opt/irods-externals/clang13.0.1-0/bin
RUN cmake -DCMAKE_BUILD_TYPE=Release -DUSE_MATCHCOMPILER=ON -Wno-dev -G Ninja -B /build/cppcheck -S /cppcheck-${CPPCHECK_VERSION} && \
    cmake --build /build/cppcheck


FROM ubuntu:22.04

SHELL ["/bin/bash", "-c"]
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
    apt-get update && \
    apt-get install -y \
        git \
        gnupg \
        lsb-release \
        wget \
    && \
    rm -rf /tmp/*

RUN wget -qO - https://packages.irods.org/irods-signing-key.asc | apt-key add - && \
    echo "deb [arch=amd64] https://packages.irods.org/apt/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/renci-irods.list

RUN wget -qO - https://core-dev.irods.org/irods-core-dev-signing-key.asc | apt-key add - && \
    echo "deb [arch=amd64] https://core-dev.irods.org/apt/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/dev-renci-irods.list

ARG irods_version=4.3.2-0~jammy
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y \
        apt-transport-https \
        g++-11 \
        gcc-11 \
        irods-dev=${irods_version} \
        irods-runtime=${irods_version} \
        irods-externals-clang13.0.1-0 \
        irods-externals-cmake3.21.4-0 \
        irods-externals-fmt-libcxx8.1.1-1 \
        irods-externals-json3.10.4-0 \
        irods-externals-jwt-cpp0.6.99.1-0 \
        irods-externals-nanodbc-libcxx2.13.0-2 \
        irods-externals-spdlog-libcxx1.9.2-2 \
        libcurl4-gnutls-dev \
        libssl-dev \
        libssl3 \
        ninja-build \
    && \
    rm -rf /tmp/*

ARG cmake_path="/opt/irods-externals/cmake3.21.4-0/bin"
ENV PATH=${cmake_path}:$PATH
ARG clang_path="/opt/irods-externals/clang13.0.1-0/bin"
ENV PATH=${clang_path}:$PATH

# TODO: This should be a feature
# Install python-lsp
# Install yaml-lsp

# -----------------
# Developer Tooling
# -----------------

ARG CPPCHECK_VERSION

# Install vim
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y vim

# Install cppcheck
RUN --mount=type=bind,target=/build/cppcheck,source=/build/cppcheck,from=cppcheck,rw \
    --mount=type=bind,target=/cppcheck-${CPPCHECK_VERSION},source=/cppcheck-${CPPCHECK_VERSION},from=cppcheck \
    cmake --install /build/cppcheck

# Install CodeChecker
RUN --mount=type=cache,target=/root/.cache/pip,sharing=locked \
    pip3 install codechecker

# Add non-root user, taken from the following link:
# https://code.visualstudio.com/remote/advancedcontainers/add-nonroot-user
ARG USERNAME=dev
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Create the user
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    groupadd --gid $USER_GID $USERNAME && \
    useradd --uid $USER_UID --gid $USER_GID -m $USERNAME && \
    apt-get update && \
    apt-get install -y sudo && \
    echo $USERNAME ALL=\(root\) NOPASSWD:ALL > /etc/sudoers.d/$USERNAME && \
    chmod 0440 /etc/sudoers.d/$USERNAME

USER $USERNAME
