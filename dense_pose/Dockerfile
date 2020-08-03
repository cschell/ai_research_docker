FROM nvidia/cuda:10.2-cudnn7-devel-ubuntu18.04

RUN apt update
RUN apt install -y git \
                   build-essential \
                   curl \
                   wget \
                   libglib2.0-0 \
                   libsm6 \
                   libxext6 \
                   libxrender-dev \
                   gcc-5

# Setup Miniconda with Python 2.7
RUN wget --quiet https://repo.anaconda.com/miniconda/Miniconda2-4.7.12-Linux-x86_64.sh -O ~/miniconda.sh && \
    /bin/bash ~/miniconda.sh -b -p /opt/conda && \
    rm ~/miniconda.sh && \
    ln -s /opt/conda/etc/profile.d/conda.sh /etc/profile.d/conda.sh && \
    echo ". /opt/conda/etc/profile.d/conda.sh" >> ~/.bashrc && \
    echo "conda activate base" >> ~/.bashrc && \
    find /opt/conda/ -follow -type f -name '*.a' -delete && \
    find /opt/conda/ -follow -type f -name '*.js.map' -delete && \
    /opt/conda/bin/conda clean -afy

ENV PATH $PATH:/opt/conda/bin

####
#### Compile PyTorch to get Caffe2 header files (takes forever)
####
RUN git clone --recursive -b v1.1.0 https://github.com/pytorch/pytorch /pytorch
WORKDIR /pytorch
RUN pip install numpy pyyaml mkl mkl-include setuptools cmake cffi typing
RUN MAX_JOBS=8 python setup.py install

ENV PYTHONPATH $PYTHONPATH:/pytorch/build


####
#### Install MKL
####
RUN wget -O- https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS-2019.PUB | apt-key add -
RUN echo deb https://apt.repos.intel.com/mkl all main > /etc/apt/sources.list.d/intel-mkl.list
RUN apt update && apt install -y intel-mkl-64bit-2020.2-108


####
#### Install the COCO API
####
RUN pip install --upgrade pip
# matplotlib >= 3.1 requires Python 3
RUN pip install "matplotlib<3.1"
RUN pip install cython
RUN git clone https://github.com/cocodataset/cocoapi.git /cocoapi
WORKDIR /cocoapi/PythonAPI
RUN make install


####
#### Install Protobuf
####
WORKDIR /
RUN wget https://github.com/protocolbuffers/protobuf/releases/download/v3.6.1/protobuf-all-3.6.1.tar.gz
RUN tar xzvf protobuf-all-3.6.1.tar.gz
WORKDIR /protobuf-3.6.1
RUN apt install -y autoconf automake libtool curl make g++ unzip
RUN ./autogen.sh
RUN ./configure
RUN make
# RUN make check
RUN make install
RUN ldconfig

####
#### Install Cmake
####
WORKDIR /
RUN wget https://github.com/Kitware/CMake/releases/download/v3.18.0/cmake-3.18.0.tar.gz
RUN tar xzvf cmake-3.18.0.tar.gz
WORKDIR  /cmake-3.18.0
RUN apt install -y libssl-dev
RUN ./bootstrap
RUN make && make install
RUN cmake --version

####
#### Install nvidia_toolkit
####
WORKDIR /
RUN wget http://developer.download.nvidia.com/compute/cuda/10.2/Prod/local_installers/cuda_10.2.89_440.33.01_linux.run
RUN sh cuda_10.2.89_440.33.01_linux.run --toolkit --silent


####
#### Compile and install DensePose
####

# Clone the Denspose repository
RUN git clone https://github.com/facebookresearch/densepose /densepose

WORKDIR /densepose

# Install Python dependencies
RUN python -m pip install -r /densepose/requirements.txt

# Set appropriate environment variables
ENV CUDNN_LIBRARY_PATH /usr/lib/x86_64-linux-gnu/
ENV CUDNN_INCLUDE_PATH /usr/include/x86_64-linux-gnu/cudnn

ENV LD_LIBRARY_PATH $LD_LIBRARY_PATH:/usr/lib/x86_64-linux-gnu/:/usr/local/lib
ENV LD_INCLUDE_PATH /pytorch/build/include/:/pytorch/torch/include/

ENV CAFFE2_INCLUDE_DIRS /pytorch/torch/include/
ENV Caffe2_DIR /pytorch/build/lib.linux-x86_64-2.7/torch/share/cmake/Caffe2/

RUN make

# add info about where to find dependencies
RUN sed -i '1s;^;include_directories(\"/pytorch/torch/include/\")\n;' CMakeLists.txt
RUN sed -i '1s;^;include_directories(\"/opt/intel/compilers_and_libraries_2020.2.254/linux/mkl/include\")\n;' CMakeLists.txt
RUN make ops

# Fetch DensePose data.
RUN bash /densepose/DensePoseData/get_densepose_uv.sh
RUN mv UV_data /densepose/DensePoseData/

# For training, download the DensePose-COCO dataset:
RUN bash /densepose/DensePoseData/get_DensePose_COCO.sh
RUN mv DensePose_COCO /densepose/DensePoseData/

# For evaluation, get the necessary files:
RUN bash /densepose/DensePoseData/get_eval_data.sh
RUN mv eval_data /densepose/DensePoseData/

RUN pip install protobuf
RUN conda install pyyaml==3.12

# Download weights
RUN mkdir -p /weights
RUN cd /weights && wget https://dl.fbaipublicfiles.com/densepose/DensePose_ResNet101_FPN_s1x-e2e.pkl

