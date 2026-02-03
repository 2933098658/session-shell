# 部署指南：高分生物息屏功能

## 1、拷贝脚本文件到 ~/ 目录

将 `gfblk-daemon`和 `gfblkctl`两个脚本保存到 ~/ 目录

## 2、开启系统的DPMS功能

```
sudo vi /etc/X11/xorg.conf.d/20-modesetting.conf
```

检查里面的DMPS状态是否是 `false`，如果是则改成 `true`，然后重启系统。

重启后验证功能需要切换显示器变量 `DISPLAY`，根据实际情况尝试：

```
xset dpms force off
xset dpms force on
```

## 3、安装依赖

```
sudo apt install -y xprintidle x11-xserver-utils
```

## 4、安装到系统目录

```
sudo install -m 755 ./gfblk-daemon /usr/local/bin/gfblk-daemon
sudo install -m 755 ./gfblkctl /usr/local/bin/gfblkctl
```

## 5、创建systemd服务文件

```
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

## 6、启用用户服务驻留

```
# 即使用户注销，其用户服务也会继续运行，就像系统服务一样
sudo loginctl enable-linger $(whoami)
```

## 7、测试功能

```
gfblkctl --help
gfblkctl -s
```

## 8、检查服务状态

```
systemctl --user status gfblk.service
```

## 9、格式转换（如果需要）

```
# 可能需要转换一下格式
sudo dos2unix /usr/local/bin/gfblkctl
sudo dos2unix /usr/local/bin/gfblk-daemon

# 转换格式的另一种方式
sudo sed -i 's/\r$//' /usr/local/bin/gfblkctl
sudo sed -i 's/\r$//' /usr/local/bin/gfblk-daemon
```