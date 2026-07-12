## Ubuntu CUDA CUDNN TRT 配置教程

#### by ZYL

##### 仅适用于x86带Nvidia独显的PC

**测试设备及环境：**
Ubuntu 20 & 22 LTS
Acer Swift X (RTX 3050 驱动版本535)
Mechrevo Aurora 15 (RTX 4060 驱动版本580)

**示例安装版本:**
CUDA 11.8.0
CUDNN 8.9.7.29-cuda11
TensorRT 8.5.1.7
TensorRT 10.7.0.23

### 安装CUDA

#### 一、准备：

在安装CUDA之前先安装好适合自己机器的独显驱动,CUDA安装包里集成的驱动版本往往较古老和存在兼容问题,安装好驱动后使用命令

```
nvidia-smi
```

可以查看显卡型号,驱动版本和当前驱动下支持的最高CUDA版本

至于怎么下载CUDA,由于NVIDIA并未给出整合的地址,默认只提供最新稳定版的CUDA,每个版本单独一个网页,所以只能根据要使用的版本搜索下载地址,这里给出CUDA11.8的地址:https://developer.nvidia.com/cuda-11-8-0-download-archive,选择Linux->x86_64->Ubuntu->20.04/22.04(系统版本无所谓,我们要下载的是run格式的)->runfile(local),选择好后给出的两条命令,第一条是下载run文件的,使用这条命令即可获得run格式的安装文件

#### 二、安装：

先给run文件赋权,否则无法运行:

```
sudo chmod a+x cuda_xxx.run
```

接着使用命令启动安装程序:

```
sudo ./cuda_xxx.run
```

终端会卡一会儿,因为要解压安装文件,等待即可,接着在弹出的安装选项选择里**注意不要安装CUDA自带的显卡驱动**（字符界面只能使用键盘操作,操作逻辑是含“[X]”的表示选中，“[ ]”表示不选中，使用空格键可以更改是否选中的状态）,其他不用管，继续执行安装程序即可

#### 三、配置环境变量：

在安装程序执行完成后,需要手动添加环境变量:

```
sudo gedit ~/.bashrc
```

在弹出的文档尾部添加:

```
export PATH=$PATH:/usr/local/cuda-11.8/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda-11.8/lib64
```

保存修改后刷新bashrc:

```
source ~/.bashrc
```

验证安装&版本查询：

```
nvcc -V
```

### 安装CUDNN：

#### 一、准备

NVIDIA官网提供了CUDNN的整合下载链接:https://developer.nvidia.com/rdp/cudnn-archive
CUDNN本质并不是独立软件,而是CUDA的扩展,所以要下载对应CUDA版本的安装包,这里依旧建议不要根据系统下载deb,而是直接选择下载Local Installer for Linux x86_64 (Tar),当然你下载的时候让你登录NVIDIA帐号,注册一个就行

#### 二、安装：

下载下来是一个tar.xz格式的压缩包,解压后进入解压出的文件夹内，打开终端，运行以下命令完成文件复制和赋权：

```
sudo cp include/cudnn*.h /usr/local/cuda-11.8/include
sudo cp lib/libcudnn* /usr/local/cuda-11.8/lib64
sudo chmod a+r /usr/local/cuda-11.8/include/cudnn*.h /usr/local/cuda-11.8/lib64/libcudnn*
```

版本验证:

```
cat /usr/local/cuda-11.8/include/cudnn_version.h | grep CUDNN_MAJOR -A 2
```

**注意:**在Ubuntu 22安装完cudnn后使用sudo ldconfig命令会出现几条libxxxx.so.8 is not a symbolic link的问题，此时需要进行手动软连接，命令：

