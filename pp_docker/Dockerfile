FROM ubuntu:22.04 AS BUILD

# docker image to serve as base for a PP singularity image
# can be done in singularity .def file but I'm more familiar with docker and docker's image cache
# is very useful as opposed to going through all the steps in singularity build every time

# update system & add package requirements for building python
RUN apt update && \
    DEBIAN_FRONTEND=noninteractive apt dist-upgrade -y && \
    DEBIAN_FRONTEND=noninteractive apt install -y make build-essential libssl-dev zlib1g-dev \
        libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncursesw5-dev xz-utils tk-dev \
        libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev && \
    apt autoclean -y && \
    apt autoremove -y && \
    rm -rf /var/lib/apt/lists/*

# build python 3.8.13, install to /python
# header files and python-config also required to build wxpython later on
RUN mkdir /tmp/pythonbuild && \
    cd /tmp/pythonbuild && \
    wget https://www.python.org/ftp/python/3.8.13/Python-3.8.13.tgz -O - | tar -xz && \
    cd Python-3.8.13/ && \
    ./configure --enable-optimizations --with-lto --prefix=/python && \
    make && \
    mkdir /python && \
#    make DESTDIR=/python install && \
    make install && \
    cd /python/bin && \
    rm -rf /tmp/pythonbuild && \
    ln -s python3.8 python && \
    ln -s python3.8-config python-config && \
    ln -s pip3.8 pip

# requirements for building PP (mostly needed for pocketsphinx and wxpython)
# can be simplified once wxpython wheels are available
RUN apt update && \
    DEBIAN_FRONTEND=noninteractive apt install -y ubuntu-restricted-extras swig libpulse-dev libasound2-dev \
        libtiff-dev libjpeg-dev libgtk-3-dev freeglut3 freeglut3-dev libwebkit2gtk-4.0-dev libsdl2-dev \
        libsecret-1-dev libnotify-dev libgstreamer-plugins-base1.0-dev gettext make libglib2.0-dev && \
    rm -rf /var/lib/apt/lists/*

# create new virtualenv (could just install into /python really but this feels cleaner)
RUN export PATH="/python/bin:$PATH" && \
    export PIP_CACHE_DIR="/pipcache" && \
    mkdir /pipcache && \
    python -m venv /venv && \
    . /venv/bin/activate && \
    python -m pip install -U pip setuptools wheel && \
    pip install psychopy wxpython && \
    rm -rf /pipcache
    
# ------------------------
# 2nd stage for running PP
FROM ubuntu:22.04

# Python 3.8 installation folder (maybe no need?) + virtualenv
COPY --from=BUILD /venv /venv
COPY --from=BUILD /python /python

# give ownership of virtualenv to singularity user; for now, assume singularity user is 1000:1000
RUN chown -R 1000:1000 /venv

# add packages required for running PP (all needed? others needed?)
RUN apt update && \
    DEBIAN_FRONTEND=noninteractive apt install -y ubuntu-restricted-extras libgstreamer-gl1.0-0 python3-gst-1.0 \
        freeglut3 libx11-6 libgtk-3-0 libsm6 libnotify4 libsdl2-2.0-0 libsecret-1-0 pulseaudio libportaudio2 \
        libglib2.0-0 libtool-bin libpulse0 libgl1-mesa-dri libglu1-mesa libjpeg8 libtiff5 libsndfile1 \
        libwebkit2gtk-4.0-37 libcanberra-gtk3-module libcanberra-gtk3-0 vlc ffmpeg && \
    rm -rf /var/lib/apt/lists/*
    
# for network testing
#RUN apt update && \
#    DEBIAN_FRONTEND=noninteractive apt install -y iproute2 iputils-ping bind9-dnsutils && \
#    rm -rf /var/lib/apt/lists/*
    
# just a reminder how to set up sec limits for PTB group, don't uncomment in this form, NOT FOR USE
#RUN addgroup --gid 1000 $USER && \
#    adduser --disabled-password --gecos "" --uid user
# groupadd --force psychopy && \
#    usermod -a -G psychopy 1000 && \
#    echo "@psychopy - nice -20" > /etc/security/limits.d/99-psychopylimits.conf && \
#    echo "@psychopy - rtprio 50" >> /etc/security/limits.d/99-psychopylimits.conf && \
#    echo "@psychopy - memlock unlimited" >> /etc/security/limits.d/99-psychopylimits.conf
    
#CMD /bin/bash

