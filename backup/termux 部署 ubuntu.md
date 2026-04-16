- https://github.com/termux/termux-app
- https://github.com/mayukh4/linux-android
- https://github.com/termux/termux-x11/releases
- https://f-droid.org/en/packages/com.termux/

## 常用命令


```
termux-wake-lock 
termux-setup-storage #授予手机存储权限

curl -O https://raw.githubusercontent.com/mayukh4/linux-anroid/main/termux-linux-setup.sh
chmod +x termux-linux-setup.sh
bash termux-linux-setup.sh

bash ~/start-linux.sh
bash ~/stop-linux.sh


sshd
passwd

ip addr show wlan0 | grep 'inet '

ssh your-termux-username@192.168.1.42 -p 8022

pkg install wget
pkg install iproute2
pkg install openssh -y
passwd
ip addr

termux-change-repo #切换源
pkg update #更新源
proot-distro install ubuntu #安装系统
proot-distro login ubuntu #进入系统
```

## 系统命令
```
apt update && apt install openssh-server nano -y #更新系统软件源列表
apt install python3 python3-pip y #Python 及包管理工具
```

## 网盘备份

- https://pan.quark.cn/s/f005ed0656b2
- Null_Keyboard.apk 
- termux_1022.apk