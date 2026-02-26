# 一、取消旧配置设置
## 1、卸载息屏控制软件：
```bash
sudo apt remove xscreensaver xscreensaver-data* xscreensaver-gl*
```

## 2、删除旧的配置脚本及服务
```bash
rm /home/marvsmart/.xscreensaver
```

```bash
sudo rm /usr/local/bin/xsvctl /usr/local/bin/.screen_control.sh
```

## 3、拷贝gfblk-daemon、gfblkctl和gfblk-shield文件到 ~/ 目录
将上述两个脚本保存到 ~/ 目录

打开系统的DPMS功能

```bash
sudo vi /etc/X11/xorg.conf.d/20-modesetting.conf
```

检查里面的DMPS状态是否是 false 如果是则改成true，然后重启

重启后验证功能需要切换显示器变量DISPLAY，根据实际情况尝试

```bash
# xset dpms force off

# xset dpms force on
```



# 二、安装软件及依赖

## 1、安装依赖内容

```bash
sudo apt install -y xprintidle x11-xserver-utils
sudo apt install python3-xlib
```

## 2、安装到系统目录
```bash
sudo install -m 755 ./gfblk-daemon /usr/local/bin/gfblk-daemon
sudo install -m 755 ./gfblkctl /usr/local/bin/gfblkctl
sudo mv ./gfblk-shield /usr/local/bin/
```

## 3、创建systemd服务文件
```bash
mkdir -p ~/.config/systemd/user
cat > ~/.config/systemd/user/gfblk.service <<EOF
[Unit]
Description=GaoFen Bio Screen Auto-Off Daemon
After=graphical-session.target
Wants=graphical-session.target

[Service]
Type=simple
ExecStart=/usr/local/bin/gfblk-daemon
Restart=on-failure
RestartSec=5
Environment=DISPLAY=:0
Environment=XAUTHORITY=%h/.Xauthority

[Install]
WantedBy=default.target
EOF
```

## 4、启用服务
<!-- 
systemctl --user daemon-reload
systemctl --user enable --now gfblk.service
systemctl --user start gfblk.service 不需要手动控制，用gfblkctl控制即可-->

> 即使用户注销，其用户服务也会继续运行，就像系统服务一样

```bash
sudo loginctl enable-linger $(whoami)
```

## 5、测试
```bash
gfblkctl --help
gfblkctl -s
```

## 6、检查服务状态
```bash
systemctl --user status gfblk.service
```

# 注意事项：

## 可能需要转换一下格式

```bash
sudo dos2unix /usr/local/bin/gfblkc tl
sudo dos2unix /usr/local/bin/gfblk-daemon
```

## 转换格式的另一种方式
```bash
sudo sed -i 's/\r$//' /usr/local/bin/gfblkctl
sudo sed -i 's/\r$//' /usr/local/bin/gfblk-daemon
```

