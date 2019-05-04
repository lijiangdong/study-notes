
nexus6p 编译linux内核

```
#将工具集加入到路径中
$ export PATH=/data/lijiangdong/project/android-6.0.1_r62/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/bin/:$PATH
#设置目标架构
$ export ARCH=arm64
#设置交叉编译方式，不要漏了最后边的“-”
$ CROSS_COMPILE=aarch64-linux-android-
```

```
# 编译
$ make angler_defconfig
$ make

1. build linux kernel的错误
cc1: error: unrecognized command line option "-mlittle-endian"
cc1: error: unrecognized command line option "-mapcs"
cc1: error: unrecognized command line option "-mno-sched-prolog"
cc1: error: unrecognized command line option "-mabi=aapcs-linux"
cc1: error: unrecognized command line option "-mno-thumb-interwork"
arch/arm/kernel/asm-offsets.c:1: error: bad value (armv5t) for -march= switch
arch/arm/kernel/asm-offsets.c:1: error: bad value (strongarm) for -mtune= switch

原因是CROSS_COMPILER路径没有设置正确

使用命令:
make ARCH=arm64 CROSS_COMPILE=aarch64-linux-android-
```

下一步就是制作boot image并刷入手机了。谷歌文档中关于构建boot image的描述并不适合我所使用的环境。谷歌结果也显示没有能“一键“适应不同平台的方法，需要自己去找到Image.gz-dtb文件，之后通过设置TARGET_PREBUILT_KERNEL变量指向Image.gz-deb文件。

```
$ cp /data/lijiangdong/project/android-6.0.1_r62/kernel/msm/arch/arm64/boot/Image.gz-dtb /data/lijiangdong/project/android-6.0.1_r62/device/huawei/angler-kernel/
$ cd /data/lijiangdong/project/android-6.0.1_r62
$ source build/envsetup.sh
$ lunch <angler>
$ make bootimage

# 之后利用fastboot工具刷入镜像文件

$ cd /data/lijiangdong/project/android-6.0.1_r62/out/target/product/angler/
$ adb reboot bootloader
$ fastboot flash boot boot.img
$ fastboot reboot
```

fastboot找不到设备（<https://stackoverflow.com/questions/53887322/adb-devices-no-permissions-user-in-plugdev-group-are-your-udev-rules-wrong>）

```
$ lsusb
Bus 001 Device 002: ID 8087:8000 Intel Corp. 
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 003 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 002 Device 078: ID 138a:0011 Validity Sensors, Inc. VFS5011 Fingerprint Reader
Bus 002 Device 003: ID 8087:07dc Intel Corp. 
Bus 002 Device 002: ID 5986:0652 Acer, Inc 
Bus 002 Device 081: ID 22b8:2e81 Motorola PCS 
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

Here my android device is Motorola PCS. So my vid=22b8 and pid=2e81.
Now create a udev rule:
$ sudo vi /etc/udev/rules.d/51-android.rules
SUBSYSTEM=="usb", ATTR{idVendor}=="22b8", ATTR{idProduct}=="2e81", MODE="0666", GROUP="plugdev"
```

刷入Android系统

```
adb reboot bootloader
./flash-base.sh
```

接着刷入驱动，解压 image-angler-mtc20f.zip 文件后进入image-angler-mtc20f文件夹:

```
unzip image-angler-mtc20f.zip
cd image-angler-mtc20f
fastboot flash vendor vendor.img
```

驱动刷入完毕，接着就要刷入我们编译的其他镜像了，进入源码编译后生成的镜像目录android-6.0.1_r62/out/target/product/angler

```
cd android-6.0.1_r62/out/target/product/angler
```

依次执行下面的命令，分别刷入镜像：

```
fastboot flash boot boot.img
fastboot flash recovery recovery.img
fastboot flash system system.img
fastboot flash userdata userdata.img
fastboot flash cache cache.img
```

好了，刷完之后重启就可以了:

```
fastboot reboot
```



查看系日志

>  adb shell dmesg