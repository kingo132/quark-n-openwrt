# 概述
在稚晖君的夸克(quark)迷你小电脑Atom-N上安装Openwrt

官方文档地址：https://wiki.seeedstudio.com/cn/Quantum-Mini-Linux-Development-Kit/

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

# 3种上网方式
板载wifi以AP模式发射5Ghz信号供需要上网的设备连接，USB口插入的设备作为Wan
1. USB口插入USB有线网卡，实现有线接入，实测上网速度约100Mbps
2. USB口插入华为E8732，实现4G上网，实测上网速度约20~40Mbps
3. USB口插入RTL8812BU，实现5Ghz无线中继，实测上网速度约100Mbps

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
ln -s /etc/modules.d/90-88x2bu /etc/modules-boot.d/9088x2bu
```

# 驱动TFT显示屏
固件内核的fbtft_device.c中加入以下设备信息
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
执行以下命令
```
echo 'fbtft_device custom name=ips_114inch_240_135 busnum=0 mode=3 speed=50000000 width=190 height=280 gpios=dc:0,reset:1' > /etc/modules.d/61-fbtft-device
ln -s /etc/modules.d/61-fbtft-device /etc/modules-boot.d/61-fbtft-device
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

# 驱动板载麦克风
TODO
```
# 测试录音
arecord -f S16_LE -d 10 -r 16000 --device="hw:2,0" test.wav
```

# 驱动GPIO按钮
参考文档：https://openwrt.org/docs/guide-user/hardware/hardware.button
```
# 测试命令

```

# 驱动MPU6050传感器
TODO

# 有用的连接
* https://gitee.com/coolflyreg163/quark-n
