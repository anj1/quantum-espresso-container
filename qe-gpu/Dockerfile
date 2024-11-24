# Start from NVIDIA CUDA base image
FROM nvcr.io/nvidia/nvhpc:24.11-devel-cuda_multi-ubuntu22.04

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive
ENV QE_VERSION="7.4"

# Install required packages
RUN apt-get update && apt-get install -y \
    build-essential \
    wget \
    git \
    python3 \
    python3-pip \
    gfortran \
    openmpi-bin \
    libopenmpi-dev \
    libfftw3-dev \
    libfftw3-mpi-dev \
    libscalapack-openmpi-dev \
    pkg-config \
    make \
    cmake \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Create working directory
WORKDIR /build

# Download and extract Quantum ESPRESSO
RUN wget https://github.com/QEF/q-e/archive/qe-${QE_VERSION}.tar.gz \
    && tar -xzf qe-${QE_VERSION}.tar.gz \
    && rm qe-${QE_VERSION}.tar.gz

# Configure and build Quantum ESPRESSO with GPU support using NVHPC
WORKDIR /build/q-e-qe-${QE_VERSION}

# Configure with NVHPC compiler
RUN ./configure \
    --enable-openmp \
    --enable-parallel \
    --with-scalapack=yes \
    --with-cuda=yes \
    --with-cuda-cc=86 \
    --with-cuda-runtime=12.6 \
    --with-cuda-mpi=yes \
    && make all -j$(nproc) \
    && make install

# # Set working directory for running calculations
WORKDIR /work

# # Add environment setup script
# RUN echo '#!/bin/bash\n\
# export CUDA_VISIBLE_DEVICES=${GPU_ID:-0}\n\
# export OMP_NUM_THREADS=${OMP_NUM_THREADS:-1}\n\
# export LD_LIBRARY_PATH="/opt/nvidia/hpc_sdk/Linux_x86_64/24.1/compilers/lib:${LD_LIBRARY_PATH}"\n\
# exec "$@"' > /entrypoint.sh \
#     && chmod +x /entrypoint.sh

# ENTRYPOINT ["/entrypoint.sh"]
# CMD ["pw.x"]