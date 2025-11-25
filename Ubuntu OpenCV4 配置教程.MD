# Ubuntu OpenCV4 with CUDA VSCode CMake C++ 环境配置教程

**by ZYL**

## 版本选择

Ubuntu 20.04 推荐 OpenCV 4.5.5 /4.8.0
Ubuntu 22.04 推荐 OpenCV 4.8.0 /4.10.0
CUDA 11/12 推荐 OpenCV 4.10.0/4.8.0+contrib
Jetpack 5 官方自带为4.5.4不带cuda增强
Jetpack 6 官方自带为4.8.0不带cuda增强，经实测可以编译4.10.0带cuda增强，4.8.0经测试无法安装带cuda增强的版本

## 安装步骤

#### 1.安装依赖

##### 先安装基础依赖：

```
sudo apt install -y g++
sudo apt install -y make
sudo apt install vim
```

```
sudo apt-get install build-essential libgtk-3-dev libavcodec-dev libavformat-dev libjpeg-dev libswscale-dev libtiff5-dev  libgtk2.0-dev
```

Ubuntu 22可能存在libgtk2.0-dev不能正常安装的问题，需要先安装aptitude来降级安装，命令：

```
sudo apt-get install aptitude
sudo aptitude install libgtk2.0-dev
```

选择可以没有gtk后面没有（未安装）字样的方案输入y，并且这样安装完gtk后4.5.5也无法使用gtk，只能安装4.8.0/4.10.0，执行完后再执行上面的命令，安装其他必需依赖

##### 安装cmake：

```
sudo apt install -y cmake
```

使用apt安装时ubuntu 20一般是3.16,ubuntu22一般是3.22

也可以编译安装 某个特定版本（**不建议使用**，apt的已经够了）：
1.先安装openssl：

```
sudo apt-get install libssl-dev
```

2.下载源码的tar.gz包，解压，进入目录打开终端，输入：

```
./configure
```

3.无报错则接着：

```
make -j$(nproc)
```

4.编译完成后：

```
sudo make install
```

5.最后验证安装：

```
cmake --version
```

##### 其他可选依赖安装命令：

```
sudo apt install libxvidcore-dev libx264-dev
sudo apt install libatlas-base-dev gfortran 
sudo apt install ffmpeg
sudo apt install libgstreamer-plugins-base1.0-dev libgstreamer1.0-dev
sudo apt install libgstreamer-plugins-base1.0-dev libgstreamer1.0-dev（streamer支持）
sudo apt install python3-dev python3-numpy（python3支持（首次安装了python的库，但make报错了，之后删了这两个库，若不使用python，建议不安装））
```

如果需要安装带cuda增强的版本，则还需要安装nvidia显卡驱动、cuda和cudnn，注意OpenCV从4.5开始才支持CUDA12,安装方法见我的另一篇文章

##### 特别的，对于Jetson开发板，可以使用SDK manger安装OpenCV，但是如果需要其他版本或者需要启用cuda增强，即使使用了sdk manager安装了opencv也需要走上面的步骤补充依赖（主要是需要libgtk2.0-dev，在jetpack6下也能正常安装无需aptitude）

#### 2.解压源码

解压opencv源码文件到一个文件夹，在该文件夹下新建一个叫build的文件夹，在此文件夹下打开终端

如果需要扩展组件，将扩展组件源码解压后复制该文件夹到opencv源码根目录下或者某个位置（一会儿需要用）

#### 3.预处理

##### build文件夹下打开终端后，在以下的命令里根据你的需要选择一种执行,部分命令需要联网并翻墙后才能正常运行：

启用pkg-config，不安装扩展组件：

```
sudo cmake -D CMAKE_BUILD_TYPE=Release -D OPENCV_GENERATE_PKGCONFIG=YES ..
```

启用pkg-config，不安装扩展组件，已安装ffmpeg：

```
sudo cmake -D CMAKE_BUILD_TYPE=Release -D OPENCV_GENERATE_PKGCONFIG=YES -D WITH_FFMPEG=ON ..
```

