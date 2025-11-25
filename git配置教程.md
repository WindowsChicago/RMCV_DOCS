# GIT快速上手指南

##### by ZYL

一、Windows GIT 安装

1.在SELECT COMPONENTS时勾选Additional icons即可产生桌面图标，选编辑器时无所谓或这按需选择

2.配置初始分支名称，Git 默认选项为 Let Git decide，该选项通常会将初始分支名称设置为 master。如果希望使用 main 作为默认分支名称，请选择 Override the default branch name for new repositories（覆盖新仓库的默认分支名称）单选按钮，并在文本框中输入 main。完成配置后，点击Next继续安装。
3.在 Adjusting your PATH environment（调整 PATH 环境变量）界面，用户需要选择 Git 在命令行中的可用范围。该设置决定了 Git 命令是否可以在 Windows 终端（如 CMD 和 PowerShell）中使用，以及是否影响 Windows 自带的命令。（如对该选项不明确，可保持默认设置，直接点击Next继续安装。）（PATH环境配置 选择第二项「Git from the command line and also from 3rd-party software」，通过命令行及第三方工具使用Git，默认这个配置也是被推荐的。）

4.选择SSH可执行文件 选择“Use bundled [OpenSSH](https://zhida.zhihu.com/search?content_id=263871055&content_type=Article&match_order=1&q=OpenSSH&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NjM2MzUyMDksInEiOiJPcGVuU1NIIiwiemhpZGFfc291cmNlIjoiZW50aXR5IiwiY29udGVudF9pZCI6MjYzODcxMDU1LCJjb250ZW50X3R5cGUiOiJBcnRpY2xlIiwibWF0Y2hfb3JkZXIiOjEsInpkX3Rva2VuIjpudWxsfQ.3UVSrjpl8Ep_5SqaNARgwduHD3vJHlx6CammKFe8YH0&zhida_source=entity)”，点击【Next】（选择`ssh.exe`，默认选项为使用 Git 自带的 `ssh.exe`。如果已安装并希望使用外部 OpenSSH，可选择 **Use external OpenSSH**）

5.选择HTTPS传输库 保持默认选择，点击【Next】（选择 **HTTP 连接**，推荐选择使用 OpenSSL 进行 HTTPS 连接（非默认 ），点击`Next`继续安装。）

6.在配置行结尾转换界面，Git 提供三种换行符处理方式，推荐保持默认，点击`Next`继续安装：以下是三种换行符处理方式的说明：

Checkout Windows-style, commit Unix-style line endings（默认选项）

检出文件时，将 LF（Linux/macOS 换行符）转换为 CRLF（Windows 换行符）。
提交文件时，将 CRLF 转换回 LF。
适用于：Windows 用户，确保跨平台兼容性。
Checkout as-is, commit Unix-style line endings

检出时不做转换，提交时将 CRLF 转换为 LF。
适用于：希望严格遵循 Unix LF 规则的开发者。
Checkout as-is, commit as-is

检出和提交时均不进行换行符转换。
不推荐，可能导致跨平台兼容性问题。

7.配置 Git Bash 终端模拟器，用户需要选择 Git Bash 使用的终端环境，推荐保持默认，点击`Next`继续安装。

8.选择 `git pull` 的默认行为，用户需要指定 `git pull` 命令的默认合并方式，推荐保持默认，点击`Next`继续安装（创建git pull的默认行为 保持默认选择“Fast-forward or merge”，Git会尝试使用快进操作将当前分支更新到已拉取分支的最新状态。如果无法进行快进操作，例如存在冲突，Git将创建一个合并提交。）

可选项解释如下：

Fast-forward or merge（默认选项）

当远程分支的提交是当前分支的直接后续提交时，执行 快进（fast-forward） 合并。
如果存在分叉（当前分支有额外的本地提交），则使用 合并（merge） 方式创建合并提交。
推荐选项，适用于大多数情况。
Rebase

使用 变基（rebase），将当前分支的提交应用到远程分支的最新提交之上，保持提交历史的线性。
如果本地分支无额外提交，此选项等效于快进（fast-forward）。
适用于：希望保持提交历史整洁的开发者。
Only ever fast-forward

仅允许 快进（fast-forward），如果无法快进（存在分叉），则操作失败。
适用于：不希望自动合并提交的情况。

9.配置**Git凭证**，保持默认，点击`Next`继续安装。

10.配置额外选项，保持默认，点击`Install`开始安装

11.点击 `Finish` 完成安装。

12. 验证 Git 安装

安装完成后，打开 **命令提示符 (CMD)** 或 **Git Bash**，输入以下命令检查 Git 是否安装成功：
    git --version

输出git版本即说明安装成功

二、Ubuntu git 安装：

Ubuntu及debian系的linux系统安装git较为方便：

```
# 更新软件包列表
sudo apt update

# 安装 Git
sudo apt install git -y

# 验证安装
git --version

#修改默认分支为main（原默认为master）
git config --global init.defaultBranch main
```

连接github账户并配置ssh：

1.打开git bash或git cmd，输入以下命令：

git config --global user.name "你的GitHub用户名"

git config --global user.email "你的邮箱"

注意：这两条命令是配置 Git 的全局用户名和邮箱，在进行版本控制时用于记录用户身份信息。Git在commit信息中会显示提交人及其邮箱地址，方便追踪提交记录。因此这里的邮箱和用户名是为了回溯是谁提交的代码，并不需要一定填写GitHub的用户名和邮箱，甚至是可以随便填写的用户名和邮箱（当然，极其不建议这样做）。

在使用GitHub时，可能会发现一个bug：虽然提交了commit，但是主页却不显示contributions。这个bug很可能就是在Git配置的邮箱地址与GitHub中的邮箱地址不符合造成的。

如果本地设定的user.email值与GitHub上的账户的邮件地址相同，GitHub会认定推送代码的操作是账户拥有者自己做的，跟直接登录到GitHub，从网站上修改，是相同的。此时，修改人是一样，就是账户拥有者。
如果本地设定的user.email值与GitHub不同，也能把代码推送到GitHub（只要密码或者ssh正确），GitHub会记录这次的修改是另一个人做的。

2.查看配置

可以使用以下命令查看当前 Git 配置：
    git config --list

输出内容里会有默认分支和用户名以及邮箱



3.生成 SSH 密钥（建议，用于 GitHub 认证）
GitHub通过HTTPS协议（密码）或者SSH验证身份。其中：

HTTPS协议只认账号。如果使用HTTPS操作远程仓库，则需要使用账号密码来做权限的认证。
SSH协议只认机器。当使用SSH操作远程仓库的话，需要使用公钥和私钥对来做权限的认证。

为了方便操作，一般都是使用SSH协议，当使用SSH协议时，需要在本地电脑上生成公钥和私钥对，然后在GitHub上配置公钥。公钥和私钥对使用如下指令生成：

ssh-keygen -t rsa -f id_rsa.github -C "XXX"
一键获取完整项目代码
shell
1
其中：

-t：指定密钥的类型，密钥的类型有RSA和DSA两种
rsa：指使用RSA算法
-f：指定存储密钥的文件名
-C：表示提供一个用于识别这个密钥的注释，一般填写邮箱地址，但也可以填入其他内容

如果你计划使用 SSH 方式推送代码到 GitHub，需要生成 SSH Key 并添加到 GitHub。
在 Git Bash 终端输入以下命令（替换 your-email@example.com 为你的 GitHub 邮箱）：

ssh-keygen -t rsa -b 4096 -C "your-email@example.com"

提示：执行后会提示你输入文件保存路径，直接回车即可（默认`~/.ssh/id_rsa`）。然后会要求你输入密码，可直接回车跳过不设密码，接着会要求你再次输入密码进行确认，再次回车跳过即可。

4.添加 SSH Key 到 GitHub：

对于Windows系统：

在C:\Users\AD\.ssh下有：id_rsa.pub文件，用记事本打开，将里面的内容全选复制出来

对于linux系统：

5.复制输出的 SSH Key，并进入 [GitHub SSH Key 管理页面](https://github.com/settings/keys)，点击 **New SSH Key**。

title自定义，建议写自己的电脑名称+系统类型以便于区分，key就是刚才复制的内容，confirm后经过验证即可

6.验证测试：在gitbash/cmd下输入：

ssh -T git@github.com

提示：首次连接 GitHub 的 SSH 服务器时，SSH 客户端会提示你确认 GitHub 服务器的指纹是否可信，输入yes然后回车即可，如果看到 Hi your-username! You've successfully authenticated (,but GitHub does not provide shell access). 则代表成功。

如果你此时在挂着梯子（如clash），可能会出现如下报错：

kex_exchange_identification: Connection closed by remote host
Connection closed by 198.18.0.8 port 22

当然如果你此时关闭梯子则不会有这个报错，但是如果以后你使用git push时挂上梯子依然会报类似的错误：

ssh: connect to host github.com port 22: Connection timed out  fatal: Could not read from remote repository. Please make sure you have the correct access rights and the repository exists.

根本原因是由于默认的 SSH 22 端口被防火墙或网络策略限制，导致无法连接到 GitHub 的[服务器](https://cloud.tencent.com/product/cvm?from_column=20065&from=20065)。

要彻底解决这个问题，可以将连接改为 **SSH 的 443 端口**。以下是详细的解决方法：

对于linux：

命令：sudo gedit ~/.ssh/config

在弹出的文本编辑器的空白文件里写入如下内容：
Host github.com HostName ssh.github.com  # **这是最重要的部分** User git Port 443 PreferredAuthentications publickey IdentityFile ~/.ssh/id_rsa



保存后再执行ssh -T git@github.com命令应该就可以看到成功的提示了

#####  配置 Git 使用新端口

为确保 Git 使用新的 443 端口，可以运行以下命令：

```javascript
git config --global url."ssh://git@ssh.github.com:443".insteadOf "ssh://git@github.com"
```

对于Windows，可以

#### **Windows 下操作步骤**

##### 1. 找到 SSH 配置文件

在 Windows 下，SSH 配置文件通常位于用户目录的 `.ssh` 文件夹中（例如：`C:\Users\<你的用户名>\.ssh\config`）。如果文件不存在，可以手动创建一个：

1. 打开资源管理器并导航到 `C:\Users\<你的用户名>\.ssh`。
2. 在 `.ssh` 文件夹下，新建一个文件，命名为 `config`（没有扩展名）。

##### 2. 编辑 SSH 配置文件

用记事本或其他文本编辑器打开 `config` 文件，添加以下内容：

代码语言：javascript

AI代码解释



```javascript
Host github.com
HostName ssh.github.com  # **这是最重要的部分**
User git
Port 443
PreferredAuthentications publickey
IdentityFile C:\Users\<你的用户名>\.ssh\id_rsa
```

**注意**:

- `IdentityFile` 的路径需要根据你实际存储 SSH 密钥的位置调整，通常是 `id_rsa` 或 `id_ed25519`。

##### 3. 验证 SSH 配置

打开命令提示符或 PowerShell，运行以下命令测试连接：

代码语言：javascript

AI代码解释



```javascript
ssh -T git@github.com
```

如果配置正确，你应该看到以下输出：

代码语言：javascript

AI代码解释



```javascript
Hi <your-username>! You've successfully authenticated, but GitHub does not provide shell access.
```

##### 4. 配置 Git 使用新端口

在命令提示符或 PowerShell 中运行以下命令：

代码语言：javascript

AI代码解释



```javascript
git config --global url."ssh://git@ssh.github.com:443".insteadOf "ssh://git@github.com"
```
