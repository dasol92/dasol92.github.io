---
title:  "Bind TensorFlow to C language"
categories: 
  - Deep-learning
---

* TOC
{:toc}

TensorFlow 를 C/C++ 에서 사용할 수 있도록 C API 가 제공된다. 
이를 활용해 C/C++ 코드를 통해 TensorFlow 모델로 추론을 해보자. 


## Install TensorFlow
WSL Ubuntu 18.04 에서 TensorFlow 설치

Python3 버전 확인, 없다면 설치
```sh
python3 --version
```
```sh
sudo apt install python3 python3-venv
```

작업용 가상환경 생성 및 활성화
```
mkdir tf-c-api
cd tf-c-api
python3 -m venv .
source bin/activate
```

TensorFlow 패키지 설치
```
pip install tensorflow
```

TensorFlow 패키지 import와 테스트용 tensor 생성 시 아래와 같은 메시지 출력된다. GPU 가 없기 때문에 CUDA 관련은 무시해도 되며, Intel CPU 의 XLA 관련 메시지는 추후 확인이 필요하다. 
```
>>> import tensorflow as tf
2021-03-03 23:09:43.780360: W tensorflow/stream_executor/platform/default/dso_loader.cc:60] Could not load dynamic library 'libcudart.so.11.0'; dlerror: libcudart.so.11.0: cannot open shared object file: No such file or directory
2021-03-03 23:09:43.780449: I tensorflow/stream_executor/cuda/cudart_stub.cc:29] Ignore above cudart dlerror if you do not have a GPU set up on your machine.

>>> 
>>> print(tf.reduce_sum(tf.random.normal([1000, 1000])))
2021-03-03 23:10:33.195135: I tensorflow/compiler/jit/xla_cpu_device.cc:41] Not creating XLA devices, tf_xla_enable_xla_devices not set
2021-03-03 23:10:33.196385: W tensorflow/stream_executor/platform/default/dso_loader.cc:60] Could not load dynamic library 'libcuda.so.1'; dlerror: libcuda.so.1: cannot open shared object file: No such file or directory
2021-03-03 23:10:33.196448: W tensorflow/stream_executor/cuda/cuda_driver.cc:326] failed call to cuInit: UNKNOWN ERROR (303)
2021-03-03 23:10:33.196580: I tensorflow/stream_executor/cuda/cuda_diagnostics.cc:156] kernel driver does not appear to be running on this host (gram-Dasol): /proc/driver/nvidia/version does not exist
2021-03-03 23:10:33.197385: I tensorflow/core/platform/cpu_feature_guard.cc:142] This TensorFlow binary is optimized with oneAPI Deep Neural Network Library (oneDNN) to use the following CPU instructions in performance-critical operations:  AVX2 FMA
To enable them in other operations, rebuild TensorFlow with the appropriate compiler flags.
2021-03-03 23:10:33.199403: I tensorflow/compiler/jit/xla_gpu_device.cc:99] Not creating XLA devices, tf_xla_enable_xla_devices not set
tf.Tensor(261.91336, shape=(), dtype=float32)
```


## Tutorial - Image Classifications

