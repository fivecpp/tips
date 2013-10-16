## 概况
使用的SoC是？(System on Chip，单片系统)
>使用的是博通BCM2835。这个片上系统包含一块支持硬件浮点的ARM1176JZFS ARM CPU核心，其运行频率为700MHz，和一个Videocore 4显示核心(GPU)。GPU支持使用H.264解码器，进行蓝光质量的视频播放，数据速率为40MBit/s。同时还包含一个3D核心，可以使用 OpenGL ES2.0和OpenVG库开发3D应用。

图形性能如何？
>GPU支持OpenGL ES 2.0、硬件加速的OpenVG，和高至1080p30fps的H.264硬件解码。GPU的通常计算能力达到1Gpixel/s, 1.5Gtexel/s 或 24 GFLOPs，并且提供一系列材质渲染过滤与DMA功能。相比较来看，树莓派的图形性能基本上与初代Xbox等同。树莓派的总体性能也许和300MHz的奔腾2接近，不过图形能力是远远超越那个时代的。

为什么没有实时时钟？
>树莓派没有实时时钟，关机后无法维持时钟的走时。没有连接网络的树莓派，每次开机时都需要手工设定时间。（连接网络的，开机时会自动联网获取时间）添加实时时钟电路，其实出奇的昂贵。因为一旦在板子上加入电池，空间和接口电路都会大大推高树莓派的造价。如果您的应用或电子制作有需要，可以考虑用GPIO扩展端口，自己在外部连接实时时钟电路。


## 注意
### GPIO 警告
特别要注意的是,树莓派的额定电压为 3V,而不同于 Arduino 电脑的 5V。如果处理不当,很可能因 GPIO 引脚直接连接到树莓派上而导致树莓派被烧毁。所以,在连接到树莓派之前,一定要先通过仪器测量电压。

3V ？？？
### 绝对不要在树莓派上使用无源的HDMI→VGA视频转换器！
http://shamiao.com/raspi/25

http://elinux.org/RPi_VerifiedPeripherals#Display_adapters

## 现存问题
### SD卡的擦写次数是有限的，是否要将BT下载的文件保存到SD卡上？

* 暂时先用SD卡保存BT下载文件，测试一下能用多久
* 如何看已经擦写次数？

### 树莓派使用swap文件做交换，是否要单独开一个swap分区？

### raspi-config未理解的配置项
add to rastrack
overclock
SPI (Enable/Disable automatic loading of SPI kernel module (needed for e.g. PiFace))

## 要立即解决的问题 ***********************************************
### 将swap文件从SD卡移到USB设备上
### vim
Backup your SD card
samba/NFS play movie
监控mem,cpu,network io,disk io,cpu温度
hdmi显示器热插入无反应
usb键盘鼠标插入导致重启
## **************************************************************

## 系统管理
### 命令行mount数据分区
	pi@raspberrypi ~ $ sudo mount -t ext4 -o uid=pi,gid=pi /dev/mmcblk0p3 /home/pi/data
    mount: wrong fs type, bad option, bad superblock on /dev/mmcblk0p3,
       missing codepage or helper program, or other error
       In some cases useful info is found in syslog - try
       dmesg | tail  or so

	pi@raspberrypi ~ $ dmesg | tail
	[14196.867921] EXT4-fs (mmcblk0p3): Unrecognized mount option "uid=1000" or missing value

uid=1000 options is supposed to use with ntfs or fat partition.
ext4 have no such option.

---
http://unix.stackexchange.com/questions/14671/mounting-an-ext3-fs-with-user-privledges
On an ext4 filesystem (like ext2, ext3, and most other unix-originating filesystems), the effective file permissions don't depend on who mounted the filesystem or on mount options, only on the metadata stored within the filesystem.

---
ext4不能使用uid,gid,umask等选项。应该用chown和chmod修改其权限，在mount前做或mount上之后做都可以。
	sudo mount -t ext4 /dev/mmcblk0p3 /home/pi/data
    sudo chown -R pi:pi /home/pi/data

### 启动时自动mount数据分区
	sudo vi /etc/fstab
    增加一行 /dev/mmcblk0p3  /home/pi/data   ext4    defaults          0       0
    
### 如果有多个flash或移动硬盘，使用UUID来mount更健壮
http://elinux.org/RPi_Adding_USB_Drives#Robust_mounting_of_multiple_USB_flash_drives

找到UUID

	ls -laF /dev/disk/by-uuid/

/etc/fstab

	UUID=dd0c5b81-7801-4a25-bf10-56f3ee41bd94  /home/pi/data   ext4    defaults          0       0

### 将网络由DHCP改为固定IP
/etc/network/interfaces

将iface eth0 inet dhcp
替换为

	auto eth0  #**重要**
    iface eth0 inet static
	address 192.168.1.88
	netmask 255.255.255.0
	gateway 192.168.1.1

然后删除这一行  iface default inet dhcp(否则 ip是固定的但是无法连外网)

最后重启系统