不启用pkg-config安装扩展组件：

```
sudo cmake -D CMAKE_INSTALL_PREFIX=/usr/local -D CMAKE_BUILD_TYPE=Release -D OPENCV_EXTRA_MODULES_PATH=../opencv_contrib/modules ..
```

启用pkg-config并安装扩展组件：

```
sudo cmake -D CMAKE_BUILD_TYPE=Release -D OPENCV_GENERATE_PKGCONFIG=ON -D CMAKE_INSTALL_PREFIX=/usr/local .. -D OPENCV_EXTRA_MODULES_PATH=../opencv_contrib/modules ..
```

不启用pkg-config，安装扩展组建并使用启用cuda增强（部分TensorRT10推理框架需要，带需要先安装好cuda和cudnn）：

```
cmake \
-D CUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-11.8 \
-D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=$(python3 -c "import sys; print(sys.prefix)")  \
-D INSTALL_PYTHON_EXAMPLES=ON  \
-D INSTALL_C_EXAMPLES=OFF  \
-D OPENCV_ENABLE_NONFREE=ON  \
-D WITH_CUDA=ON \
-D WITH_CUDNN=ON  \
-D ENABLE_FAST_MATH=1  \
-D CUDA_FAST_MATH=1  \
-D CUDA_ARCH_BIN=8.6  \
-D WITH_CUBLAS=0 \
-D OPENCV_EXTRA_MODULES_PATH=/home/ad/opencv-4.10.0/opencv_contrib-4.10.0/modules \
-D BUILD_EXAMPLES=ON \
-D PYTHON_EXECUTABLE=$(which python3) ..
```

启用pkg-config，安装扩展组建并使用启用cuda增强：

```
cmake \
-D CUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-11.8 \
-D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_INSTALL_PREFIX=$(python3 -c "import sys; print(sys.prefix)")  \
-D INSTALL_PYTHON_EXAMPLES=ON  \
-D INSTALL_C_EXAMPLES=OFF  \
-D OPENCV_ENABLE_NONFREE=ON  \
-D WITH_CUDA=ON \
-D WITH_CUDNN=ON  \
-D ENABLE_FAST_MATH=1  \
-D CUDA_FAST_MATH=1  \
-D CUDA_ARCH_BIN=8.6  \
-D WITH_CUBLAS=0 \
-D OPENCV_EXTRA_MODULES_PATH=/home/ad/opencv-4.10.0/opencv_contrib-4.10.0/modules \
-D BUILD_EXAMPLES=ON \
-D OPENCV_GENERATE_PKGCONFIG=YES \
-D PYTHON_EXECUTABLE=$(which python3) ..
```

##### 命令说明：

-D OPENCV_GENERATE_PKGCONFIG=YES：OpenCV4以上默认不使用pkg-config，该编译选项开启生成opencv4.pc文件，支持pkg-config功能

-D OPENCV_EXTRA_MODULES_PATH=/home/ad/opencv-4.10.0/opencv_contrib-4.10.0/modules：启用扩展组件并指明扩展组件位置（位置为第二部你解压扩展组件源码的位置）

-D CUDA_TOOLKIT_ROOT_DIR=/usr/local/cuda-11.8 ：指定cuda位置，可以不写版本

-D CUDA_ARCH_BIN=8.6 ：指定显卡cuda arch数值，不同架构的显卡其数值不同，具体数值查看https://blog.csdn.net/u010087338/article/details/136580245

-D WITH_CUBLAS=0 ：不启用cublas，有些文章的命令里会置1或true来启用，但是很多时候安装cuda时不会安装cublas，启用会导致编译报错，所以设置为不启用

##### 启用cuda增强模式时编译可能遇到的问题：

```
cc: fatal error: cannot execute ‘cc1plus’: execvp: 没有那个文件或目录
compilation terminated.
nvcc fatal   : Failed to preprocess host compiler properties.
```

