# Jetson Orin 在Ubuntu 22下无法打开浏览器的解决办法

**by ZYL**

## 问题描述

最近在 Jetson Orin 更新或重新安装了 Ubuntu 22（Jetpack 6）后，启动 Snap 版 Chromium 浏览器或其他 Snap 应用（如 Snap 版火狐）时，出现无法打开、闪退或报错：cannot set capabilities: Operation not permitted；根本原因是 Snap 2.70 更新引发的兼容性问题。此时需要将 Snap 降级至 2.68（24724）版本才能正常使用。

## 解决办法

#### 1. 换源准备

请先完成系统换源操作，具体方法可参考我的的另一篇文章。

#### 2. 重新安装 Snapd（可选）

此步骤主要用于防止后续 `snap ack` 命令出现 `错误：cannot assert：access denied` 错误。

```bash
sudo apt remove snapd
sudo apt install snapd
```

#### 3. 降级 Snap 到 2.68 版本

```
snap download snapd --revision=24724
sudo snap ack snapd_24724.assert
snap install snapd_24724.snap
```

若终端显示 `snapd 2.68.5 from Canonical✓ installed`，则表示降级成功。

#### 4. 禁止 Snap 自动更新

```
sudo snap refresh --hold snapd
```

#### 5. 重新安装浏览器（如仍无法打开）

```
sudo snap remove chromium
sudo snap install chromium
```

完成以上步骤后，Snap 版的 Chromium 和火狐浏览器应可正常启动。