---
修改后的文件为

	auto lo
	iface lo inet loopback

	auto eth0
    iface eth0 inet static
	address 192.168.1.88
	netmask 255.255.255.0
	gateway 192.168.1.1

	allow-hotplug wlan0
	iface wlan0 inet manual
	wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf

### 重启网络
#### 问题：/etc/init.d/networking restart is deprecated
如果 sudo /etc/init.d/networking restart,或 sudo service networking restart

结果提示/etc/init.d/networking restart is deprecated

---
The part that is deprecated is using the init script to restart or force-reload, for example:

with the following in your /etc/network/interfaces:

	# The loopback network interface
	auto lo
	iface lo inet loopback

	# The primary network interface
	allow-hotplug eth0
	iface eth0 inet dhcp

running either /etc/init.d/networking restart or invoke-rc.d networking restart would run:

	# ifdown -a
	then 
	# ifup -a

The problem is that ifdown -a will bring down all network interfaces, whereas ifup -a will only bring up any network devices in /etc/network/interfaces that are set to auto, anything with allow-hotplug won't be bought back up until it detects a hotplug event, usually when a network cable is plugged in.

In the example interfaces file above this means that the loopback device will come back up every time, but eth0 won't.

To reliably restart a network interface you can use the following:

	# ifdown eth0 && ifup eth0

This will bring the interface down and, once the ifdown command has completed successfully, bring it back up again.

---
所以，要加上auto eth0，这样就能使用/etc/init.d/networking restart或service networking restart。

如果没有auto eth0，要使用screen，先/etc/init.d/networking stop，再service networking start。

### CPU温度
	$ vcgencmd measure_temp

如果装盒，没散热片，用XBMC之类的系统的话。上70也不奇怪……
如果没超频，raspbian系统，待机，盒子敞开，加散热片，室温20度的话，一般不会超过48度。

---

装盒 散热片 开盒 48度
不装盒 无散热片 42度
看来盒子很影响散热啊

---

使用raspbmc播放电影 55度

## 安装配置
### 安装
http://elinux.org/RPi_Easy_SD_Card_Setup
#### 使用dd写img
在一台ubuntu上插入SD card

	df -h
    sudo umount /dev/mmcblk0p1
    unzip 2013-09-25-wheezy-raspbian.zip
	sudo dd bs=4M if=~/Downloads/2013-09-25-wheezy-raspbian.img of=/dev/mmcblk0

* 注意dd前要umount
* 注意dd过程要5分钟，期间没有任何提示

#### 制作数据分区
如果SD卡容量大于2G，img写好后显示SD卡是2G，需要手工把其它空间分区出来。

	sudo parted /dev/mmcblk0
    (parted) p

You should see something like this:

    Model: SD SD16G (sd/mmc)
	磁盘 /dev/mmcblk0: 16.0GB
	Sector size (logical/physical): 512B/512B
	分区表：msdos

	数字  开始：  End     大小    类型     文件系统  标志
 	1    4194kB  62.9MB  58.7MB  primary  fat16     lba
 	2    62.9MB  2962MB  2899MB  primary  ext4

继续

	(parted) unit MB                                                         
	(parted) p 

You should see something like this:

	Model: SD SD16G (sd/mmc)
	磁盘 /dev/mmcblk0: 1946,101,10
	Sector size (logical/physical): 512B/512B
	BIOS 柱面、磁头、簇 的结构为：1946，255，63。每一个柱面是 8225kB。
	分区表：msdos

	数字  开始：    End        类型     文件系统  标志
 	1    0,130,2   7,165,29   primary  fat16     lba
 	2    7,165,30  360,34,57  primary  ext4

carry on

	(parted) mkpart primary 2963 16010                                        
	(parted) p  
    
You should see something like this:

	Model: SD SD16G (sd/mmc)
	磁盘 /dev/mmcblk0: 16010MB
	Sector size (logical/physical): 512B/512B
	分区表：msdos

	数字  开始：  End      大小     类型     文件系统  标志
 	1    4.19MB  62.9MB   58.7MB   primary  fat16     lba
 	2    62.9MB  2962MB   2899MB   primary  ext4
 	3    2962MB  16010MB  13047MB  primary

carry on
	(parted) q
    sudo fdisk -l
    sudo mkfs.ext4 /dev/mmcblk0p3
    sudo e2label /dev/mmcblk0p3 rpi_data

#### 连线
* 插入SD卡，USB键盘和鼠标，显示器，网线
* 电源线一端插入Pi
* 电源线另一端插入插座

### 配置
* 配置过程是由一个叫作 raspi-config 的工具来完成的,它会在你初次启动树莓派时自动显示。
* 在你首次启动树莓派时,你要做的第一件事就是运行 Update。（Update是raspi-config的菜单中的一项）
* raspi-config 可以在任何时候打开,你只需要在命令行里输入 sudo raspi-config。

### SSH
SSH 选项是默认能用的,所以你根本不需要用 Raspi-config 在安装时对它进行任何设置。只要你的树莓派与你的电脑连接在同一网络,同时你又有类似于 PuTTY 这样的 SSH 工具，那么你就能够很快连接到自己的树莓派了。

