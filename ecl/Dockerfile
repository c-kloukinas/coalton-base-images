FROM debian:13

LABEL org.opencontainers.image.source=https://github.com/coalton-lang/base-images
LABEL org.opencontainers.image.description="ECL base image for Coalton CI"
LABEL org.opencontainers.image.licenses=MIT

ARG ECL_MAJOR_VERSION=26
ARG ECL_MEDIUM_VERSION=3
ARG ECL_MINOR_VERSION=27
ARG ECL_VERSION=${ECL_MAJOR_VERSION}.${ECL_MEDIUM_VERSION}.${ECL_MINOR_VERSION}
# ARG ECL="ecl --norc -q"
ARG ECL="ecl -q"
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
RUN ( curl -L -o ecl-binary.tgz \
      https://ecl.common-lisp.dev/static/files/release/ecl-${ECL_VERSION}.tgz \
    || curl --insecure -o ecl-binary.tgz \
      https://ecl.common-lisp.dev/static/files/release/ecl-${ECL_VERSION}.tgz \
    ) \
 && tar xzf ecl-binary.tgz \
 && gcc --version \
 && gcc-14 --version \
 && (cd ecl-${ECL_VERSION} && ./configure --prefix=/ && make && make install) \
 && rm -r ecl-binary.tgz ecl-${ECL_VERSION} \
 && ${ECL} --eval "(print 'success)" --shell /dev/null

# Download and install quicklisp
RUN curl -O https://beta.quicklisp.org/quicklisp.lisp \
 && ${ECL} --load quicklisp.lisp --eval "(quicklisp-quickstart:install)" --shell /dev/null \
 && rm quicklisp.lisp && mkdir -p /root/

ENV ECL_HOME=/usr/local/lib/ecl-${ECL_VERSION}
COPY eclrc /.eclrc
COPY eclrc /root/.eclrc
RUN chmod 644 /.eclrc /root/.eclrc

# workaround for old quicklisp and new FSET
WORKDIR /extras
RUN git clone https://github.com/melisgl/named-readtables.git
RUN git clone https://github.com/slburson/misc-extensions.git
RUN git clone https://github.com/slburson/fset.git
RUN git clone --single-branch --branch fix-define-test-package-for-abcl \
              https://github.com/appleby/fiasco.git fiasco

# fetch all needed QL libraries
ARG ALL_LIBS="acclimation \
alexandria \
bordeaux-threads \
cl-ppcre \
closer-mop \
coalton \
computable-reals \
concrete-syntax-tree \
documentation-utils \
eclector \
fiasco \
float-features \
global-vars \
html-entities \
in-nomine \
introspect-environment \
iterate-release \
lisp-namespace \
mgl-pax \
mt19937 \
parse-declarations \
parse-number \
serapeum \
spinneret \
split-sequence \
string-case \
trivia \
trivial-arguments \
trivial-cltl2 \
trivial-features \
trivial-file-size \
trivial-garbage \
trivial-gray-streams \
trivial-indent \
trivial-macroexpand-all \
type-i \
yason"

WORKDIR /

RUN ( echo '(progn (format t "Loading libs~%")' \
     ; for f in ${ALL_LIBS} ; do \
         echo '  (cl:handler-case (ql:quickload :$f) (cl:error (c) (declare (ignore c) (format t "*** Failed to load ~A!!! ***~%" :$f))))' \
     ; done \
     ; echo '(format t "Finished loading libs~%"))' \
    ) > /load-all-libs.lisp
RUN wc -l /load-all-libs.lisp
RUN ${ECL} --load /.eclrc --shell /load-all-libs.lisp

WORKDIR /lisp

CMD ["ecl"]