```
sudo ln -sf /usr/local/cuda-11.8/targets/x86_64-linux/lib/libcudnn_adv_infer.so.8.9.7 /usr/local/cuda-11.8/targets/x86_64-linux/lib/libcudnn_adv_infer.so.8
sudo ln -sf /usr/local/cuda-11.8/targets/x86_64-linux/lib/libcudnn_ops_infer.so.8.9.7 /usr/local/cuda-11.8/targets/x86_64-linux/lib/libcudnn_ops_infer.so.8
sudo ln -sf /usr/local/cuda-11.8/targets/x86_64-linux/lib/libcudnn.so.8.9.7 /usr/local/cuda-11.8/targets/x86_64-linux/lib/libcudnn.so.8
sudo ln -sf /usr/local/cuda-11.8/targets/x86_64-linux/lib/libcudnn_cnn_infer.so.8.9.7 /usr/local/cuda-11.8/targets/x86_64-linux/lib/libcudnn_cnn_infer.so.8
sudo ln -sf /usr/local/cuda-11.8/targets/x86_64-linux/lib/libcudnn_adv_train.so.8.9.7 /usr/local/cuda-11.8/targets/x86_64-linux/lib/libcudnn_adv_train.so.8
sudo ln -sf /usr/local/cuda-11.8/targets/x86_64-linux/lib/libcudnn_ops_train.so.8.9.7 /usr/local/cuda-11.8/targets/x86_64-linux/lib/libcudnn_ops_train.so.8
sudo ln -sf /usr/local/cuda-11.8/targets/x86_64-linux/lib/libcudnn_cnn_train.so.8.9.7 /usr/local/cuda-11.8/targets/x86_64-linux/lib/libcudnn_cnn_train.so.8
```

### 安装TensorRT 8/10

#### 一、准备：

NVIDIA官网提供了TensorRT的整合下载链接:https://developer.nvidia.com/tensorrt
TensorRT依赖CUDA和CUDNN,所以和CUDNN一样也要下载对应CUDA版本的安装包,这里依旧建议不要根据系统下载deb版,而是直接选择下载TAR Package,当然下载的时候也需要,让你登录NVIDIA帐号

#### 二、安装：

网络上往往会让你解压后复制到系统内部,但是为了多版本共存和切换方便,直接将下载下来的压缩包解压到你固定放这些依赖的文件夹里就行,我直接放到我的home根目录下

#### 三、配置环境变量：

在安装程序执行完成后,也需要手动添加环境变量:

```
sudo gedit ~/.bashrc
```

在弹出的文档尾部添加:

```
export PATH=/home/ad/TensorRT-8.5.1.7/bin:$PATH（这行可以不加）
export LD_LIBRARY_PATH=$PATH:/home/ad/TensorRT-8.5.1.7/lib:$LD_LIBRARY_PATH
export LIBRARY_PATH=$PATH:/home/ad/TensorRT-8.5.1.7/lib::$LIBRARY_PATH
```

或

```
export PATH=$PATH:/home/ad/TensorRT-8.5.1.7/bin
export LD_LIBRARY_PATH=/home/ad/TensorRT-8.5.1.7/lib:$LD_LIBRARY_PATH
export LIBRARY_PATH=$LIBRARY_PATH:/home/ad/TensorRT-8.5.1.7/lib
```

注意这里的TensorRT版本和位置根据你实际存放的位置来定,不要无脑照抄我的位置

保存修改后刷新bashrc:

```
source ~/.bashrc
```

验证安装,进入tensorrt所在文件夹下的samples/sampleOnnxMNIST文件夹下打开终端：

```
make -j12
```

编译完成后回到tensorrt文件夹下的bin文件夹下打开终端

```
./sampleOnnxMNIST
```

终端出现一个字符拼成数字形状的输出即说明安装成功

**注意:**Ubuntu 22 安装 TensorRT 8 后运行该验证示例无法正常make，会提示几条如下类型的报错，无视即可，其他软件能调用就行,TensorRT 10 无此问题

```
CUDA_INSTALL_DIR variable is not specified, using /usr/local/cuda by default, use CUDA_INSTALL_DIR=<cuda_directory> to change
```

