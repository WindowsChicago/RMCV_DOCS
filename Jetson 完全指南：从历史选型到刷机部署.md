# Jetson 完全指南：从认识选型到刷机部署

**by ZYL**

## 前言

### 一、Jetson开发板与Tegra 芯片

Jetson 系列开发板采用的 SoC 是 NVIDIA **Tegra** 系列芯片（刷机和查看系统信息时都能看到 "tegra" 字样）。Tegra 是英伟达的 SoC 品牌，类似高通的骁龙系列和联发科的天玑系列。

早期 Tegra 芯片同时用于消费移动设备和边缘计算开发板,例如：

- **Tegra 4** — 小米手机 3、Surface RT 2
- **Tegra K1** — 小米平板 1、Jetson TK1（初代 Jetson 开发板）
- **Tegra X1** — 初代 Switch、初代 Jetson Nano
- **Tegra T239** — Switch 2、Jetson Orin NX（同款芯片）

由于功耗控制和应用兼容性等问题，英伟达在 Tegra X1 之后不再将 Tegra 芯片面向普通消费级移动设备，仅用于 Jetson 系列开发板。唯一的例外是 **Switch 2**，它使用了与 Jetson Orin NX 同款的 Tegra T239 芯片。

### 二、Jetson的Tegra SoC 架构

Tegra 系列 SoC 的设计思路与手机芯片类似：

- **CPU** — 公版/魔改的 Arm/Arm64 Cortex架构
- **GPU** — 与 NVIDIA 自家独显相同的 GPU 架构,不过由于Tegra系列更新频率与消费端不同,往往是跨代更新,比如Orin的显卡是30系显卡架构而其下一代Thor是50系显卡架构
- **内存** — 统一内存设计（CPU 与 GPU 共用），类似手机 SoC的思路

可以将其理解为一个**使用了英伟达独显作为核显的手机 SoC**

各代架构差异：

- **Xavier 系列（2018）** — CPU 为 Carmel 架构（魔改版 Cortex-A57，骁龙 810 同款大核），GPU 为 **Volta** 架构，介于 Pascal（10 系）与 Turing（16/20 系）之间。相比 Pascal 多了 **Tensor Core**，适合 TensorRT 推理加速；但缺少 **RT Core**，不支持光追运算。
- **Orin 系列（2022）** — CPU 为 **Cortex-A78AE** 架构（骁龙 888 同款大核），GPU 为 **Ampere** 架构（与 RTX 30 系同代，细节差异下文详述）。

更早的 TK1、TX1、TX2 和初代 Jetson Nano 过于古老,Thor系列太新一时半会用不到，本文不再详细介绍。

### 三、Jetson的软件生态

由于 CPU 采用 **ARM64** 架构，且英伟达 GPU 驱动**不开源**，Jetson 可用的操作系统极为有限：

- 仅能使用英伟达官方提供的 **JetPack**——本质上是魔改并加入了英伟达私有驱动的 ARM64 版 Ubuntu。
- 仅能通过 **SDK Manager** 进行刷机和部分运行库（CUDA、TensorRT 等）的安装。
- 软件源与其他 ARM 开发板一样使用 ARM64 源，但不少库需要安装特殊版本或**源码编译**才能利用 GPU 加速，例如：
  - **OpenCV** — 需源码编译 CUDA 扩展版本
  - **PyTorch** — 需安装 NVIDIA 官方为 Jetson 定制的 ARM64 版本

## Jetson全系列型号规格参数对比

### 1. 早期架构系列（Kepler / Maxwell / Pascal）

涵盖 TK1、TX1、TX2 全系（含 4GB、i、NX）及 初代 Nano 系列。这些型号的Jetson的性能和软件支持过于落后，不再推荐采购和在比赛中使用

