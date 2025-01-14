#*******************************************************************************
#    (c) 2020 Zondax GmbH
# 
#   Licensed under the Apache License, Version 2.0 (the "License");
#   you may not use this file except in compliance with the License.
#   You may obtain a copy of the License at
# 
#       http://www.apache.org/licenses/LICENSE-2.0
# 
#   Unless required by applicable law or agreed to in writing, software
#   distributed under the License is distributed on an "AS IS" BASIS,
#   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
#   See the License for the specific language governing permissions and
#   limitations under the License.
#*******************************************************************************
#Download base ubuntu image
FROM ubuntu:20.04
ENV DEBIAN_FRONTEND noninteractive
ENV LANG C.UTF-8

RUN apt-get update && \
    apt-get -y install build-essential git wget sudo udev zip \
                       curl cmake software-properties-common

RUN apt-get update && \
    apt-get -y install libudev-dev libusb-1.0-0-dev 

# Install Python
RUN apt-get update && apt-get -y install python3 python3-pip
RUN ln -s /usr/bin/python3 /usr/bin/python
RUN pip3 install -U setuptools ledgerblue pillow flask

# Speculos dependencies
RUN apt-get update && apt-get -y install qemu-user-static python3-pyqt5 python3-construct \
                                 python3-mnemonic python3-pyelftools gcc-arm-linux-gnueabihf \
                                 libc6-dev-armhf-cross gdb-multiarch libvncserver-dev \
                                 mesa-utils libgl1-mesa-glx qemu-user-static 

RUN pip3 install -U jsonschema pyelftools mnemonic construct flask flask-restful pillow requests

# Create zondax user
RUN adduser --disabled-password --gecos "" -u 1000 zondax
RUN echo "zondax ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers
WORKDIR /home/zondax

####################################
####################################
USER zondax

# Speculos
RUN git clone -b master https://github.com/Zondax/speculos
RUN cd /home/zondax/speculos && git checkout d7d74564f083edccb70e4222f1a86fb27c75042c
RUN mkdir -p /home/zondax/speculos/build
RUN cd /home/zondax/speculos && cmake -Bbuild -H. -DWITH_VNC=0
RUN make -C /home/zondax/speculos/build/

# Patch proxy to connect to all interfaces
RUN sed -i "s/HOST = '127.0.0.1'/HOST = '0.0.0.0'/g" speculos/tools/ledger-live-http-proxy.py

# Open TCP ports
# gdb
EXPOSE 1234/tcp
EXPOSE 1234/udp
# device keyboard
EXPOSE 1235/tcp
EXPOSE 1235/udp
# APDU RAW
EXPOSE 9999/tcp
EXPOSE 9999/udp
# HTTP APDU PROXY
EXPOSE 9998/tcp
EXPOSE 9998/udp
# RFB
EXPOSE 8001/tcp
EXPOSE 8001/udp
# API Speculos
EXPOSE 5000/tcp
EXPOSE 5000/udp

# ENV
ADD entrypoint.sh /home/zondax/entrypoint.sh

# START SCRIPT
ENTRYPOINT ["/home/zondax/entrypoint.sh"]
