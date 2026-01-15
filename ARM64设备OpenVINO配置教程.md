## ARM64设备源码/非源码安装OpenVINO C++版教程

#### by ZYL

##### 适用于Jetson/树莓派/ARM64的Linux开发版

**测试环境：**

Jetson Orin Nano 8GB 

JetPack 6.2 MAXN Mode （Ubuntu 22 ARM64）

### 方案一：源码编译安装

现在OpenVINO官方对ARM设备的兼容性好了很多，安装好依赖后可以直接编译安装

#### 一、获取源码：

先拉取：

```
git clone https://github.com/openvinotoolkit/openvino.git
```

然后根据需要切换到目标版本

```
git checkout 2024.6.0  
```

也可以直接拉取时指定版本：

```
git clone -b 2025.3.0 https://github.com/openvinotoolkit/openvino.git
```

其他版本拉取源码的命令可以访问：https://www.intel.cn/content/www/cn/zh/developer/tools/openvino-toolkit/download.html?PACKAGE=OPENVINO_BASE&VERSION=v_2025_3_0&OP_SYSTEM=LINUX&DISTRIBUTION=GITHUB

选择版本->linux->git/gitee源

然后补充需要的内容子模块，会比较漫长，因为本质是把openvino的整个仓库拉下来，再从中检出需要的子模块，对比补充前后的区别主要是thirdparty文件夹变大到300m左右，以及.git文件夹变成3g多（这个文件夹下载完可以删除），并且不能挂梯子否则反而会报错：

```
git submodule init
git submodule update 
```

也可以替换为gitee源：

不过搞笑的是gitee源无梯子下载也很慢，得开梯子才能正常下载，不过比使用git init成功率高的多也快的多

#### 二、安装依赖：

需要：node.js、scons、npm、build-essential、glog、gflags（全部apt即可）

```
sudo apt -y install nodejs
sudo apt install npm
sudo apt install scons
sudo apt install build-essential 
sudo apt-get install libgoogle-glog-dev libgflags-dev
```

并且经实践，**x86设备上下载的3rd party不能用在arm64设备上**

#### 三、编译安装：

在完成版本切换并补充依赖后在源码文件夹下创建build

```
cmake .. 
```

(需要联网并挂梯子):这种编译方法会的默认安装位置不在/opt/intel/openvino202x下，部分openvino程序编译是会找不到安装好的ov，可以再单独安装非编译版

```
cmake -DCMAKE_INSTALL_PREFIX=/opt/intel/openvino_2024 ..
```

（也需要挂梯子）即可安装到opt（很多ov程序默认寻找位置）

```
make -j6
sudo make install
```

#### 四、加载环境：

安装完成后调用时需要在以下命令加载环境（不加载cmake会报错找不到ov）：

```
source /opt/intel/openvino_2024/setupvars.sh
```

也可以在bashrc里加上该命令即可免去每次开终端加载的问题

```
sudo gedit ~/.bashrc
```

粘贴环境加载语句：

```
source /opt/intel/openvino_2024/setupvars.sh
```

保存后刷新bashrc

```
source ~/.bashrc
```

**注意：**部分程序正常调用源码版openvino编译成功，但是运行是报错：

```
 error while loading shared libraries: /opt/intel/openvino_2024/runtime/3rdparty/tbb/lib/libtbb.so.12: file too short
```

 其实是是软链接没有成功建立，需要手动链接同目录下的同名文件
 进入/opt/intel/openvino_2024/runtime/3rdparty/tbb/lib/
 sudo rm libtbb.so.12
 sudo ln -s libtbb.so.12.13 libtbb.so.12

### 方案二：非源码版安装：

#### 一、创建文件夹

先在opt文件夹下创建intel文件夹

```
sudo mkdir /opt/intel
```

#### 二、下载预编译包：

先访问仓库

[storage.openvinotoolkit.org](https://storage.openvinotoolkit.org/repositories/openvino/packages/)

下载带arm64（不是armf）字样debian/ubuntu（通用）的包，例如：l_openvino_toolkit_ubuntu20_2024.6.0.17404.4c0f47d2335_arm64.tgz

#### 三、解压安装

解压文件释放出文件夹，在该文件夹所在父目录（比如下载文件夹）打开终端，执行命令：

```
sudo mv l_openvino_toolkit_ubuntu20_2024.6.0.17404.4c0f47d2335_arm64 /opt/intel/openvino_2024.6.0
```

该命令将会该文件夹移动到opt/intel下并改名为openvino_2024.6.0
在openvino_2024.6.0下打开终端，执行依赖安装脚本：

```
sudo -E ./install_dependencies/install_openvino_dependencies.sh
```

如果需要的是python版本的就运行下面命令（可选）

```
python3 -m pip install -r ./python/requirements.txt
```

接着在opt/intel文件夹下创建文件夹链接

```
sudo ln -s openvino_2024.6.0 openvino_2024
```

#### 四、加载环境：

安装完成后调用时需要在以下命令加载环境（不加载cmake会报错找不到ov）：

```
source /opt/intel/openvino_2024/setupvars.sh
```

也可以在bashrc里加上该命令即可免去每次开终端加载的问题

```
sudo gedit ~/.bashrc
```

粘贴环境加载语句：

```
source /opt/intel/openvino_2024/setupvars.sh
```

保存后刷新bashrc：

```
source ~/.bashrc
```

使用非源码版可以替代源码编译版，节省时间





