# Dockerfile for building PyKaldi image from Ubuntu 18.04 image

# docker build  --tag pykaldi/pykaldi -f Dockerfile ..
FROM ubuntu:18.04

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

WORKDIR /pykaldi

# Install pykaldi
RUN python setup.py install

# Create wheel package
RUN python setup.py bdist_wheel

WORKDIR /pykaldi/tools

# Source paths to libs and binaries
RUN . ./path.sh

# Test pykaldi
RUN python -c "import kaldi; print(kaldi.__version__)"
