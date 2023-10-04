---
title:  "BeagleBone AI chip(AM57x) SDK"
categories: 
  - Embedded
tags:
  - Texas Instruments
  - Beaglebone
  - TI SDK
---

- [1. Install SDK](#1-install-sdk)
  - [1.1. Download SDK](#11-download-sdk)
  - [1.2. Install SDK](#12-install-sdk)
- [2.  Set Environment, Path by Toolchain](#2--set-environment-path-by-toolchain)
- [3. SDK NFS mount to BeagleBoneAI](#3-sdk-nfs-mount-to-beagleboneai)

**SDK 버전: 6.3.0.106**

## 1. Install SDK

### 1.1. Download SDK

이곳에서 AM57x 의 SDK 를 다운로드 받을 수 있다. 
https://www.ti.com/tool/PROCESSOR-SDK-AM57X#downloads

[Linux Processor SDK for AM57x](https://software-dl.ti.com/processor-sdk-linux/esd/AM57X/latest/index_FDS.html) 가 지원하는 대표적인 SoC, EVM 은 아래와 같다.

- SoC : [AM5729](https://www.ti.com/product/AM5729), AM5728 ...
- EVM : [BeagleBone AI](https://www.ti.com/tool/BEAGLE-3P-BBONE-AI), AM572x EVM ...

다운로드 할 파일은 아래와 같다. 

- SDK Essential 설치파일 
  - `ti-processor-sdk-linux-am57xx-evm-06.03.00.106-Linux-x86-Install.bin` 
- ARM Toolchain
  - `gcc-arm-8.3-2019.03-x86_64-arm-linux-gnueabihf.tar.xz`

### 1.2. Install SDK

```
chmod +x ti-processor-sdk-linux-am57xx-evm-06.03.00.106-Linux-x86-Install.bin
./ti-processor-sdk-linux-am57xx-evm-06.03.00.106-Linux-x86-Install.bin
```

설치 마법사를 따라서 설치경로를 지정한다. (설치경로를 이하 `${SDK_DIR]` 로 칭한다.)

> 설치 도중 `${SDK_DIR]/sdk-install.sh` 를 자동으로 실행한 다음, `sdk-install.sh` 파일은 삭제를 해준다. 

SDK의 경로 설정

```
./setup.sh
```

설치 완료.

## 2.  Set Environment, Path by Toolchain

툴체인 활성화

```
source ${PSDK_LINUX}/linux-devkit/environment-setup
```

## 3. SDK NFS mount to BeagleBoneAI

BeagleBone AI Board 와 Host 간 NFS 연결

BeagleBone 이 client 가 되어서 Host 의 SDK targetNFS 를 mount

```
# On BeagleBone
$ mkdir ~/targetNFS
$ sudo mount -t nfs 192.168.1.81:$HOME/workspace/BeagleBoneAI/ti-processor-sdk-linux-am57xx-evm-06.03.00.106/targetNFS ~/targetNFS
```