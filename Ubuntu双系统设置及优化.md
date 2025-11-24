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
