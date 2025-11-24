## Ubuntu OpenCV4 CUDA VSCode CMake C++ 环境配置教程

##### Ubuntu 20.04推荐OpenCV 4.5.5 /4.8.0

##### Ubuntu 22.04推荐OpenCV 4.8.0 /4.10.0

CUDA 11/12 推荐OpenCV 4.10.0+contrib，4.8.0大概率

Jetpack 5 官方为4.5.4不带cuda增强

Jetpack 6 官方为4.8.0不带cuda增强，经实测可以使用4.10.0+cuda

##### 1.安装依赖

命令：

```
sudo apt install -y g++
sudo apt install -y make
sudo apt install vim
```

Ubuntu 20:

```
sudo apt-get install build-essential libgtk-3-dev libavcodec-dev libavformat-dev libjpeg-dev libswscale-dev libtiff5-dev  libgtk2.0-dev
```

Ubuntu 22:可能存在libgtk2.0-dev不能正常安装的问题，需要先安装aptitude来降级安装，命令：

```
sudo apt-get install aptitude
sudo aptitude install libgtk2.0-dev
```

选择可以没有gtk后面没有（未安装）字样的方案输入y，并且这样安装完gtk后4.5.5也无法使用gtk，只能安装4.8.0，执行完后再执行上面的命令，安装上面的其他必需依赖

Cmake：sudo apt install -y cmake（旧版,ubuntu 20一般是3.16,ubuntu22一般是3.22）
               手动安装新版（不建议使用，旧版已经够了）方法：
               		1.先安装openssl：sudo apt-get install libssl-dev
                        2.下载源码的tar.gz包，解压，进入目录打开终端，输入：./configure
                        3.无报错则接着：make -j$(nproc)
                        4.编译完成后：sudo make install
                        5.最后验证安装：cmake --version
                        
其他可选依赖安装命令：

```
sudo apt install libxvidcore-dev libx264-dev
sudo apt install libatlas-base-dev gfortran 
sudo apt install ffmpeg
```

安装python3支持（首次安装了python的库，但make报错了，之后删了这两个库，若不使用python，建议不安装）:

```
sudo apt install python3-dev python3-numpy
```

安装streamer支持:

```
sudo apt install libgstreamer-plugins-base1.0-dev libgstreamer1.0-dev
```

可选的依赖

```
sudo apt install libpng-dev libopenexr-dev libtiff-dev libwebp-dev
```

##### 2.解压opencv压缩文件到一个文件夹，在该文件夹下运行终端

命令：

```
mkdir build
cd build
```

##### 3.预处理

命令：
启用pkg-config，不安装扩展组件（建议）：

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

命令中“opencv_contrib”为扩展组件文件夹名，将其放置在opencv本体文件夹下

说明：
-D OPENCV_GENERATE_PKGCONFIG=YES
OpenCV4以上默认不使用pkg-config，该编译选项开启生成opencv4.pc文件，支持pkg-config功能

##### 4.编译&安装

命令：sudo make -j8
其中数字为编译占用cpu线程数，可以改变，比如4/16等
走完进度条便编译完成
命令：
sudo make install
安装编译完成的组件

##### 5.配置pkg-config环境

先打开文件，点击其他位置，点击计算机，进入系统根目录，查看/usr/local/lib/pkgconfig/下是否存在opencv4.pc文件，若不存在则到opencv安装包解压目录的build/unix-install下寻找并复制到pkgconfig下
接着命令：
sudo gedit /etc/profile.d/pkgconfig.sh
等待一会儿会弹出文档编辑工具打开该文件，在文件中写入：
export PKG_CONFIG_PATH=/usr/local/lib/pkgconfig:$PKG_CONFIG_PATH
保存退出文档编辑工具之后
命令：
source /etc/profile
使设置生效
命令：
pkg-config --libs opencv4
返回一串含opencv字样的文字即安装成功

##### 6.配置动态库环境

命令：
sudo vim /etc/ld.so.conf.d/opencv4.conf
会使用Vim打开该文件，按insert进入插入模式，在文档中写入：
/usr/local/lib
按esc退出插入模式，输入“：wq”保存并退出
命令：
sudo ldconfig
使配置生效

##### 7.更新 PKG_CONFIG_PATH（用于Ubuntu 22&OpenCV 4.8.0,且可选）

命令：
sudo gedit /etc/bash.bashrc
在文本末添加：
PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/lib/pkgconfig
export PKG_CONFIG_PATH
保存后，命令：
source /etc/bash.bashrc
接着安装mlocate来执行updatedb命令：
sudo apt install mlocate
sudo updatedb

##### 8.测试

cd 到/opencv/samples/cpp/example_cmake目录下，执行如下命令：
cmake .
make
./opencv_example
会弹出含opencv字样的框说明安装成功

##### 9.配置VSCode C++调用opencv功能（可选，非必须）

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

##### 10.CMake使用opencv配置

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

