FROM debian:13

LABEL org.opencontainers.image.source=https://github.com/coalton-lang/base-images
LABEL org.opencontainers.image.description="ECL base image for Coalton CI"
LABEL org.opencontainers.image.licenses=MIT

ARG ECL_MAJOR_VERSION=24
ARG ECL_MEDIUM_VERSION=5
ARG ECL_MINOR_VERSION=10
ARG ECL_VERSION=${ECL_MAJOR_VERSION}.${ECL_MEDIUM_VERSION}.${ECL_MINOR_VERSION}
ARG ECL="ecl --norc -q"
ARG CC=gcc-14
ARG CXX=g++-14

# Install dependencies
ARG DEBIAN_FRONTEND=noninteractive
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
 && rm -r ecl-binary.tgz ecl-${ECL_VERSION} \
 && ${ECL} --eval "(print 'success)" --shell /dev/null

# Download and install quicklisp
RUN curl -O https://beta.quicklisp.org/quicklisp.lisp \
 && ${ECL} --load quicklisp.lisp --eval "(quicklisp-quickstart:install)" --shell /dev/null \
 && rm quicklisp.lisp

RUN for d in / /root /home/github ; do \
      for f in eclrc .eclrc ; do \
        if [ -f ${d}/${f} ]; then \
          echo ${d}/${f} ; cat ${d}/${f} ; \
        fi ; \
      done ; \
    done
RUN cat /no-such-file
ENV ECL_HOME=/usr/local/lib/ecl-${ECL_VERSION}
COPY eclrc /.eclrc

# workaround for old quicklisp and new FSET
WORKDIR /extras
RUN git clone https://github.com/melisgl/named-readtables.git
RUN git clone https://github.com/slburson/fset.git

WORKDIR /lisp
CMD ["ecl"]