这个报错乍一看好像是cuda的问题，其实并不是，而是gcc和g++版本对不上导致的,我之前的文章里写到有些时候Ubuntu22安装NVIDIA显卡驱动需要gcc12,但是那时候升级gcc时，我没有将gcc降回gcc11,导致gcc和g++版本对不上，并且也不能将g++更新到12，否则会有如下报错：

```
CMake Error at cuda_compile_1_generated_gpu_mat.cu.o.RELEASE.cmake:222 (message):
  Error generating
  /home/ad/opencv-4.10.0/build/modules/core/CMakeFiles/cuda_compile_1.dir/src/cuda/./cuda_compile_1_generated_gpu_mat.cu.o
```

因为cuda 11.8的nvcc不支持使用gcc12和g++12进行cuda程序的编译,所以g++不用动还是默认的11,把gcc降下去即可，查看当前默认gcc/g++版本命令：

```
gcc -v
g++ -v
```

发现g++是11.4而gcc是12.3，所以需要修改gcc默认版本，先确保你的设备有gcc11，没有则安装即可：

```
sudo apt install gcc-11
```

然后设置优先级数值（之前按我的文章安装nvidia驱动并已经安装gcc12的大概率gcc11还在），这个命令里的“100”的含义见下面的附录单独解释

```
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 100
```

设置当前gcc备用选组：

```
sudo update-alternatives --config gcc
```

查看表单里gcc11选项前是否有*，有*的为默认版本，不是则按提示修改gcc11为默认版本，之后再：

```
gcc -v
g++ -v
```

版本都是11.4则设置成功

另外我还发现如果使用gcc12安装驱动，而且在没有切回gcc11的情况下安装cuda，cmake是在最后可能有如下警告：

```
CMake Warning at samples/samples_utils.cmake:10 (add_executable):
  Cannot generate a safe runtime search path for target
  example_opencl_opencl-opencv-interop because files in some directories may
  conflict with libraries in implicit directories:

runtime library [libOpenCL.so.1] in /usr/lib/x86_64-linux-gnu may be hidden by files in:
  /usr/local/cuda/lib64

  Some of these libraries may not be found correctly.
Call Stack (most recent call first):
  samples/opencl/CMakeLists.txt:33 (ocv_define_sample)
```

出现这个警告后再切换回gcc11后，重新cmake仍然会存在，不过不会引起编译出错，想没有这个警告可以重新在gcc11下安装一遍cuda

#### 4.编译&安装

##### 编译命令：

```
sudo make -j8
```

其中数字为编译占用cpu线程数，可以改变，比如4/16等，注意盯着点运行内存占用，线程数太多编译时可能吃满内存而导致系统卡死
走完进度条便编译完成

##### 安装命令：

```
sudo make install
```

安装编译完成的组件

#### 5.配置pkg-config环境

先打开文件，点击其他位置，点击计算机，进入系统根目录，查看/usr/local/lib/pkgconfig/下是否存在opencv4.pc文件，若不存在则到opencv安装包解压目录的build/unix-install下寻找并复制到pkgconfig下
接着命令：

```
sudo gedit /etc/profile.d/pkgconfig.sh
```

等待一会儿会弹出文档编辑工具打开该文件，在文件中写入：

```
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH
```

保存退出文档编辑工具之后
命令：

```
source /etc/profile
```

使设置生效
命令：

```
pkg-config --libs opencv4
```

返回一串含opencv字样的文字即安装成功

#### 6.配置动态库环境

命令：

```
sudo vim /etc/ld.so.conf.d/opencv4.conf
```

会使用Vim打开该文件，按insert进入插入模式，在文档中写入：

```
/usr/local/lib
```

按esc退出插入模式，输入“：wq”保存并退出
命令：

```
sudo ldconfig
```

使配置生效

#### 7.更新 PKG_CONFIG_PATH（用于Ubuntu 22&OpenCV 4.8.0,且可选）

命令：

```
sudo gedit /etc/bash.bashrc
```

在文本末添加：

```
PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/lib/pkgconfig
export PKG_CONFIG_PATH
```

保存后，命令：

```
source /etc/bash.bashrc
```

