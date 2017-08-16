# Description

Setup development environment upon Ubuntu Desktop 16.04 with LightDM GUI

## Install Ubuntu through USB or CDROM

## Step 1 - Login X-Window
* Press ``` CTRL-ALT-T ``` to launch Terminal shell 
* Install OpenSSH server, allow remote ssh login
    ```
    $ sudo apt-get install -y openssh-server
    ```

* Verify remote ssh connection at notebook, and copy public key.
* Install essential toolkits
    ```
    $ sudo apt-get install -y build-essential curl unzip dkms pkg-config linux-headers-generic vim git tree 
    ```

* You might modify /etc/default/locale, change all ``` lzh_TW ``` to ``` en_US.UTF-8 ``` 


## Step 2 - Download NVIDIA cuda drivers
* Download from local server, or from NVDIA developer program
    ```
    $ scp -r isaac:/home/dataset/GPU .
    ```

* Remove *incompatible* open source nvidia driver from kernel
    ```
    $ sudo bash -c "echo -e \"blacklist nouveau\nalias nouveau off\" > /etc/modprobe.d/nvidia.conf"
    $ sudo update-initramfs -u
    $ sudo reboot
    ```

## Step 3 - Install NVIDIA drivers
* Use SSH connect to target workstation
* Stop X-Window
    ```
    $ sudo service lightdm stop
    ```

* NVidia 381 driver is incompatible with Ubuntu X session, resulting login failure on GUI IDE. Thus failback to Ubuntu default 371 driver 
    ```
    $ sudo ubuntu-drivers autoinstall
    $ sudo reboot
    ```

* Use SSH connect to target workstation again
* Verify driver installed correctly. 
    ```
    $ nvidia-smi
    ```

* Install CUDA driver and its patch
    ```
    //// Note: 
    ////    1. Do NOT install Graphics Driver, use above latest driver instead 
    ////    2. Did NOT install sample code
    ////    3. Yes to other options
    $ sudo sh GPU/cuda_8.0.61_375.26_linux-run
    $ sudo sh GPU/cuda_8.0.61.2_linux-run
    ```

* Install CUDNN driver for Neural Network accerlation.
    ```
    $ sudo tar zxvf GPU/cudnn-8.0-linux-x64-v5.1.tgz -C /usr/local/
    ```

* Add cuda to library path, so other program can load library correctly
    ```
    $ sudo bash -c "echo /usr/local/cuda/lib64/ > /etc/ld.so.conf.d/cuda.conf"
    $ sudo ldconfig
    ```

* Add cuda to execution path
    ```
    $ echo "export PATH=\"/usr/local/cuda/bin:\$PATH\"" >> ~/.bashrc
    ```

## Step 4 - Install Python packages

* Install Python 3
    ```
    $ sudo apt-get update
    $ sudo apt-get install -y python3 python3-pip python3-setuptools python3-tk
    $ echo "alias python=\"python3\"" >> ~/.bashrc
    $ echo "alias pip=\"pip3\"" >> ~/.bashrc
    ```

* Install 3rd party Python library
    ```
    $ sudo pip3 install --upgrade pip numpy ipykernel jupyter matplotlib Pillow
    $ sudo pip3 install --upgrade pandas scipy scikit-learn
    ```

* Fix pip format error
    ```
    $ mkdir -p ~/.pip && echo -e "[list]\nformat = columns" > ~/.pip/pip.conf
    ```

* Install OpenCV
    ```
    $ sudo apt-get install -y ffmpeg openexr webp libgtk2.0-0
    $ sudo pip3 install https://bazel.blob.core.windows.net/opencv/opencv_python-3.3.0-cp35-cp35m-linux_x86_64.whl
    ```

* Install Tensorflow with GPU support
    ```
    $ sudo pip3 install --upgrade https://bazel.blob.core.windows.net/tensorflow-gpu/tensorflow-1.2.1-cp35-cp35m-linux_x86_64.whl
    ```

* Test if everything works fine, launch python with below scripts
    ```Python
    import numpy as np
    import cv2
    import tensorflow as tf
    hello = tf.constant('Hello, TensorFlow!')
    sess = tf.Session()
    print(sess.run(hello))
    ```

## Step 5 - Burn GPU and monitor 

* Download ` gpu-bench.py ` script
* Compute N iteration of matrix multiplication, say ` 2000 `
    ```
    $ time python gpu-bench.py 2000
    ```

* Open another terminal or SSH shell to watch GPU utilization, focus on GPU-Util percentage and Power voltage
    ```
    $ watch -n 1 nvidia-smi

    +-----------------------------------------------------------------------------+
    | NVIDIA-SMI 375.66                 Driver Version: 375.66                    |
    |-------------------------------+----------------------+----------------------+
    | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
    | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
    |===============================+======================+======================|
    |   0  GeForce GTX 108...  Off  | 0000:01:00.0     Off |                  N/A |
    | 20%   51C    P2   237W / 250W |  10695MiB / 11172MiB |    100%      Default |
    +-------------------------------+----------------------+----------------------+
                                                                                
    +-----------------------------------------------------------------------------+
    | Processes:                                                       GPU Memory |
    |  GPU       PID  Type  Process name                               Usage      |
    |=============================================================================|
    |    0      1062    G   /usr/lib/xorg/Xorg                              62MiB |
    |    0      1780    C   python3                                      10629MiB |
    +-----------------------------------------------------------------------------+
    ```
