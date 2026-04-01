FROM debian:13
# Debian 13 is trixie
# Debian 12 is bookworm
# Debian 11 is bullseye
# Debian 10 is buster
# Debian  9 is stretch 	# miktex may need this one

LABEL org.opencontainers.image.source=https://github.com/coalton-lang/base-images
LABEL org.opencontainers.image.description="ECL base image for Coalton CI"
LABEL org.opencontainers.image.licenses=MIT

ARG ECL_MAJOR_VERSION=24
ARG ECL_MEDIUM_VERSION=5
ARG ECL_MINOR_VERSION=10
ARG ECL_VERSION=${ECL_MAJOR_VERSION}.${ECL_MEDIUM_VERSION}.${ECL_MINOR_VERSION}
# ARG ECL="ecl --norc -q"
ARG ECL="ecl -q"
ARG CC=gcc-14
ARG CXX=g++-14

# Install dependencies
ARG DEBIAN_FRONTEND=noninteractive

## From https://hub.docker.com/r/miktex/miktex/dockerfile
# 1 Install dependencies for miktex
RUN    apt-get update \
    && apt-get install -y --no-install-recommends \
           apt-transport-https \
           ca-certificates \
           dirmngr \
           ghostscript \
           gnupg \
           gosu \
           make \
           perl
# 2 Install miktex's repository
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys D6BC243565B2087BC3F897C9277A7293F59E4889
RUN echo "deb http://miktex.org/download/debian trixie universe" | tee /etc/apt/sources.list.d/miktex.list
RUN    apt-get update -y \
    && apt-get install -y --no-install-recommends \
           miktex

# 3 Setup miktex
RUN    miktexsetup finish \
    && initexmf --admin --set-config-value=[MPM]AutoInstall=1 \
    && mpm --admin --update-db \
    && mpm --admin \
           --install amsfonts \
           --install biber-linux-x86_64 \
    && initexmf --admin --update-fndb

# 4 Define some miktex environment variables
ENV MIKTEX_USERCONFIG=/miktex/.miktex/texmfs/config
ENV MIKTEX_USERDATA=/miktex/.miktex/texmfs/data
ENV MIKTEX_USERINSTALL=/miktex/.miktex/texmfs/install

# Ecl Dependencies
RUN apt-get update && apt-get install --no-install-recommends -y \
  ca-certificates \
  curl \
  git \
  bzip2 \
  libgmp10 \
  libmpfr6 \
  \
  build-essential \
  libgmp-dev \
  libmpfr-dev \
  netbase \
  \
  && rm -rf /var/lib/apt/lists/*

WORKDIR /src

# Download and install ecl
RUN curl -L -o ecl-binary.tgz \
 https://common-lisp.net/project/ecl/static/files/release/ecl-${ECL_VERSION}.tgz \
 && tar xzf ecl-binary.tgz \
 && gcc --version \
 && gcc-14 --version \
 && (cd ecl-${ECL_VERSION} && ./configure --prefix=/ && make && make install) \
 && (echo Building the Ecl PDF manual ; cd ecl-${ECL_VERSION} && make doc && ( cd build/doc/ ; make manual && cd manual ; ( make ecl-manual.pdf && cp -p ecl-manual.pdf / ) || echo PDF manual building failed ) ) \
 && rm -r ecl-binary.tgz ecl-${ECL_VERSION} \
 && ${ECL} --eval "(print 'success)" --shell /dev/null

# Download and install quicklisp
RUN curl -O https://beta.quicklisp.org/quicklisp.lisp \
 && ${ECL} --load quicklisp.lisp --eval "(quicklisp-quickstart:install)" --shell /dev/null \
 && rm quicklisp.lisp

ENV ECL_HOME=/usr/local/lib/ecl-${ECL_VERSION}
COPY eclrc /.eclrc
COPY eclrc /root/.eclrc
RUN chmod 644 /.eclrc /root/.eclrc

# workaround for old quicklisp and new FSET
WORKDIR /extras
RUN git clone https://github.com/melisgl/named-readtables.git
RUN git clone https://github.com/slburson/fset.git

WORKDIR /lisp
CMD ["ecl"]