Tutorial 의 꽃 이미지 Classification 예제 [링크](https://www.tensorflow.org/tutorials/images/classification)

```python
import matplotlib.pyplot as plt
import numpy as np
import os
import PIL
import tensorflow as tf

from tensorflow import keras
from tensorflow.keras import layers
from tensorflow.keras.models import Sequential
```

기본적으로 TensorFlow 만 설치한 상태에서는 찾지 못하는 패키지
`matplotlib`
```sh
pip install matplotlib
```


## Saved model 활용
[참고](https://www.tensorflow.org/guide/saved_model?hl=ko)

```
tf.saved_model.save(pretrained_model, "./flower/1/")
```


저장한 모델의 tag 설정 
```
$ saved_model_cli show --dir /home/dskim/tf-c-api/flower/1 --tag_set serve --signature_def serving_default
2021-03-06 23:26:34.309029: W tensorflow/stream_executor/platform/default/dso_loader.cc:60] Could not load dynamic library 'libcudart.so.11.0'; dlerror: libcudart.so.11.0: cannot open shared object file: No such file or directory
2021-03-06 23:26:34.310318: I tensorflow/stream_executor/cuda/cudart_stub.cc:29] Ignore above cudart dlerror if you do not have a GPU set up on your machine.
The given SavedModel SignatureDef contains the following input(s):
  inputs['sequential_1_input'] tensor_info:
      dtype: DT_FLOAT
      shape: (-1, 180, 180, 3)
      name: serving_default_sequential_1_input:0
The given SavedModel SignatureDef contains the following output(s):
  outputs['dense_3'] tensor_info:
      dtype: DT_FLOAT
      shape: (-1, 5)
      name: StatefulPartitionedCall:0
Method name is: tensorflow/serving/predict
```


## TensorFlow 소스 빌드 by Bazel 

### 1. WSL에 nvm 설치
> [참고 링크](https://docs.microsoft.com/ko-kr/windows/nodejs/setup-on-wsl2)

```
sudo apt install curl
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
```
### 2. Node.js 설치
```
nvm install node
```
node, npm 버전 확인
```
node --version
npm --version
```
### 3. Bazelisk 설치
> [Bazel 참고 링크](https://docs.bazel.build/versions/master/install-bazelisk.html)

```
npm install -g @bazel/bazelisk
```
설치 확인
```
npm list --global
/home/dskim/.nvm/versions/node/v15.11.0/lib
├── @bazel/bazelisk@1.7.5
└── npm@7.6.0
```
Bazel, Bazelisk 확인
```
$ which bazel
/home/dskim/.nvm/versions/node/v15.11.0/bin/bazel
```
```
$ which bazelisk
/home/dskim/.nvm/versions/node/v15.11.0/bin/bazelisk
```

### 4. TensorFlow 소스 빌드
```
git clone https://github.com/tensorflow/tensorflow.git
```
Bazel 이 있는 상황에서, TensorFlow 소스 구성(configure)
```
cd tensorflow
./configure
```

나의 configure 세션 내용
```
./configure 
You have bazel 3.7.2 installed.
Please specify the location of python. [Default is /home/dskim/.virtualenvs/tf2/bin/python3]: 


Found possible Python library paths:
/home/dskim/.virtualenvs/tf2/lib/python3.6/site-packages
Please input the desired Python library path to use.  Default is [/home/dskim/.virtualenvs/tf2/lib/python3.6/site-packages]

Do you wish to build TensorFlow with ROCm support? [y/N]: 
No ROCm support will be enabled for TensorFlow.

Do you wish to build TensorFlow with CUDA support? [y/N]: 
No CUDA support will be enabled for TensorFlow.

Do you wish to download a fresh release of clang? (Experimental) [y/N]: 
Clang will not be downloaded.

Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -Wno-sign-compare]: 


Would you like to interactively configure ./WORKSPACE for Android builds? [y/N]: 
Not configuring the WORKSPACE for Android builds.

Preconfigured Bazel build configs. You can use any of the below by adding "--config=<>" to your build command. See .bazelrc for more details.
        --config=mkl            # Build with MKL support.
        --config=mkl_aarch64    # Build with oneDNN support for Aarch64.
        --config=monolithic     # Config for mostly static monolithic build.
        --config=numa           # Build with NUMA support.
        --config=dynamic_kernels        # (Experimental) Build kernels into separate shared objects.
        --config=v2             # Build TensorFlow 2.x instead of 1.x.
Preconfigured Bazel build configs to DISABLE default on features:
        --config=noaws          # Disable AWS S3 filesystem support.
        --config=nogcp          # Disable GCP support.
        --config=nohdfs         # Disable HDFS support.
        --config=nonccl         # Disable NVIDIA NCCL support.
Configuration finished
```

pip 패키지 빌드
```
bazel build //tensorflow/tools/pip_package:build_pip_package
```

빌드 후, `tensorflow/bazel-bin/tensorflow/` 밑에 아래와 같이 라이브러리들이 생성 되었다. 
```
libtensorflow_framework.so.2.5.0-2.params*
libtensorflow_framework.so.2.5.0*
libtensorflow_framework.so.2 -> libtensorflow_framework.so.2.5.0*
libtensorflow_framework.so -> libtensorflow_framework.so.2*
```
위 라이브러리들을 환경변수 $LD_LIBRARY_PATH 에 등록해주자.

```sh
$ export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:~/tf-c-api/tensorflow/bazel-bin/tensorflow
$ export LIBRARY_PATH=$LIBRARY_PATH:~/tf-c-api/tensorflow/bazel-bin/tensorflow
```

## Hello TensorFlow C API library
아래 샘플 코드를 빌드 
```c
#include <stdio.h>
#include <tensorflow/c/c_api.h>

int main() {
    printf("Hello from TensorFlow C library version %s\n", TF_Version());
    return 0;
}
```

```
g++ hello_tf.cpp -ltensorflow_cc -o hello_tf -Itensorflow
```

hello_tf 실행
```
./hello_tf 
Hello from TensorFlow C library version 2.5.0
```

