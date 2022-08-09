FROM ubuntu:20.04
# LABEL about the custom image
LABEL maintainer="b.r.salehe@reading.ac.uk"
LABEL version="0.1"
LABEL description="This is custom Docker Image for \
the MultiFOLD which is a component method of IntFOLD \
server for predicting protein complexes"

# CMD ["bash"]
ENV NVARCH=x86_64
ENV NVIDIA_REQUIRE_CUDA=cuda>=11.2 brand=tesla,driver>=418,driver<419
ENV NV_CUDA_CUDART_VERSION=11.2.72-1
ENV NV_CUDA_COMPAT_PACKAGE=cuda-compat-11-2
ARG TARGETARCH
#
RUN |1 TARGETARCH=amd64 RUN apt-get update  \
		&& apt-get install -y --no-install-recommends gnupg2 curl ca-certificates  \
		&& curl -fsSL https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/${NVARCH}/3bf863cc.pub | apt-key add -  \
		&& echo "deb https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/${NVARCH} /" > /etc/apt/sources.list.d/cuda.list  \
		&& apt-get purge --autoremove -y curl  \
		&& rm -rf /var/lib/apt/lists/* # buildkit
ENV CUDA_VERSION=11.2.0
RUN |1 TARGETARCH=amd64 RUN apt-get update  \
		&& apt-get install -y --no-install-recommends cuda-cudart-11-2=${NV_CUDA_CUDART_VERSION} ${NV_CUDA_COMPAT_PACKAGE}  \
		&& ln -s cuda-11.2 /usr/local/cuda  \
		&& rm -rf /var/lib/apt/lists/* # buildkit
RUN |1 TARGETARCH=amd64 RUN echo "/usr/local/nvidia/lib" >> /etc/ld.so.conf.d/nvidia.conf  \
		&& echo "/usr/local/nvidia/lib64" >> /etc/ld.so.conf.d/nvidia.conf # buildkit
ENV PATH=/usr/local/nvidia/bin:/usr/local/cuda/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
ENV LD_LIBRARY_PATH=/usr/local/nvidia/lib:/usr/local/nvidia/lib64
COPY NGC-DL-CONTAINER-LICENSE / # buildkit
		NGC-DL-CONTAINER-LICENSE

ENV NVIDIA_VISIBLE_DEVICES=all
ENV NVIDIA_DRIVER_CAPABILITIES=compute,utility

## Install colabfold
RUN apt update && \
		apt upgrade && \
		apt -y install curl git wget && \
		wget https://raw.githubusercontent.com/YoshitakaMo/localcolabfold/main/install_colabbatch_linux.sh && \
		bash install_colabbatch_linux.sh && \
		rm ./install_colabbatch_linux.sh && \
		echo "PATH='/colabfold_batch/bin:$PATH'" >> ~/.bashrc && \
		source ~/.bashrc && \
    apt clean && \
		mkdir -p /colabfold_batch/input_files /colabfold_batch/ouput_files

## DockQ
## /bin/bash
RUN git clone https://github.com/bjornwallner/DockQ && \
		cd DockQ/ && \
		apt-get install build-essential && \
		make && \
		mkdir -p DockQ/input_model DockQ/native_model && \

VOLUME ["/colabfold_batch/bin/", "/colabfold_batch/input_files", "/colabfold_batch/output_files", "/DockQ/input_model", "/DockQ/", "/DockQ/input_model/", "/DockQ/native_model/"]

WORKDIR /colabfold_batch/bin/
CMD ["colabfold_batch"]
## /bin/bash
