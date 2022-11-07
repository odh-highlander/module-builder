FROM quay.io/centos/centos:stream9

ARG ARCH=x86_64

# Install Dependencies
USER root
RUN dnf install -y dnf-utils && \
    dnf config-manager --enable crb && \
    dnf -y install epel-release epel-next-release && \
    dnf -y update && \
    INSTALL_PKGS="ant \
        autoconf \
        automake \
        bzip2 \
        cmake \
        file \
        flex \
        gcc \
        gcc-c++ \
        git \
        help2man \
        http-parser \
        iproute \
        libtool  \
        lua \
        lua-devel \
        lua-filesystem \
        lua-posix \
        make \
        nano \
        ncurses \
        openssl-devel \
        pam-devel \
        patch \
        perl \
        pip \
        python \
        python-unversioned-command \
        rdma-core-devel \
        rust \
        sudo \
        tar \
        tcl \
        tmux \
        unzip \
        which \
        xz \
        zlib-devel" && \
    dnf -y install  --setopt=tsflags=nodocs $INSTALL_PKGS && \
    dnf -y clean all --enablerepo='*' && \
    rm -rf /var/cache/dnf

# Build LMOD
ENV LMOD_VER 8.7.13

RUN mkdir -p /build/Lmod-${LMOD_VER} && \
    cd /build/Lmod-${LMOD_VER} && \
    curl -L http://github.com/TACC/Lmod/archive/${LMOD_VER}.tar.gz -o /build/Lmod-${LMOD_VER}.tar.gz && \
    tar xvf /build/Lmod-${LMOD_VER}.tar.gz -C /build && \
    ./configure --prefix=/opt/apps --with-fastTCLInterp=no --with-mode=755 --with-useDotConfigDirOnly=yes && \
    make install && \
    chmod -R 755 /opt/apps/lmod && \
    rm -rf /build && \
    ln -s /opt/apps/lmod/lmod/init/profile /etc/profile.d/z00_lmod.sh

# Add easybuild user and let it run the show
RUN adduser easybuild && \
    echo "easybuild:easybuild" | chpasswd && usermod -aG wheel easybuild && \
    echo "easybuild ALL=(ALL:ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/easybuild

ENV APP_ROOT=/opt/app-root \
    HOME=/opt/app-root/src \
    PLATFORM="el9"

COPY . /opt/app-root/src/
COPY fix-permissions /usr/bin/fix-permissions

COPY .bash_profile /opt/app-root/src/.bash_profile

RUN pip install --no-cache-dir --disable-pip-version-check wheel python-graph-core python-graph-dot && \
    chown -R easybuild:0 ${APP_ROOT} && \
    fix-permissions ${APP_ROOT} -P

ENV PATH="/opt/apps/easybuild/bin:${PATH}"
ENV MODULEPATH="/opt/apps/easybuild/modules:${MODULEPATH}"
ENV PYTHONPATH="/opt/apps/easybuild/lib/python3.9/site-packages:${PYTHONPATH}"
ENV XDG_CONFIG_DIRS="/opt/apps/easybuild"

# Prevent activate script to be run at each command, scraping the PATH
ENV PROMPT_COMMAND=""

USER easybuild

WORKDIR /opt/app-root/src

# Set the default CMD to print the usage of the language image.
CMD /opt/app-root/src/usage
