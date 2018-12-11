# Install MXNet 1.3.1 on Nvidia Jetson TX2

Build from source
---

1. Prepare your board with Jetpack 3.3. In this tutorial, it's going to be using CUDA 9.0 and cuDNN 7.1.5. Jetpack 3.3 is the latest version supported on TX2 when this instruction is authored.

2. Login to your device or ssh into your device.
    ```
    ssh nvidia@<device-ip-address>
    ```

3. Install requirements    
    These requirements are listed on MXNet website:
    ```
    sudo apt-get update
    sudo apt-get -y install git build-essential libatlas-base-dev libopencv-dev graphviz python-pip
    sudo pip install pip --upgrade
    sudo pip install setuptools numpy --upgrade
    sudo pip install graphviz jupyter
    ```
    In this tutorial, we use openblas, so we also need to install that lib:
    ```
    sudo apt-get -y install libopenblas-dev 
    ```

4. In the home of user ```nvidia```, create work environment.
    ```
    mkdir mxnet-workspace
    cd mxnet-workspace
    wget https://github.com/apache/incubator-mxnet/releases/download/1.3.1/apache-mxnet-src-1.3.1.rc0-incubating.tar.gz
    tar xvzf apache-mxnet-src-1.3.1.rc0-incubating.tar.gz
    cd apache-mxnet-src-1.3.1.rc0-incubating
    ```

5. Create build configuration.
    ```
    cp make/crosscompile.jetson.mk config.mk

    # Make sure CUDA path is configured correctly.
    # For example: /usr/local/cuda-9.0/targets/aarch64-linux
    cat config.mk | grep -e '^USE_CUDA_PATH ='

    # Make sure NVCC path is configured correctly.
    # For example: /usr/local/cuda-9.0/bin/nvcc
    cat config.mk | grep -e '^NVCC ='
    ```
    Also, append below line in your config.mk if it's not defined in the MXNet source. This is used for nvcc compilation.
    ```
    export CUDA_ARCH=-gencode arch=compute_53,code=sm_53 -gencode arch=compute_62,code=sm_62
    ```
    ```compute_53``` and ```sm_53``` are used for TX1 and ```compute_62``` and ```sm_62``` are used for TX2. More reading about mitigating JIT caching with NVCC:    
    https://devblogs.nvidia.com/cuda-pro-tip-understand-fat-binaries-jit-caching/

6. Change line 166 (in source of release 1.3.1) in ```3rdparty/mshadow/make/mshadow.mk``` to be
    ```
    MSHADOW_CFLAGS += -DMSHADOW_USE_PASCAL=1
    ```

7. Start compilation
    ```
    make
    ```
    or use all cores:
    ```
    make -j $(nproc)
    ```

8. Installation. There are a few ways to install the package. 
    * Install in the python virtual environment.   
        First, install ```virtualenv``` if you don't have it and create a virtual environment.
        ```
        sudo pip install virtualenv
        mkdir ~/mxnet-env
        cd ~/mxnet-env
        virtualenv .
        source bin/activate
        ```
        and install MXNet in your virtual environment:
        ```
        cd ~/mxnet-workspace/apache-mxnet-src-1.3.1.rc0-incubating/python
        pip install -e .
        ```
    * Install in the root.
        ```
        sudo pip install -e .
        ```

9. Verify the MXNet is installed. Note, python version used by ```pip``` and ```python``` must be same. You can also do ```pip2```, ```pip3``` in last steps. Now assume the MXNet was installed on python lib path cmd ```pip``` was used in last step, we can run:
    ```
    # This should print 1.3.1
    python -c 'import mxnet; print mxnet.__version__'
    ```
    You can also run tests in the folder ```~/mxnet-workspace/apache-mxnet-src-1.3.1.rc0-incubating/tests/python``` to verify your compilation and installation. You can use ```nose``` to do it.

10. Create distribution python package. MXNet provided script to generate wheel file to share your complied MXNet distributions.
    ```
    cd tools/pip_package
    python setup.py bdist_wheel
    ```

Pip installation
---
Python 2
```
pip2 install --user https://s3-us-west-2.amazonaws.com/github-litivecoder/mxnet/1.3.1/jetson_tx2/cuda_9.0/mxnet-1.3.1-cp27-cp27mu-linux_aarch64.whl
```

Python 3
```
pip3 install --user
https://s3-us-west-2.amazonaws.com/github-litivecoder/mxnet/1.3.1/jetson_tx2/cuda_9.0/mxnet-1.3.1-cp35-cp35m-linux_aarch64.whl
```