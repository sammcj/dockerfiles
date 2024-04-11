# Sets the base image for subsequent instructions
ARG ARG_BUILD_FROM="nvcr.io/nvidia/cuda:12.3.2-devel-ubuntu22.04"
FROM $ARG_BUILD_FROM as base

ARG TARGETARCH
ARG GOFLAGS="'-ldflags=-w -s'"
# Sets the time zone within the container
ENV TZ="Australia/Melbourne"
# ENV APT_PROXY=https://apt.local.domain
# ENV APT_PROXY_HTTP=http://apt:3142
# ENV APT_PROXY_HTTPS=https://apt.local.domain
# ENV PIP_INDEX_URL=https://pip.local.domain/index/
# ENV PIP_TRUSTED_HOST=pip.local.domain
# ENV NPM_CONFIG_REGISTRY=https://npm..local.domain/
ENV TORCH_CUDA_ARCH_LIST='8.6;8.6+PTX'
ENV CMAKE_CUDA_ARCHITECTURES='86'
ENV CMAKE_ARGS="-DLLAMA_BLAS_VENDOR=AOCL-mt -DLLAMA_BLAS=ON -DLLAMA_CUBLAS=on -DLLAMA_F16C=on -DLLAMA_CUDA_F16=on -DLLAMA_AVX512=on -DLLAMA_AVX=on -DLLAMA_AVX2=on -DLLAMA_AVX512_VBMI=on -DLLAMA_AVX512_VNNI=on -DLLAMA_FMA=on -DCUDA_USE_TENSOR_CORES=on -DLLAMA_CURL=on -DGIN_MODE=release -DDCMAKE_BUILD_TYPE=Release"
ENV GIN_MODE=release
ENV CUDA_USE_TENSOR_CORES=on

RUN apt-get update && apt-get install -y git build-essential cmake ccache libclblast-dev aria2 libcurl4 libcurlpp-dev libcurlpp0 libcurl4-openssl-dev
RUN /usr/sbin/update-ccache-symlinks
RUN export PATH="/usr/lib/ccache:$PATH" && export GIN_MODE=release

ADD https://dl.google.com/go/go1.22.0.linux-$TARGETARCH.tar.gz /tmp/go1.22.0.tar.gz
RUN mkdir -p /usr/local && tar xz -C /usr/local </tmp/go1.22.0.tar.gz

# Clones the repository into the container
RUN git clone --recurse-submodules -j6 --depth=1 https://github.com/jmorganca/ollama.git /go/src/github.com/jmorganca/ollama
# Sets the working directory
WORKDIR /go/src/github.com/jmorganca/ollama

## Gross hacks because the script from ollama is broken ##
RUN sed -i 's/LLAMA_NATIVE=off/LLAMA_NATIVE=on/g' ./llm/generate/gen_linux.sh
RUN sed -i 's/LLAMA_AVX=off/LLAMA_AVX=on/g' ./llm/generate/gen_linux.sh
RUN sed -i 's/LLAMA_AVX2=off/LLAMA_AVX2=on/g' ./llm/generate/gen_linux.sh
RUN sed -i 's/LLAMA_AVX512=off/LLAMA_AVX512=on/g' ./llm/generate/gen_linux.sh
RUN sed -i 's/LLAMA_AVX512_VNNI=off/LLAMA_AVX512_VNNI=on/g' ./llm/generate/gen_linux.sh
RUN sed -i 's/LLAMA_AVX512_VBMI=off/LLAMA_AVX512_VBMI=on/g' ./llm/generate/gen_linux.sh
RUN sed -i 's/LLAMA_FMA=off/LLAMA_FMA=on/g' ./llm/generate/gen_linux.sh
RUN sed -i 's/LLAMA_F16C=off/LLAMA_F16C=on/g' ./llm/generate/gen_linux.sh
RUN sed -i 's/LLAMA_SCHED_MAX_COPIES=4/LLAMA_SCHED_MAX_COPIES=6/g' ./llm/generate/gen_linux.sh
RUN sed -i 's/LLAMA_CURL=off/LLAMA_CURL=on/g' ./llm/generate/gen_linux.sh
RUN sed -i 's/LLAMA_AVX512=on /LLAMA_AVX512=on -DLLAMA_CURL=on -DLLAMA_AVX512_VNNI=on -DLLAMA_AVX512_VBMI=on -DCUDA_USE_TENSOR_CORES=on -DGIN_MODE=release /g' ./llm/generate/gen_linux.sh
# now in the ./llm/llama.cpp/CMakeLists.txt source:
RUN sed -i 's/\(option(LLAMA_AVX512.*"llama: enable AVX512".*\)\(OFF\)/\1ON/g' ./llm/llama.cpp/CMakeLists.txt
RUN sed -i 's/\(option(LLAMA_AVX512_VBMI.*"llama: enable AVX512-VBMI".*\)\(OFF\)/\1ON/g' ./llm/llama.cpp/CMakeLists.txt
RUN sed -i 's/\(option(LLAMA_AVX512_VNNI.*"llama: enable AVX512-VNNI".*\)\(OFF\)/\1ON/g' ./llm/llama.cpp/CMakeLists.txt
RUN sed -i 's/\(option(LLAMA_NATIVE.*"llama: enable march=native".*\)\(OFF\)/\1ON/g' ./llm/llama.cpp/CMakeLists.txt
RUN sed -i 's/\(option(CUDA_USE_TENSOR_CORES.*"llama: enable CUDA tensor cores".*\)\(OFF\)/\1ON/g' ./llm/llama.cpp/CMakeLists.txt
RUN sed -i 's/\(option(GIN_MODE.*"llama: enable GIN mode".*\)\(OFF\)/\1ON/g' ./llm/llama.cpp/CMakeLists.txt
RUN sed -i 's/\(option(LLAMA_F16C.*"llama: enable F16C".*\)\(OFF\)/\1ON/g' ./llm/llama.cpp/CMakeLists.txt
RUN sed -i 's/\(option(LLAMA_AVX.*"llama: enable AVX".*\)\(OFF\)/\1ON/g' ./llm/llama.cpp/CMakeLists.txt
RUN sed -i 's/\(option(LLAMA_AVX2.*"llama: enable AVX2".*\)\(OFF\)/\1ON/g' ./llm/llama.cpp/CMakeLists.txt
RUN sed -i 's/\(option(LLAMA_SCHED_MAX_COPIES.*"llama: maximum copies for scheduling".*\)\(4\)/\16/g' ./llm/llama.cpp/CMakeLists.txt
RUN sed -i 's/\(option(LLAMA_CURL.*"llama: enable CURL".*\)\(OFF\)/\1ON/g' ./llm/llama.cpp/CMakeLists.txt
RUN sed -i 's/\(option(LLAMA_FMA.*"llama: enable FMA".*\)\(OFF\)/\1ON/g' ./llm/llama.cpp/CMakeLists.txt

