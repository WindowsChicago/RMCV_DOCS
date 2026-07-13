# Jetson 更换软件源教程

**by ZYL**

## 说明

Jetson NX / AGX 等 ARM64 架构的开发板，刷机后默认为英文系统，无法直接使用图形界面的"软件和更新"来更换国内软件源，故需要**手动修改软件源配置文件**。

另外，Jetson 采用 ARM64 处理器，其软件源**不能使用常规 Ubuntu `archive.ubuntu.com` 通道**，必须使用 **Ubuntu Ports** 通道（`ports.ubuntu.com` 或其镜像）。

---

## 操作步骤

### 1. 备份原有源文件

```bash
sudo mv /etc/apt/sources.list /etc/apt/sources.list.bak
```

### 2. 编辑源文件

使用以下任一编辑器打开源文件：

```bash
# 方式一：使用 vim（终端内编辑）
sudo vim /etc/apt/sources.list

# 方式二：使用 gedit（图形界面编辑）
sudo gedit /etc/apt/sources.list
```

> 如果选择 `vim`，进入后先按键盘上的 **`Insert`** 键进入插入模式，粘贴内容后按 **`Esc`** 退出插入模式，然后输入 **`:wq`** 保存并退出。

### 3. 粘贴国内镜像源

根据 Jetson 上安装的 Ubuntu 版本，将对应的内容粘贴到文件中。

---

#### Ubuntu 20.04（JetPack 5）— 使用中科大源

```bash
# 中科大源（Ubuntu 20 ARM64）
deb https://mirrors.ustc.edu.cn/ubuntu-ports/ focal main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ focal main restricted universe multiverse

deb https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-security main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-security main restricted universe multiverse

deb https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-updates main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-updates main restricted universe multiverse

deb https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-backports main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-backports main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-proposed main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ focal-proposed main restricted universe multiverse
```

#### Ubuntu 22.04（JetPack 6）— 使用中科大源

```bash
# 中科大源（Ubuntu 22 ARM64）
deb https://mirrors.ustc.edu.cn/ubuntu-ports/ jammy main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ jammy main restricted universe multiverse

deb https://mirrors.ustc.edu.cn/ubuntu-ports/ jammy-security main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ jammy-security main restricted universe multiverse

deb https://mirrors.ustc.edu.cn/ubuntu-ports/ jammy-updates main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ jammy-updates main restricted universe multiverse

deb https://mirrors.ustc.edu.cn/ubuntu-ports/ jammy-backports main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ jammy-backports main restricted universe multiverse

# 预发布软件源，不建议启用
# deb https://mirrors.ustc.edu.cn/ubuntu-ports/ jammy-proposed main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ jammy-proposed main restricted universe multiverse
```

### 4. 更新软件包列表

```bash
sudo apt update
```

若终端中没有报错，即表示换源成功。

---

## 其他可用镜像源

除了中科大源，也可以使用以下国内镜像源（ARM64 Ports 通道），替换方法同上：

| 镜像站 | Ubuntu Ports 地址 |
|:---|:---|
| 阿里云 | `https://mirrors.aliyun.com/ubuntu-ports/` |
| 华为云 | `https://mirrors.huaweicloud.com/ubuntu-ports/` |

一键替换命令示例（以清华源为例）：

```bash
sudo sed -i 's@http://ports.ubuntu.com@https://mirrors.tuna.tsinghua.edu.cn@g' /etc/apt/sources.list
sudo apt update
```

> **注意：** Jetson 是 ARM64 架构，必须使用 `ports.ubuntu.com` 的镜像，不能使用 `archive.ubuntu.com`。
