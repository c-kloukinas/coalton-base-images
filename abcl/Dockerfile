FROM debian:13

LABEL org.opencontainers.image.source=https://github.com/coalton-lang/base-images
LABEL org.opencontainers.image.description="ABCL base image for Coalton CI"
LABEL org.opencontainers.image.licenses=MIT

ARG ABCL_MAJOR_VERSION=1
ARG ABCL_MEDIUM_VERSION=9
ARG ABCL_MINOR_VERSION=2
ARG ABCL_VERSION=${ABCL_MAJOR_VERSION}.${ABCL_MEDIUM_VERSION}.${ABCL_MINOR_VERSION}
ARG ABCL_TAR=abcl-bin-${ABCL_VERSION}.tar.gz
# ARG ABCL="abcl --noinit --noinform --nosystem --batch"
ARG ABCL="abcl --noinform --batch"

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
  default-jre-headless \
  \
  && rm -rf /var/lib/apt/lists/*

WORKDIR /src

# Download and install abcl
RUN curl -o ${ABCL_TAR} \
    -fsSL https://abcl.org/releases/${ABCL_VERSION}/${ABCL_TAR} \
 && tar xzf ${ABCL_TAR} \
 && mv abcl-bin-${ABCL_VERSION}/abcl.jar /bin/ \
 && echo '#! /bin/sh -' > /bin/abcl \
 && echo 'exec java -cp /bin/abcl.jar:"$CLASSPATH" org.armedbear.lisp.Main "$@"' >> /bin/abcl \
 && chmod 755 /bin/abcl \
 && ${ABCL} --eval "(print 'success)" \
 && rm -rf ${ABCL_TAR} abcl-bin-${ABCL_VERSION}/

# Download and install quicklisp
RUN curl -O https://beta.quicklisp.org/quicklisp.lisp \
 && ${ABCL} --load quicklisp.lisp --eval "(quicklisp-quickstart:install)" \
 && rm quicklisp.lisp

RUN for d in / /root /home/github ; do \
      for f in abclrc .abclrc ; do \
        if [ -f ${d}/${f} ]; then \
          echo ${d}/${f} ; cat ${d}/${f} ; \
        fi ; \
      done ; \
    done
RUN cat /no-such-file
COPY abclrc /.abclrc

# workaround for old quicklisp and new FSet
WORKDIR /extras
RUN git clone https://github.com/melisgl/named-readtables.git
RUN git clone https://github.com/slburson/fset.git

WORKDIR /lisp
CMD ["abcl"]