接着安装mlocate来执行updatedb命令：

```
sudo apt install mlocate
sudo updatedb
```

#### 8.测试

cd 到/opencv/samples/cpp/example_cmake目录下，执行如下命令：

```
cmake .
make
./opencv_example
```

会弹出含opencv字样的框说明安装成功

### 以下内容为可选步骤

#### 9.配置VSCode C++调用opencv功能（可选，非必须）

VSCode安装微软的c/c++扩展组件，打开一个空文件夹，新建一个main.cpp,写入如下代码:

```
#include <opencv2/opencv.hpp>

using namespace std;
using namespace cv;

int main(int argc, char* argv[]) {
    const char* imagename = "test.jpg";//此处为你自己的图片路径

    //从文件中读入图像
    Mat img = imread(imagename, 1);

    //如果读入图像失败
    if (img.empty()) {
        fprintf(stderr, "Can not load image %s\n", imagename);
        return -1;
    }
    //显示图像
    imshow("image", img);

    //此函数等待按键，按键盘任意键就返回
    waitKey();
    return 0;

}
```

安装c/c++扩展后会发现打开cpp文件左上角有一个三角形的调试按钮，点击会弹出编译器选择，选择“c++：gcc生成和调试文件”会发现报错后在文件夹栏出现“task.json”文件点击打开，替换写入如下内容：

```
{
    "tasks": [
        {
            "type": "cppbuild",
            "label": "C/C++: g++ build active file",  /* 与launch.json文件里的preLaunchTask的内容保持一致 */
            "command": "/usr/bin/g++",
            "args": [
                "-std=c++11",
                "-g",
                //"${file}",   /* 编译单个文件 */
                "${fileDirname}/*.cpp",  /* 编译多个文件 */
                "-o",
                "${fileDirname}/${fileBasenameNoExtension}",  /* 输出文件路径 */

                /* 项目所需的头文件路径 */
                "-I","${workspaceFolder}/",
                "-I","/usr/local/include/",
                "-I","/usr/local/include/opencv4/",
                "-I","/usr/local/include/opencv4/opencv2",

                /* 项目所需的库文件路径 */
                "-L", "/usr/local/lib",

                /* OpenCV的lib库 */
                "/usr/local/lib/libopencv_*",

            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "Task generated by Debugger."
        }
    ],
    "version": "2.0.0"

}
```

保存后，在主界面选择“运行”->“添加配置”发现生成“launch.json”文件，替换原内容，写入如下内容：

```
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "name": "g++ - Build and debug active file",
            "type": "cppdbg",
            "request": "launch",
            "program": "${fileDirname}/${fileBasenameNoExtension}",  //程序文件路径
            "args": [],  //程序运行需传入的参数
            "stopAtEntry": false,
            "cwd": "${fileDirname}",
            "environment": [],
            "externalConsole": true,   //运行时是否显示控制台窗口
            "MIMode": "gdb",
            "setupCommands": [
                {
                    "description": "Enable pretty-printing for gdb",
                    "text": "-enable-pretty-printing",
                    "ignoreFailures": true
                }
            ],
            "preLaunchTask": "C/C++: g++ build active file",
            "miDebuggerPath": "/usr/bin/gdb"
        }
    ]
}
```

保存，在主界面按“ctrl+shift+p”调出命令面板，搜索c/c++，出现“c/c++编辑配置UI/JSON”选json版便发现出现了“c_cpp_properties.json”文件，替换写入如下内容：

```
{
    "configurations": [
        {
            "name": "Linux",
            "includePath": [
                "${workspaceFolder}/**",


                "/usr/local/include/opencv4"
            ],
            "defines": [],
            "compilerPath": "/usr/bin/gcc",
            "cStandard": "gnu11",
            "cppStandard": "gnu++14",
            "intelliSenseMode": "linux-gcc-x64"
        }
    ],
    "version": 4

}
```

保存后会发现主cpp文件的报错消失，可以使用VSCode运行调试opencv程序

#### 10.CMake使用opencv配置

