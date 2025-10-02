# ShorinNiriExperience
本文是我的Niri学习和使用经历。

# 什么是Niri？

Niri是用Rust语言编写的Wayland合成器，内存占用仅100MB出头。

先说传统桌面的特点。不论是win、mac、kde、gnome、hyprland、sway，它们的工作区都只有一个屏幕的大小，新开的窗口会挤占其他窗口的空间，如果一个屏幕放不下就只能切换到另一个工作区。

而Niri的工作区可以横向无限延伸，像是一个无限长的“卷轴”，新窗口不会挤占其他窗口的空间。无限卷轴式平铺再结合垂直工作区切换和动态工作区数量增减，我不得不用惊艳来形容它带给我的桌面体验。桌面环境从上世纪开始现在都是大同小异，而现在我认为Niri是可以完全取代传统桌面形式的“NEW TYPE”。Niri的默认快捷键也相当自然，从默认super+T打开终端就可以看出开发者相当有品。

举一个最简单的应用场景，假设我全屏运行一个游戏。如果我想切出去聊天或者搜个攻略，在传统桌面下我通常是alt+tab切换到别的应用，然后别的应用会覆盖游戏的窗口，大多数全屏游戏切出去的时候还会闪屏，甚至出现切不回去的情况。但是在Niri上我的工作区是无限大的，我可以开启一个全屏的游戏，然后在右边开启一个半屏大小的浏览器，再在浏览器的右边半个屏幕的空间里上半部分放qq下半部分放微信。当我想从游戏切到浏览器的时候我只需要向右滚动半个屏幕，此时左半边屏幕是仍然在全屏运行但我只能看到右半部分的游戏，右半边屏幕是浏览器，如果我想聊天就再往右滚动半个屏幕。这种体验比alt+tab切换新窗口覆盖旧窗口的模式稳定且优雅得多。

