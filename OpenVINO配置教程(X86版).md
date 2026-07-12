## X86设备安装OpenVINO C++版教程

#### by ZYL

##### 适用于X86架构的PC,6代以上的Intel处理器在安装驱动后支持调用核显GPU加速(驱动安装方法见附录),其它x86处理器仅支持纯CPU推理

### 方案一：APT安装

Intel官网给Ubuntu默认提供的就是这个方法,参考链接:https://www.intel.cn/content/www/cn/zh/developer/tools/openvino-toolkit/download.html?PACKAGE=OPENVINO_BASE&VERSION=v_2026_1_0&OP_SYSTEM=LINUX&DISTRIBUTION=APT,缺点是官网只提供最新的两个版本,但是其前几步是全版本通用的

#### 1.下载 GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB

命令：

```
wget https://apt.repos.intel.com/intel-gpg-keys/GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB
```

#### 2.将此密钥添加到系统密钥环

命令：

```
sudo apt-key add GPG-PUB-KEY-INTEL-SW-PRODUCTS.PUB
```

#### 3.通过以下命令添加存储库

命令:

```
# Ubuntu 20
echo "deb https://apt.repos.intel.com/openvino ubuntu20 main" | sudo tee /etc/apt/sources.list.d/intel-openvino.list
# Ubuntu 22
echo "deb https://apt.repos.intel.com/openvino ubuntu22 main" | sudo tee /etc/apt/sources.list.d/intel-openvino.list
# Ubuntu 24
echo "deb https://apt.repos.intel.com/openvino ubuntu24 main" | sudo tee /etc/apt/sources.list.d/intel-openvino.list
```

#### 4.使用更新命令更新软件包列表

命令:

```
sudo apt update
```

#### 5.验证 APT 存储库的设置是否正确。使用 apt-cache 命令查看包含所有可用 OpenVINO 程序包和组件的列表

命令:

```
apt-cache search openvino
```

#### 6.完成上面几步之后可以自由安装存在的OpenVINO版本

命令:

```
# OpenVINO 2024.6.0
sudo apt install openvino-2024.6.0

#或其他版本,比如2026.1.0
sudo apt install openvino-2026.1.0
```

### 方案二：非源码版安装：

#### 1.创建文件夹

先在opt文件夹下创建intel文件夹

```
sudo mkdir /opt/intel
```

#### 2.下载预编译包：

先访问仓库

[storage.openvinotoolkit.org](https://storage.openvinotoolkit.org/repositories/openvino/packages/)

下载x86_64字样debian/ubuntu（通用）的包，例如：l_openvino_toolkit_ubuntu22_2024.6.0.17404.4c0f47d2335_x86_64.tgz

#### 3.解压安装

解压文件释放出文件夹，在该文件夹所在父目录（比如下载文件夹）打开终端，执行命令：

```
sudo mv l_openvino_toolkit_ubuntu22_2024.6.0.17404.4c0f47d2335_x86_64 /opt/intel/openvino_2024.6.0
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

#### 4.加载环境：

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

### 附录：Intel处理器安装Ubuntu核显驱动

#### 1.对于7-10代处理器(UHD架构系列核显)

如果是intel Core 7-10代处理器的核显本质都是一样的,架构较老,一般ubuntu可以直接驱动核显，无需单独下载驱动，但是Ubuntu不默认安装OpenCL，OpenVINO无法直接进行GPU加速，所以需要先安装OpenCL
命令：

```
sudo apt install intel-opencl-icd
```

可以下载intel的显卡信息监测工具监视显卡占用：
安装：

```
sudo apt-get install intel-gpu-tools
```

启动：

```
sudo intel_gpu_top
```

当然如果还是无法调用GPU就检查是否安装了intel-ocloc intel-gmmlib intel-igc-core intel-igc-opencl,以及依赖包libnumal和ocl-icd-libopencl1

#### 2.对于11-13代处理器(XE架构系列核显)

命令:

```
mkdir neo  
cd neo  

wget https://github.com/intel/intel-graphics-compiler/releases/download/igc-1.0.13463.18/intel-igc-core_1.0.13463.18_amd64.deb  
wget https://github.com/intel/intel-graphics-compiler/releases/download/igc-1.0.13463.18/intel-igc-opencl_1.0.13463.18_amd64.deb  
wget https://github.com/intel/compute-runtime/releases/download/23.09.25812.14/intel-level-zero-gpu-dbgsym_1.3.25812.14_amd64.ddeb  
wget https://github.com/intel/compute-runtime/releases/download/23.09.25812.14/intel-level-zero-gpu_1.3.25812.14_amd64.deb  
wget https://github.com/intel/compute-runtime/releases/download/23.09.25812.14/intel-opencl-icd-dbgsym_23.09.25812.14_amd64.ddeb  
wget https://github.com/intel/compute-runtime/releases/download/23.09.25812.14/intel-opencl-icd_23.09.25812.14_amd64.deb  
wget https://github.com/intel/compute-runtime/releases/download/23.09.25812.14/libigdgmm12_22.3.0_amd64.deb  
wget https://github.com/intel/compute-runtime/releases/download/23.09.25812.14/ww09.sum  

sha256sum -c ww09.sum  
sudo dpkg -i *.deb  
```

