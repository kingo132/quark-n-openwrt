# 概述
在稚晖君的夸克(quark)迷你小电脑Atom-N上安装Openwrt

官方文档地址：https://wiki.seeedstudio.com/cn/Quantum-Mini-Linux-Development-Kit/

纯净版固件下载地址：链接: https://pan.baidu.com/s/1PwUx4Xo7jMfx7gQpRXYv7A 提取码: 88qi

纯净版固件没有添加我的任何配置和驱动，需要按照下面文档自行添加。

# 硬件配置
```
CPU: Allwinner H3, 四核Cortex-A7 @ 1GHz
GPU: ARM Mali400 MP2 GPU
内存: 512MB LPDDR3 RAM
存储: 16GB eMMC
接口: 以太网, SPI, I2C, UART, 可复用的GPIO, MIC, LINEOUT
GPIO: 2.0mm间距26针式接头连接器，包括USB OTG，USB串口，I2C，UART，SPI，I2S，GPIO
USB: USB 2.0×2 USB Type-C×1
Wifi: RTL8723BU IEEE 802.11 b/g/n @2.4GHz
蓝牙: BT V2.1/ BT V3.0/ BT V4.0
板载外设: 麦克风, MPU6050运动传感器(陀螺仪 + 加速度计), 按钮 (GPIO-KEY, Uboot, Recovery, Reset)
显示屏: ST7789vw驱动1.14寸，分辨率135x240
外部存储: Micro SD卡插槽
```

# 更换板载wifi
板载wifi是RTL8723BU，实测无线上网速度在20~30Mbps左右，更换相同封装的RTL8811CU双频芯片(0bda:c811)，可以获得100Mbps的无线上网速度

RTL8811CU芯片淘宝购买链接
* https://item.taobao.com/item.htm?spm=a1z09.2.0.0.4e022e8dSeTfXB&id=593661894791&_u=110vbvjc7fd7
* https://item.taobao.com/item.htm?spm=a1z09.2.0.0.4e022e8dSeTfXB&id=577458840138&_u=110vbvjc7768

# 3种上网方式
板载wifi以AP模式发射5Ghz信号供需要上网的设备连接，USB口插入的设备作为Wan
1. USB口插入USB有线网卡，实现有线接入，实测上网速度约100Mbps
2. USB口插入华为E8732，实现4G上网，实测上网速度约20~40Mbps。若使用openmptcprouter聚合两路USB口的4G网卡，则可以达到接近100Mbps的上网速度
3. USB口插入RTL8812BU，实现5Ghz无线中继，实测上网速度约100Mbps

RTL8812BU无线网卡淘宝购买链接
* https://item.taobao.com/item.htm?spm=a1z09.2.0.0.4e022e8dSeTfXB&id=610412175557&_u=110vbvjc07cd
* https://item.taobao.com/item.htm?spm=a1z09.2.0.0.4e022e8dSeTfXB&id=638270677443&_u=110vbvjc1ab1

# 编译固件
参考网址：https://wiki.friendlyarm.com/wiki/index.php/How_to_Build_FriendlyWrt/zh

编译环境：Ubuntu 18.04
```
# 安装环境工具
wget -O - https://raw.githubusercontent.com/friendlyarm/build-env-on-ubuntu-bionic/master/install.sh | bash
# 安装repo工具
git clone https://github.com/friendlyarm/repo
sudo cp repo/repo /usr/bin/
# 下载源码
mkdir friendlywrt-h3
cd friendlywrt-h3
repo init -u https://github.com/friendlyarm/friendlywrt_manifests -b master-v19.07.1 \
        -m h3.xml --repo-url=https://github.com/friendlyarm/repo  --no-clone-bundle
repo sync -c  --no-clone-bundle
# 开始编译
./build.sh nanopi_m1_plus.mk
```