```
CMakelist内容如下：

# cmake needs this line

cmake_minimum_required(VERSION 3.1)

# Define project name

project(ImageShow)

# Find OpenCV, you may need to set OpenCV_DIR variable

# to the absolute path to the directory containing OpenCVConfig.cmake file

# via the command line or GUI

find_package(OpenCV REQUIRED)

# Enable C++11

set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED TRUE)

# Declare the executable target built from your sources

add_executable(ImageShow main.cpp)

# Link your application with OpenCV libraries

target_link_libraries(ImageShow PRIVATE ${OpenCV_LIBS})
```



### 附录：gcc/g++多版本切换与共存详解：

Ubuntu不同版本有其默认的gcc/g++，比如20 LTS默认是gcc/g++9,22 LTS默认是gcc/g++11,也可以单独下载其他版本的gcc/g++
命令：

```
sudo apt install gcc-9 g++-9 gcc-10 g++-10 gcc-11 g++-11 g++-12 gcc-12
```

但是apt下载其他版本下来后会发现仍然会使用默认的版本，因为Ubuntu使用候选项表的方式管理gcc/g++包，每个版本的包需要指派一个“优先级”数值，在未手动设置默认版本的情况下，优先级数值最大的为默认版本，查询候选表会发现这个包有两行，有一行的状态为“自动模式”，其他的状态为“手动模式”，只有一个自带版本的gcc/g++时，没有候选表，使用刚才的apt命令下载其他版本后也不会自动生成候选表和优先级，而是要手动设置创建候选表并将目标版本写入候选表并设置优先级：
命令：

```
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 100 --slave /usr/bin/g++ g++ /usr/bin/g++-12 --slave /usr/bin/gcov gcov /usr/bin/gcov-12
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 80 --slave /usr/bin/g++ g++ /usr/bin/g++-11 --slave /usr/bin/gcov gcov /usr/bin/gcov-11
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-10 60 --slave /usr/bin/g++ g++ /usr/bin/g++-10 --slave /usr/bin/gcov gcov /usr/bin/gcov-10
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 40 --slave /usr/bin/g++ g++ /usr/bin/g++-9 --slave /usr/bin/gcov gcov /usr/bin/gcov-9
```

其中“100”，“80”，“60”，“40”的数值可以自定义任意数值，“--slave”前的包为“主包”，后的为“从包”，使用这种方法可以绑定gcc和g++的版本使其一致，但是如果出现以下这种报错：

```
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/bin 4.9 60 --slave /usr/bin/g g /usr/bin/g-4.9
update-alternatives: error: alternative g cannot be a slave of gcc: it is a primary choice
```

一般是某个gcc版本在当前系统没有对应的g++版本，手动下载缺失的gcc/g++即可，也可以单独分开设置gcc/g++的候选表和优先级：

单独设置gcc优先级：

```
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-11 100
```

单独设置g++优先级：

```
sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-11 100
```

如果有提示：

```
update-alternatives: 警告: 链接组 gcc 已损坏，故强制重新安装候选项 /usr/bin/gcc-11
```

说明你的gcc11的优先级数值和某个已经存在的gcc优先级数值相同了，此时再赋个其他值就行

如果想要在不改变优先级数值的情况下，手动设置默认gcc/g++版本（手动设置后根据优先级排序的“自动模式”将失效，而以手动设置的默认版本为准）可以使用 update-alternatives 命令重新配置gcc/g++默认版本：

```
sudo update-alternatives --config gcc
sudo update-alternatives --config g++
```

上述命令将显示安装的 GCC 版本列表，只需输入对应的版本编号，然后按「回车」键即可将其设置为默认版本。命令会自动创建指向 GCC 和 G ++ 特定版本的符号链接

如果运行命令显示报错： 

```
update-alternatives：错误：无gcc的候选项
```

说明目前系统只有一个版本的gcc或者没有使用上面的优先级设置命令加入备选组并设置优先级数值，先按上面的方法将已经下载的gcc/g++设置个优先级数值加入候选表中。
