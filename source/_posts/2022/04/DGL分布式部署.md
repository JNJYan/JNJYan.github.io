---
title: DGL分布式部署
date: 2022-04-02 10:40:49
tags:
categories:
---

DGL安装脚本
```shell
#/bin/bash

# deps
sudo apt-get update && sudo apt-get install -y build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev wget libbz2-dev

# install python
wget https://www.python.org/ftp/python/3.8.13/Python-3.8.13.tar.xz && tar -xf Python-3.8.13.tar.xz && cd Python-3.8.13 && ./configure --prefix=/usr/local/python3 --enable-optimizations && sudo make -j 72 && sudo make install

sudo mv /usr/bin/python3 /usr/bin/python3.bak && sudo ln -s /usr/local/python3/bin/python3 /usr/bin/python3

# install dgl
cd ~/ && git clone https://github.com/dmlc/dgl.git && cd dgl && git checkout v0.7.0 && git submodule update --init --recursive && mkdir build && cd build && cmake .. && make -j 72 && cd ../python && sudo python3 setup.py install

# install torch
pip3 install torch==1.10.0
```

ParMETIS安装脚本
```shell
#/bin/bash

# install GKlib
cd ~ && git clone https://github.com/KarypisLab/GKlib.git && cd GKlib && rm -rf build && make config prefix=/usr/local && sudo make install

# install METIS [deps on GKlib]
cd ~ && git clone https://github.com/KarypisLab/METIS.git && cd METIS && git submodule update --init --recursive && make config shared=1 cc=gcc prefix=/usr/local i64=1 && sudo make install

# install mpicc
cd ~ && wget https://www.mpich.org/static/downloads/4.0/mpich-4.0.tar.gz && tar -xf mpich-4.0.tar.gz && cd mpich-4.0 && ./configure -prefix=/usr/local --disable-fortran && make -j 72 && sudo make install

# install ParMETIS [deps on mpicc and METIS]
cd ~ && git clone --branch dgl https://github.com/KarypisLab/ParMETIS.git && cd ParMETIS && rm -rf build && make config cc=mpicc prefix=/usr/local && sudo make install
```