### sp_vision_25运行环境部署问题及解决办法

------

**1.Eigen版本和Ceres的问题：**虽然ceres 2.2.0在Eigen3.3.7下也能完成编译，但是该视觉程序本身所需的Eigen版本为3.4以上版本（及配置文档里所给的apt版eigen），但是在已经安装Eigen3.3.7后再安装apt版Eigen会在编译时引起冲突，所以需要把编译版Eigen3.3.7卸载，命令:

```
//先把apt版也删除(主要是先尽量清理干净)
sudo apt remove libeigen3-dev

//查看版本号
pkg-config --modversion eigen3

//查看eigen3位置
sudo updatedb
locate eigen3

//删除相关文件
sudo rm -rf /usr/include/eigen3
sudo rm -rf /usr/lib/cmake/eigen3
sudo rm -rf /usr/local/include/eigen3
sudo rm -rf /usr/local/lib/cmake/eigen3
sudo rm -rf /usr/share/doc/libeigen3-dev 
sudo rm -rf /usr/local/share/pkgconfig/eigen3.pc /usr/share/pkgconfig/eigen3.pc /var/lib/dpkg/info/libeigen3-dev.list /var/lib/dpkg/info/libeigen3-dev.md5sums
sudo rm -rf /usr/local/lib/pkgconfig/eigen3.pc
sudo rm -rf /usr/local/share/eigen3

//刷新查看是否删除彻底
sudo updatedb
locate eigen3
pkg-config --modversion eigen3

//再安装所需的源码版Eigen3.4或apt版，命令：
sudo apt install libeigen3-dev
```

对于Ceres的问题，我之前给你们的都是源码版的ceres1.14,源码编译版的ceres可以互相替换，以最后一次make install的版本为准，所以只需要在安装完eigen后下载2.2.0的源码编译安装即可，命令：

```
cmake ..
make -j16
sudo make install
```

**2.spdlog和fmt的问题：**spdlog依赖于fmt，而该视觉程序需要fmt8.1.1（即apt下载的默认版本），但是我之前给你们的基本都是fmt10,会导致编译时报错，不过由于都是源码版，可以直接替换，直接下载源码版fmt8.1.1编译安装即可，apt版和源码版可以共存,命令：

```
cmake ..
make -j16
sudo make install
```

安装apt版和spdlog（1.9.2）和fmt（8.1.1），命令：

```
sudo apt install libfmt-dev
sudo apt install libspdlog-dev
```

又由于spdlog与fmt版本的强关联性，该程序所使用的spdlog版本是1.9.2（即apt下载的默认版本）,依赖fmt8.1.1,所以在apt安装时若之前没安装fmt会发现apt自动安装fmt8.1.1,并且由于spdlog程序本身内置了fmt，如果之前源码编译安装过fmt11,会导致该视觉程序编译时错误的选择fmt而导致出现fmt::11相关的报错（经测试，fmt10降级安装到8不会出现该问题），此时需要在尽管将fmt降级到8.1.1之后仍然无法解决问题，此时需要修改该视觉程序的cmakelists，在其任意位置（尽量靠前）添加：

```
add_definitions(-DSPDLOG_FMT_EXTERNAL)    # 强制 spdlog 用系统安装的 fmt
```

这样即可避免fmt方面的编译错误。

**3.OpenVINO方面的问题：**该程序可以在Jetson之类的arm64设备上编译运行，但是arm64的OpenVINO只能使用编译版本的，其文件所处位置与x64版不同，其安装位置一般是"/usr/local/runtime/",注意除了主cmakelists，tasks下的子cmakelists的内容注意也要修改。

**4.ROS方面的问题：**该程序对ROS的需求仅在io部分，其中ament_cmake、rclcpp、std_msgs、rosidl_typesupport_cpp均为ROS的标准包，使用apt即可下载补全，但是sp_msgs疑似sp队伍的自己的包且并未开源，故ROS部分无法完成编译,不过其ROS部分仅与导航通信相关，不参与编译亦可正常使用