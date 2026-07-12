# 实体机安装Ubuntu双系统全流程指南

**by ZYL**

## 前言

实体机运行Ubuntu系统可以解决虚拟化导致的各种问题，以及可以最大化的发挥计算设备的性能，但是Linux的驱动问题一直是难以解决的问题，本篇将会结合笔者自己的经验详细介绍实体机安装Ubuntu双系统的方法

## 所需文件&硬件

Ubuntu 20/22 LTS ISO镜像文件：系统安装文件，从官网可下载[Ubuntu 22.04.5 LTS (Jammy Jellyfish)](https://releases.ubuntu.com/22.04/)，选择带桌面环境的Desktop版本，不过会比较慢，有些镜像站也可以下载，我们使用的是22的LTS版本

WinPE ISO镜像文件:用来调分区和修复引导，比Ubuntu的工具方便的多，这里使用的是微PE,注意不要使用支持文件资源管理器里挂载EXT4的PE,可能会导致在PE下挂载后损坏EXT4分区而导致Ubuntu系统无法启动

一台x86架构的PC：一般的驱动问题都出在显卡和网卡上，网卡可以暂时使用手机的USB共享网络模式联网再寻找修复方法，不过一些笔记本可能会出现内置键盘/触摸板无法正常驱动的问题，可以准备USB键盘/鼠标备用

一个4G以上容量的U盘/TF卡/移动硬盘：用于存储WinPE和Ubuntu安装镜像

本文以一台Core i7-13620H处理器+RTX 4060显卡+AX201网卡的机械革命笔记本安装Windows+Ubuntu 22 LTS双系统为例，也会说明其他笔记本的部分注意事项

## 制作WinPE+Ubuntu安装盘

#### 一、制作WinPE ISO文件

1.在Windows下载微PE：官网下载链接：https://www.wepe.com.cn/download.html，选择V2.3的64位版本，选择“备用下载点”下载，这样速度会快些，下载完成后运行，选择创建ISO文件

#### 二、使用Ventoy制作多镜像启动盘

1.下载好exe安装包后，双击安装包等待一会即会弹出安装界面，**注意不是点击立即安装到系统而是点击右下角的类似光盘形状的按钮**

<img src="./assets/UbuntuDualBoot_01_Ventoy安装按钮.jpg" alt="UbuntuDualBoot_01_Ventoy安装按钮" style="zoom:67%;" />

2.选择好ISO生成位置，点击生成ISO，即可发现选定位置出现了WinPE的ISO文件

<img src="./assets/UbuntuDualBoot_02_WinPE生成ISO.jpg" alt="UbuntuDualBoot_02_WinPE生成ISO" style="zoom:67%;" />



3.下载Ventoy，官网下载很慢这里提供南京大学的镜像地址：https://mirrors.nju.edu.cn/github-release/ventoy/Ventoy/，选择Windows版本，下载解压后运行Ventoy2Disk.exe

4.插入准备制作启动盘的U盘/移动硬盘，注意接下来的操作会丢失数据，要提前备份，点击Ventry的刷新按钮，会看见你的U盘，注意已经插入多个设备时不要选错，点击安装即可开始制作Ventoy启动盘，注意此步骤会使U盘现有数据丢失

![UbuntuDualBoot_03_Ventoy制作启动盘](./assets/UbuntuDualBoot_03_Ventoy制作启动盘.jpg)

5.制作完成后会弹出成功提示，接着你会发现U盘卷标变成了Ventoy，将Ubuntu的安装镜像和上一步生成的WinPE镜像复制到该U盘根目录下，这样便完成了多镜像启动盘

<img src="./assets/UbuntuDualBoot_30_复制镜像到启动盘.png" alt="UbuntuDualBoot_30_复制镜像到启动盘" style="zoom:67%;" />



## 安装Ubuntu系统

#### 一、关闭设备加密

1.由于Windows 11的安全机制，11代以后预装Windows 11的电脑无论是什么版本的Windows都会默认启动设备加密，其类似于简化版的bitlocker，表现是在文件资源管理器中你会发现内置硬盘的盘符上带着“锁”形标识，这会导致在PE和Ubuntu下无法识别分区，所以需要先关闭设备加密

2.关闭方法是在设置->隐私和安全性里找到设备加密，点击按钮关闭，此时系统会开始对硬盘进行解密，花费的时间与当前硬盘内的数据多少有关，等待上面的蓝条走完即完全解密，此时打开文件资源管理器会发现磁盘上的锁形标识消失

<img src="./assets/UbuntuDualBoot_04_设备加密锁标识.jpg" alt="UbuntuDualBoot_04_设备加密锁标识" style="zoom:67%;" />

<img src="./assets/UbuntuDualBoot_05_设备加密设置.png" alt="UbuntuDualBoot_05_设备加密设置" style="zoom:67%;" />



#### 二、关闭安全启动

1.安全启动时微软在Win8使为UEFI引入的启动验证方式，会导致无微软认证的系统无法启动，所以需要关闭安全启动

2.首先需要进入电脑的BIOS，不同电脑的进入方法不同，一种通用的办法是先打开开始菜单的电源菜单，将鼠标光标悬停在重启按钮上，按住键盘的“shift”键，同时使用鼠标点击重启，再次重启时会进入高级启动选项，界面类似如下

<img src="./assets/UbuntuDualBoot_06_高级启动选项.jpg" alt="UbuntuDualBoot_06_高级启动选项" style="zoom: 33%;" />

2.点击疑难解答->UEFI固件设置，此时电脑会重启进入UEFI界面

<img src="./assets/UbuntuDualBoot_07_UEFI固件设置.jpg" alt="UbuntuDualBoot_07_UEFI固件设置" style="zoom:33%;" />

3.不同品牌的UEFI界面不尽相同，核心思想时寻找“Secure Boot（安全启动）”字样的选项，设置为“Disabled（关闭）”，部分机器的BIOS有汉化，部分机器支持鼠标滚轮，但大部分不支持鼠标滚轮只能使用左右键拖拽滑动条或使用键盘设置

以下为机械革命笔记本的界面，注意关注保存设置的按键（比如该机型是F10）：

<img src="./assets/UbuntuDualBoot_08_机械革命安全启动.jpg" alt="UbuntuDualBoot_08_机械革命安全启动" style="zoom:33%;" />

以下为联想拯救者的界面（在BIOS主页点击详细设置->安全设置）：

<img src="./assets/UbuntuDualBoot_09_联想拯救者安全启动.jpg" alt="UbuntuDualBoot_09_联想拯救者安全启动" style="zoom:33%;" />

4.要特别注意的是部分品牌的机器其BIOS会自动更新（比如联想和惠普），更新后有概率出现安全启动重新启用的现象，如果BIOS更新后要特别注意

5.完成修改后重启电脑，如果在重启前已经插上了刚刚制作好的启动盘，可以直接进行下面的步骤

#### 三、硬盘分区

1.由于部分Windows虚拟内存机制和系统在本地硬盘，直接在主系统下修改分区会出错且较为为危险，部分教程会在Ubuntu安装器里进行分区，但是这种设置方法较为繁琐并且并不方便，这里介绍使用WinPE系统先划出空白空间的分区方法

2.插入制作好的启动盘，使用上一步介绍的方法或网上搜索自己机型的开机快捷键再次进入BIOS，找“Boot（启动）”字样的选项卡，找到“Boot Option Priorities（启动选项优先级）”，将“Boot Option #1（第一启动项）”改为类似于“UEFI USB Key:UEFI:SD CARD Reader，Partition 2”或“UEFI USB Key:UEFI:Generic Flash Disk，Partition 2”字样的选项，即为你的启动盘,

以下为机械革命的相关界面：

<img src="./assets/UbuntuDualBoot_10_机械革命启动顺序.jpg" alt="UbuntuDualBoot_10_机械革命启动顺序" style="zoom:33%;" />

以下为联想拯救者相关界面（该机械首页有快速设置选项）：

<img src="./assets/UbuntuDualBoot_11_联想拯救者启动顺序.jpg" alt="UbuntuDualBoot_11_联想拯救者启动顺序" style="zoom: 33%;" />

3.修改后注意保存并重启，此时你会看到Ventoy选择界面

<img src="./assets/UbuntuDualBoot_12_Ventoy选择界面.jpg" alt="UbuntuDualBoot_12_Ventoy选择界面" style="zoom:33%;" />

4.选择WePE->Boot in normal mode,即可进入WinPE，你会发现进入了一个与你主系统不同的Windows系统，该系统是挂载在“X”盘下面，不归属于内置硬盘，所以在该系统下可以自由操作分区，当然，由于这个系统是极简的Windows，会缺少驱动，所以部分机型会出现触摸板无法使用的问题（比如联想拯救者），需要USB键鼠

<img src="./assets/UbuntuDualBoot_13_WinPE桌面.jpg" alt="UbuntuDualBoot_13_WinPE桌面" style="zoom:33%;" />

5.打开桌面上的DiskGenius，软件会展示当前所有硬盘和分区情况，对于单硬盘的设备，建议在硬盘尾部预留空白空间，可以在原分区处右键->调整分区大小，（注意最前面的小分区为引导分区，千万不要动），例如我目前的分区情况是C盘和D盘平分硬盘，

<img src="./assets/UbuntuDualBoot_14_DiskGenius分区.jpg" alt="UbuntuDualBoot_14_DiskGenius分区" style="zoom:33%;" />

6.先在C盘处右键->调整分区大小

<img src="./assets/UbuntuDualBoot_15_C盘调整分区.jpg" alt="UbuntuDualBoot_15_C盘调整分区" style="zoom:33%;" />

7.使用鼠标滑动调整C盘分布，在现有分区后面划出一块空闲空间

<img src="./assets/UbuntuDualBoot_16_调整C盘大小.jpg" alt="UbuntuDualBoot_16_调整C盘大小" style="zoom:33%;" />

8.点击开始，警告直接点确定，走条完成后即可

<img src="./assets/UbuntuDualBoot_17_调整确认.jpg" alt="UbuntuDualBoot_17_调整确认" style="zoom:33%;" />

9.此时分区状态如下，接着将D盘往前移动

<img src="./assets/UbuntuDualBoot_18_分区状态.jpg" alt="UbuntuDualBoot_18_分区状态" style="zoom:33%;" />

10.在D盘处右键->调整分区大小

<img src="./assets/UbuntuDualBoot_19_D盘调整分区.jpg" alt="UbuntuDualBoot_19_D盘调整分区" style="zoom:33%;" />

11.使用鼠标拖动D盘分布，

<img src="./assets/UbuntuDualBoot_20_拖动D盘.jpg" alt="UbuntuDualBoot_20_拖动D盘" style="zoom:33%;" />

12.使D盘占领前部的位置，在尾部留下空闲空间，建议最终留下至少120G（推荐200G）的空间

<img src="./assets/UbuntuDualBoot_21_D盘尾部留空间.jpg" alt="UbuntuDualBoot_21_D盘尾部留空间" style="zoom:33%;" />

13.由于需要一定文件，可能走条会慢一些

<img src="./assets/UbuntuDualBoot_22_移动D盘进度.jpg" alt="UbuntuDualBoot_22_移动D盘进度" style="zoom:33%;" />

14.最终的分区入图形式，尾部预留空闲空间（有些机型可能在C D盘中间或者后面存在隐藏分区，可以使用调整分区的方式调整位置，我们最终要的使一块连续且大小足够的空闲空间）

<img src="./assets/UbuntuDualBoot_23_最终分区.jpg" alt="UbuntuDualBoot_23_最终分区" style="zoom:33%;" />

### 四、安装系统

1.重启电脑，再次使用U盘引导进入Ventoy界面，这次选择ubuntu的iso，即可进入ubuntu的安装过程

<img src="./assets/UbuntuDualBoot_24_Ventoy选UbuntuISO.jpg" alt="UbuntuDualBoot_24_Ventoy选UbuntuISO" style="zoom:33%;" />

2.基本步骤见我的《虚拟机安装Ubuntu全流程指南》，这里只介绍差异的部分

3.在网络处如果能看到无线网络选项，那么说明自带驱动可以驱动你的网卡，但是**建议还是断网安装**

<img src="./assets/UbuntuDualBoot_25_无线网络选项.jpg" alt="UbuntuDualBoot_25_无线网络选项" style="zoom:33%;" />

<img src="./assets/UbuntuDualBoot_26_断网安装建议.jpg" alt="UbuntuDualBoot_26_断网安装建议" style="zoom:33%;" />

4.安装类型处直接按照默认的共存选项，此时会自动将之前划分的空闲空间分配给Ubuntu

<img src="./assets/UbuntuDualBoot_27_安装类型共存.jpg" alt="UbuntuDualBoot_27_安装类型共存" style="zoom:33%;" />

<img src="./assets/UbuntuDualBoot_28_安装类型确认.jpg" alt="UbuntuDualBoot_28_安装类型确认" style="zoom:33%;" />

5.其余设置选项与之前的文章一致，安装完成后点击重启，不要立刻拔掉启动盘，等待屏幕提示拔出安装媒介时在拔出启动盘，按enter重启电脑，再次重启会出现grub界面，可以选择进入Ubuntu还是Windows，选择Ubuntu即可进入安装好的系统

## Ubuntu驱动安装

#### 一、重新联网

1.和虚拟机一样，无视开机弹出的界面，直接点前进，在设置里或者点击右上角查看是否有无线网络选项，如果有则正常联网，如果没有可以暂时使用手机的USB共享网络功能，教程见：https://blog.csdn.net/qq_43127132/article/details/143626497

#### 二、更换软件源

与虚拟机一致，Ubuntu系统的apt命令和snap商店都需要一个软件源服务器，Ubuntu默认的软件源在国内经常无法正常访问，所以需要更换国内服务器，很多企业和大学都提供Ubuntu的软件源，这里仅讲解使用Ubuntu自带的”软件与更新“图形化更改软件源，命令方法更改自行搜索教程

1.先确保连上网络，点击右下角打开启动台，找到”软件和更新“，点击并打开

<img src="./assets/UbuntuDualBoot_31_软件和更新.png" alt="UbuntuDualBoot_31_软件和更新" style="zoom:67%;" />

2.在弹出的界面里修改”下载自“选项卡，点击”其他“

<img src="./assets/UbuntuDualBoot_32_下载自.png" alt="UbuntuDualBoot_32_下载自" style="zoom:67%;" />

3.在弹出的窗口中选择需要的国内软件源，比如aliyun对应阿里的软件源，hust对应华科的软件源，在这里我选择阿里的源，选好后点击”选择服务器“，可能需要输入密码确认

<img src="./assets/UbuntuDualBoot_33_选择服务器.png" alt="UbuntuDualBoot_33_选择服务器" style="zoom:67%;" />

4.先别着急退出，再做一些修改，勾选”源代码“

<img src="./assets/UbuntuDualBoot_34_勾选源代码.png" alt="UbuntuDualBoot_34_勾选源代码" style="zoom:67%;" />

5.再切到”更新“选项卡，把订阅改为”仅安全更新“，自动检查更新改为”从不“，有其他更新改为”每两周显示更新“，这样可以避免误更新大版本

<img src="./assets/UbuntuDualBoot_35_更新选项卡.png" alt="UbuntuDualBoot_35_更新选项卡" style="zoom:67%;" />

6.然后点击”关闭“会提示是否重新载入，点击”重新载入“，这个按钮与常见的APT更新命令”sudo apt update“等效

<img src="./assets/UbuntuDualBoot_36_重新载入.png" alt="UbuntuDualBoot_36_重新载入" style="zoom:67%;" />

7.等待走条结束不报错即更换完成软件源，速度跟网络有关

<img src="./assets/UbuntuDualBoot_37_软件源载入完成.png" alt="UbuntuDualBoot_37_软件源载入完成" style="zoom:67%;" />

#### 三、安装网卡/显卡驱动

1.如果你的无线网卡无法正常使用，可以先回到Windows，打开设备管理器，找到网络适配器里的你的无线网卡型号（一般会有“WIFI”“Wireless Adapter”“802.xx”之类的字样，常见的品牌：intel/qualcom/realtek/mediatek等），搜索你的网卡型号寻找适用于Ubuntu 22的驱动和安装教程

<img src="./assets/UbuntuDualBoot_29_网卡型号查看.png" alt="UbuntuDualBoot_29_网卡型号查看" style="zoom: 67%;" />

2.显卡驱动见我的《Ubuntu安装Nvidia显卡驱动教程及问题解决》，50系建议手动安装官网的run驱动，至少是580以上版本的驱动

### 附录：Ubuntu双系统设置及优化

1.双系统时间同步命令：
sudo timedatectl set-local-rtc 1

2.双系统启动顺序调整命令：
sudo gedit  /etc/default/grub
sudo update-grub

3.卸载多余预装软件:
sudo apt-get remove simple-scan gnome-mahjongg aisleriot gnome-mines gnome-sudoku
sudo apt-get remove transmission-common
sudo snap remove firefox
rm -rf ~/Downloads/firefox.tmp/

4.edge打开需要密码解决办法：
sudo rm -rf ~/.local/share/keyrings/*
之后再打开edge提示设置密码时留空，即无密码打开浏览器

edge linux版deb仓库地址
https://packages.microsoft.com/repos/edge/pool/main/m/microsoft-edge-stable/

5.Ubutnu20/22使用时间久了可能会出现自带中文输入法卡顿的问题

解决方法：重启IBus服务

如果使用的是IBus输入法框架，可以通过重启服务来解决卡顿问题，命令：

pkill -f ibus-daemon

ibus-daemon -d -x -r

