---
title:  "BeagleBone AI 보드 시작하기"
categories: 
  - Embedded
tags:
  - Texas Instruments
  - Beaglebone
  - Deep learning
---

BeagleBone AI 보드 시작하기 [참고](https://beagleboard.org/getting-started)

- [1. BeagleBone AI Features](#1-beaglebone-ai-features)
- [2. Board Bring-up](#2-board-bring-up)
- [3. Cloud9 IDE](#3-cloud9-ide)
- [4. Update System](#4-update-system)
  - [4.1. Download Images](#41-download-images)
  - [4.2. Update board with latest software](#42-update-board-with-latest-software)
    - [4.2.1. Flash Image to micro SD](#421-flash-image-to-micro-sd)
    - [4.2.2. Flash eMMC](#422-flash-emmc)
      - [4.2.2.1. Change boot environment](#4221-change-boot-environment)
- [5. Get Connected to the Internet](#5-get-connected-to-the-internet)
  - [5.1. Ethernet](#51-ethernet)
  - [5.2. WiFi](#52-wifi)
- [6. Update the boot-up scripts and Linux kernel](#6-update-the-boot-up-scripts-and-linux-kernel)
- [7. Update distribution compenents (packages)](#7-update-distribution-compenents-packages)
- [8. Update examples in the Cloud9 IDE workspace](#8-update-examples-in-the-cloud9-ide-workspace)
- [9. TIDL Examples](#9-tidl-examples)
- [10. SDK](#10-sdk)

## 1. BeagleBone AI Features

Main Processor Features of the **AM5729** Within BeagleBone® AI

- Dual 1.5GHz ARM® Cortex®-A15 with out-of-order speculative issue 3-way superscalar execution pipeline for the fastest execution of existing 32-bit code
- 2 C66x Floating-Point VLIW DSP supported by OpenCL
- **4 Embedded Vision Engines (EVEs) supported by TIDL machine learning library**
- 2x Dual-Core Programmable Real-Time Unit (PRU) subsystems (4 PRUs total) for ultra low-latency control and software generated peripherals
- 2x Dual ARM® Cortex®-M4 co-processors for real-time control
- IVA-HD subsystem with support for 4K @ 15fps H.264 encode/decode and other codecs @ 1080p60
- Vivante® GC320 2D graphics accelerator
- Dual-Core PowerVR® SGX544™ 3D GPU

위 설명과 같이, TI 사의 딥러닝 라이브러리(TIDL)를 지원한다. 

## 2. Board Bring-up

기본 제공된 eMMC flashed image 로 부팅할 수 있다. 

BeagleBoneAI 의 USB-TypeC 포트를 Host PC 와 연결하면, 전원 공급과 함께 USB 네트워크 연결이 된다. 

## 3. Cloud9 IDE

USB-TypeC 포트를 Host 와 연결하면, BeagleBone 이 Web 서버 역할을 하게되어, http://localhost/ide.html 로 접속하면 타겟의 `/var/lib/cloud9` 밑으로 접속할 수 있다. 

코드 편집과 쉘 접속이 함께 지원된다. 

## 4. Update System
### 4.1. Download Images

https://beagleboard.org/latest-images

- [Buster TIDL, without GUI Image](https://debian.beagleboard.org/images/am57xx-debian-10.3-iot-tidl-armhf-2020-04-06-6gb.img.xz)
  
  - GUI 가 없는 이미지

- [Buster TIDL, with GUI Image](https://groups.google.com/g/beagleboard/c/FYAxTxxZLgg)
  
  - GUI (X11 backend based LXQt) 포함된 이미지
  - 부팅 후 LXQt Desktop GUI 를 사용할 수 있다. 

### 4.2. Update board with latest software

#### 4.2.1. Flash Image to micro SD

다운로드 받은 이미지 파일을 uSD 카드에 [balenaEtcher](https://www.balena.io/etcher/) 로 Flash

uSD 카드를 보드에 삽입 후 전원 인가 (USB C type 포트 연결)

이렇게 SD 카드를 부팅 디바이스로 사용하면서 filesystem 을 쓸 수도 있지만, BeagleBoneAI 에는 eMMC 가 있기 때문에 eMMC 에 부팅 이미지를 flash 할 수 있다. 

#### 4.2.2. Flash eMMC

##### 4.2.2.1. Change boot environment

이미지가 플래쉬 된 SD 카드를 삽입한 상태에서 부팅이 완료되면, cloud9 를 통해 Host 에서 쉘 접속을 할 수 있다. 

그 후 eMMC flash 를 위한 boot environment 를 수정해준다. 

`/boot/uEnv.txt`

```
##enable x15: eMMC Flasher:
##make sure, these tools are installed: dosfstools rsync
#cmdline=init=/opt/scripts/tools/eMMC/init-eMMC-flasher-v3-no-eeprom.sh
```

Change to:

```
##enable x15: eMMC Flasher:
##make sure, these tools are installed: dosfstools rsync
cmdline=init=/opt/scripts/tools/eMMC/init-eMMC-flasher-v3-no-eeprom.sh
```

수정을 완료하고 재부팅하면 자동으로 eMMC flash 가 시작된다. 약 10분 소요 

Flash 가 완료 되면 전원이 꺼지고 4 USRx LED 모두 꺼진다. 그 후 uSD 카드 제거 후 전원 재인가 

업데이트가 완료 된 것을 확인할 수 있다. 

> 기본 제공 이미지에서는 tidl 예제가 없었으나, 업데이트 후 tidl 예제 코드가 생겼음

## 5. Get Connected to the Internet

image 파일을 통해 업데이트를 완료했지만, tidl example 이나 기타 다른 패키지들을 최신 버전으로 업데이트 할 수 있다. 

Debian linux 가 설치된 BeagleBoneAI 에서 직접 패키지 업데이트를 위해 인터넷과 연결한다. 

### 5.1. Ethernet

유선 연결, MAC 주소 확인

```
$ ifconfig eth0
eth0: flags=-28669<UP,BROADCAST,MULTICAST,DYNAMIC>  mtu 1500
        ether 24:7d:4d:34:8a:6a  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 126
```

### 5.2. WiFi

```
$ sudo connmanctl
[sudo] password for debian: temppwd
connmanctl> scan wifi
connmanctl> scan wifi
Scan completed for wifi
connmanctl> services
    lab_2.4GHz     wifi_80913349fa97_76616461735f6c61625f322e3447487a_managed_psk
    lab_5GHz       wifi_80913349fa97_76616461735f6c61625f3547487a_managed_psk
    DIRECT-jZM2070 Series wifi_80913349fa97_4449524543542d6a5a4d3230373020536572696573_managed_psk
    ...
connmanctl> agent on
Agent registered
connmanctl> connect wifi_80913349fa97_76616461735f6c61625f3547487a_managed_psk
Connected wifi_80913349fa97_76616461735f6c61625f3547487a_managed_psk
connmanctl> quit
```

```
$ ifconfig wlan0
wlan0: flags=-28605<UP,BROADCAST,RUNNING,MULTICAST,DYNAMIC>  mtu 1500
        inet 192.168.3.58  netmask 255.255.255.0  broadcast 192.168.3.255
        inet6 fe80::8291:33ff:fe49:fa97  prefixlen 64  scopeid 0x20<link>
        ether 80:91:33:49:fa:97  txqueuelen 1000  (Ethernet)
        RX packets 59  bytes 5924 (5.7 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 66  bytes 12588 (12.2 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

## 6. Update the boot-up scripts and Linux kernel

boot-up sciprts 와 Linux kernel 을 업데이트

현재 Linux kernel 버전은 아래와 같다. (4.14.108-ti-r131)

```
$ uname -a
Linux beaglebone 4.14.108-ti-r131 #1buster SMP PREEMPT Tue Mar 24 19:18:36 UTC 2020 armv7l GNU/Linux
```

boot-up sciprts 업데이트 (pull)

```
$ cd /opt/sciprts
$ git pull
```

커널 업데이트 실행. 해당 아래 스크립트는 apt 계열 명령어를 통해 커널을 업데이트 한다. 

```sh
debian@beaglebone:/opt/scripts$ sudo tools/update_kernel.sh
[sudo] password for debian:
info: checking archive
2021-04-08 04:15:51 URL:https://rcn-ee.com/repos/latest/buster-armhf/LATEST-ti [194/194] -> "LATEST-ti" [1]
-----------------------------
Kernel Options:
ABI:1 LTS41 4.1.30-ti-r70
ABI:1 LTS44 4.4.155-ti-r155
ABI:1 LTS49 4.9.147-ti-r121
ABI:1 LTS414 4.14.108-ti-r140
ABI:1 LTS419 4.19.94-ti-r61
ABI:1 LTS54 5.4.106-ti-r26
ABI:1 LTS510 5.10.21-ti-r1
-----------------------------
Kernel version options:
-----------------------------
LTS414: --lts-4_14
LTS419: --lts-4_19
LTS54: --lts-5_4
LTS510: --lts-5_10
-----------------------------
info: you are running: [4.14.108-ti-r131], latest is: [4.14.108-ti-r140] updating...
Get:1 http://deb.debian.org/debian buster InRelease [121 kB]                                              
...         
```

커널 업데이트 완료 (4.14.108-ti-r140)

```
debian@beaglebone:/var/lib/cloud9$ uname -a
Linux beaglebone 4.14.108-ti-r140 #1buster SMP PREEMPT Wed Mar 31 11:12:16 UTC 2021 armv7l GNU/Linux
```

## 7. Update distribution compenents (packages)

```sh
$ sudo apt update
$ sudo apt upgrade
```

TIDL 예제 실행을 위해 다음 패키지를 설치한다. 

```sh
$ sudo apt install ti-tidl mjpg-streamer-opencv-python
```

## 8. Update examples in the Cloud9 IDE workspace

```sh
$ cd /var/lib/cloud9
$ git pull
```

## 9. TIDL Examples

- `/var/lib/cloud9/BeagleBone/AI/tidl`
  - 21.04 기준으로 Classification 예제만 제공
- `/usr/share/ti` 
  - [여기](/embedded/beaglebone-ai-tidl) 참고
  - Classfication, Object Detection 등의 예제 제공

## 10. SDK

[여기](/embedded/beaglebone-ai-sdk/) 참고

---