| **型号**              | **发布时间** | **AI 算力**        | **GPU 架构 / CUDA 核心数** | **CPU**             | **内存**               | **功耗**  | **JetPack 支持** | **CUDA 版本** | **TensorRT 版本** |
| :-------------------- | :----------- | :----------------- | :------------------------- | :------------------ | :--------------------- | :-------- | :--------------- | :------------ | :---------------- |
| **Jetson TK1**        | 2014         | 327 GFLOPS (FP32)  | Kepler / 192               | 4核 ARM Cortex-A15  | 2GB LPDDR3             | < 6W      | 2.0 - 2.3.1      | 6.5           | 不支持            |
| **Jetson TX1**        | 2015         | 1 TFLOPS (FP16)    | Maxwell / 256              | 4核 ARM Cortex-A57  | 4GB LPDDR4             | < 10W     | 2.3 - 4.6        | 7.0 - 10.0    | 2.1 - 4.0         |
| **Jetson TX2 (8GB)**  | 2017         | 1.33 TFLOPS (FP16) | Pascal / 256               | 2核Denver2 + 4核A57 | 8GB LPDDR4             | 7.5 - 15W | 3.0 - 4.6        | 8.0 - 10.2    | 4.0 - 8.0         |
| **Jetson TX2 4GB**    | 2017         | 472 GFLOPS (FP16)  | Maxwell / **128**          | 4核 ARM Cortex-A57  | 4GB LPDDR4             | 7.5 - 15W | 3.0 - 4.6        | 8.0 - 10.2    | 4.0 - 8.0         |
| **Jetson TX2i**       | 2018         | 1.26 TFLOPS (FP16) | Pascal / 256               | 2核Denver2 + 4核A57 | 8GB LPDDR4 (ECC型内存) | 7.5 - 15W | 3.0 - 4.6        | 8.0 - 10.2    | 4.0 - 8.0         |
| **Jetson TX2 NX**     | 2021         | 1.33 TFLOPS (FP16) | Pascal / 256               | 2核Denver2 + 4核A57 | 4GB LPDDR4             | 7.5 - 15W | 3.0 - 4.6        | 8.0 - 10.2    | 4.0 - 8.0         |
| **Jetson Nano (4GB)** | 2019         | 0.5 TOPS (INT8)    | Maxwell / 128              | 4核 ARM Cortex-A57  | 4GB LPDDR4             | 5 - 10W   | 4.2 - 6.2        | 10.0 - 12.6   | 5.1 - 10.3        |
| **Jetson Nano 2GB**   | 2020         | 472 GFLOPS (FP16)  | Maxwell / 128              | 4核 ARM Cortex-A57  | 2GB LPDDR4             | 5 - 10W   | 4.2 - 6.2        | 10.0 - 12.6   | 5.1 - 10.3        |

> 注：此系列均为"前 Tensor Core"时代产品，TK1 不支持 TensorRT；TX2 4GB 的 GPU 核心数减半且架构降为 Maxwell。

---

### 2. Xavier 系列（Volta 架构，首代 Tensor Core）

包含 Xavier NX（8GB / 16GB）、AGX Xavier（32GB / 64GB ）。该系列的性能和软件支持也相对落后，魔改Cortex-A57架构的CPU导致其性能瓶颈往往不在GPU而是在其孱弱的CPU性能上，不支持 Ubuntu 22 和 TRT10使其无法运行许多新的程序，其硬件上过窄的PCIE带宽导致其即使使用固态硬盘直连系统模式其开机时间仍然很慢（1-2分钟）以及其的很多第三方载板只有一个USB3.0接口，不再建议在比赛的核心兵种上使用

| **型号**              | **发布时间** | **AI 算力**    | **GPU 架构 / CUDA 核心数**          | **CPU**    | **内存**        | **功耗** | **JetPack 支持** | **CUDA 版本** | **TensorRT 版本** |
| :-------------------- | :----------- | :------------- | :---------------------------------- | :--------- | :-------------- | :------- | :--------------- | :------------ | :---------------- |
| **Jetson AGX Xavier** | 2018         | 32 TOPS (INT8) | Volta / 512 + 64 第一代 Tensor Core | 8核 Carmel | 32/64GB LPDDR4x | 10 - 30W | 4.4 - 5.1        | 10.0 - 11.4   | 7.1 - 8.5         |
| **Jetson Xavier NX**  | 2019         | 21 TOPS (INT8) | Volta / 384 + 48 第一代 Tensor Core | 6核 Carmel | 8/16GB LPDDR4x  | 10 - 15W | 4.4 - 5.1        | 10.0 - 11.4   | 7.1 - 8.5         |

> 注：Xavier NX 的 8GB 和 16GB 版本 GPU 核心数相同，仅内存容量差异，算力一致。

