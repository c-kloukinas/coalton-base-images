FROM debian:12

LABEL org.opencontainers.image.source=https://github.com/coalton-lang/base-images
LABEL org.opencontainers.image.description="CLASP base image for Coalton CI"
LABEL org.opencontainers.image.licenses=MIT

ARG CLASP="clasp --non-interactive --norc"

# Install dependencies
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install --no-install-recommends -y \
  ca-certificates \
  curl \
  git \
  bzip2 \
  libgmp10 \
  libmpfr6 \
  build-essential \
  libgmp-dev \
  libmpfr-dev \
  gpg \
  netbase

# # Try to get the latest llvm packages - https://apt.llvm.org/
# RUN bash -c "$(wget -O - https://apt.llvm.org/llvm.sh)"

# RUN apt-get install --no-install-recommends -y software-properties-common
 
RUN rm -rf /var/lib/apt/lists/*

WORKDIR /src

# Download and install clasp
 ## && add-apt-repository "deb https://www.thirdlaw.tech/pkg/bookworm/ ./" \

RUN curl -fsSL https://www.thirdlaw.tech/pkg/key.gpg | gpg --yes --dearmor --output /etc/apt/trusted.gpg.d/thirdlaw.gpg \
 && echo 'deb [signed-by=/etc/apt/trusted.gpg.d/thirdlaw.gpg] https://www.thirdlaw.tech/pkg/bookworm/ ./' | tee /etc/apt/sources.list.d/thirdlaw.list \
 && apt-get update \
 && apt-get install --no-install-recommends -y clasp-cl \
 && rm -rf /var/lib/apt/lists/* \
 && ${CLASP} --eval "(print 'success)"

# Download and install quicklisp
RUN curl -O https://beta.quicklisp.org/quicklisp.lisp \
 && ${CLASP} --load quicklisp.lisp --eval "(quicklisp-quickstart:install)" \
 && rm quicklisp.lisp

RUN for d in / /root /home/github ; do \
      for f in clasprc .clasprc ; do \
        if [ -f ${d}/${f} ]; then \
          echo ${d}/${f} ; cat ${d}/${f} ; \
        fi ; \
      done ; \
    done
RUN cat /no-such-file
COPY clasprc /.clasprc

# workaround for old quicklisp and new FSet
WORKDIR /extras
RUN git clone https://github.com/melisgl/named-readtables.git
RUN git clone https://github.com/slburson/fset.git

WORKDIR /lisp
CMD ["clasp"]