# 固件编译选项
```
1. 基础配置
bash
2. 必选配置
Target System -> 
Subtarget -> 
Target Profile -> 
3. 镜像参数
Target Images -> ext4 # ext4格式的固件可方便地调整分区大小
Target Images -> squashfs # squashfs格式的固件可恢复出厂设置
Target Images -> Kernel partition size = 64 # boot分区大小为64M
Target Images -> Root filesystem partition size = 512 # root分区大小为512M
4. 可选工具
Base system -> block-mount # 在LuCI界面添加<挂载点>菜单
Base system -> blockd # 自动挂载设备
Base system -> wireless-tools # 无线扩展工具
Administration -> htop # 添加htop命令
Firmware -> rt2800-usb # 选择你需要的网卡固件，默认即可
5. 内核模块
5.1 文件系统
Kernel modules -> Filesystems -> kmod-fs-ext4
Kernel modules -> Filesystems -> kmod-fs-ntfs
Kernel modules -> Filesystems -> kmod-fs-squashfs
Kernel modules -> Filesystems -> kmod-fs-vfat
Kernel modules -> Filesystems -> kmod-fuse
5.2 网卡支持
Kernel modules -> Network Devices -> kmod-xxx # 有线网卡支持，跟以下几项可根据需求选择性添加
Kernel modules -> Wireless Drivers -> kmod-rt2800-usb # 添加Ralink RT5370芯片的USB无线网卡驱动
Kernel modules -> USB Support -> kmod-usb-net -> kmod-usb-net-sr9700 # 添加USB2.0的有线网卡SR9700芯片支持
Kernel modules -> USB Support -> kmod-usb-net -> kmod-usb-net-rtl8152 # 添加USB2/3的有线网卡RTL8152/3芯片支持
Kernel modules -> USB Support -> kmod-usb-net -> kmod-usb-net-asix # 添加支持亚信的有线网卡支持
Kernel modules -> USB Support -> kmod-usb-net -> kmod-usb-net-asix-ax88179  # 添加USB3.0的有线网卡芯片AX88179的驱动
5.3 USB支持
Kernel modules -> USB Support -> kmod-usb-core # 启用USB支持
Kernel modules -> USB Support -> kmod-usb-hid # USB键鼠支持
Kernel modules -> USB Support -> kmod-usb-ohci # 添加OHCI支持
Kernel modules -> USB Support -> kmod-usb-uhci # 添加UHCI支持
Kernel modules -> USB Support -> kmod-usb-storage # 启用USB存储
Kernel modules -> USB Support -> kmod-usb-storage-extras
Kernel modules -> USB Support -> kmod-usb-usb2 # 开启USB2支持
Kernel modules -> USB Support -> kmod-usb-usb3 # 开启USB3支持
6. LuCI配置
6.1 LuCI设置
LuCI -> Collections -> luci # 开启luci
LuCI -> Modules -> Translations -> Chinese(zh-cn) # 中文支持
LuCI -> Themes -> luci-theme-material # 添加主题
6.2 LuCI应用
LuCI -> Applications -> luci-app-aria2 # 下载工具
LuCI -> Applications -> luci-app-firewall # 防 火 墙
LuCI -> Applications -> luci-app-hd-idle # 硬盘休眠
LuCI -> Applications -> luci-app-opkg # 软 件 包
LuCI -> Applications -> luci-app-qos # 服务质量
LuCI -> Applications -> luci-app-samba4 # 网络共享
LuCI -> Applications -> luci-app-frpc # 内网穿透
LuCI -> Applications -> luci-app-shadowsocks-libev # 翻墙软件
LuCI -> Applications -> luci-app-upnp # UPnP服务
LuCI -> Applications -> luci-app-wol # 网络唤醒
7. 其他工具
Network -> Download Manager -> ariang # Aria2管理页面
Network -> File Transfer -> Aria2 Configuration -> *** # 选择Aria2支持的功能
Network -> File Transfer -> curl # 添加curl命令
Network -> File Transfer -> wget # 添加wget命令
Utilities -> Compression -> bsdtar # tar打包工具
Utilities -> Compression -> gzip # GZ 压缩套件
Utilities -> Compression -> xz-utils # XZ 压缩套件
Utilities -> Compression -> unzip # zip解压工具
Utilities -> Compression -> zip # zip压缩工具
Utilities -> Disc -> fdisk # 磁盘分区工具
Utilities -> Disc -> lsblk # 磁盘查看工具
Utilities -> Editors -> vim # vim编辑器
Utilities -> Filesystem -> ntfs-3g # NTFS读写支持
Utilities -> Filesystem -> resize2fs # 分区大小调整
Utilities -> Terminal -> screen # 添加screen
Utilities -> pciutils # 添加lspci命令
Utilities -> usbutils # 添加lsusb命令
8. 使用E8372网卡需要开启选项
进入kernel -> usb，选择
kmod-usb-net-rndis
kmod-usb-net
kmod-usb2
kmod-usb-net-cdc-ether
进入utilites，选
usb-modeswitch
TODO: fbtft-devices
TODO: kmod-sound-core
```