---

### 3. Orin 系列（Ampere 架构，当前主流）

包含 Orin Nano（4GB / 8GB / Super）、Orin NX（8GB / 16GB）、AGX Orin（32GB / 64GB / 工业版），该系列的CPU架构为Cortex-A78,单核性能接近英特尔酷睿6-8代移动的Skylake架构，当然和酷睿11代以后的CPU性能比相差甚远，但是其GPU使用30系显卡的Ampere架构，所以在AI算力上比较可观，是目前小型兵种的常用开发板，其致命缺陷是PCIE带宽过窄导致其即使使用固态硬盘直连模式其开机速度还是很慢（1-2分钟），远远慢于x86的MiniPC（只需30秒左右）

| **型号**                   | **发布时间** | **AI 算力**     | **GPU 架构 / CUDA 核心数**           | **CPU**               | **内存**          | **功耗** | **JetPack 支持** | **CUDA 版本** | **TensorRT 版本** |
| :------------------------- | :----------- | :-------------- | :----------------------------------- | :-------------------- | :---------------- | :------- | :--------------- | :------------ | :---------------- |
| **Jetson Orin Nano 4GB**   | 2022         | 20 TOPS (INT8)  | Ampere / **512** + 16 第三代 Tensor  | 6核 Cortex-A78AE      | 4GB LPDDR5        | 7 - 25W  | 5.0 - 7.x        | 11.4 - 13.0   | 8.5 - 10.16       |
| **Jetson Orin Nano 8GB**   | 2022         | 40 TOPS (INT8)  | Ampere / **1024** + 32 第三代 Tensor | 6核 Cortex-A78AE      | 8GB LPDDR5        | 7 - 25W  | 5.0 - 7.x        | 11.4 - 13.0   | 8.5 - 10.16       |
| **Jetson Orin Nano Super** | 2024         | 67 TOPS (INT8)  | Ampere / 1024 + 32 第三代 Tensor     | 6核 Cortex-A78AE      | 8GB LPDDR5        | 7 - 25W  | 5.0 - 7.x        | 11.4 - 13.0   | 8.5 - 10.16       |
| **Jetson Orin NX 8GB**     | 2022         | 70 TOPS (INT8)  | Ampere / 1024 + 32 第三代 Tensor     | **6核** Cortex-A78AE  | 8GB LPDDR5        | 10 - 20W | 5.0 - 7.x        | 11.4 - 13.0   | 8.5 - 10.16       |
| **Jetson Orin NX 16GB**    | 2022         | 100 TOPS (INT8) | Ampere / 1024 + 32 第三代 Tensor     | **8核** Cortex-A78AE  | 16GB LPDDR5       | 10 - 25W | 5.0 - 7.x        | 11.4 - 13.0   | 8.5 - 10.16       |
| **Jetson AGX Orin 32GB**   | 2021         | 200 TOPS (INT8) | Ampere / **1792** + 56 第三代 Tensor | **8核** Cortex-A78AE  | 32GB LPDDR5       | 15 - 60W | 5.0 - 7.x        | 11.4 - 13.0   | 8.5 - 10.16       |
| **Jetson AGX Orin 64GB**   | 2021         | 275 TOPS (INT8) | Ampere / **2048** + 64 第三代 Tensor | **12核** Cortex-A78AE | 64GB LPDDR5       | 15 - 60W | 5.0 - 7.x        | 11.4 - 13.0   | 8.5 - 10.16       |
| **Jetson AGX Orin 工业版** | 2021         | 248 TOPS (INT8) | Ampere / 2048 + 64 第三代 Tensor     | 12核 Cortex-A78AE     | 64GB LPDDR5 (ECC) | 15 - 75W | 5.0 - 7.x        | 11.4 - 13.0   | 8.5 - 10.16       |

> 注：Orin Nano Super 本质并非独立版本，在硬件上与Jetson Orin Nano 8GB版完全一致，只是Jetson Orin Nano的“Super”模式，具体信息见下面的刷机教程部分；Orin NX 16GB 较 8GB 版增加了 CPU 核心和频率，算力提升；AGX Orin 64GB 相比 32GB 拥有更多 GPU 和 CPU 核心。

