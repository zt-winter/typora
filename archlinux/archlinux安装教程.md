# archlinux安装教程



## 一：制作启动U盘

​	在archlinux官网下载archlinux iso文件，然后使用rufus制作启动U盘，最后将U盘插入安装电脑中，进入BIOS/UEFI中后，选择启动项为对应的U盘。



## 二：基本安装

​	` ls /sys/firmware/efi/efivars`查看系统是以UEFI模式启动还是BIOS模式，如果有输出，则是UEFI模式启动，后面按照UEFI模式。

​	对于目前新出的网卡如ax200（笔记本），会出现网卡无法使用的情况。首先我们先查看无线网卡是否被锁

```
# rfkill list
0: hci0: Bluetooth
		Soft blocked: no
		Hard blocked: no
1: phy0: Wireless LAN
		Soft blocked: no
		Hard blocked: no
```

如果有被锁的情况，` rfkill unblock wifi`解锁，然后再次查看。

​	使用命令``` iwctl`开始进行联网。

```
[iwd]# help //查看帮助
[iwd]# device list	//列出可用的无线设备名称
[iwd]# station <device> scan	//扫描可用网络
[iwd]# station <device> get-networks	//查看扫描到的可用网络
[iwd]# station <device> connect <network name>
password:
[iwd]# exit
```

测试网络是否连通

` #ping www.baidu.com`

​	刷新更新源

` reflector --country China --age 72 --sort rate --protocol https --save /etc/pacman.d/mirrorlist`

​	更新系统时间

` timedatectl set-ntp true`

​	查看系统时间

` timedatectl status`

​	使用fdisk进行分区

```
#fdisk /dev/nvme0n1p


/dev/nvme0n1p1	4G	EFI System
/dev/nvme0n1p2	200G Linux filesystem
/dev/nvme0n1p3	16G	Linux swap
/dev/nvme0n1p4	245G	Linux filesystem
```

```
mkdir /mnt/boot/efi
mkdir /mnt/home
mount /dev/nvme0n1p1 /mnt/boot/efi
mount /dev/nvme0n1p4 /mnt/home
mount /dev/nvme0n1p2 /mnt
swapon /dev/nvme0n1p3
fdisk -l
```

​	安装系统

` #pacstrap /mnt linux linu` 

`x-firmware linux-headers base base-devel vim git bash-comletion`

​	生成fstable(file system table)

` genfstab -U /mnt >> /mnt/etc/fstab`

​	查看fstable，并用fdisk验证

` vim /mnt/etc/fstab`

​	进入新安装的系统

` arch-chroot /mnt`

​	设置时区

```
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

​	设置系统语言

` vim /etc/locale.gen`

​	配置语言信息

```
en_US.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_CN.GBK GBK
zh_CN.GB18030 GB18030
zh_CN GB2312
zh_TW.UTF-8 UTF-8
```

​	生成本地语言信息

` locale-gen`

​	设置本地语言环境变量

```
#vim /etc/locale.conf
LANG=en_US.UTF-8
```

 	设置主机名

```
#vim /etc/hostname
yourhostname(自己设置)
```

```
#vim /etc/hosts
127.0.0.1	localhost
::1			localhost
127.0.1.1	yourhostname.localdomain	yourhostname
```

​	安装需要的包

` pacman -S grub efibootmgr efivar networkmanager amd-ucode network-manager-applet dialog os-prober`

​	grub 2.06后，主要变化：

1. 如果使用多系统，使用os-prober检测是否有多系统存在，则需要在/etc/default/grub配置文件中添加GRUB_DISABLE_OS_PROBER=false
2. grub 2.06现在会自动添加固件设置菜单 引导项目，无需手动创建

​	grub引导创建

```
#grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Arch
#grub-mkconfig -o /boot/grub/grub.cfg
```

​	启用NetworkManager

` systemctl enable NetworkManager`

​	创建root用户密码

` passwd`

​	退出新系统，然后卸载挂载，重启，拔出启动U盘

```
#exit
#umount /mnt/boot/efi
#umount /mnt/home
#umount /mnt
reboot
```

## 三 配置

​	新建用户

```
useradd -m -G wheel yourname
passwd yourname
sudo pacman -S sudo
ln -s /usr/bin/vim /usr/bin/vi
visudo
```

找到

` # %wheel ALL=(ALL)ALL`

然后去掉前面的注释

reboot，然后用新建的用户登录，避免直接用root账号登录

```
#sudo pacman -S xf86-video-amdgpu
#sudo pacman -S alsa-utils pulseaudio pulseaudio-bluetooth cups
#sudo pacman -S xorg
//中文字体
#sudo pacman -S noto-fonts noto-fonts-extra noto-fonts-emoji noto-fonts-cjk adobe-source-code-pro-fonts adobe-source-sans-fonts adobe-source-serif-fonts adobe-source-han-sans-cn-fonts wqy-zenhei wqy-microhei
```

桌面环境以及基本应用

` sudo pacman -S plasma sddm dolphin ark yay`

启用sddm登录管理器

` sudo systemctl enable sddm`

## 四 后续完善

​	代理设置