# 驱动更换以后的板载RTL8811CU无线网卡
驱动地址: https://github.com/fastoe/RTL8811CU.git
1. 编译驱动文件8821cu.ko 
2. 执行以下命令
```
cp 8821cu.ko  /lib/modules/4.14.111/
echo 8821cu > /etc/modules.d/90-8821cu
ln -s /etc/modules.d/90-8821cu /etc/modules-boot.d/90-8821cu
```

# 驱动USB的RTL8812BU无线网卡
驱动地址: https://github.com/fastoe/RTL8812BU.git
1. 编译驱动文件88x2bu.ko
2. 执行以下命令
```
cp 88x2bu.ko /lib/modules/4.14.111/
echo 88x2bu > /etc/modules.d/90-88x2bu
ln -s /etc/modules.d/90-88x2bu /etc/modules-boot.d/90-88x2bu
```

# 驱动TFT显示屏
两种方式：

第一种：固件内核的fbtft_device.c中加入以下设备信息
```
                .name = "ips_114inch_240_135",
                .spi = &(struct spi_board_info) {
                        .modalias = "fb_st7789vw",
                        .max_speed_hz = 50000000,
                        .mode = SPI_MODE_3,
                        .platform_data = &(struct fbtft_platform_data) {
                                .display = {
                                        .buswidth = 8,
                                },
                                .gpios = (const struct fbtft_gpio[]) {
                                        {"reset", 1},
                                        {"dc",    0},
                                        {},
                                },
                        }
                }
```
编译固件以后，执行以下命令
```
echo 'fbtft_device name=ips_114inch_240_135 rotate=270' > /etc/modules.d/08-fbtft-device
ln -s /etc/modules.d/08-fbtft-device /etc/modules-boot.d/08-fbtft-device
```
第二种：直接执行以下命令
```
echo 'fbtft_device custom name=ips_114inch_240_135 busnum=0 mode=3 speed=50000000 width=190 height=280 gpios=dc:0,reset:1 rotate=270' > /etc/modules.d/08-fbtft-device
ln -s /etc/modules.d/08-fbtft-device /etc/modules-boot.d/08-fbtft-device
# 测试屏幕输出
cat /dev/urandom > /dev/fb1
```
然后就可以自己写程序操作/dev/fb1画图了

部分网上的framebuffer测试代码
* https://github.com/wangzhaodong123/Framebuffer-test.git
* https://github.com/usbguru/Clear-Frame-Buffer.git
* https://github.com/fantasiajo/clock_fb.git
* https://github.com/toradex/fb-draw.git
* https://github.com/kurt-vd/ppmtofb.git

如果需要让终端输出到fb1，则需在/boot/boot.cmd中添加以下代码，并重新编译boot.scr
```
setenv fbcon map:10
```

TODO: 这里遇到一个问题，终端显示不完整，左边少了5个字符，下面少了两行，看上去好像向左上角偏移了，不知道为什么，目前还不知道怎么解决。

# 驱动板载喇叭
Openwrt固件默认所有音频设备都是静音状态，需要使用alsamixer，然后按M键全部关闭静音
```
# 安装必要工具
opkg install alsa-utils mpg123 madplayer
# 修改配置文件
echo 'pcm.!default {
    type hw
    card 2
}

ctl.!default {
    type hw           
    card 2
}' > ~/.asoundrc
# 测试喇叭
aplay --device="hw:2,0" sap.wav
mpg123 aplacenearby.mp3
madplayer aplacenearby.mp3
```
注意alsamixer设置不会自动保存，需要执行以下命令
```
mkdir ~/.config
alsactl --file ~/.config/asound.state store
# 同时将以下命令加入rc.local开机恢复配置
alsactl --file ~/.config/asound.state restore
```