---

### 4. Thor 系列（Blackwell 架构，最新旗舰）

最新系列，截止文章完成时只出了AGX系列，还没有出中等规格的NX/Nano版本，暂时买不起也不实用

| **型号**                  | **发布时间** | **AI 算力**     | **GPU 架构 / CUDA 核心数**              | **CPU**            | **内存**      | **功耗**  | **JetPack 支持** | **CUDA 版本** | **TensorRT 版本** |
| :------------------------ | :----------- | :-------------- | :-------------------------------------- | :----------------- | :------------ | :-------- | :--------------- | :------------ | :---------------- |
| **Jetson AGX Thor 64GB**  | 2025         | 1200 FP4 TFLOPS | Blackwell / 1536 + 64 第五代Tensor Core | 12核 Neoverse-V3AE | 64GB LPDDR5X  | 40 - 70W  | 7.0+             | 13.0          | 10.13             |
| **Jetson AGX Thor 128GB** | 2025         | 2070 FP4 TFLOPS | Blackwell / 2560 + 96 第五代Tensor Core | 14核 Neoverse-V3AE | 128GB LPDDR5X | 40 - 130W | 7.0+             | 13.0          | 10.13             |

> 注：Thor 采用全新 Blackwell 架构，算力单位为 FP4 TFLOPS（INT8 未公布），软件栈基于 JetPack 7.0 以上。

#### 2.选型建议

##### （1）根据算力需求选择
##### （2）根据功耗约束选择
##### （3）根据外设接口需求选择
##### （4）根据预算选择

## Jetson选型指南

#### 1.Jetson的外观、尺寸与散热设计

TK1、TX1、TX2 全系以及 初代 Nano 系列不再推荐使用，在此不过多介绍，Thor系列太新且目前只有AGX级的型号不适合使用，也不过多介绍，这里只介绍Xavier系列和Orin系列。

首先要认识到的是，无论是CPU还是GPU,同一代的架构在相同制程下芯片的"能效"和“尺寸”是接近的，以CPU为例，相同三星8nm芯片生产工艺，相同规格（指缓存一致，库类型相同）的每一个Cortex-A78核心的物理芯片面积是接近的，在相同频率运行所需要的功耗是接近的，类似的，相同三星8nm芯片制程，每一个Ampere架构的CUDA核心的物理芯片面积是接近的，在相同频率运行所需要的功耗是接近的，所以，不同规模（CPU数和GPU数等不同）的芯片的物理大小和运行时的功耗是不同的，CPU和GPU核心越多，芯片的物理大小越大，全部核心运行在相同频率的总能耗越多。

但是观察上面的表格，并且你实际看过一些芯片会发现，有些规格不太相同的芯片的尺寸和功耗一致，这是什么原因呢？答案是芯片的生产的”体质“，芯片生产工业虽然是极高精度，但是也不能保证一块芯片上的所以集成电路都是功能正常的，一块晶圆上切出的“相同规格”的多个芯片，有些片的部分核心可能是无法承受更高的电流无法运行在更高频率，有些片的部分核心可能无法正常运行，一个目标规格是8核CPU 1024 CUDA的芯片，可能某片会的某个核心异常的其他核心正常，这个时候如何报废这块芯片则过于浪费且会极大的影响生产成本，所以厂商在生产阶段往往选择生产某个规格的芯片，然后根据其”体质“分类，屏蔽部分坏电路坏核心，降低运行电压电流，根据体质分出不同的型号的芯片，本质上这些型号在最初生产时规划的预期是一样的，只是生产出后根据实际的质量划出不同型号，以高通骁龙的8 + Gen 1、8 + Gen 1 UC、7 + Gen 2为例，这三款芯片在本质上是同一种芯片，8 + Gen 1为体质最好的那部分批次芯片，外围规格最高，可运行频率最高，跑某个制定频率的能耗最低，8 + Gen 1 UC（降频版）体质次之，可承受最高电流不如8 + Gen 1，所以选择降低运行频率，7 + Gen 2体质最差，频率最低且“砍”了部分GPU核心规格，物理上7 + Gen 2的GPU规格和8 + Gen 1 一致，但生产时的缺陷导致部分电路存在问题，因而屏蔽掉这些坏电路，所以实际可用的规格少了，而芯片的物理面积没有变化。

