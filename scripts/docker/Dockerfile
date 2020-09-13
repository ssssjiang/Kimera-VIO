# Use an official Python runtime as a parent image
FROM ubuntu:18.04

MAINTAINER Antoni Rosinol "arosinol@mit.edu"

RUN sed -i 's#http://archive.ubuntu.com/#http://mirrors.163.com/#' /etc/apt/sources.list

# To avoid tzdata asking for geographic location...
ENV DEBIAN_FRONTEND noninteractive

# Set the working directory to /root
ENV DIRPATH /root
WORKDIR $DIRPATH

#Install build dependencies
RUN apt-get update && apt-get install -y --no-install-recommends apt-utils
RUN apt-get update && apt-get install -y git cmake

# Install xvfb to provide a display to container for GUI related testing.
RUN apt-get update && apt-get install -y xvfb

# Install GTSAM
RUN apt-get update && apt-get install -y libboost-all-dev
RUN git clone https://github.com/borglab/gtsam.git
RUN cd gtsam && \
    git fetch && \
    mkdir build && \
    cd build && \
    cmake -DCMAKE_INSTALL_PREFIX=/usr/local -DGTSAM_BUILD_TESTS=OFF -DGTSAM_BUILD_EXAMPLES_ALWAYS=OFF -DCMAKE_BUILD_TYPE=Release -DGTSAM_BUILD_UNSTABLE=ON .. && \
    make -j$(nproc) install

# dev
RUN apt-get update && apt-get install -y \
    libboost-all-dev \
    freeglut3-dev \
    libglew-dev \
    ffmpeg libavcodec-dev libavutil-dev libavformat-dev libswscale-dev \
    libjpeg-dev libpng-dev libtiff5-dev libopenexr-dev libv4l-dev \
    libeigen3-dev \
    libopencv-dev \
    libsuitesparse-dev libatlas-base-dev \
    libgoogle-glog-dev \
    lua5.3 liblua5.3-dev \
    valgrind kcachegrind google-perftools graphviz \
    ninja-build \
    google-mock \
    libcairo2-dev \
    zlib1g-dev \
    cppcheck-gui \
    htop

# Install Open_GV
RUN git clone https://github.com/laurentkneip/opengv
RUN cd opengv && \
      mkdir build
RUN cd opengv/build && \
      cmake -DCMAKE_BUILD_TYPE=Release \
      -DCMAKE_INSTALL_PREFIX=/usr/local \
      -DEIGEN_INCLUDE_DIRS=$DIRPATH/gtsam/gtsam/3rdparty/Eigen \
      -DEIGEN_INCLUDE_DIR=$DIRPATH/gtsam/gtsam/3rdparty/Eigen .. && \
      make -j$(nproc) install

# Install DBoW2
RUN git clone https://github.com/dorian3d/DBoW2.git
RUN cd DBoW2 && \
      mkdir build && \
      cd build && \
      cmake .. && \
      make -j$(nproc) install

RUN apt-get update && \
      apt-get install python-pip -y
RUN pip install --upgrade pip
RUN pip install pyrsistent

# Install RobustPGO
ADD https://api.github.com/repos/MIT-SPARK/Kimera-RPGO/git/refs/heads/master version.json
RUN git clone https://github.com/MIT-SPARK/Kimera-RPGO.git
RUN cd Kimera-RPGO && \
      mkdir build && \
      cd build && \
      cmake .. && \
      make -j$(nproc)

# [Optional] Install spark_vio_evaluation from PyPI
RUN apt-get update && apt-get install -y python-pip python-dev python-tk
# Hack to avoid Docker's cache when spark_vio_evaluation master branch is updated.
ADD https://api.github.com/repos/ToniRV/spark_vio_evaluation/git/refs/heads/master version.json
RUN git clone https://github.com/ToniRV/spark_vio_evaluation.git
RUN cd spark_vio_evaluation && pip install .

# Add credentials on build
ARG SSH_PRIVATE_KEY
RUN mkdir /root/.ssh/
RUN echo "${SSH_PRIVATE_KEY}" > /root/.ssh/id_rsa
RUN chmod 600 ~/.ssh/id_rsa

# Make sure your domain is accepted
RUN touch /root/.ssh/known_hosts
RUN ssh-keyscan -t rsa github.com >> /root/.ssh/known_hosts

# Install glog, gflags
RUN apt-get update && apt-get install -y libgflags2.2 libgflags-dev libgoogle-glog0v5 libgoogle-glog-dev