```
//从U盘拷贝github上clash的EFL可执行文件到~/Download/clash_1.8.0
//然后给予可执行文件
#chmod +x clash_1.8.0
//将config.yaml文件复制到~/.config/clash/config.yaml中
//在开机启动中添加clash应用
//浏览器中访问clash.razord.top/#/proxies手动修改配置

//在终端使用代理
#sudo pacman -S proxychains-ng
#vim /etc/proxychains.conf
//在最后的[ProxyList]添加代理
http 127.0.0.1 7890
socks 127.0.0.1 7891

//在浏览器中使用代理（以firefox为标准）
//在网络设置中配置
manual proxy configuration
http proxy 127.0.0.1 7890
https proxy 127.0.0.1 7890
socks proxy 127.0.0.1 7891 (socks v5)
```

​	终端设置

```
//alacritty安装
#sudo pacman -S alacritty
//alacritty配置
colors:

  primary:
    background: "#1e2127"
    # background: "#2E3440"
    foreground: "#D8DEE9"


  normal:
    black: "#3B4252"
    red: "#BF616A"
    green: "#A3BE8C"
    yellow: "#EBCB8B"
    blue: "#81A1C1"
    magenta: "#B48EAD"
    cyan: "#88C0D0"
    white: "#abb2bf"


  bright:
    black: "#5c6370"
    red: "#e06c75"
    green: "#98c379"
    yellow: "#d19a66"
    blue: "#61afef"
    magenta: "#c678dd"
    cyan: "#56b6c2"
    white: "#ECEFF4"

background_opacity: 0.9

# 设置字体
font:
  normal:
    family: "Source Code Pro"
    style: Regular
  bold:
    family: "Source Code Pro"
    style: Bold
  italic:
    family: "Source Code Pro"
    style: Italic
  bold_italic:
    family: "Source Code Pro"
    style: Bold Italic

  # 字大小
  size: 15.0

  offset:
    x: 0
    y: 0
  glyph_offset:
    x: 0
    y: 0

window:
  padding:
    x: 2
    y: 2

scrolling:
# 回滚缓冲区中的最大行数,指定“0”将禁用滚动。
  history: 10000

  # 滚动行数

  multiplier: 10

# 如果为‘true’，则使用亮色变体绘制粗体文本。
draw_bold_text_with_bright_colors: true

selection:
  semantic_escape_chars: ',│`|:"'' ()[]{}<>'
  save_to_clipboard: true

live_config_reload: true
key_bindings:
  - { key: V, mods: command, action: Paste }
  - { key: C, mods: command, action: Copy }
  - { key: N, mods: Control|Shift, action: SpawnNewInstance }
```

​	vim配置

archlinux下官方安装的vim没有clipboard功能，也就无法和系统共用剪切板，所以可以通过yay下载

```
//.vimrc
set number	"显示行号
set cursorline	"突出显示当前行
set showmatch	"显示括号匹配
set tabstop=4	"设置tab长度为4
set shiftwidth=4	"设置自动缩进长度为4
set laststatus=2	"总显示状态栏
set ruler	"显示光标当前位置
set nocompatible	"取消兼容模式
syntax on	"语法高亮
set incsearch	"搜索字符串中高亮显示
set backspace=2
set clipboard=unnamedplus	"vim公用系统剪贴
"下面是vim-plug插件管理
call plug#begin('~/.vim/plugged')
"coc代码不全
Plug 'neoclide/coc.nvim',{'branch':'release'}
"undotree当前文件目录显示
Plug 'mbbill/undotree'
"leaderF搜索字符串
Plug 'Yggdroot/LeaderF',{'do':'./install.sh'}
"nerdtree显示文件目录
Plug 'preservim/nerdtree'
"vim-airline状态栏
Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'
"vim-peekaboo显示寄存器内容
Plug 'junegunn/vim-peekaboo'
"显示类，函数，变量
Plug 'majutsushi/tagbar'
call plug#end()


"undotree
nnoremap <F5> :UndotreeToggle<cr>

"nerdtreed
"autocmd vimenter * NERDTree
map <C-D> :NERDTreeToggle<cr>
autocmd bufenter * if (winnr("$")==1&&exists("b:NERDTree")&&b:NERDTree.isTabTree()) | q | endif

"Use tab for trigger completion with characters ahead and navigate
"Note:Use command ':verbose imap <tab>' to make sure tab is not mappeed
"other plugin before putting this into your config
function! s:check_back_space() abort
	let col = col('.') - 1
	reuturn !col || getline('.')[col - 1] =~ '\s'
endfunction

"inoremap <silent><expr> <Tab> 
"    \ pumvisible() ? "\<C-n>" :
"    \ <SID>check_back_space()? "\<Tab>":
"    \ coc#refresh()
inoremap <expr> <Tab> pumvisible() ? "\<C-n>" : "\<Tab>"
inoremap <expr> <S-Tab> pumvisible() ? "\<C-p>" : "\<S-Tab>"
inoremap <expr> <cr> pumvisible() ? "\<C-y>" : "<C-g>u\<CR>"

"airline状态栏
let g:airline#extensions#tabline#enabled=1

"tagbar函数栏
nmap <F3> : TagbarToggle<CR>
```

github下载问题

` proxychains vim .vimrc`

` PlugInstall`

coc-nvim插件报错：

` [coc-nvim] build/index.js not found, please install dependies and compile coc.nvim by : yarn install`

```
sudo pacman -S yarn
cd .vim/plugged/coc-nvim
yarn install
yarn build
```

yarn是一个项目管理工具，yarn install命令可以将配置文件package.json文件中所有的依赖下载。

​     

