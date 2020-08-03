# Based on https://github.com/esemeniuc/openpose-docker
FROM nvidia/cuda:10.0-cudnn7-devel-ubuntu18.04

# Install system dependecies
RUN apt update
RUN DEBIAN_FRONTEND=noninteractive apt install -y --no-install-recommends \
python3-dev python3-pip git g++ wget make libprotobuf-dev protobuf-compiler libopencv-dev \
libgoogle-glog-dev libboost-all-dev libcaffe-cuda-dev libhdf5-dev libatlas-base-dev

# Run Python dependecies
RUN pip3 install numpy opencv-python

# replace cmake as old version has CUDA variable bugs
WORKDIR /tmp
RUN wget https://github.com/Kitware/CMake/releases/download/v3.16.0/cmake-3.16.0-Linux-x86_64.tar.gz
RUN tar xzf cmake-3.16.0-Linux-x86_64.tar.gz -C /opt
RUN rm cmake-3.16.0-Linux-x86_64.tar.gz
ENV PATH=/opt/cmake-3.16.0-Linux-x86_64/bin:$PATH

# Download OpenPose
RUN git clone https://github.com/CMU-Perceptual-Computing-Lab/openpose /openpose
WORKDIR /openpose

# Compile
WORKDIR /openpose/build
RUN cmake -DBUILD_PYTHON=ON .. && make -j `nproc`

WORKDIR /openpose/models
RUN ./getModels.sh

Workdir /openpose
