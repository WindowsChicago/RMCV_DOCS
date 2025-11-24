运算平台：Jetson Orin NX/NANO
系统：Jetpack 6 （Ubuntu 22 ARM64）
1.使用fishros安装ros2-humble桌面版
2.安装完成后编译rm_vision_hs需要补充的组件
命令：

```
sudo apt install ros-humble-camera-info-manager
sudo apt install ros-humble-serial-driver
sudo apt install ros-humble-asio-cmake-module
sudo apt install ros-humble-xacro
```

3.安装海康相机驱动（deb版不再需要手动复制lib文件）
4.修复ch340驱动问题：由于ubuntu22自带一个名为brltty的组件会占用插入的串口，导致无法进行串口赋权，故需要先卸载该组件
命令：
sudo apt remove brltty
x86版的Ubuntu22卸载该组件后重新拔插串口或者重启即可正常赋权使用串口，但Jetson所运行的arm版ubuntu22默认不带ch34x系列的串口驱动，需要单独编译安装驱动