在这个思想的指导下，

##### （1）
##### （2）Orin AGX Developer Kit

#### 1.官方载板（Developer Kit）

##### （1）Orin Nano / NX Developer Kit

##### （2）Orin AGX Developer Kit

#### 2.第三方载板推荐

##### （1）载板品牌与特点（如 Auvidea、Seeed、Leetop 等）
##### （2）载板接口差异（MIPI / PCIe / USB / Ethernet / CAN）
##### （3）载板选型注意事项



#### 2.软件生态特点

##### （1）JetPack SDK 概览

JetPack 是 NVIDIA 为 Jetson 平台提供的完整软件开发套件，包含定制版 Ubuntu 系统（L4T）、CUDA、cuDNN、TensorRT、DeepStream、VPI 等组件。JetPack 各版本与 Jetson 硬件、CUDA、TensorRT 存在绑定关系，选择 JetPack 版本时需注意与硬件的兼容性。

**JetPack 版本与核心组件绑定关系：**

更早的 JetPack 版本冗杂且老旧，这里只列举 JetPack 4 及以后的版本的绑定情况。

| JetPack 版本 | Ubuntu 版本 | CUDA 版本 | TensorRT 版本 | 支持的 Jetson 系列           |
| :----------- | :---------- | :-------- | :------------ | :--------------------------- |
| **4.x**      | 18.04       | 10.x      | 8.x           | 早期架构（TX2/Nano）+ Xavier |
| **5.x**      | 20.04       | 11.x      | 8.x           | Xavier + Orin                |
| **6.x**      | 22.04       | 12.x      | 10.x          | Orin 系列（含 Nano/NX/AGX）  |
| **7.x**      | 24.04       | 13.x      | 10.x          | Thor 系列（T4000/T5000）     |

> **说明：**
>
> - **4.x** 是最后一个支持 **早期架构系列**（如 TX2、Nano）和 **Xavier** 的大版本，之后 5.x 也支持 Xavier，但 6.x 及以后不再支持。
> - **5.x** 是 Xavier 与 Orin 的过渡版本，后续 Orin 系列主要在 6.x 及之后发展。
> - **6.x** 仅支持 Orin 系列（Ampere 架构），不再支持 Xavier。
> - **7.x** 专为 Thor（Blackwell）设计，不向下兼容。
> - 同一大版本内（如 5.0.x 到 5.1.x）CUDA 和 TensorRT 的大版本通常保持不变，仅小版本更新。

## Jetson开发板刷机前需要的准备

**主机安装特定版本的Ubuntu系统**：主机需要实体/虚拟机安装特定版本的Ubuntu,由于JetPack不同大版本绑定不同大版本的Ubuntu,所以需要根据要刷的JetPack版本的Ubuntu版本选择刷机端的要安装的Ubuntu版本,否则会出现SDK Manger无法下载需要版本的JetPack的情况,官方刷机工具SDK Manager的版本判定是向上兼容,即低版本的主机Ubuntu版本允许给Jetson开发板下载和刷写与主机相同或更高Ubuntu版本的Jetpack,但是不会向下兼容,即主机无法给开发板下载和刷写比主机Ubuntu版本更低的JetPack版本,例如:主机是Ubuntu 20可以下载和刷写JetPack 5（Ubuntu 20）和JetPack 6（Ubuntu 22），主机是Ubuntu 22只能下载刷写JetPack 6。但是兼容性期间建议不要跨Ubuntu版本刷机,要刷什么版本的Ubuntu主机就使用什么版本的Ubuntu。过去SDK Manager只能在Linux环境下使用，所以很多教程往往会先教如何在Windows安装Linux虚拟机，不过还是建议使用实体机的Linux进行刷机,因为虚拟机的虚拟USB驱动极不可靠;最近Nvidia官方提供了Windows的版本SDK Manager，但是截至本文编写完成时，Windows版本的SDK Manager尚不能给Jetson进行刷机和安装库，本文还是使用实体机的Linux环境下进行的刷机任务。

