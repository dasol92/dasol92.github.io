---
title:  "BeagleBone AI 보드로 딥러닝 모델 추론"
categories: 
  - Embedded
tags:
  - Texas Instruments
  - Beaglebone
  - Deep learning
---
BeagleBone AI 는 TI SoC (AM5729) 를 탑재하여, 내장된 가속기 (EVE, DSP) 를 이용해 딥러닝 연산을 수행한다. 

- [1. TIDL\_API Examples](#1-tidl_api-examples)
  - [1.1. SSD\_multibox with Default model](#11-ssd_multibox-with-default-model)
    - [1.1.1. Default Input SSD](#111-default-input-ssd)
    - [1.1.2. Video Input SSD](#112-video-input-ssd)
      - [1.1.2.1. (Error) connot open display](#1121-error-connot-open-display)
      - [1.1.2.2. (Error) "Duplicate option categories"' failed.](#1122-error-duplicate-option-categories-failed)
- [2. Build TIDL\_API](#2-build-tidl_api)
  - [2.1. TIDL\_API Library](#21-tidl_api-library)
  - [2.2. TIDL\_API Examples](#22-tidl_api-examples)
- [3. TIDL\_API EVE, DSP](#3-tidl_api-eve-dsp)
  - [3.1. EVE 1개, DSP 1개로 실행](#31-eve-1개-dsp-1개로-실행)
  - [3.2. EVE 4개, DSP 1개로 실행](#32-eve-4개-dsp-1개로-실행)
- [DL 성능 결과에 대하여](#dl-성능-결과에-대하여)


BeagleBone AI 보드에서 실행 가능한 모델은 TIDL 변환 모델이다. 

> TI 학습 모델을 제공하는데, Jacinto7 (TDA4) 용으로는 PyTorch 모델도 제공 하지만, Jacinto6 (TDA2, TDA3 ...) 에서는 Caffe-jacinto, TensorFlow to TIDL 모델만 제공한다. 

NN Layer 중에서 TIDL 로 지원(변환) 되는 종류는 여기 [링크](https://software-dl.ti.com/processor-sdk-linux/esd/docs/06_03_00_106/linux/Foundational_Components/Machine_Learning/tidl.html#neural-network-layers-supported-by-tidl) 에서 확인 가능하다. 

## 1. TIDL_API Examples

타겟 보드의 다음 경로에 TIDL 예제가 있다. `/usr/share/ti/examples/tidl`

### 1.1. SSD_multibox with Default model

#### 1.1.1. Default Input SSD

SSD 기반의 JdetNet 모델을 사용, TIDL_API 를 사용해 Object Detection 수행하는 예제이다. TIDL_API 를 수정/빌드 하고 싶다면 [2. Build TIDL_API](#2-build-tidl_api) 참조

TIDL_API 예제는 OpenCL의 메모리 접근 동작과 관련해 `/dev/mem` 파일을 열어야 하기 때문에, root 권한으로 실행한다. 

```sh
$ cd /usr/share/ti/examples/tidl/ssd_multibox
$ sudo ./ssd_multibox
Input: ../test/testvecs/input/horse_768x320.y
Saving frame 0 to: frame_0.png
Saving frame 0 with SSD multiboxes to: multibox_0.png
Loop total time (including read/write/opencv/print/etc):  889.8ms
ssd_multibox PASSED
```

기본 입력 영상은 `../test/testvecs/input/horse_768x320.y` 로, 해당 파일은 B,G,R 8bit planar 데이터를 담고 있는 파일이다. (일반적인 preset YUV 혹은 RGB 포맷 아님)

#### 1.1.2. Video Input SSD

예제 실행 시 argument 옵션 `-i` 로 입력 영상 지정이 가능하다. 

> Argument 옵션 목록

```sh
./ssd_multibox -h
```

```c
void DisplayHelp()
{
    std::cout <<
    "Usage: ssd_multibox\n"
    "  Will run partitioned ssd_multibox network to perform "
    "multi-objects detection\n"
    "  and classification.  First part of network "
    "(layersGroupId 1) runs on EVE,\n"
    "  second part (layersGroupId 2) runs on DSP.\n"
    "  Use -c to run a different segmentation network.  Default is jdetnet_voc.\n"
    "Optional arguments:\n"
    " -c <config>          Valid configs: jdetnet_voc, jdetnet \n"
    " -d <number>          Number of dsp cores to use\n"
    " -e <number>          Number of eve cores to use\n"
    " -i <image>           Path to the image file as input\n"
    "                      Default are 9 frames in testvecs\n"
    " -i camera<number>    Use camera as input\n"
    "                      video input port: /dev/video<number>\n"
    " -i <name>.{mp4,mov,avi}  Use video file as input\n"
    " -l <objects_list>    Path to the object classes list file\n"
    " -f <number>          Number of frames to process\n"
    " -w <number>          Output image/video width\n"
    " -p <number>          Output probability threshold in percentage\n"
    "                      Default is 25 percent or higher\n"
    " -v                   Verbose output during execution\n"
    " -h                   Help\n";
}
```

동영상 (`ssd_multibox/clips/*.mp4 `) 파일은 아래와 같은 에러로 실행 불가

##### 1.1.2.1. (Error) connot open display

display 로 실행결과를 확인하기 위해서 with GUI 이미지(X11 based LXQt) 로 부팅하였음에도 불구하고, 
아래와 같은 메시지와 함께 display 를 열 수 없음

```sh
$ sudo ./ssd_multibox -i clips/pexels_videos_3623.mp4
Input: clips/pexels_videos_3623.mp4
Unable to init server: Could not connect: Connection refused

(SSD_Multibox:1871): Gtk-WARNING **: 05:25:37.637: cannot open display:
```

`export DISPLAY=:0.0` 설정 후에도 다음과 같이 디스플레이를 open 하지 못하는 건 마찬가지이다. 

```sh
Input: clips/pexels_videos_3623.mp4
No protocol specified
Unable to init server: Could not connect: Connection refused

(SSD_Multibox:1877): Gtk-WARNING **: 05:26:26.752: cannot open display: :0.0
```

##### 1.1.2.2. (Error) "Duplicate option categories"' failed.

직접 키보드/마우스를 연결한 후 터미널을 로컬 접속해서 명령어 입력했으나 OCL 관련 에러로 실행 불가

```sh
$ sudo ./ssd_multibox -i clips/pexels_videos_3623.mp4
Input: clips/pexels_videos_3623.mp4

ssd_multibox: /build/ti-llvm-3.6-oHqxVY/ti-llvm-3.6-3.6-git20190111.1/lib/Support/CommandLine.cpp:141: void llvm::cl::OptionCategory::registerCategory(): Assertion `std::count_if(RegisteredOptionCategories->begin(), RegisteredOptionCategories->end(), [this](const OptionCategory *Category) { return getName() == Category->getName(); }) == 0 && "Duplicate option categories"' failed.
```


## 2. Build TIDL_API

예제에서 사용하고 있는 TIDL_API 라이브러리 빌드

> TIDL_API 라고 이름 붙인 라이브러리는 종류가 여러가지이다. 본 예제에서 동작하는 라이브러리는 Jacinto6 시리즈 용이며, TDA4 등의 Jacinto7 에서 동작하는 TIDL 라이브러리도 별도로 존재한다. 

- 경로
  - TargetBoard
    - `/usr/share/ti/tidl/tidl_api` 
    - TIDL_API 헤더, 빌드 된 라이브러리 파일 존재
  - SDK 
    - `${SDK_DIR}/linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/usr/share/ti/tidl/tidl_api`
    - SDK (AM57x PSDK Linux) 설치는 [여기](/embedded/beaglebone-ai-sdk/) 참조
    - TIDL_API 헤더, **소스(cpp) 파일** 존재 
      ↪ TIDL_API 라이브러리의 수정을 위해서는 SDK의 소스를 빌드하면 된다. 

빌드를 위해 툴체인 활성화

```sh
source ${SDK_DIR}/linux-devkit/environment-setup
```

sdk의 targetNFS 를 `TARGET_ROOTDIR` 환경 변수에 설정 해준다. TIDL_API 빌드 스크립트에서 필요하다. 

```sh
export TARGET_ROOTDIR=${SDK_DIR}/linux-devkit/targetNFS
```

이 경로를 타겟보드에서 NFS 를 통해 마운트, 접근할 것이다. 

### 2.1. TIDL_API Library

TIDL_API 빌드

```sh
$ cd ${SDK_DIR}/linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/usr/share/ti/tidl/tidl_api
$ make
```

BeagleBone 타겟에서 TIDL_API 예제들은 static 라이브러리 (`tidl_api.a`) 를 참조한다. 

> (`exmples/make.common` 에서 설정 됨)
> 
> ```
> TIDL_API_LIB_NAME = tidl_api.a
> ```

> BeagleBoneAI 보드는 기본적으로 보드 내에 컴파일러를 가지고 있다. 따라서 별도 환경 구축 없이 타겟 위에서 직접 빌드가 가능하다. 
> 만약 BeagleBone 타겟 상에서 TIDL_API 예제를 빌드하고 싶다면 TIDL_API 빌드 결과물 중 `tidl_api.a` 를 타겟의 `/usr/share/ti/tidl/tidl_api/` 밑으로 복사 후 TIDL_API 예제를 빌드 시 변경 사항이 반영 된다. 

### 2.2. TIDL_API Examples

Host PC 에서 TIDL_API 예제를 빌드 하기 위해서도 동일하게 환경 셋팅이 필요하다. 위와 같이 툴체인을 활성화 하고, `TARGET_ROOTDIR` 환경 변수를 설정해준다.

SSD_multibox 예제를 빌드해보자. 

```sh
$ cd ${SDK_DIR}/linux-devkit/sysroots/armv7at2hf-neon-linux-gnueabi/usr/share/ti/tidl/tidl_api/examples/ssd_multibox
$ make
```

빌드 결과물을 targetNFS 로 복사해주기 위해, `Makefile` 에 `install` 타겟을 추가한다. 

```Makefile
# ssd_multibox/Makefile
...
install:
    cp -v $(EXE) $(TARGET_ROOTDIR)/usr/share/ti/tidl/examples/ssd_multibox/
```

빌드된 결과물은 `make install` 시에 `$(TARGET_ROOTDIR)` 밑으로 복사된다. 

BeagleBoneAI 타겟보드에 있는 공유 라이브러리 버전과, SDK 에서 사용한 라이브러리 버전의 차이로 몇몇 라이브러리는 복사가 필요하다. 

`${SDK_DIR}/linux-devkit/targetNFS/usr/lib` 에 있는 아래 라이브리들을, 
타겟의 라이브러리 참조 경로인 `/usr/lib/arm-linux-gnueabihf/` 으로 복사해준다. 

```
libopencv_*.so.3.1*
libwebp.so.7*
libjson-c.so.4*
libQt5Test.so.5*
libgstapp-1.0.so.0*
libgst*-1.0.so.0*
```

> 타겟에서의 라이브러리 참조 경로는 `/etc/ld.so.conf`, `/etc/ld.so.conf.d/arm-linux-gnueabihf.conf` 에서 확인할 수 있다. 

위의 라이브러리를 모두 복사하고 나서는 라이브러리를 찾지 못한다는 메시지는 없는데, 아래와 같이 특정 심볼을 찾지 못한다. 즉 라이브러는 있는 데 이 심볼이 없다는 것. grep 으로 `/usr/lib` 에서 `__free_ddr` 을 검색해보니 OpenCL 라이브러리와 관련이 있다. 

```sh
$ ./ssd_multibox: symbol lookup error: ./ssd_multibox: undefined symbol: __free_ddr
```

실행파일에서도 라이브러리를 어떻게 참조 하고 있는지 확인해본다. 

```sh
$ ldd ./ssd_multibox
...
        libOpenCL.so.1 => /usr/lib/arm-linux-gnueabihf/libOpenCL.so.1 (0xb6ece000)
...
```

OpenCL 라이브러리는 BeagleBoneAI 에서는 `libTIOpenCL.so.1.2` 의 이름을 가지고 있고, AM57x SDK 에서는 `libOpenCL.so.1` 이기 때문에 빌드한 실행 파일도 같은 이름의 `libOpenCL.so.1` 을 참조하고 있다. 

그런데 현재 심볼을 찾지 못하는 실행파일이 참조하는 OpenCL 라이브러리의 버전이 1.0.0 으로 확인 된다. 

```sh
$ ll /usr/lib/arm-linux-gnueabihf/libOpenCL.so.1*
lrwxrwxrwx 1 root root    18 Dec 17  2018 /usr/lib/arm-linux-gnueabihf/libOpenCL.so.1 -> libOpenCL.so.1.0.0
-rw-r--r-- 1 root root 26020 Dec 17  2018 /usr/lib/arm-linux-gnueabihf/libOpenCL.so.1.0.0
```

즉 현재 실행파일은 SDK 에서 빌드될 때 `libOpenCL.so.1` 을 가진 1.2 버전으로 정상적으로 빌드가 되었지만, 타겟보드에서 `libOpenCL.so.1` 의 이름을 가진 라이브러리는 1.0.0 버전인 상황이다. 

따라서 SDK 안에 있던 libOpenCL.so.1.2 버전을 타겟으로 복사해준다. 이 때 타겟의 라이브러리 참조 경로 중 기존 `/usr/lib/arm-linux-gnueabihf/libOpenCL.so.1` 을 덮어쓰기 하지 않기 위해서, 다른 경로 `/lib/arm-linux-gnueabihf/` 밑으로 복사해준다. 

```sh
sudo cp -av targetNFS/usr/lib/libOpenCL.so.1.2* /lib/arm-linux-gnueabihf/
```

`/lib/arm-linux-gnueabihf/` 경로가 기존 1.0 버전 라이브러리가 있던 경로보다 우선순위가 높다. 라이브러리 참조 경로를 설정해놓은 `/etc/ld.so.conf.d/arm-linux-gnueabihf.conf` 파일에서 더 상위 라인에 있기 때문이다. 

```
# Multiarch support
/usr/local/lib/arm-linux-gnueabihf
/lib/arm-linux-gnueabihf
/usr/lib/arm-linux-gnueabihf
```

위와 같이 설정했기 때문에 1.2 버전을 1.0 버전보다 더 먼저 찾을 것이다. 
링크 관계를 다시 lddconfig 로 링크를 다시 만들어준다.

```sh
$ sudo ldconfig
```

실행파일에서 참조하고 있는 OpenCL 라이브러리를 확인해보면 1.2 버전이 있는 곳으로 경로가 바뀐 것을 확인할 수 있다. 

```sh
$ ldd ./ssd_multibox
...
        libOpenCL.so.1 => /lib/arm-linux-gnueabihf/libOpenCL.so.1 (0xb6c71000)
```

## 3. TIDL_API EVE, DSP

TIDL_API 예제를 실행할 때, EVE 코어와 DSP 코어에서 잘 실행되고 있는지 확인을 해보자. 

- 코어 확인

`Executor::GetDeivce(DeviceType)` 를 통해서 현재 시스템의 EVE, DSP 의 개수를 알아낼 수 있다. 

- 코어별 점유율 확인

앱을 실행 할 때 `-v` (verbose) 옵션을 설정하면 `Configuration::ApiTrace` 와 `TRACE::enabled` 가 true 로 설정되어 `TRACE::print` 가 터미널로 출력 된다.

### 3.1. EVE 1개, DSP 1개로 실행

```
debian@beaglebone:~/nfs_host/usr/share/ti/tidl/examples/ssd_multibox$ sudo ./ssd_multibox -i ../test/testvecs/input/roads/test_1.png -c jdetnet -e 1 -d 1 -v
[DEBUG] GetNumDevices num_eves, num_dsps: 4 2
[DEBUG] Set Using num_eves, num_dsps: 1 1
Input: ../test/testvecs/input/roads/test_1.png
[DEBUG] c.runFullNet: 0
[DEBUG] : Setup camera/video input done: 0 0
[DEBUG] : Setup preprocessed input done: 0
-> Executor::Executor()
        OCL Device: EVE created
...
```

### 3.2. EVE 4개, DSP 1개로 실행

```
debian@beaglebone:~/nfs_host/usr/share/ti/tidl/examples/ssd_multibox$ sudo ./ssd_multibox -i ../test/testvecs/input/roads/test_1.png -c jdetnet -e 4 -d 1 -v
[DEBUG] GetNumDevices num_eves, num_dsps: 4 2
[DEBUG] Default Using num_eves, num_dsps: 4 1
Input: ../test/testvecs/input/roads/test_1.png
[DEBUG] c.runFullNet: 0
[DEBUG] : Setup camera/video input done: 0 0
[DEBUG] : Setup preprocessed input done: 0
-> Executor::Executor()
        OCL Device: EVE created
```

## DL 성능 결과에 대하여

Object Detection 예제 테스트 진행

TDA2x 에서도 사용한 모델을 테스트 했으나, 속도 성능에 차이가 있다. 

- 모델 : JDetNet 모델
- 속도성능
  - BeagleBoneAI : 4 FPS
  - TDA2x : 29-30 FPS

두 플랫폼간의 차이는 다음과 같다. 

1. 사용하는 TIDL 라이브러리 차이
   1. TDA2x 에서 사용하는 TIDL
      * 버전 01.01 ("TI Deep learning Library on EVE and DSP Version 01.01")
      * IVISION interface 기반 (extension of the eXpressDSP Algorithm Interface Standard (XDAIS))
      * VisionSDK (PROCESSOR-SDK-VISION 3.X ) 와 함께 제공, C 로만 작성 됨
      * 주요 API 로는 algInit(), algProcess() ... 등
      * VisionSDK 앱 내에서 TIDL Link 가 API 를 호출 함
      * 입력 영상 리사이즈는 VPE Link (on IPU core) 에서 수행
    2. BeagleBoneAI 에서 사용하는 TIDL
      * "TIDL API 1.4"
      * EVE와 DSP 에서 DL 동작을 위해 TI's OpenCL 사용
      * 다른 framework (ex. OpenCV) 와 통합하기 쉬움
      * PROCESSOR-SDK-LINUX 6.3.X 와 함께 제공, C++ 로 작성 됨
      * 주요 API 로는 Executor(), ProcessFrameStartAsync() .. 등
      * 입력 영상 리사이즈는 OpenCV (on ARM core) 에서 수행
      * TIDL API 가이드 문서 상 속도 성능 (on AM57x)
        * JDetNet 768x320 모델, 약 7 FPS
        * BeagleBoneAI 에서 동일 모델 테스트 결과 약 4 FPS 로 동작
2. OS 환경 차이
    1. TDA2x 
       * Linux + RTOS
    2. BeagleBoneAI 
       * BeagleBone 에서 제공한 공식 이미지는 Linux Only (Debian)
       * TI 에서 제공한 Linux + RTOS 이미지 설치 가능
