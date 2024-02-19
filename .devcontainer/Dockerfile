
FROM python:alpine3.19

ARG USER=user
ARG WORKDIR=workspaces
ARG HOST_ARCH=x86_64
ARG TARGET=stm32
ARG TOOLCHAIN=arm-zephyr-eabi
ARG ZEPHYR_VERSION=3.3.0
ARG ZSDK_VERSION=0.16.1

# Install base packages
RUN apk add --update --no-cache \
	bash sudo \
	build-base pkgconfig cmake git \
	cairo-dev cairo cairo-tools \
	gobject-introspection-dev \
	rust cargo

# Initialise system locale
#RUN locale-gen en_US.UTF-8
ENV LANG=en_US.UTF-8
ENV LANGUAGE=en_US:en
ENV LC_ALL=en_US.UTF-8

# Install Python dependencies
RUN python3 -m pip install -U --no-cache-dir pip && \
	pip3 install -U --no-cache-dir wheel setuptools && \
	pip3 install --no-cache-dir pygobject && \
	pip3 install --no-cache-dir \
		-r https://raw.githubusercontent.com/zephyrproject-rtos/zephyr/main/scripts/requirements.txt \
		-r https://raw.githubusercontent.com/zephyrproject-rtos/mcuboot/main/scripts/requirements.txt \
		GitPython imgtool junitparser numpy protobuf PyGithub \
		pylint sh statistics west && \
	pip3 check

# Create 'user' account
# https://wiki.alpinelinux.org/wiki/Setting_up_a_new_user
RUN adduser -D $USER \
    && echo "$USER ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/$USER \
    && chmod 0440 /etc/sudoers.d/$USER


# Install Zephyr SDK (minimal + toolchain + host tools)
RUN mkdir -p /opt/toolchains && \
	cd /opt/toolchains && \
	wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v${ZSDK_VERSION}/zephyr-sdk-${ZSDK_VERSION}_linux-${HOST_ARCH}_minimal.tar.xz && \
	tar xf zephyr-sdk-${ZSDK_VERSION}_linux-${HOST_ARCH}_minimal.tar.xz && \
	rm zephyr-sdk-${ZSDK_VERSION}_linux-${HOST_ARCH}_minimal.tar.xz && \
	# toolchain
	cd zephyr-sdk-${ZSDK_VERSION} && \
	wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.16.1/toolchain_linux-${HOST_ARCH}_${TOOLCHAIN}.tar.xz && \
    tar xvf toolchain_linux-${HOST_ARCH}_${TOOLCHAIN}.tar.xz && \
    rm toolchain_linux-${HOST_ARCH}_${TOOLCHAIN}.tar.xz && \
    # host tools
    wget https://github.com/zephyrproject-rtos/sdk-ng/releases/download/v0.16.1/hosttools_linux-${HOST_ARCH}.tar.xz && \
    tar xvf hosttools_linux-${HOST_ARCH}.tar.xz && \
    rm hosttools_linux-${HOST_ARCH}.tar.xz
    # run it
    #./zephyr-sdk-${HOSTTYPE}-hosttools-standalone-0.9.sh

# Run the Zephyr SDK setup script as 'user' in order to ensure that the
# `Zephyr-sdk` CMake package is located in the package registry under the
# user's home directory.
USER ${USER}

#RUN sudo -E -- /bin/sh -c \
#	'/opt/toolchains/zephyr-sdk-${ZSDK_VERSION}/setup.sh -c && \
#	chown -R $USER:$USER /home/$USER/.cmake'
RUN sudo /opt/toolchains/zephyr-sdk-${ZSDK_VERSION}/setup.sh -c && \
	sudo chown -R ${USER}:${USER} /home/${USER}/.cmake

WORKDIR /${WORKDIR}

RUN \
	west init -m https://github.com/cooked/vscode-zephyr-template --mf .manifests/west-${TARGET}.yml && \
    west update && \
    python3 -m pip install -r zephyr/scripts/requirements.txt

# Set the locale
ENV PKG_CONFIG_PATH=/usr/lib/i386-linux-gnu/pkgconfig
ENV OVMF_FD_PATH=/usr/share/ovmf/OVMF.fd
ENV LC_ALL=C.UTF-8
ENV LANG=C.UTF-8
ENV XDG_CACHE_HOME=/${WORKDIR}/.cache
ENV ZEPHYR_TOOLCHAIN_VARIANT=zephyr
ENV ZEPHYR_SDK_INSTALL_DIR=/${WORKDIR}/zephyr-sdk
ENV ZEPHYR_BASE=/${WORKDIR}/zephyr
ENV PATH="${ZEPHYR_BASE}/scripts:${PATH}"