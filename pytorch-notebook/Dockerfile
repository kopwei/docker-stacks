# Copyright (c) Jupyter Development Team.
# Distributed under the terms of the Modified BSD License.
ARG BASE_CONTAINER=kopkop/jupyter-cuda-notebook

FROM $BASE_CONTAINER as dev-base
USER root
RUN --mount=type=cache,id=apt-dev,target=/var/cache/apt \
    apt-get update && apt-get install -y --no-install-recommends \
        build-essential \
        ca-certificates \
        ccache \
        cmake \
        curl \
        git \
        libjpeg-dev \
        libpng-dev && \
    rm -rf /var/lib/apt/lists/*
RUN /usr/sbin/update-ccache-symlinks
RUN mkdir /opt/ccache && ccache --set-config=cache_dir=/opt/ccache
ENV PATH /opt/conda/bin:$PATH

FROM dev-base as conda-installs
USER $NB_UID
ARG PYTHON_VERSION=3.8
ARG CUDA_VERSION=11.0
ARG CUDA_CHANNEL=defaults
#ARG INSTALL_CHANNEL=pytorch-nightly
ARG INSTALL_CHANNEL=pytorch
ARG PYTORCH_VERSION=1.7.1
ENV CONDA_OVERRIDE_CUDA=${CUDA_VERSION}
RUN /opt/conda/bin/conda install -c "${INSTALL_CHANNEL}" -c "${CUDA_CHANNEL}" -y python=${PYTHON_VERSION}  "cudatoolkit=${CUDA_VERSION}" && \
    /opt/conda/bin/conda clean -ya && \
    /opt/conda/bin/pip install torch===1.7.1+cu110 torchvision===0.8.2+cu110 torchaudio===0.7.2 torchtext==0.8.1 -f https://download.pytorch.org/whl/torch_stable.html

RUN /opt/conda/bin/pip install torchelastic jupyter-tensorboard-proxy==0.1.1
RUN fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"

FROM ${BASE_CONTAINER} as final
LABEL maintainer="Jupyter Project <jupyter@googlegroups.com>"
ARG PYTORCH_VERSION=1.7.1
LABEL com.nvidia.volumes.needed="nvidia_driver"
USER root
RUN --mount=type=cache,id=apt-final,target=/var/cache/apt \
    apt-get update && apt-get install -y --no-install-recommends \
        ca-certificates \
        libjpeg-dev \
        libpng-dev && \
    rm -rf /var/lib/apt/lists/*
COPY --from=conda-installs /opt/conda /opt/conda
USER $NB_UID
RUN fix-permissions "${CONDA_DIR}" && \
    fix-permissions "/home/${NB_USER}"
ENV PATH /opt/conda/bin:$PATH
ENV NVIDIA_VISIBLE_DEVICES all
ENV NVIDIA_DRIVER_CAPABILITIES compute,utility
ENV LD_LIBRARY_PATH /usr/local/nvidia/lib:/usr/local/nvidia/lib64
ENV PYTORCH_VERSION ${PYTORCH_VERSION}