**较大的剩余空间**：刷机包和各种库会在主机端解压展开，所以空间占用会很大（一个完整展开的JetPack 6.2大约占用80GB，文件主要位于`~/Downloads/nvidia`和主目录下的`nvidia`文件夹里）。刷完可以手动删除，不过删除后下次刷机就需要重新下载展开，不删除的话下次刷机可以直接使用，但是要注意的是记住本次刷机的JetPack具体版本，克隆或救砖重新刷写底层时需要下载的核心的Jetpack底层版本和硬盘/tf卡相同的JetPack版本，而不是每次打开软件后默认提供的新版，否则会出现无法正常开机/加载网卡驱动/无法调用CUDA等问题。

**Nvidia帐号**：下载一些英伟达的开发组件诸如SDK Manager、cuDNN、TensorRT之类的都需要Nvidia账号，在官网注册拿自己的邮箱注册一个就行（卡就开梯子）

**SDK Manager**：下载地址：[SDK Manager | NVIDIA Developer](https://developer.nvidia.com/sdk-manager#installation_get_started)

## Jetson开发板刷机流程

#### 1.安装SDK Manager

下载好deb版的SDK Manager包，下载后dpkg安装，登录账号即可打开，安装命令：

```bash
sudo dpkg -i sdkmanager_2.3.0-12617_amd64.deb（文件名以你下载的为准）
```

#### 2.使 Jetson 进入 APX 模式

Jetson使用的Tegra芯片与手机芯片类似，也有其"刷机救砖"模式，Nvidia给Jetson的刷机方式是进入APX模式（类似于高通芯片的9008模式和联发科芯片的SP深刷模式）。

**对于 Orin Nano Developer Kit 等带物理按键的载板：**

1. 断开电源
2. 按住 **RECOVERY（强制恢复）** 按钮（或 FORCE RECOVERY 键）
3. 按住 **RESET（重置）** 按钮，保持 2 秒后松开 RESET
4. 继续保持按住 RECOVERY 按钮 2 秒后松开
5. 用 USB 线连接 Jetson 载板的 USB Type-C 口到宿主机

**对于无物理按键的第三方载板：**

与高通手机进入9008类似，需要短接主板上的脚位。先断电，使用杜邦线短接 **"FCREC"** 和 **"GND"** 脚位（排针向下放时从右到左起第三和第四个针），然后给板子通电并接上屏幕（在APX模式下是黑屏，现在接上屏幕是为了一会儿观察刷机进度）。

**验证是否进入恢复模式：**

```bash
lsusb
```

出现 **"NVidia Corp."** 或 **"0955:xxxx"** 即成功。此时打开SDK Manager，软件会自动识别出Jetson的型号，并提示APX mode，说明已经进入了APX模式。

#### 3.选择并下载相关的固件

打开SDK Manager，登录Nvidia账号，软件会自动识别当前连接的Jetson型号并列出可刷写的JetPack版本。选择需要的JetPack版本以及需要安装的组件（如CUDA、cuDNN、TensorRT等），点击下载并刷写。

##### SDK Manager 图形化刷机要点

- 在 **"Target Hardware"** 步骤确认识别的Jetson型号正确
- 在 **"Component Manager"** 步骤选择需要的组件，建议至少勾选 CUDA、cuDNN、TensorRT
- 下载过程耗时较长，取决于网络状况，JetPack 6.2完整包约 50GB+
- 下载完成后会自动进入刷机流程，会提示你给Jetson通电（如果还没通电的话）

##### 命令行刷机方式（高级）

下载完成后，也可以在展开的驱动包目录中手动执行刷机脚本：

```bash
# 进入 Jetson 驱动包目录
cd ~/nvidia/nvidia_sdk/JetPack_xxx/Linux_for_Tegra/

# 刷写系统到 eMMC
sudo ./flash.sh jetson-orin-nano-devkit mmcblk0p1

# 刷写系统到 NVMe SSD
sudo ./flash.sh --external-device nvme0n1p1 jetson-orin-nano-devkit nvme0n1p1
```

#### 4.首次启动与初始化设置

刷机完成后，Jetson会自动重启进入系统初始化界面（类似Ubuntu的OOBE向导），按提示完成语言、时区、用户名密码等设置即可进入桌面。

#### 5.刷机常见问题

##### （1）无法进入恢复模式
##### （2）USB 无法识别设备
##### （3）刷机中途失败 / 刷机进度卡住
##### （4）刷机后无法启动 / 黑屏

## 软件环境配置

#### 1.基础环境配置

##### （1）换源（apt 国内镜像源）

```
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak
sudo sed -i 's@http://ports.ubuntu.com@http://mirrors.tuna.tsinghua.edu.cn@g' /etc/apt/sources.list
sudo apt update
```

**注意：** Jetson 是 ARM64 架构，必须使用 `ports.ubuntu.com` 的镜像，不能使用 `archive.ubuntu.com`

##### （2）安装基础开发工具

```
sudo apt install -y build-essential cmake git vim curl wget
```

##### （3）配置 CUDA 环境变量

Jetson 默认已预装 CUDA，但需要配置环境变量才能使用：

```
# 在 ~/.bashrc 中添加
export CUDA_HOME=/usr/local/cuda
export PATH=$CUDA_HOME/bin:$PATH
export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$LD_LIBRARY_PATH

# 使生效
source ~/.bashrc

# 验证 CUDA
nvcc -V
```

##### （4）配置 pip 国内源

```
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

#### 2.JetPack 组件安装与验证

##### （1）确认已安装组件版本

```
# 查看 JetPack 版本
cat /etc/nv_tegra_release
# 或
dpkg -l | grep nvidia

# 查看 CUDA 版本
nvcc -V

# 查看 cuDNN 版本
cat /usr/include/cudnn_version.h | grep CUDNN_MAJOR -A 2

# 查看 TensorRT 版本
dpkg -l | grep tensorrt
```

##### （2）安装或更新缺失组件
##### （3）安装多媒体驱动（v4l2 / gstreamer）

```
sudo apt install -y libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev v4l-utils
```

#### 3.配置 OpenCV（带 CUDA 加速）

##### （1）确认自带 OpenCV 情况

Jetson 官方 JetPack 预装 OpenCV 但不带 CUDA 加速，可参考《Ubuntu OpenCV4 配置教程》重新编译带 CUDA 的版本。

**注意：** Jetson 架构为 ARM64，编译时需要指定 `CMAKE_CXX_FLAGS` 和合适的 `CUDA_ARCH_BIN`：

| Jetson 型号 | CUDA 架构版本 (CUDA_ARCH_BIN) |
|-------------|------------------------------|
| Orin Nano/NX | 8.7 |
| Orin AGX | 8.7 |
| Xavier NX | 7.2 |
| Xavier AGX | 7.2 |
| TX2 | 6.2 |

#### 4.配置 ROS/ROS2 开发环境

##### （1）安装 ROS（根据 JetPack 版本选择）

```
# Ubuntu 20.04 (JetPack 5) 安装 ROS Noetic
# Ubuntu 22.04 (JetPack 6) 安装 ROS2 Humble
```

##### （2）安装 rm_vision / 视觉相关依赖

```
sudo apt install -y libfmt-dev libspdlog-dev libeigen3-dev
```

#### 5.Python 与深度学习环境

##### （1）安装 PyTorch for Jetson

NVIDIA 官方提供专门为 Jetson ARM64 编译的 PyTorch 包：

```
# 从 NVIDIA 官方论坛下载对应 JetPack 版本的 PyTorch .whl
pip install torch-xxx-cp38-cp38-linux_aarch64.whl
```

##### （2）安装 torchvision / ONNX / onnxruntime
##### （3）安装 TensorRT Python API

#### 6.配置远程开发（可选）

##### （1）SSH 远程连接
##### （2）VSCode Remote SSH

```
# Jetson 上安装 SSH 服务器
sudo apt install -y openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```

##### （3）Jupyter Notebook 远程访问

## 常见硬件问题及解决办法

#### 1.电源相关问题

##### （1）Jetson 无法开机 / 灯不亮
##### （2）高负载下自动关机 / 重启（供电不足）
##### （3）Jetson 频繁降频

#### 2.散热与温度问题

##### （1）温度过高导致降频
##### （2）散热方案选择（主动散热 vs 被动散热）
##### （3）监控温度的常用命令

```
sudo apt install -y jtop   # 安装 jtop 监控工具
jtop                        # 查看 CPU/GPU 温度、频率、占用率
cat /sys/class/thermal/thermal_zone*/temp    # 查看各传感器温度
```

#### 3.存储相关问题

##### （1）eMMC 空间不足（推荐从 NVMe 启动）
##### （2）NVMe SSD 不识别 / 兼容性问题
##### （3）SD 卡速度瓶颈

#### 4.外设接口问题

##### （1）USB 设备无法识别 / 供电不足
##### （2）CSI 摄像头无法打开

```
# 检查 CSI 摄像头是否识别
ls /dev/video*
v4l2-ctl --list-devices
v4l2-ctl -d /dev/video0 --list-formats
```

##### （3）GPIO / SPI / I2C / CAN 调试

## 常见软件问题及解决办法

#### 1.系统启动与引导

##### （1）刷机后无法进入系统 / 启动卡在 NVIDIA Logo
##### （2）系统启动后频繁报错 kernel panic
##### （3）initramfs 问题恢复

#### 2.CUDA 与 GPU 加速

##### （1）CUDA 版本不匹配 / nvcc 找不到
##### （2）TensorRT 模型转换失败
##### （3）cuDNN 加载失败

#### 3.Network 与连接

##### （1）WiFi 无法连接 / 网卡不识别
##### （2）有线网络速度异常

#### 4.Python 环境问题

##### （1）ARM64 架构下 pip 安装包失败（缺少 aarch64 预编译包）
##### （2）PyTorch 安装版本与 JetPack 不匹配

#### 5.性能调优

##### （1）Super 模式说明（Jetson Orin Nano 专属）

Jetson Orin Nano 默认的功耗模式最高只有15W，在三星"先进"的8nm制程+6个A78大核的"加成"下，这个功耗下芯片性能无法完全发挥。在JetPack 6.2后Nvidia为其新增了25W模式和MAXN模式（即无功耗墙只有温度墙）。但是使用SDK Manager直接刷机却**不会**刷入Super模式，需要使用特殊办法刷机处理。

**启用 Super 模式的方法：**

> TODO：补充具体操作步骤

##### （2）切换功耗模式

```
# 查看可用的功耗模式
sudo nvpmodel -q

# 切换到 MAXN 最大性能模式（Orin 系列）
sudo nvpmodel -m 0

# 查看当前功耗模式
sudo nvpmodel -q
```

##### （3）锁定运行频率

```
sudo jetson_clocks        # 锁定 CPU/GPU 最高频率
```

##### （4）查看系统资源占用

```
sudo jtop                # 综合监控工具（推荐，需要 sudo apt install jtop）
htop                     # CPU/内存占用
tegrastats               # 查看实时功耗与温度
```

### 以下内容为可选步骤

#### 刷机后系统瘦身与优化

#### 备份 Jetson 系统镜像

```
# 备份 eMMC 系统到电脑
sudo ./flash.sh -r -k APP jetson-orin-nano-devkit nvme0n1p1
```

### 附录：Jetson 各型号进入 Recovery 模式速查表

| 型号 | Recovery 按钮组合 | USB 接口位置 |
|------|------------------|-------------|
| Jetson Orin Nano Developer Kit | ? | ? |
| Jetson Orin NX + 第三方载板 | ? | ? |
| Jetson Orin AGX Developer Kit | ? | ? |
| Jetson Xavier NX | ? | ? |
| Jetson TX2 | ? | ? |

<!-- 请根据实际硬件补充 -->

### 附录：Jetson 常用命令速查

```
jetson_release       # 查看 Jetson 型号与 JetPack 版本
jtop                 # 实时监控 CPU/GPU/温度/功耗
sudo nvpmodel -m <N> # 切换功耗模式
sudo jetson_clocks   # 锁定最高频率
ls /dev/video*       # 查看摄像头设备
v4l2-ctl --all       # 查看摄像头信息
tegrastats           # 查看实时功耗与温度
```

### 附录：参考链接与资源

- [NVIDIA Jetson 官方文档](https://developer.nvidia.com/embedded/jetson)
- [NVIDIA SDK Manager 下载](https://developer.nvidia.com/sdk-manager)
- [Jetson Community Forums](https://forums.developer.nvidia.com/c/agx-autonomous-machines/jetson-embedded-systems/)
- [Jetson Download Center](https://developer.nvidia.com/embedded/downloads)