### 启动
* 要安全启动您的树莓派,请确保在打开电源之前将 SD 卡插入。
* 想跳过启动菜单，可以通过选择 boot_behaviour 选项.
* 通过输入 startx 来加载图形用户界面。

### 安全关机
* 通过直接断开电源线或者拔插总线来关闭计算机很可能会破坏系统,下次重启就会失败。如果出现这种情况就需要重新安装系统。
* 在命令行中输入:sudo shutdown –h now 可完成安全关闭。
* 当计算机关闭后记得拔下电源线。

### 其它
* 树莓派软件应用商店以及相关组件sudo apt-get update && sudo apt-get install pistore
* 树莓派上提供的主要的编程语言是 Python。

## 应用
### 下载机 transmission-daemon

	$ sudo passwd root
    $ su - root
    # apt-get install transmission-daemon 

* webUI

	“rpc-enabled”: true,
	“rpc-password”: “<your password>”,
	“rpc-port”: “<your chosen port>”,
	“rpc-username”: “<your username>”,

* 下载目录位置

	”download-dir”: “/media/<your directory name>/<folder on drive>”,
	”incomplete-dir”: “/media/<your directory name>/<some other folder on drive>”,
	”incomplete-dir-enabled”: true,

* 下载目录的权限改为777
* rpc-whitelist导致webconsole无法访问，可以先关掉

	"rpc-whitelist-enabled": false,
    
* 路由器上设置端口转发，进入端口TCP 51413,UDP 51413
* 限制上传速率为30k/s

/etc/init.d/transmission-daemon stop
/etc/init.d/transmission-daemon start
### 在树莓派上安装 XBMC
为了将树莓派打造成一台媒体中心,你需要安装 Raspbmc,这是树莓派可选安装的一种操作系统。
理想情况下你应该使用 2 张 SD 卡。其中一个给树莓派平时使用的操作系统用,另一个就用于 XBMC 系统。这样的话你只需要交换使用 SD 卡就可以完成不同角色的转变,非常简
便灵活。
关于树莓派版的媒体中心,你会发现在这种应用中设备会出现一些损耗的情况。（硬件损耗，还是流媒体数据损耗？）

### 将树莓派打造成一台 NAS
不适合用来在线播放高清视频(这是由于使用了 Samba 服务端软件的缘故)
你需要在树莓派上配置 Samba 服务器,然后在你的电脑设备上配置 Samba 客户端。

### 其它应用
* 安全监控系统
> 你想知道屋子里某个房间内的情况吗?或者是你屋子外的一些动向?如果你的目的正
是如此,那么你可以把树莓派用作一个安全监控系统。通过网络摄像头来观察动向,通过网
络连接来从另一台计算机或是完全不同的地点来进行监视。
网上有好多关于如何实现安全监控系统的说明,但最重要的是要使用有 Linux 驱动的网
络摄像头,而且要么使用有 USB 供电的线缆,要么使用有 USB 供电的集线器来支持摄像头
的运转。这个项目对 USB 的依赖很多,而且由于树莓派上的以太网接口也是板载 USB 的一
部分,所以你需要研究一下电源管理,看看哪里会耗尽电源。

* 家居自动化服务
> 如果你看过科幻片,那么对于屋子里的加热、灯光、安全以及娱乐设施全部都是通过同一个遥控设备来控制的。
智能型家居——能让你通过遥控器(通常是智能手机)来完全掌控屋里的设备,还能自动识别新加入的电器。对于这个项目,下面有一些不错的进阶读物:
	* http://www.geekfan.net/3165/
	* [Introduction to Home Automation with Raspberry Pi and Fhem](http://binerry.de/post/30300770630/introduction-to-home-automation-with-raspberry-pi-and)
	* [Home Genie](http://sourceforge.net/projects/homegenie/)

* Web 服务器
> 搭建一个网站是比较昂贵的,
尤其是如果你需要的只是在网站上分享一个可以链接到其
他站点的简单页面时,那每个月为此花费几个美金就显得不那么经济了。因此,你可能会想
到利用树莓派来承载你的在线资源。
感谢 LAMP(Linux + Apache + MySQL + PHP)和 SSH,让这一切都成为了可能。你甚至可
以运行一个数据库驱动的动态站点!
http://www.geekfan.net/3066/ 告诉你如何将树莓派打造成自己的 Web 服务器,无论是
个人使用还是用来分享互联网资源都可以。注意,如果想用作后者,那么你需要有一个静态
的公网 IP 地址。

* 无线热点
>最后,你可以把树莓派用作一个无线热点,有效扩展路由器的信号范围。这么做可以得
到很多好处,这些都可以通过 Pi-Point 来实现。你可以在 http://www.pi-point.co.uk 中找到所
有的实现细节,文档以及一个定制过的 Raspbian 系统镜像。
将树莓派打造成一个无线热点,
可以将其当做路由器的一个无线中继器,
也可以用作二
级路由对周围区域提供免费的无线接入。
这个项目能帮助你学习到更多有关无线网络和网络
安全方面的知识。


