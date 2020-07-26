ARG GCC_VERSION=gcc-10.2.0
FROM debian:buster-slim as build

ARG DEBIAN_FRONTEND=noninteractive
RUN apt update -y -q && apt upgrade -y -q && apt upgrade -y -q && apt install -y -q \
        bison \
        bzip2 \
        curl \
        file \
        gawk \
        g++ \
        gcc \
        git \
        make \
        patch \
        wget \
        xz-utils \
        texinfo \
        && rm -rf /var/lib/apt/lists/*
ARG MAKE_OPTS=-j12

ARG GCC_VERSION
ARG GCC_MIRROR=https://ftp.gnu.org/gnu/gcc
ARG GCC_BUILD_ARGS="--target=avr --disable-nls"

ARG BINUTILS_VERSION=binutils-2.34
ARG BINUTILS_MIRROR=https://ftp.gnu.org/gnu/binutils
ARG BINUTILS_BUILD_ARGS="--target=avr --disable-nls"
ARG AVR_LIBC_VERSION=avr-libc-2.0.0


RUN set -ex ; mkdir -p /gcc_build && cd /gcc_build && \
        curl -SL ${BINUTILS_MIRROR}/${BINUTILS_VERSION}.tar.xz | tar -xJC /gcc_build && \
        mkdir -p "${BINUTILS_VERSION}-build" && cd "${BINUTILS_VERSION}-build" && \
        /gcc_build/${BINUTILS_VERSION}/configure --prefix=/opt/${GCC_VERSION} ${BINUTILS_BUILD_ARGS} && \
        make ${MAKE_OPTS} && make install && cd / && export PATH=/opt/${GCC_VERSION}/bin:${PATH} && \
        curl -SL ${GCC_MIRROR}/${GCC_VERSION}/${GCC_VERSION}.tar.xz | tar -xJC /gcc_build && \
        cd /gcc_build/${GCC_VERSION}/ && ./contrib/download_prerequisites && cd /gcc_build && \
        mkdir -p "${GCC_VERSION}-build" && cd "${GCC_VERSION}-build" && \
        /gcc_build/${GCC_VERSION}/configure --prefix=/opt/${GCC_VERSION} ${GCC_BUILD_ARGS} --enable-languages=c,c++ && \
        make ${MAKE_OPTS} && make install && \
        curl -SL https://download.savannah.gnu.org/releases/avr-libc/${AVR_LIBC_VERSION}.tar.bz2 | tar -xjC /gcc_build && \
        mkdir -p "${AVR_LIBC_VERSION}-build" && cd "${AVR_LIBC_VERSION}-build" && \
        /gcc_build/${AVR_LIBC_VERSION}/configure --prefix=/opt/ --target=avr --host=avr && \
        make ${MAKE_OPTS} && make install &&\
        rm -Rf /gcc_build


FROM python:3.8-slim
ARG GCC_VERSION

RUN apt-get update && apt-get install -y make git && rm -rf /var/lib/apt/lists/*

COPY --from=build /opt/${GCC_VERSION} /opt/${GCC_VERSION}
RUN ln -sf /opt/${GCC_VERSION}/bin /opt/gcc

RUN mkdir -p /opt/conan && \
        gcc_stripped_version=${GCC_VERSION#*-} ; gcc_short_stripped_version=${gcc_stripped_version%.?} ;\
        echo "COMPILER_DIR=/opt/${GCC_VERSION}/bin\n\n[settings]\ncompiler=gcc\ncompiler.version=${gcc_short_stripped_version}\n\n[env]\nCOMPILER_DIR=\$COMPILER_DIR\n"> /opt/conan/compiler_local
RUN export uid=1000 gid=1000 && \
        mkdir -p /home/runner && \
        mkdir -p /etc/sudoers.d/ && \
        echo "runner:x:${uid}:${gid}:Runner,,,:/home/runner:/bin/bash">> /etc/passwd && \
        echo "runner:x:${uid}:" >> /etc/group && \
        echo "runner ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/runner && \
        chmod 0440 /etc/sudoers.d/runner && \
        chown ${uid}:${gid} -R /home/runner

USER runner
ENV HOME /home/runner
ENV PATH "/opt/${GCC_VERSION}/bin/:${PATH}:/home/runner/.local/bin/"