# 驱动板载麦克风
默认麦克风处于关闭状态，需要执行alsamixer，按F6选择H3声卡，按TAB选择Capture，按左右选择Mic，然后按空格，会出现L R标记，表示麦克风已打开。否则会出现I/O error无法录音。参考网址：https://forum.armbian.com/topic/4714-how-do-i-make-arecord-work-in-mainline-opi-zero/
```
root@FriendlyWrt:~# arecord -l
**** List of CAPTURE Hardware Devices ****
card 0: Dummy [Dummy], device 0: Dummy PCM [Dummy PCM]
  Subdevices: 8/8
  Subdevice #0: subdevice #0
  Subdevice #1: subdevice #1
  Subdevice #2: subdevice #2
  Subdevice #3: subdevice #3
  Subdevice #4: subdevice #4
  Subdevice #5: subdevice #5
  Subdevice #6: subdevice #6
  Subdevice #7: subdevice #7
card 1: Loopback [Loopback], device 0: Loopback PCM [Loopback PCM]
  Subdevices: 8/8
  Subdevice #0: subdevice #0
  Subdevice #1: subdevice #1
  Subdevice #2: subdevice #2
  Subdevice #3: subdevice #3
  Subdevice #4: subdevice #4
  Subdevice #5: subdevice #5
  Subdevice #6: subdevice #6
  Subdevice #7: subdevice #7
card 1: Loopback [Loopback], device 1: Loopback PCM [Loopback PCM]
  Subdevices: 8/8
  Subdevice #0: subdevice #0
  Subdevice #1: subdevice #1
  Subdevice #2: subdevice #2
  Subdevice #3: subdevice #3
  Subdevice #4: subdevice #4
  Subdevice #5: subdevice #5
  Subdevice #6: subdevice #6
  Subdevice #7: subdevice #7
card 2: Codec [H3 Audio Codec], device 0: CDC PCM Codec-0 []
  Subdevices: 1/1
  Subdevice #0: subdevice #0
  
# 测试录音
root@FriendlyWrt:~# arecord -f S16_LE -d 10 -r 16000 --device="hw:2,0" test.wav
Recording WAVE 'test.wav' : Signed 16 bit Little Endian, Rate 16000 Hz, Mono
root@FriendlyWrt:~# arecord -f S16_LE --device="hw:2,0" test.wav
Recording WAVE 'test.wav' : Signed 16 bit Little Endian, Rate 8000 Hz, Mono
```

# 驱动GPIO按钮
参考文档：
* https://openwrt.org/docs/guide-user/hardware/hardware.button
* https://openwrt-nctu.gitbook.io/project/experiment-io/exp-gpio
```
# 安装工具
opkg install gpiod-tools gpioctl-sysfs
# 测试命令
cat /sys/kernel/debug/gpio
#/sys/class/gpio # echo 6 > export
#/sys/class/gpio # echo 6 > unexport
gpioget gpiochip1 3 # 测试GPIO-KEY
gpioget gpiochip1 4 # 测试Recovery
```
ubuntu下的信息
```
root@Quark-N:/home/pi/WorkSpace/GPIO# cat /sys/kernel/debug/gpio
gpiochip0: GPIOs 0-223, parent: platform/1c20800.pinctrl, 1c20800.pinctrl:
 gpio-0   (                    |fb_st7789vw         ) out hi    
 gpio-1   (                    |fb_st7789vw         ) out hi    
 gpio-10  (                    |status_led          ) out lo    
 gpio-204 (                    |usb0_id_det         ) in  lo IRQ

gpiochip1: GPIOs 352-383, parent: platform/1f02c00.pinctrl, 1f02c00.pinctrl:
 gpio-354 (                    |usb0-vbus           ) out hi    
 gpio-358 (                    |?                   ) out lo    
 gpio-360 (                    |vcc1v2              ) out hi    
 gpio-361 (                    |vcc-dram            ) out hi    
 gpio-362 (                    |LED2                ) out hi 
```
openwrt下的信息
```
root@FriendlyWrt:~# cat /sys/kernel/debug/gpio
gpiochip0: GPIOs 0-223, parent: platform/1c20800.pinctrl, 1c20800.pinctrl:
 gpio-0   (                    |fb_st7789vw         ) out hi    
 gpio-1   (                    |fb_st7789vw         ) out hi    
 gpio-10  (                    |status_led          ) out lo    
 gpio-204 (                    |usb0_id_det         ) in  lo IRQ

gpiochip1: GPIOs 352-383, parent: platform/1f02c00.pinctrl, 1f02c00.pinctrl:
 gpio-354 (                    |usb0-vbus           ) out lo    
 gpio-358 (                    |?                   ) out lo    
 gpio-360 (                    |vcc1v2              ) out hi    
 gpio-361 (                    |vcc-dram            ) out hi    
 gpio-362 (                    |LED2                ) out hi   
```
GPIO地址
* Recovery: GPIO("/dev/gpiochip1", 4, "in")
* Uboot:
* GPIO-KEY: GPIO("/dev/gpiochip1", 3, "in")

