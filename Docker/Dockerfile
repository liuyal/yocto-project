FROM ubuntu:24.04

# Prevent interactive prompts during package installation
ENV DEBIAN_FRONTEND=noninteractive

# Install host dependencies
RUN apt-get update && apt-get install -y \
    gawk wget git diffstat unzip texinfo gcc build-essential \
    chrpath socat cpio python3 python3-pip python3-pexpect \
    xz-utils debianutils iputils-ping python3-git python3-jinja2 \
    libegl1 libsdl1.2-dev pylint xterm lz4 \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Install Zscaler Root CA
COPY ZscalerRootCA.crt /usr/local/share/ca-certificates/ZscalerRootCA.crt
RUN update-ca-certificates

# Yocto/bitbake must not run as root — create a build user
RUN useradd -m -s /bin/bash yocto
USER yocto
WORKDIR /home/yocto

# Clone Poky and check out the scarthgap release
RUN git clone https://git.yoctoproject.org/poky && \
    cd poky && \
    git checkout scarthgap

# Set MACHINE in local.conf after the build environment is initialised
RUN /bin/bash -c "source poky/oe-init-build-env build && \
    echo 'MACHINE = \"qemux86-64\"' >> conf/local.conf"

# Allow unprivileged user namespaces (required by some build tasks)
# Note: the host must also run: sudo sysctl -w kernel.apparmor_restrict_unprivileged_userns=0
# The container flag is set here for documentation; it requires privileged mode at runtime.

# Default working directory is the pre-initialised build directory
WORKDIR /home/yocto/build

# Entrypoint sources the build environment so bitbake is on PATH
ENTRYPOINT ["/bin/bash", "-c", "source /home/yocto/poky/oe-init-build-env /home/yocto/build && exec \"$@\"", "--"]

CMD ["bash"]