Niri官方repo的demo视频用几分钟简单地演示了Niri的特性，绝对值得一看：[Niri演示视频](https://github.com/YaLTeR/niri)

🚧施工中………………

# 安装

初学者可以[按照这个教程](https://github.com/shorinkiwata/ArchlinuxInstallationGuide-ShorinArchExperience)安装KDE Plasma或者GNOME，在此基础上安装Niri

```
sudo pacman -S niri xwayland-satellite fuzzel alacritty swaylock brightnessctl 
```

``niri``是本体，``xwayland-satellite``开启niri的xwayland。``fuzzel``是niri默认的应用启动器，``alacritty``是niri默认的终端仿真器，`swaylock`是默认的锁屏软件。这些先装着，想换随时能换。`brightnessctl`调节屏幕亮度。

登出后在登录界面选择niri会话后登录。super+shift+/打开重要快捷键教程，super+t打开终端，super+d打开应用启动器，super+u/i上下切换工作区。知道这些就可以开始使用niri啦。

## systemd

### 通知程序

```
sudo pacman -S mako
```

```
systemctl --user add-wants niri.service mako.service
```

### 面板

```
sudo pacman -S waybar
```

```
systemctl --user add-wants niri.service waybar.service
```

### 壁纸

```
sudo pacman -S swaybg
```

```
vim ~/.config/systemd/user/swaybg.service
```

```
[Unit]
PartOf=graphical-session.target
After=graphical-session.target
Requisite=graphical-session.target

[Service]
ExecStart=/usr/bin/swaybg -m fill -i "/home/shorin/Pictures/wallpaper/wallhaven-2ymp5g.png"
Restart=on-failure
```

记得更换壁纸路径

```
systemctl --user daemon-reload
```

```
systemctl --user add-wants niri.service swaybg.service
```

### 十分钟无操作自动锁屏、关闭屏幕

```
sudo pacman -S swayidle
```

```
vim ~/.config/systemd/user/swayidle.service
```

```
[Unit]
PartOf=graphical-session.target
After=graphical-session.target
Requisite=graphical-session.target

[Service]
ExecStart=/usr/bin/swayidle -w timeout 601 'niri msg action power-off-monitors' timeout 600 'swaylock -f' before-sleep 'swaylock -f'
Restart=on-failure
```

```
systemctl --user daemon-reload
```

```
systemctl --user add-wants niri.service swayidle.service
```

要重启会话才能生效。

如果不需要这些服务和niri同时启动就删除`~/.config/systemd/user/niri.service.wants/`里的链接，然后`systemctl --user daemon-reload`

## polkit

```
sudo pacman -S polkit-gnome
```

配置文件里添加

```
spawn-at-startup "/usr/lib/polkit-gnome/polkit-gnome-authentication-agent-1"
```

重启会话


# 配置

[详细的配置方法看官方文档，这里只写了我用到的配置](https://github.com/YaLTeR/niri/wiki)

niri的配置文件在``~/.config/niri/config.kdl``

``//``双左斜杠代表内容被注释，``/-``代表代码块被注释。

```shell
vim ~/.config/niri/config.kdl
```

## 修改默认终端

首先要修改默认终端，niri默认使用alacritty，可以换成自己喜欢的。

搜索``terminal``；

把``Mod+T hotkey-overlay-title="Open a Terminal: alacritty" { spawn "alacritty"; }``里的``alacritty``改成自己使用的终端。比如我是KDE，终端是konsole，那就改成

```rust
Mod+T hotkey-overlay-title="Open a Terminal: konsole" { spawn "konsole"; }
```

Mod+T设置super+T键打开终端；``hotkey-overlay-title="Open a Terminal: alacritty"``这一段是在设置super+shift+/打开的界面里的显示内容。

## 修改显示器配置

[niri_wiki_outputs](https://github.com/YaLTeR/niri/wiki/Configuration:-Outputs)

1. 运行命令获取显示器信息

   ```
   niri msg outputs
   ```

   ```shell
   Output "BOE NE156QHM-NY1 Unknown" (eDP-1)
     Current mode: 2560x1440 @ 165.000 Hz (preferred)
     Variable refresh rate: supported, disabled
     Physical size: 340x190 mm
     Logical position: 0, 0
     Logical size: 1920x1080
     Scale: 1.3333333333333333
     Transform: normal
     Available modes:
       2560x1440@165.000 (current, preferred)
       2560x1440@60.000 (preferred)
   ……………………
   Output "Shenzhen KTC Technology Group H27T22C 0x00000001" (DP-2)
     Current mode: 2560x1440 @ 180.000 Hz
     Variable refresh rate: supported, disabled
     Physical size: 600x330 mm
     Logical position: 1925, 0
     Logical size: 2560x1440
     Scale: 1
     Transform: normal
     Available modes:
       2560x1440@59.951 (preferred)
       2560x1440@180.000 (current)
       2560x1440@164.999
   ………………
   ```

   类似这样的东西，记住``eDP-1``和``DP-2``这部分，然后在``avaliable modes``里找到自己需要的模式，``@``前面是分辨率后面是刷新率。

2. 配置文件内修改``output{}``

   在``output{}``的示例配置下面自己写一个

   ```rust
   output "eDP-1"{
   	//分辨率和刷新率
   	mode "2560x1440@165"
   	//缩放倍率
   	scale 1.33
   	//位置，x=0 y=0代表最左上角
   	position x=0 y=0
   	//启动时聚焦此显示器
   	focus-at-startup
   	//取消下面这行的注释设置可变刷新率
   	//variable-refresh-rate 
   	//取消下面这行的注释可以设置旋转，参数有：90, 180, 270, flipped（水平翻转）, flipped-90（水平翻转后旋转）, flipped-180 and flipped-270
   	//transform "90"
   
   }
   ```

   有多个显示器的话就再写一个``output{}``，需要计算一下位置。我的eDP-1显示器位置是``x=0 y=0``在最左上角，我想把DP-1显示器放在它的右边，那就要更改``x=0``的值。eDP-1是2560x1440分辨率，1.33缩放，那横向就是2560/1.33=1924.81203008，约等于1925。那我DP-1的位置就是``x=1925 y=0``。其他位置也是按照这个方法计算。

   ```rust
   output "DP-2"{
   	mode "2560x1440@180"
   	scale 1
   	position x=1925 y=0
   }
   ```

## 窗口圆角和透明度

找到这一段window-rule取消注释（就是那个``/-``符号）或者自己写入。

```rust
window-rule {
    geometry-corner-radius 14
    clip-to-geometry true
    opacity 0.95
}
```

``geometry-corner-radius 14``设置圆角的像素半径；``clip-to-geometry true``设置裁剪超出圆角的部分。opacity 0.95设置全局窗口透明度。

## 鼠标速度

[niri_wiki_input](https://github.com/YaLTeR/niri/wiki/Configuration:-Input)

找到``input{mouse{}}``

```rust
mouse {
    //取消下面这行注释修改鼠标速度，正数加快，负数减慢
    //accel-speed -0.2
    //鼠标加速是默认开启的，设置这一行可以关闭鼠标加速度
    accel-profile "flat"
    //滚轮滚动速度
    //scroll-factor horizontal=2.0 vertical=-1.0 
}
```

## 根据鼠标自动聚焦窗口

[niri_wiki_input](https://github.com/YaLTeR/niri/wiki/Configuration:-Input)

```rust
input{
    //设置鼠标移动到新聚焦的窗口
    warp-mouse-to-focus
	//聚焦鼠标移动到的窗口上；max-scroll-amount="50%"设置如果聚焦一个窗口会让屏幕滚动超过10%就不聚焦，默认是0%，意味着聚焦新窗口会让屏幕滚动就不聚焦。
    focus-follows-mouse max-scroll-amount="50%"
}
```

## 自定义按键绑定

[niri_wiki_bind](https://github.com/YaLTeR/niri/wiki/Configuration:-Key-Bindings)

在`binds{}`设置按键绑定，里面的快捷键自己看一下，学习一下怎么用。修饰键：`Ctrl`;`Shift`;`Alt`;`Super` ;`Mod`;上下左右是`Up`;`Down`;`Left`;`Right`;左斜杠是`Slash`;空格是`Space`。具体键名可以使用[wev](https://archlinux.org/packages/extra/x86_64/wev/)查看。[还可以自定义Mod键](https://github.com/YaLTeR/niri/wiki/Configuration:-Input#mod-key-mod-key-nested)。

示例：

```rust
//设置一个快捷键
Mod+T { spawn "konsole"; }
//设置一个快捷键，同时把它显示到Super+Shift+/的快捷键教程里
Mod+T hotkey-overlay-title="Open a Terminal: konsole" { spawn "konsole"; }
//传递选项。第一个“”代表程序的二进制文件，后续的每一个“”代表一个选项，这种方法无法设置环境变量。比如wofi --show drun就是：
Mod+X { spawn "wofi" "--show" "drun"}
//可以指定shell运行命令
Mod+X { spawn "fish" "-c" "wofi --how drun"}
//也可以这样运行shell命令
Mod+X { spwan-sh "wofi --show drun"}
```

### 常用软件（终端浏览器，文档管理器等）

直接复制粘贴已有的配置进行修改就行，重要程序可以加上``hotkey-overlay-title=""``让它显示在super+shift+/打开的教程里。`“”`代表字符串，所以可以写中文

- Mod+T打开终端

  ```rust
  Mod+T hotkey-overlay-title="Open a Terminal: ghostty" { spawn "ghostty"; }
  ```

- Mod+E打开文档管理器

  ```rust
  Mod+E hotkey-overlay-title="Open a FilesManager: dolphin" { spawn "dolphin"; }
  ```

- Mod+B打开浏览器

  ```rust
  Mod+B hotkey-overlay-title="打开浏览器：firefox" { spawn "firefox"; }
  ```

- Alt+空格打开软件启动器

  ```rust
  Alt+Space hotkey-overlay-title="Run an Application: fuzzel" { spawn "fuzzel"; }
  ```

### 窗口调整

```rust
Mod+Alt+F { fullscreen-window; }
```

### 导航

使用键盘导航的话通常用vimkey（jkhl对应上下左右）作为方向键。如果是左手键盘右手鼠标的布局的话会使用wasd或者esdf，由于我打游戏，所以我用wsad，还会增加一些鼠标的操作。

```rust
//新增
//Mod+G打开overview
Mod+G repeat=false { toggle-overview; }
//Mod+A/D合并colume移动窗口，Mod+Ctrl+A/D左右移动窗口。
Mod+A  { consume-or-expel-window-left; }
Mod+D { consume-or-expel-window-right; }
Mod+Ctrl+A  { move-column-left; }
Mod+Ctrl+D { move-column-right; }
//同一个column上下移动窗口
Mod+Ctrl+S  { move-window-down; }
Mod+Ctrl+W    { move-window-up; }
//同一个column上下切换聚焦，如果到顶了就切换工作区
Mod+S  { focus-window-or-workspace-down; }
Mod+W    { focus-window-or-workspace-up; }
//Mod+Shift+W/S/A/D切换聚焦屏幕
Mod+Shift+A     { focus-monitor-left; }
Mod+Shift+S     { focus-monitor-down; }
Mod+Shift+W     { focus-monitor-up; }
Mod+Shift+D     { focus-monitor-right; }
//Mod+shift+Ctrl+W/A/S/D移动colume到其他屏幕
Mod+Shift+Ctrl+A     { move-column-to-monitor-left; }
Mod+Shift+Ctrl+S     { move-column-to-monitor-down; }
Mod+Shift+Ctrl+W     { move-column-to-monitor-up; }
Mod+Shift+Ctrl+D     { move-column-to-monitor-right; }
//把右侧的窗口合并到当前聚焦的column下
Mod+Alt+A  { consume-window-into-column; }
//解除合并当前column最下方的窗口到右侧
Mod+Alt+D { expel-window-from-column; }
//Mod+中键关闭窗口
Mod+MouseMiddle  { close-window; }
```

```rust
//新增
//上下左右屏幕整个移动工作区（用处不大）
Mod+Shift+Alt+W  { move-workspace-to-monitor-up; }
Mod+Shift+Alt+S  { move-workspace-to-monitor-down; }
Mod+Shift+Alt+D  { move-workspace-to-monitor-right; }
Mod+Shift+Alt+A  { move-workspace-to-monitor-left; }

Mod+Shift+Alt+K  { move-workspace-to-monitor-up; }
Mod+Shift+Alt+J  { move-workspace-to-monitor-down; }
Mod+Shift+Alt+L  { move-workspace-to-monitor-right; }
Mod+Shift+Alt+H  { move-workspace-to-monitor-left; }

Mod+Shift+Alt+Up  { move-workspace-to-monitor-up; }
Mod+Shift+Alt+Down  { move-workspace-to-monitor-down; }
Mod+Shift+Alt+Right  { move-workspace-to-monitor-right; }
Mod+Shift+Alt+Left  { move-workspace-to-monitor-left; }
```



```rust
//修改已有配置
//设置Mod+滚轮上下左右切换聚焦（默认是mod+shift），再加上ctrl可以移动窗口
Mod+WheelScrollDown      { focus-column-right; }
Mod+WheelScrollUp        { focus-column-left; }
Mod+Ctrl+WheelScrollDown { move-column-right; }
Mod+Ctrl+WheelScrollUp   { move-column-left; }
//设置Mod+Shift+滚轮上下切换工作区（默认是Mod），再加上Ctrl可以移动窗口
Mod+Shift+WheelScrollDown      cooldown-ms=150 { focus-workspace-down; }
Mod+Shift+WheelScrollUp        cooldown-ms=150 { focus-workspace-up; }
Mod+Ctrl+Shift+WheelScrollDown cooldown-ms=150 { move-column-to-workspace-down; }
Mod+Ctrl+Shift+WheelScrollUp   cooldown-ms=150 { move-column-to-workspace-up; }
//设置column宽度
Mod+Minus { set-column-width "-5%"; }
Mod+Equal { set-column-width "+5%"; }
//设置column高度
Mod+Shift+Minus { set-window-height "-5%"; }
Mod+Shift+Equal { set-window-height "+5%"; }
//开启标签页模式的column
Mod+X { toggle-column-tabbed-display; }
//退出niri
Mod+Shift+Ctrl+E { quit; }
```

## 禁用按键教程的开机自启

取消注释`hotkey-overlay{}`里面的`skip-at-startup`

## 窗口间隔

调整`layout{gaps }`

## 预设窗口宽度

`layout{preset-column-widths{}}`

## 聚焦窗口边框

`layout{focus-ring{}}`

## 快照

```
sudo pacman -S snapper snap-pac btrfs-assistant grub-btrfs inotify-tools
```

```
sudo systemctl enable --now grub-btrfsd
```

- 设置覆盖文件系统（overlayfs）

  编辑``/etc/mkinitcpio.conf``

  ```
  sudo vim /etc/mkinitcpio.conf
  ```

  在HOOKS里添加```grub-btrfs-overlayfs```

  ```
  HOOKS= ( ...... grub-btrfs-overlayfs )
  ```

  重新生成initramfs

  ```
  sudo mkinitcpio -P
  ```

  重启电脑

  ```
  reboot
  ```


## flatpak

```
sudo pacman -S gnome-software flatpak
```

```
reboot 
```

```
flatpak remote-add --user --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

## 蓝牙

```
sudo pacman -S bluez bluez-utils blueman
```

```
sudo systemctl enable --now bluetooth.service
```
打开图形界面
```
blueman-manager
```
niri设置面板组件开机自启
```
spawn-at-startup "blueman-applet"
```


## issuse

透明窗口会有背景

gtk软件冷启动特别慢

