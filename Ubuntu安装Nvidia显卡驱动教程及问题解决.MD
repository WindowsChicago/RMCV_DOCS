# Ubuntu安装20/30/40/50系显卡驱动教程及部分问题解决

1.nvidia官网下载对应显卡的.run文件，复制到主文件夹并修改成便于输入的文件名，如“4050.run”。
2.将默认图形化软件修改为lightdm
命令：sudo apt install lightdm
会出现选择图形化默认软件的选项框，选lightdm，完成后输入
命令：sudo dpkg-reconfigure lightdm
验证是否切换，之后重启会发现登陆界面的变化。注：根据实践，20系显卡完成这步可以直接使用自带的535驱动，无需手动安装公版驱动
3.安装vim，gcc，make
命令：

```
sudo apt install vim
sudo apt install gcc
sudo apt install make
```

4.禁用默认开源显卡驱动
命令：sudo vim /etc/modprobe.d/blacklist.conf
在vim打开文档的界面按insert进入插入编辑模式
在文档最后加上如下两行：

```
blacklist nouveau
options nouveau modeset=0
```

按esc退出插入模式，键入”:“”wq“实现保存修改并退出
重启，终端验证是否禁用成功
命令：

```
lsmod | grep nouveau
```

无任何反馈则禁用成功。
5.关闭桌面环境
命令：

```
sudo service lightdm stop
```

桌面关闭，左上角会出现光标，输入ctrl+alt+f1，输入用户名和密码，登入命令行模式，有必要的话使用root模式
命令：

```
sudo password root
```

输入密码便可获得root权限
6.安装驱动
命令：

```
sudo chmod a+x 4050.run
```

可赋予主文件夹下的4050.run文件运行能力
命令：

```
sudo ./4050.run -no-x-check -no-nouveau-check (-no-opengl-files可选)
```

接着会进入驱动安装界面，注意不安装32位组件库，xconfig选择是，中间的报错先无视，若显示完成安装，键入ctrl+alt+delete重启。
7.完成安装
30系和50系无xconfig的提示，重启便完成了驱动安装，40系会出问题，解决方法如下：
先进bios关闭安全启动，再次进入桌面发现驱动并未完成安装，nvidia-smi命令报错
命令：

```
sudo apt-get install dkms
```

安装dkms
命令：

```
ls -l /usr/src
```

发现有nvidia字样
命令：

```
sudo dkms install -m nvidia -v 555.58.02(对应之前的驱动版本号)
```

若未关闭安全启动，会发现弹出关于安全启动的图形界面，强制关闭终端，重启关闭安全启动即可，重启后无需重复操作，便可发现nvidia-smi可以正常反馈信息，驱动安装完成



8.最近重新安装Ubuntu22发现无法安装apt版驱动，错误提示如下：

```
Loading new nvidia-550.40.07 DKMS files…
Building for 6.5.0-15-generic
Building for architecture x86_64
Building initial module for 6.5.0-15-generic
ERROR: Cannot create report: [Errno 17] File exists: ‘/var/crash/nvidia-dkms-550.0.crash’
Error! Bad return status for module build on kernel: 6.5.0-15-generic (x86_64)
Consult /var/lib/dkms/nvidia/550.40.07/build/make.log for more information.
dpkg: error processing package nvidia-dkms-550 (–configure):
installed nvidia-dkms-550 package post-installation script subprocess returned error exit status 10
Setting up libnvidia-encode-550:amd64 (550.40.07-0ubuntu0~gpu23.10.1) …
Setting up libnvidia-encode-550:i386 (550.40.07-0ubuntu0~gpu23.10.1) …
dpkg: dependency problems prevent configuration of nvidia-driver-550:
nvidia-driver-550 depends on nvidia-dkms-550 (<= 550.40.07-1); however:
Package nvidia-dkms-550 is not configured yet.
nvidia-driver-550 depends on nvidia-dkms-550 (>= 550.40.07); however:
Package nvidia-dkms-550 is not configured yet.
dpkg: error processing package nvidia-driver-550 (–configure):
dependency problems - leaving unconfigured
```

也无法安装run版驱动，在编译时出错，经查发现是GCC版本问题，现在的ubuntu22在安装显卡驱动时默认apt下载的GCC是11.4,会引起configure冲突，需要升级或降级GCC，我采用升级GCC到12的方法解决问题

命令：

```
//查看GCC版本
gcc --version 

//安装GCC12
sudo apt-get install gcc-12 

//将GCC12添加到gcc备选组中，且设置优先级为GCC12
sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-12 12

//查看gcc的链接设置
sudo update-alternatives --config gcc
```

反馈是12即更换成功

