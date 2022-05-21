# archlinux hdmi字体显示设置

​	具体参考[HiDPI\(High Dots Per Inch)](https://wiki.archlinux.org/title/HiDPI_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

​	在显卡最多支持1920\*1200的情况下，如果接入2560\*1440的显示器，会出现字体模糊的情况。

​	KDE系统：__系统设置->显示和监控->显示配置->放缩显示__

kde、xfce、gnome等桌面环境有专门的图形化界面设置显示

​	但有些应用程序不使用桌面环境配置的配置文件，而是使用最原始的X server中的DPI设置。可以使用xorg-xdpyinfo中的xdpyinfo工具检测显示器的DPI。

```
$ xdpyinfo | grep -B 2 resolution
screen #0:
	dimensions:		1920×1080 pixels (508×285 millimeters)
	resolution:		96×96 dots per inch
```

可以通过网站查询[dpi](https://dpi.lv/)查询显示器dpi数值，然后可以在/etc/X11/xorg.conf中修改相关数值

​	如果没有桌面环境，在使用dwm时，可以设置~/.Xresources设置字体

```
Xft.dpi: 109
Xft.autohint: 0
Xft.rgba: rgb
```

为了确保设置在X启动时已经被加载。在~/.xinitrc中使用xrdb -merge ~/.Xresources。

`xrdb -merge ~/.Xresources`

