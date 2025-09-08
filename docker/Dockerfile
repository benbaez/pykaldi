# Dockerfile for building PyKaldi image from Ubuntu 18.04 image

# docker build  --tag pykaldi/pykaldi --build-arg SRILM_NAME=myname  \
# --build-arg SRILM_ORG=myorg --build-arg SRILM_EMAIL=myemail \
# --build-arg SRILM_ADDRESS=myaddress -f Dockerfile ..
FROM ubuntu:18.04

# SRI Required infor for download of package
ARG SRILM_NAME
ARG SRILM_ORG
ARG SRILM_EMAIL
ARG SRILM_ADDRESS

RUN test -n "${SRILM_NAME}" || (echo "Error: SRILM_NAME must be set!" && exit 1)
RUN test -n "${SRILM_ORG}" || (echo "Error: SRILM_ORG must be set!" && exit 1)
RUN test -n "${SRILM_EMAIL}" || (echo "Error: SRILM_EMAIL must be set!" && exit 1)
RUN test -n "${SRILM_ADDRESS}" || (echo "Error: SRILM_ADDRESS must be set!" && exit 1)

ARG PY3_VER=3.7

# To avoid tzdata location prompt in Python 3.8 and later 
ENV TZ="Etc/UTC"

# Python 3.8 and later versions warn when running as root
# ARG PIP_ROOT_USER_ACTION=ignore

# Install necessary system packages
RUN apt-get update \
    && apt-get install -y software-properties-common \
    && add-apt-repository -y ppa:deadsnakes/ppa \
    && apt-get update \
    && apt-get install -y \
    python${PY3_VER} \
    python3-pip \
    python2.7 \
    apt-utils \
    autoconf \
    automake \
    cmake \
    curl \
    g++ \
    gfortran \
    git \
    graphviz \
    libatlas3-base \
    libffi-dev \
    liblbfgs-dev \
    liblbfgsb-dev \
    libpython${PY3_VER}-dev \
    libtool \
    make \
    pkg-config \
    python3-cffi \
    sox \
    subversion \
    unzip \
    vim-tiny \
    wget \
    zlib1g-dev

RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Make python3 default
RUN update-alternatives --install /usr/bin/python3 python3 /usr/bin/python${PY3_VER} 1

# Link to use python3 and pip3
RUN ln -s /usr/bin/python3 /usr/bin/python \
    && ln -s /usr/bin/pip3 /usr/bin/pip

# Not sure this is needed when apt install above
# RUN pip install cffi

# Needs to run separate or else get can't find /tmp/... for last pip package to install
RUN pip install --upgrade pip

# Install necessary Python packages
RUN pip install \
    numpy \
    setuptools \
    pyparsing \
    jupyter \
    ninja

# Copy pykaldi directory into the container
COPY . /pykaldi

WORKDIR /pykaldi/tools

# ENV DEBIAN_FRONTEND=dialog

RUN ./check_dependencies.sh

# Separate layers to make debugging faster
RUN ./install_mkl.sh
RUN ./install_protobuf.sh
RUN ./install_clif.sh
RUN ./install_kaldi.sh

# SRILM for ngram-count tool
WORKDIR /pykaldi/kaldi/tools
RUN ./extras/install_srilm.sh SRILM_NAME SRILM_ORG SRILM_EMAIL SRILM_ADDRESS

#
WORKDIR /pykaldi

# Install pykaldi
RUN python setup.py install

# Create wheel package
RUN python setup.py bdist_wheel

# Build srilm
WORKDIR /pykaldi/tools/kaldi/tools
RUN ./extras/install_srilm.sh

# Source paths to libs and binaries
ENV KALDI_ROOT=kaldi

ENV LD_LIBRARY_PATH=$KALDI_ROOT/src/lib:$KALDI_ROOT/tools/openfst-1.6.7/lib:$LD_LIBRARY_PATH

ENV PATH=$KALDI_ROOT/src/lmbin/:$KALDI_ROOT/../kaldi_lm/:./utils/:$KALDI_ROOT/src/bin:$KALDI_ROOT/tools/openfst/bin:$KALDI_ROOT/src/fstbin/:$KALDI_ROOT/src/gmmbin/:$KALDI_ROOT/src/featbin/:$KALDI_ROOT/src/lm/:$KALDI_ROOT/src/sgmmbin/:$KALDI_ROOT/src/sgmm2bin/:$KALDI_ROOT/src/fgmmbin/:$KALDI_ROOT/src/latbin/:$KALDI_ROOT/src/nnetbin:$KALDI_ROOT/src/nnet2bin/:$KALDI_ROOT/src/online2bin/:$KALDI_ROOT/src/ivectorbin/:$KALDI_ROOT/src/kwsbin:$KALDI_ROOT/src/nnet3bin:$KALDI_ROOT/src/chainbin:$KALDI_ROOT/tools/sph2pipe_v2.5/:$KALDI_ROOT/src/rnnlmbin:.:$PATH

# Add ngram-count to path
ENV PATH=/pykaldi/tools/srilm-1.7.3/bin/i686-m64:$PATH

# Test pykaldi
RUN python -c "import kaldi; print(kaldi.__version__)"