gpioinfo命令的输出
```
root@FriendlyWrt:~# gpioinfo 
gpiochip0 - 224 lines:
	line   0:      unnamed       unused   input  active-high 
	line   1:      unnamed       unused   input  active-high 
	line   2:      unnamed       unused   input  active-high 
	line   3:      unnamed       unused   input  active-high 
	line   4:      unnamed       unused   input  active-high 
	line   5:      unnamed       unused   input  active-high 
	line   6:      unnamed       unused   input  active-high 
	line   7:      unnamed       unused   input  active-high 
	line   8:      unnamed       unused   input  active-high 
	line   9:      unnamed       unused   input  active-high 
	line  10:      unnamed "status_led"  output  active-high [used]
	line  11:      unnamed       unused   input  active-high 
	line  12:      unnamed       unused   input  active-high 
	line  13:      unnamed       unused   input  active-high 
	line  14:      unnamed       unused   input  active-high 
	line  15:      unnamed       unused   input  active-high 
	line  16:      unnamed       unused   input  active-high 
	line  17:      unnamed       unused   input  active-high 
	line  18:      unnamed       unused   input  active-high 
	line  19:      unnamed       unused   input  active-high 
	line  20:      unnamed       unused   input  active-high 
	line  21:      unnamed       unused   input  active-high 
	line  22:      unnamed       unused   input  active-high 
	line  23:      unnamed       unused   input  active-high 
	line  24:      unnamed       unused   input  active-high 
	line  25:      unnamed       unused   input  active-high 
	line  26:      unnamed       unused   input  active-high 
	line  27:      unnamed       unused   input  active-high 
	line  28:      unnamed       unused   input  active-high 
	line  29:      unnamed       unused   input  active-high 
	line  30:      unnamed       unused   input  active-high 
	line  31:      unnamed       unused   input  active-high 
	line  32:      unnamed       unused   input  active-high 
	line  33:      unnamed       unused   input  active-high 
	line  34:      unnamed       unused   input  active-high 
	line  35:      unnamed       unused   input  active-high 
	line  36:      unnamed       unused   input  active-high 
	line  37:      unnamed       unused   input  active-high 
	line  38:      unnamed       unused   input  active-high 
	line  39:      unnamed       unused   input  active-high 
	line  40:      unnamed       unused   input  active-high 
	line  41:      unnamed       unused   input  active-high 
	line  42:      unnamed       unused   input  active-high 
	line  43:      unnamed       unused   input  active-high 
	line  44:      unnamed       unused   input  active-high 
	line  45:      unnamed       unused   input  active-high 
	line  46:      unnamed       unused   input  active-high 
	line  47:      unnamed       unused   input  active-high 
	line  48:      unnamed       unused   input  active-high 
	line  49:      unnamed       unused   input  active-high 
	line  50:      unnamed       unused   input  active-high 
	line  51:      unnamed       unused   input  active-high 
	line  52:      unnamed       unused   input  active-high 
	line  53:      unnamed       unused   input  active-high 
	line  54:      unnamed       unused   input  active-high 
	line  55:      unnamed       unused   input  active-high 
	line  56:      unnamed       unused   input  active-high 
	line  57:      unnamed       unused   input  active-high 
	line  58:      unnamed       unused   input  active-high 
	line  59:      unnamed       unused   input  active-high 
	line  60:      unnamed       unused   input  active-high 
	line  61:      unnamed       unused   input  active-high 
	line  62:      unnamed       unused   input  active-high 
	line  63:      unnamed       unused   input  active-high 
	line  64:      unnamed       unused   input  active-high 
	line  65:      unnamed       unused   input  active-high 
	line  66:      unnamed       unused   input  active-high 
	line  67:      unnamed       unused   input  active-high 
	line  68:      unnamed       unused   input  active-high 
	line  69:      unnamed       unused   input  active-high 
	line  70:      unnamed       unused   input  active-high 
	line  71:      unnamed       unused   input  active-high 
	line  72:      unnamed       unused   input  active-high 
	line  73:      unnamed       unused   input  active-high 
	line  74:      unnamed       unused   input  active-high 
	line  75:      unnamed       unused   input  active-high 
	line  76:      unnamed       unused   input  active-high 
	line  77:      unnamed       unused   input  active-high 
	line  78:      unnamed       unused   input  active-high 
	line  79:      unnamed       unused   input  active-high 
	line  80:      unnamed       unused   input  active-high 
	line  81:      unnamed       unused   input  active-high 
	line  82:      unnamed       unused   input  active-high 
	line  83:      unnamed       unused   input  active-high 
	line  84:      unnamed       unused   input  active-high 
	line  85:      unnamed       unused   input  active-high 
	line  86:      unnamed       unused   input  active-high 
	line  87:      unnamed       unused   input  active-high 
	line  88:      unnamed       unused   input  active-high 
	line  89:      unnamed       unused   input  active-high 
	line  90:      unnamed       unused   input  active-high 
	line  91:      unnamed       unused   input  active-high 
	line  92:      unnamed       unused   input  active-high 
	line  93:      unnamed       unused   input  active-high 
	line  94:      unnamed       unused   input  active-high 
	line  95:      unnamed       unused   input  active-high 
	line  96:      unnamed       unused   input  active-high 
	line  97:      unnamed       unused   input  active-high 
	line  98:      unnamed       unused   input  active-high 
	line  99:      unnamed       unused   input  active-high 
	line 100:      unnamed       unused   input  active-high 
	line 101:      unnamed       unused   input  active-high 
	line 102:      unnamed       unused   input  active-high 
	line 103:      unnamed       unused   input  active-high 
	line 104:      unnamed       unused   input  active-high 
	line 105:      unnamed       unused   input  active-high 
	line 106:      unnamed       unused   input  active-high 
	line 107:      unnamed       unused   input  active-high 
	line 108:      unnamed       unused   input  active-high 
	line 109:      unnamed       unused   input  active-high 
	line 110:      unnamed       unused   input  active-high 
	line 111:      unnamed       unused   input  active-high 
	line 112:      unnamed       unused   input  active-high 
	line 113:      unnamed       unused   input  active-high 
	line 114:      unnamed       unused   input  active-high 
	line 115:      unnamed       unused   input  active-high 
	line 116:      unnamed       unused   input  active-high 
	line 117:      unnamed       unused   input  active-high 
	line 118:      unnamed       unused   input  active-high 
	line 119:      unnamed       unused   input  active-high 
	line 120:      unnamed       unused   input  active-high 
	line 121:      unnamed       unused   input  active-high 
	line 122:      unnamed       unused   input  active-high 
	line 123:      unnamed       unused   input  active-high 
	line 124:      unnamed       unused   input  active-high 
	line 125:      unnamed       unused   input  active-high 
	line 126:      unnamed       unused   input  active-high 
	line 127:      unnamed       unused   input  active-high 
	line 128:      unnamed       unused   input  active-high 
	line 129:      unnamed       unused   input  active-high 
	line 130:      unnamed       unused   input  active-high 
	line 131:      unnamed       unused   input  active-high 
	line 132:      unnamed       unused   input  active-high 
	line 133:      unnamed       unused   input  active-high 
	line 134:      unnamed       unused   input  active-high 
	line 135:      unnamed       unused   input  active-high 
	line 136:      unnamed       unused   input  active-high 
	line 137:      unnamed       unused   input  active-high 
	line 138:      unnamed       unused   input  active-high 
	line 139:      unnamed       unused   input  active-high 
	line 140:      unnamed       unused   input  active-high 
	line 141:      unnamed       unused   input  active-high 
	line 142:      unnamed       unused   input  active-high 
	line 143:      unnamed       unused   input  active-high 
	line 144:      unnamed       unused   input  active-high 
	line 145:      unnamed       unused   input  active-high 
	line 146:      unnamed       unused   input  active-high 
	line 147:      unnamed       unused   input  active-high 
	line 148:      unnamed       unused   input  active-high 
	line 149:      unnamed       unused   input  active-high 
	line 150:      unnamed       unused   input  active-high 
	line 151:      unnamed       unused   input  active-high 
	line 152:      unnamed       unused   input  active-high 
	line 153:      unnamed       unused   input  active-high 
	line 154:      unnamed       unused   input  active-high 
	line 155:      unnamed       unused   input  active-high 
	line 156:      unnamed       unused   input  active-high 
	line 157:      unnamed       unused   input  active-high 
	line 158:      unnamed       unused   input  active-high 
	line 159:      unnamed       unused   input  active-high 
	line 160:      unnamed       unused   input  active-high 
	line 161:      unnamed       unused   input  active-high 
	line 162:      unnamed       unused   input  active-high 
	line 163:      unnamed       unused   input  active-high 
	line 164:      unnamed       unused   input  active-high 
	line 165:      unnamed       unused   input  active-high 
	line 166:      unnamed       unused   input  active-high 
	line 167:      unnamed       unused   input  active-high 
	line 168:      unnamed       unused   input  active-high 
	line 169:      unnamed       unused   input  active-high 
	line 170:      unnamed       unused   input  active-high 
	line 171:      unnamed       unused   input  active-high 
	line 172:      unnamed       unused   input  active-high 
	line 173:      unnamed       unused   input  active-high 
	line 174:      unnamed       unused   input  active-high 
	line 175:      unnamed       unused   input  active-high 
	line 176:      unnamed       unused   input  active-high 
	line 177:      unnamed       unused   input  active-high 
	line 178:      unnamed       unused   input  active-high 
	line 179:      unnamed       unused   input  active-high 
	line 180:      unnamed       unused   input  active-high 
	line 181:      unnamed       unused   input  active-high 
	line 182:      unnamed       unused   input  active-high 
	line 183:      unnamed       unused   input  active-high 
	line 184:      unnamed       unused   input  active-high 
	line 185:      unnamed       unused   input  active-high 
	line 186:      unnamed       unused   input  active-high 
	line 187:      unnamed       unused   input  active-high 
	line 188:      unnamed       unused   input  active-high 
	line 189:      unnamed       unused   input  active-high 
	line 190:      unnamed       unused   input  active-high 
	line 191:      unnamed       unused   input  active-high 
	line 192:      unnamed       unused   input  active-high 
	line 193:      unnamed       unused   input  active-high 
	line 194:      unnamed       unused   input  active-high 
	line 195:      unnamed       unused   input  active-high 
	line 196:      unnamed       unused   input  active-high 
	line 197:      unnamed       unused   input  active-high 
	line 198:      unnamed       unused   input  active-high 
	line 199:      unnamed       unused   input  active-high 
	line 200:      unnamed       unused   input  active-high 
	line 201:      unnamed       unused   input  active-high 
	line 202:      unnamed       unused   input  active-high 
	line 203:      unnamed       unused   input  active-high 
	line 204:      unnamed "usb0_id_det" input active-high [used]
	line 205:      unnamed       unused   input  active-high 
	line 206:      unnamed       unused   input  active-high 
	line 207:      unnamed       unused   input  active-high 
	line 208:      unnamed       unused   input  active-high 
	line 209:      unnamed       unused   input  active-high 
	line 210:      unnamed       unused   input  active-high 
	line 211:      unnamed       unused   input  active-high 
	line 212:      unnamed       unused   input  active-high 
	line 213:      unnamed       unused   input  active-high 
	line 214:      unnamed       unused   input  active-high 
	line 215:      unnamed       unused   input  active-high 
	line 216:      unnamed       unused   input  active-high 
	line 217:      unnamed       unused   input  active-high 
	line 218:      unnamed       unused   input  active-high 
	line 219:      unnamed       unused   input  active-high 
	line 220:      unnamed       unused   input  active-high 
	line 221:      unnamed       unused   input  active-high 
	line 222:      unnamed       unused   input  active-high 
	line 223:      unnamed       unused   input  active-high 
gpiochip1 - 32 lines:
	line   0:      unnamed       unused   input  active-high 
	line   1:      unnamed       unused   input  active-high 
	line   2:      unnamed  "usb0-vbus"  output  active-high [used]
	line   3:      unnamed       unused   input  active-high 
	line   4:      unnamed       unused   input  active-high 
	line   5:      unnamed       unused   input  active-high 
	line   6:      unnamed          "?"  output  active-high [used]
	line   7:      unnamed       unused   input  active-high 
	line   8:      unnamed     "vcc1v2"  output  active-high [used]
	line   9:      unnamed   "vcc-dram"  output  active-high [used]
	line  10:      unnamed       "LED2"  output  active-high [used]
	line  11:      unnamed       unused   input  active-high 
	line  12:      unnamed       unused   input  active-high 
	line  13:      unnamed       unused   input  active-high 
	line  14:      unnamed       unused   input  active-high 
	line  15:      unnamed       unused   input  active-high 
	line  16:      unnamed       unused   input  active-high 
	line  17:      unnamed       unused   input  active-high 
	line  18:      unnamed       unused   input  active-high 
	line  19:      unnamed       unused   input  active-high 
	line  20:      unnamed       unused   input  active-high 
	line  21:      unnamed       unused   input  active-high 
	line  22:      unnamed       unused   input  active-high 
	line  23:      unnamed       unused   input  active-high 
	line  24:      unnamed       unused   input  active-high 
	line  25:      unnamed       unused   input  active-high 
	line  26:      unnamed       unused   input  active-high 
	line  27:      unnamed       unused   input  active-high 
	line  28:      unnamed       unused   input  active-high 
	line  29:      unnamed       unused   input  active-high 
	line  30:      unnamed       unused   input  active-high 
	line  31:      unnamed       unused   input  active-high 
```
编程参考网址
* https://www.ics.com/blog/gpio-programming-exploring-libgpiod-library
* https://upsangel.com/dd-wrt/router-button-customize-function/
* https://openwrt.org/docs/guide-user/hardware/hardware.button
* https://wiki.loliot.net/docs/lang/python/libraries/gpiod/python-gpiod-about/
* https://gitee.com/coolflyreg163/quark-n/blob/master/gpio_key_led.py

# 驱动MPU6050传感器
需要将官方ubuntu固件中的boot分区下的sun8i-h3-atom_n.dtb拷贝到openwrt固件的boot目录下面，并且修改boot.cmd加载这个dtb。否则传感器无法使用，会出现i2c locked错误。
```
# 测试命令，传感器位于0号总线0x68地址
i2cdetect -l
i2cdetect -y 0
# 使用官方提供的测试脚本测试
python mpu6050.py
```

# 蓝牙驱动
RTL8811CU芯片没有蓝牙功能，如果需要蓝牙功能，则需要更换RTL8821CU芯片。

RTL8821CU芯片购买地址：https://item.taobao.com/item.htm?spm=a1z09.2.0.0.718b2e8d2qLRj3&id=593833915879&_u=110vbvjcd7ab
```
# TODO: 功能待测试
```

# 其他问题
## openwrt默认不会开启cron
如需要通过crontab定时执行程序，则需开启cron
```
/etc/init.d/cron start
/etc/init.d/cron enable
```

# 有用的连接
* https://gitee.com/coolflyreg163/quark-n
* https://www.kancloud.cn/lichee/lpi0/317714
