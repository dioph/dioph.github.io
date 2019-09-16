---
title: "Instalando o OpenCV no Ubuntu 18.04"
date: 2019-08-02
tags: [python, opencv]
category: casual-code
---
<!--more-->

## Download do OpenCV

```shell
$ wget -O opencv.zip https://github.com/opencv/opencv/archive/4.1.1.zip
$ wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.1.1.zip
$ unzip opencv.zip
$ unzip opencv_contrib.zip
```

## Build

```shell
$ cd opencv-4.1.1
$ mkdir build
$ cd build
```

```shell
$ cmake -D CMAKE_BUILD_TYPE=RELEASE \
      -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D INSTALL_PYTHON_EXAMPLES=ON \
      -D INSTALL_C_EXAMPLES=ON \
      -D WITH_TBB=ON \
      -D WITH_V4L=ON \
      -D OPENCV_ENABLE_NONFREE=ON \
      -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib-4.1.1/modules \
      -D PYTHON_EXECUTABLE=/usr/bin/python \
      -D PYTHON_INCLUDE_DIR=/usr/include/python3.6 \
      -D PYTHON_LIBRARY=/usr/lib/python3.6/config-3.6m-x86_64-linux-gnu/libpython3.6.so \
      -D BUILD_EXAMPLES=ON ..
```

```shell
$ make -j4
$ sudo make install
```
