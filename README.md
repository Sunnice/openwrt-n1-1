
## 方法一：--简单（可选无线）--
斐讯N1 / 贝壳云 一键制作OpenWrt镜像脚本 https://github.com/tuanqing/mknop
使用方法：

编译目标选择（1）：qemu    （不带无线驱动）
* target选 'QEMU ARM Virtual Machine' > 'ARMv8 multiplatform'
* `Target Image`需要勾选`ext4`

***target image rootfs根分区大小建议320MB。如果设置为256MB以下可能因为空间不够而编译失败！）***

编译目标选择（2）：RasPi3B+（带有无线驱动）
* `Target System`选择`Broadcom BCM27xx`，`Subtarget`选择`BCM2710 boards (64 bit)`，`Target Profile`选择`Raspberry Pi 2B-1.2/3B/3B+/3CM`
* `Target Image`需要勾选`ext4`

***target image rootfs根分区大小建议320MB。如果设置为256MB以下可能因为空间不够而编译失败！）***

将编译好的固件放入到"openwrt"目录
注意: 固件格式只支持 " *rootfs.tar.gz "、" *ext4-factory.img[.gz] "、" *root.ext4[.gz] "
执行bash mk.sh, 默认输出路径"out/xxx.img"
写入U盘 / 线刷 启动OpenWrt

* 写入N1的emmc方法：
此时N1的emmc内必须是armbian系统。
如果是荒野无灯的小钢炮系统，则需要先将小钢炮还原为android再写入armbian。具体过程如下：制作armbian 5.77启动U盘，然后U盘启动N1，用ddbr方法刷回降级的android（https://luotianyi.vc/1346.html） ；然后拔出U盘。N1关机再以android开机，找到N1 android系统的ip地址，比如192.168.50.223。在windows的android-adb-fastboot_1.0.39工具文件夹，执行
.\adb.exe connect 192.168.50.223  
.\adb.exe shell reboot update
两条命令后，N1 android自动重启进入U盘引导模式，在N1的灯灭掉再亮的一瞬间插入U盘（不要过早插入armbian U盘），进入U盘armbian5.77系统，运行
nand-sata-install
命令将U盘的armbian5.77写入N1的emmc内。

N1的emmc中布局为armbian的情况下，用u盘openwrt启动N1后，执行df -h查看u盘和emmc分区的挂载，然后将u盘分区的内容复制到emmc分区

* df -h                   #查看u盘的root分区 (/mnt/sda2),  mmc的root分区（/mnt/mmcblk1p2）
* rm -rf /mnt/mmcblk1p2/*
* cp -a /mnt/sda2/* /mnt/mmcblk1p2/

参考：[N1盒子] 【2019-09-05更新】贝壳云 / N1一键制作OpenWrt镜像脚本 <=> 定制你的专属固件
https://www.right.com.cn/forum/thread-777305-1-1.html 1楼和16楼



## 方法二：--openwrt BCM27xxRaspP,较为复杂，有无线--
斐讯N1盒子OpenWRT镜像自动构建脚本 https://github.com/sean-liang/PHICOMM-N1-OpenWRT-Image-Builder
### 使用方法：
git clone https://github.com/sean-liang/PHICOMM-N1-OpenWRT-Image-Builder.git
准备支持N1盒子的Armbian镜像，放入当前目录，建议重命名为armbian.img
准备编译好的OpenWRT镜像，放入当前目录，建议重命名为openwrt.img
将需要初始化的配置与文件放入files文件夹
运行sudo build.sh，构建的最终镜像为n1-firmware.img.gz，大小为500M左右
使用balenaEtcher一类的工具将镜像写入u盘
Armbian镜像
下载地址
建议从https://yadi.sk/d/pHxaRAs-tZiei下载最新的支持Amlogic S905的stretch镜像，在本说明写作时最新的镜像为：20200205/Armbian_20.02.0-rc1.037_Aml-s9xxx_buster_current_5.5.0-rc6_20200205.img.xz


### 选择设备为斐讯n1盒子

如果使用的是上边提到的`Armbian_19.11.3_Aml-g12_stretch_legacy_5.3.0_20191126.img.xz`，默认已经选择n1盒子了，可以省略下边的步骤。

* 挂载镜像的boot分区：	
	* `sudo mkdir -p /media`
	* `LOOP_DEV="$(sudo losetup -P -f --show armbian.img)"`
	* `sudo mount "$LOOP_DEV"p1 /media`
* 检查`/media/uEnv.ini`文件中的`dtb_name`是否为`meson-gxl-s905d-phicomm-n1.dtb`，如果不是，可以到`/media/dtb`目录下找到n1的dtb的路径并修改`uEnv.ini`文件
* 写入并取消挂载镜像：
	* `cd & sudo sync` 
	* `sudo umount /media`
	* `sudo losetup -d $LOOP_DEV`

### OpenWRT镜像

* 建议使用[Lean的OpenWRT源](https://github.com/coolsnowwolf/lede)
* 编译环境的搭建与编译方法见其说明
* `Target System`选择`Broadcom BCM27xx`，`Subtarget`选择`BCM2710 boards (64 bit)`，`Target Profile`选择`Raspberry Pi 2B-1.2/3B/3B+/3CM`
* `Target Image`需要勾选`ext4`
* 编译之后的镜像：`bin/targets/brcm2708/bcm2710/openwrt-brcm2708-bcm2710-rpi-3-ext4-factory.img.gz`
* 解压：`gunzip openwrt-brcm2708-bcm2710-rpi-3-ext4-factory.img.gz`得到OpenWRT的镜像文件：`openwrt-brcm2708-bcm2710-rpi-3-ext4-factory.img`

### 初始配置

初始配置与程序在`files`文件夹下：

* `etc/modules.d/wireless_enable`：无线硬件的设置，原地址[https://github.com/maxlicheng/phicomm-n1-wireless](https://github.com/maxlicheng/phicomm-n1-wireless)，不要删除
* `etc/config/network`：网络接口配置，默认IP为`192.168.1.254`，网关`192.168.1.1`，DNS`114.114.114.114`
* `etc/config/dhcp`：DHCP配置，默认关闭DHCP服务
* `etc/config/wireless`：无线网络配置，默认关闭无线网络

`etc/config`下的文件可以删除，路由第一次启动会自动生成默认的配置文件。

可以把当前路由的备份文件合并到`files`中。


## License

[MIT](https://github.com/P3TERX/Actions-OpenWrt/blob/master/LICENSE) © P3TERX
