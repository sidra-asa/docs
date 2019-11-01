> Author: sidra-asa

# Installation

## OS Environment

OS: Ubuntu 18.04

## Graphic Card Driver

### Check if your graphic card supports CUDA.

[link](https://developer.nvidia.com/cuda-gpus#compute)

If Nouveau kernel driver may already in yout system.
Stop it:

    # vim /etc/modprobe.d/blacklist-nouveau.conf 

    Add:

    blacklist nouveau
    options nouveau modeset=0

    $ sudo update-initramfs -u
    $ sudo reboot

### Choose CUDA version

I choose CUDA 10.0 so I downloaded cuda-repo-ubuntu1804_10.0.130-1_amd64.deb and installed it.

[link](https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-repo-ubuntu1804_10.0.130-1_amd64.deb)

You can choose what you want from [here](https://developer.download.nvidia.com/compute/cuda/repos/) .

    $ wget cuda-repo-ubuntu1804_10.0.130-1_amd64.deb
    $ sudo dpkg -i cuda-repo-ubuntu1804_10.0.130-1_amd64.deb

Install apt key file.

    $ sudo apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/7fa2af80.pub
    $ sudo apt update

Install machine learning repo package.

     $ wget http://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64/nvidia-machine-learning-repo-ubuntu1804_1.0.0-1_amd64.deb
    $ sudo apt install ./nvidia-machine-learning-repo-ubuntu1804_1.0.0-1_amd64.deb
    $ sudo apt update

### Download corresponding driver.

My graphic card is GeForce GTX 1070. Check if any available drivers.

    $ ubuntu-drivers devices
    
    == /sys/devices/pci0000:00/0000:00:02.0/0000:03:00.0 ==
    vendor   : NVIDIA Corporation
    modalias : pci:v000010DEd00001B81sv00001043sd00008597bc03sc00i00
    driver   : nvidia-410 - third-party non-free
    driver   : xserver-xorg-video-nouveau - distro free builtin
    driver   : nvidia-418 - third-party non-free
    driver   : nvidia-415 - third-party free
    driver   : nvidia-387 - third-party non-free
    driver   : nvidia-396 - third-party non-free
    driver   : nvidia-384 - third-party non-free
    driver   : nvidia-390 - third-party non-free
    driver   : nvidia-430 - third-party free recommended

I choose nvidai-410 due to I got some problem while nvidia-418.

    $ sudo apt install nvidia-410 cuda-10-0 libcudnn7-dev

Enjoy it !

## Refferences

* [Ubuntu 18.04 安裝 NVIDIA Driver 418 & CUDA 10 & Miniconda & TensorFlow 1.13](http://tiny.cc/i0wifz)
* [ msis/install_nvidia_dependencies.md](https://gist.github.com/msis/108a74d08f55eed48d8521fa968851ea)

