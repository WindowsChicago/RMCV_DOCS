#### ROS1安装教程（仅支持Ubuntu 20）：

**推荐方法：使用fishros安装**
**1.**获取小鱼儿软件安装服务
命令：
wget http://fishros.com/install -O fishros

**2.**命令： 
. fishros
开始启动小鱼儿软件安装服务，选择ROS1即可，介于兼容问题，最好选择安装ros桌面版，安装桌面版期间大概率会出现包版本冲突安装失败，脚本会提示使用apitude进行部分软件包降级容易造成怪异的现象（如ubuntu桌面环境被降级导致图标变得复古，触摸板驱动无效，桌面图标无效，状态栏消失等），所以选择降级方案时务必选择在可以安装包的情况下尽可能少的降级的方案

------

#### ROS2 安装教程（支持Ubuntu20（foxy版本，不推荐使用）/22（humble版本））:

方法一：使用fishros安装
仍旧使用上面介绍的方法，选择ROS2即可，Ubuntu20选foxy，22只能选Humble，20可以自动使ROS1（noetic）和2（foxy）共存
方法二：使用apt安装（humble版教程链接地址：https://fishros.org/doc/ros2/humble/Installation/Ubuntu-Install-Debians.html）