# Edit the ./llm/generate/gen_common.sh to add a return right the line 'git_module_setup() {'
RUN sed -i 's/\(git_module_setup() {\)/\1\n  return/g' ./llm/generate/gen_common.sh

## END HACKS ##

# Sets the environment variables for building the binary
ENV GOARCH=$TARGETARCH
ENV GOFLAGS=$GOFLAGS

# Get AMD AOCL (see https://github.com/ollama/ollama/issues/2205#issuecomment-1958505548)
RUN aria2c https://download.amd.com/developer/eula/aocl/aocl-4-2/aocl-linux-aocc-4.2.0_1_amd64.deb && \
  dpkg -i aocl-linux-aocc-4.2.0_1_amd64.deb && \
  rm -f aocl-linux-aocc-4.2.0_1_amd64.deb

RUN export GIN_MODE=release && \
  CMAKE_ARGS=${CMAKE_ARGS} /usr/local/go/bin/go generate ./... && \
  sed -i 's/\(git_module_setup() {\)/\1\n  return/g' ./llm/generate/gen_common.sh && \
  CMAKE_ARGS=${CMAKE_ARGS} /usr/local/go/bin/go build -v .

FROM nvidia/cuda:12.3.2-runtime-ubuntu22.04 as app
# ENV APT_PROXY=https://apt.local.domain
# ENV APT_PROXY_HTTP=http://apt:3142
# ENV APT_PROXY_HTTPS=https://apt.local.domain
# ENV PIP_INDEX_URL=https://pip.local.domain/index/
# ENV PIP_TRUSTED_HOST=pip.local.domain
# ENV NPM_CONFIG_REGISTRY=https://npm.local.domain/
ENV TORCH_CUDA_ARCH_LIST='8.6;8.6+PTX'
ENV CMAKE_CUDA_ARCHITECTURES='86'
ENV CUDA_USE_TENSOR_CORES=on
ENV GIN_MODE=release

# Setting this argument prevents interactive prompts during the build process
ARG DEBIAN_FRONTEND=noninteractive
# Updates the image and installs necessary packages
RUN apt-get update --fix-missing \
  && apt-get install -y curl wget aria2 jq ca-certificates \
  # Cleans up unnecessary packages to reduce image size
  && apt-get autoremove -y \
  && apt-get clean -y
# Copies the binary from the base image into the app image
COPY --from=base /go/src/github.com/jmorganca/ollama/ollama /bin/ollama

# Setting ARGs and ENVs for user creation and workspace setup
ARG ARG_USERNAME="llm"
ARG ARG_USER_UID=1337
ARG ARG_USER_GID=$ARG_USER_UID
ARG ARG_WORKSPACE_ROOT="/home/llm"

ENV OLLAMA_HOST 0.0.0.0
ENV USERNAME $ARG_USERNAME
ENV USER_UID $ARG_USER_UID
ENV USER_GID $ARG_USER_GID
ENV WORKSPACE_ROOT $ARG_WORKSPACE_ROOT

# Creates a non-root user with sudo privileges
# check if user exists and if not, create user
RUN \
  groupadd --gid $USER_GID $USERNAME && \
  adduser --uid $USER_UID --gid $USER_GID $USERNAME

WORKDIR $WORKSPACE_ROOT

# Changes ownership of the workspace to the non-root user
RUN chown -R $USERNAME:$USERNAME $WORKSPACE_ROOT

VOLUME [ "$WORKSPACE_ROOT/.ollama" ]

USER $USERNAME

CMD ["/bin/ollama", "serve"]
