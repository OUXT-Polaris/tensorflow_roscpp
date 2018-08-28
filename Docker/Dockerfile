FROM nvidia/cuda:9.0-cudnn7-devel-ubuntu16.04

LABEL maintainer="Masaya Kataoka(ms.kataoka@gmail.com)"

RUN apt-get update
RUN apt-get install -y \
    bash-completion \
    python-numpy python-dev python-pip python-wheel \
    openjdk-8-jdk \
    wget \
    zlib1g-dev
RUN wget https://github.com/bazelbuild/bazel/releases/download/0.16.1/bazel_0.16.1-linux-x86_64.deb
RUN dpkg -i bazel_0.16.1-linux-x86_64.deb

RUN apt-get install -y git
RUN git clone https://github.com/NVIDIA/nccl.git
RUN cd nccl && make install -j4
RUN git clone https://github.com/tensorflow/tensorflow.git
ENV CI_BUILD_PYTHON=python \
    LD_LIBRARY_PATH=/usr/local/cuda/extras/CUPTI/lib64:${LD_LIBRARY_PATH} \
    CUDNN_INSTALL_PATH=/usr/lib/x86_64-linux-gnu \
    NCCL_INSTALL_PATH=/usr/local \
    PYTHON_BIN_PATH=/usr/bin/python \
    PYTHON_LIB_PATH=/usr/local/lib/python2.7/dist-packages \
    TF_NEED_CUDA=1 \
    TF_CUDA_VERSION=9.0 \
    TF_CUDA_COMPUTE_CAPABILITIES=3.0,3.5,5.2,6.0,6.1,7.0 \
    TF_CUDNN_VERSION=7
RUN cp /usr/local/lib/libnccl.so /usr/local/lib/libnccl.so.2
RUN cp /usr/local/cuda-9.0/targets/x86_64-linux/lib/stubs/libcuda.so /usr/local/cuda-9.0/targets/x86_64-linux/lib/stubs/libcuda.so.1
ENV LD_LIBRARY_PATH=/usr/local/cuda-9.0/targets/x86_64-linux/lib/stubs:${LD_LIBRARY_PATH}
RUN cd tensorflow && ./configure
RUN cd tensorflow && bazel build -c opt --config=cuda --copt=-march=native --config monolithic tensorflow:libtensorflow_cc.so
RUN mkdir -p /build/x64/lib
RUN mkdir -p /build/x64/include
RUN cp /tensorflow/bazel-bin/tensorflow/libtensorflow_cc.so /build/x64/lib
RUN find /tensorflow/tensorflow/core -follow -type f -name "*.h" -exec cp --parents {} /build/x64/includeR \;
RUN find /tensorflow/tensorflow/cc   -follow -type f -name "*.h" -exec cp --parents {} /build/x64/include \;
RUN find /tensorflow/tensorflow/c    -follow -type f -name "*.h" -exec cp --parents {} /build/x64/include \;
RUN find /tensorflow/third_party/eigen3 -follow -type f -exec cp --parents {} /build/x64/include \;
RUN cd /tensorflow/bazel-genfiles && find tensorflow -follow -type f -name "*.h" -exec cp --parents {} /build/x64/include \;
RUN cd /tensorflow/bazel-tensorflow/external/protobuf_archive/src && find google -follow -type f -name "*.h" -exec cp --parents {} /build/x64/include \;
RUN cd /tensorflow/bazel-tensorflow/external/eigen_archive && find Eigen -follow -type f -exec cp --parents {} /build/x64/include \;
RUN cd /tensorflow/bazel-tensorflow/external/eigen_archive && find unsupported -follow -type f -exec cp --parents {} /build/x64/include \;