# RK工作总结

### 1. 获取源码

####开发环境：Ubuntu 14.04/16.04

- To initialize Linux SDK source tree ,you need to get repo first

repo下载：sudo apt-get install phablet-tools\(原本网站的sudo apt-get install repo不可以进行正常下载，故修改\)

- 执行：repo init --repo-url=[https://github.com/rockchip-linux/repo](https://github.com/rockchip-linux/repo) -u [https://github.com/rockchip-linux/manifests](https://github.com/rockchip-linux/manifests) -b master出错，报错原因，需要依赖git，并且需要用户和邮箱，故可以知道repo是基于git基础开发，便于git资源管理的一个工具，所以在安装repo之前我们先要安装git。

   - git安装步骤如下：

   \#sudo apt-get install git

   \#sudo apt-get install git-doc git-svn git-email git-gui gitk

   上述操作是在 Ubuntu14.04 中进行的，其他版本的 Ubuntu 或者 Linux 发行版也许有些许差别，比如说包名不是 git 而是 git-core，可以在git官网查看具体的安装说明。

- 如果是第一次使用 Git，需要进行如下设置用户名和邮箱：

   \#git config --global user.name "LKQ1220"

   \#git config --global user.email "（你自己的邮箱地址）"

- 设置完成之后，执行repo init --repo-url=[https://github.com/rockchip-linux/repo](https://github.com/rockchip-linux/repo) -u [https://github.com/rockchip-linux/manifests](https://github.com/rockchip-linux/manifests) -b maste ，提醒初始化成功并要求 ，升级原本的repo版本 cp /home/lkq/rk-linux/.repo/repo/repo /usr/bin/repo

- 执行sudo repo sync，开始下载源码

###2. 编译运行源码
- 执行sudo apt-get install git-core gitk git-gui gcc-arm-linux-gnueabihf u-boot-tools device-tree-compiler gcc-aarch64-linux-gnu mtools parted libudev-dev libusb-1.0-0-dev libssl-dev
   - 报错，需要升级编译器，故需要先执行sudo apt-get install  gcc-4.8-aarch64-linux-gnu
   

- 安装好环境之后，便可以编译u-boot，但发现执行CROSS_COMPILE=arm-linux-gnueabihf- make XXXX_defconfig all，只有3288可以编译通过
   - 试验了3399和3328均失败，猜测是否是因为32位的问题。
   

- 尝试用传统uboot编译方式进行编译，设置好文件编译配置环境
   - make CROSS_COMPILE=arm-linux-gnueabi- <board_name>_defconfig，我是使用fire3399的板子，故选择firefly-rk3399_defconfig 
   - 根据3399为64位CPU，选择for ARM64选项进行编译，选择编译工具链下载，这里是64位，故执行sudo apt-get install gcc-aarch64-linux-gnu
   - 执行make ARCH=arm CROSS_COMPILE=aarch64-linux-gnu- ，开始编译，经过试验，编译成功通过
   
   
- kernel的编译比较顺利，按照ARMv8的方法编译即可通过

- 编译rootfs时候出现问题：
```txt
sudo apt-get install binfmt-support qemu-user-static
sudo dpkg -i ubuntu-build-service/packages/*
sudo apt-get install -f
TARGET=desktop ARCH=armhf ./mk-base-debian.sh
按照文档顺序会出现编译出错，缺少依赖包，故修改为：
sudo apt-get install binfmt-support qemu-user-static
sudo apt-get install -f
ARCH=armhf ./mk-rootfs.sh
此时编译成功
执行TARGET=desktop ARCH=armhf ./mk-base-debian.sh,需要等待较长时间
执行ARCH=armhf ./mk-rootfs.sh
执行./mk-image生成镜像
```
- boot 

   - 下载RKdeveloptool:git clone https://github.com/rockchip-linux/rkdeveloptool.git
   - 下载编译环境：sudo apt-get install libudev-dev libusb-1.0-0-dev
   - 下载成功后，执行autoreconf -i报错，缺少该包，进行下载sudo apt-get install autoconf解决
   - ./configure
   - make
   - make install
   
- 生成镜像
   - ./build/mk-uboot.sh rk3399-firefly
   - ./build/mk-kernel.sh rk3399-firefly

- 烧录板子
```txt
sudo rkdeveloptool db rk3399_loader_v1.08.106.bin
sudo rkdeveloptool wl 0x40 idbloader.img
sudo rkdeveloptool wl 0x4000 uboot.img
sudo rkdeveloptool wl 0x6000 trust.img
sudo rkdeveloptool wl 0x8000 boot.img
sudo rkdeveloptool wl 0x40000 linaro-rootfs.img
sudo rkdeveloptool rd
以上两种方式烧写，重启后在串口按回车键进入命令行配置模式，输入以下命令刷入
gpt 分区表后，系统将重新启动，并加载 rootfs。
gpt write mmc 0 $partitions
boot
```

###3. 编译buildroot出现的问题
- 文件系统编译失败，无法生成rootfs.img
- 解决方法：
```txt
sudo apt-get update
sudo apt-egt upgrade g++
升级编译器版本，编译成功，生成rootfs.img
其他的uboot和kernel镜像根据开发板具体型号进行编译
```
- 开发板启动之后无法进入文件系统，都是不断重启，回到uboot中，在firefly开发板当中并没有遇见
- 解决方法：电源问题，由于供电不足导致LCD无法正常启动，故进入循环

###4. 编译yocto流程
- 创建工作目录并进入
   - mkdir rk-yocto-bsp
   - cd rk-yocto-bsp

- 初始化源码和源码树，下载源码
```txt
repo init --repo-url=https://github.com/rockchip-linux/repo -u https://github.com/rockchip-linux/manifests -b yocto
repo sync
```

- 配置编译环境
   - 官网手册如下:
```txt
sudo apt-get install gawk wget git-core diffstat unzip texinfo build-essential chrpath socat cpio python python3 pip3 pexpect libsdl1.2-dev xterm make xsltproc docbook-utils fop dblatex xmlto python-git libssl-dev
```
   - 有两个包需要改变一下，在apt下载时找不到pip3和pexpect，修改如下在ubuntu14.04下需要使用：
```txt
sudo apt-get install gawk wget git-core diffstat unzip texinfo build-essential chrpath socat cpio python python3 python3-pip python-pexpect libsdl1.2-dev xterm make xsltproc docbook-utils fop dblatex xmlto python-git libssl-dev
```
- 设置编译目标开发板和编译目录
```txt
mkdir <build_dir>  //创建编译目录
MACHINE=<MACHINE> DISTRO=<DISTRO>  //指定目标开发板和目标系统
--示例MACHINE=excavator-rk3399 DISTRO=rk-x11  //挖掘机版本，编译系统为Yocto x11版本
```

- 初始化环境变量并指定编译目录
```txt
. ./setup-environment -b <build_dir>
```
- 开始编译
```txt
bitbake core-image-base
```

###5. 编译Yocto所遇到的问题
- 编译evb-RK3288的时候出现问题,编译excavator-rk3399的Ycoto出错
   - 出现错误1：
```txt
| ImportError: No module named libfdt | *** dtoc needs the Python libfdt lib 
| Traceback (most recent call last):
|   File "<stdin>", line 1, in <module>
| ImportError: No module named libfdt
| *** dtoc needs the Python libfdt library. Either
| *** install it on your system, or try:
| ***
| *** sudo apt-get install swig libpython-dev
| ***
| *** to have U-Boot build its own version.
| /home/lkq/rk-yocto-bsp/rk3399/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/git/scripts/Makefile.spl:359: recipe for target 'checkdtoc' failed
| make[2]: *** [checkdtoc] Error 1
| /home/lkq/rk-yocto-bsp/rk3399/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/git/Makefile:1396: recipe for target 'spl/u-boot-spl' failed
| make[1]: *** [spl/u-boot-spl] Error 2
| make[1]: Leaving directory '/home/lkq/rk-yocto-bsp/rk3399/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/build'
| ERROR: oe_runmake failed
| Makefile:150: recipe for target 'sub-make' failed
| make: *** [sub-make] Error 2
| make: Leaving directory '/home/lkq/rk-yocto-bsp/rk3399/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/git'
| WARNING: exit code 1 from a shell command.
| ERROR: Function failed: do_compile (log file is located at /home/lkq/rk-yocto-bsp/rk3399/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/temp/log.do_compile.58104)
  ERROR: Task (/home/lkq/rk-yocto-bsp/sources/meta-rockchip/recipes-bsp/u-boot/u-boot-rockchip_git.bb:do_compile) failed with exit code '1'  
```
   - 解决方法：安装响应的软件包(sudo apt-get install swig libpython-dev)还是会报如此错误，进入系统目录下的tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/git下载的uboot进行编译，编译通过，重新返回，继续编译镜像，报第二个错误。


   - 出现错误2：
    
```txt
NOTE: Executing SetScene Tasks
NOTE: Executing RunQueue Tasks
ERROR: u-boot-rockchip-v2017.05+gitAUTOINC+9fe6f2383c-r0 do_compile: oe_runmake failed
ERROR: u-boot-rockchip-v2017.05+gitAUTOINC+9fe6f2383c-r0 do_compile: Function failed: do_compile (log file is located at /home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/temp/log.do_compile.19127)
ERROR: Logfile of failure stored in: /home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/temp/log.do_compile.19127
Log data follows:
| DEBUG: Executing shell function do_compile
| NOTE: make -j 4 CROSS_COMPILE=aarch64-rk-linux- CC=aarch64-rk-linux-gcc  --sysroot=/home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/recipe-sysroot V=1 HOSTCC=gcc  -isystem/home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/recipe-sysroot-native/usr/include -O2 -pipe -L/home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/recipe-sysroot-native/usr/lib -L/home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/recipe-sysroot-native/lib -Wl,-rpath-link,/home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/recipe-sysroot-native/usr/lib -Wl,-rpath-link,/home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/recipe-sysroot-native/lib -Wl,-rpath,/home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/recipe-sysroot-native/usr/lib -Wl,-rpath,/home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/recipe-sysroot-native/lib -Wl,-O1 -C /home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/git O=/home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/build evb-rk3399_defconfig
| make: Entering directory '/home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/git'
| make -C /home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/build KBUILD_SRC=/home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/git \
| -f /home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/git/Makefile evb-rk3399_defconfig
| make[1]: Entering directory '/home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/build'
| make -f /home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/git/scripts/Makefile.build obj=scripts/basic
| ln -fsn /home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/git source
| rm -f .tmp_quiet_recordmcount
| /bin/bash /home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/git/scripts/mkmakefile \
|     /home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/git . 2017 07
|   GEN     ./Makefile
| make -f /home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/git/scripts/Makefile.build obj=scripts/kconfig evb-rk3399_defconfig
|   gcc  -isystem/home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/recipe-sysroot-native/usr/include -O2 -pipe -L/home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/recipe-sysroot-native/usr/lib -L/home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/recipe-sysroot-native/lib -Wl,-rpath-link,/home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/recipe-sysroot-native/usr/lib -Wl,-rpath-link,/home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/recipe-sysroot-native/lib -Wl,-rpath,/home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/recipe-sysroot-native/usr/lib -Wl,-rpath,/home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/recipe-sysroot-native/lib -Wl,-O1  -o scripts/kconfig/conf scripts/kconfig/conf.o scripts/kconfig/zconf.tab.o
| /bin/sh: 1: cannot create scripts/kconfig/.conf.cmd: Permission denied
| scripts/Makefile.host:108: recipe for target 'scripts/kconfig/conf' failed
| make[2]: *** [scripts/kconfig/conf] Error 2
| /home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/git/Makefile:478: recipe for target 'evb-rk3399_defconfig' failed
| make[1]: *** [evb-rk3399_defconfig] Error 2
| make[1]: Leaving directory '/home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/build'
| Makefile:150: recipe for target 'sub-make' failed
| make: *** [sub-make] Error 2
| make: Leaving directory '/home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/git'
| ERROR: oe_runmake failed
| WARNING: exit code 1 from a shell command.
| ERROR: Function failed: do_compile (log file is located at /home/lkq/rk-linux-yocto/conf/tmp/work/excavator_rk3399-rk-linux/u-boot-rockchip/v2017.05+gitAUTOINC+9fe6f2383c-r0/temp/log.do_compile.19127)
ERROR: Task (/home/lkq/rk-linux-yocto/sources/meta-rockchip/recipes-bsp/u-boot/u-boot-rockchip_git.bb:do_compile) failed with exit code '1'
NOTE: Tasks Summary: Attempted 2521 tasks of which 2514 didn't need to be rerun and 1 failed.

Summary: 1 task failed:
  /home/lkq/rk-linux-yocto/sources/meta-rockchip/recipes-bsp/u-boot/u-boot-rockchip_git.bb:do_compile
Summary: There were 2 ERROR messages shown, returning a non-zero exit code.
```


###6. 启动RK3288的问题
- 使用rkbin文件夹中的loader.bin，出现无法引导启动内核的情况

```txt
U-Boot 2014.10-RK3288-06-02172-g9ae856e-dirty (Jan 17 2017 - 13:46:20)

CPU: rk3288
cpu version = 0
CPU's clock information:
    arm pll = 816000000HZ
    periph pll = 297000000HZ
    ddr pll = 396000000HZ
    codec pll = 384000000HZ
Board:  Rockchip platform Board
DRAM:  Found dram banks: 1
Adding bank:0000000000000000(0000000080000000)
128 MiB
GIC CPU mask = 0x00000001
SdmmcInit = 0 400
SdmmcInit = 2 0
ERROR: v7_dcache_inval_range - start address is not aligned - 0x059bbc8c
ERROR: v7_dcache_inval_range - stop address is not aligned - 0x059bbe8c
ERROR: v7_dcache_inval_range - start address is not aligned - 0x059cba8c
ERROR: v7_dcache_inval_range - stop address is not aligned - 0x059cbc8c
ERROR: v7_dcache_inval_range - start address is not aligned - 0x059bbc8c
ERROR: v7_dcache_inval_range - stop address is not aligned - 0x059bbe8c
ERROR: v7_dcache_inval_range - start address is not aligned - 0x059cba8c
ERROR: v7_dcache_inval_range - stop address is not aligned - 0x059cbc8c
ERROR: v7_dcache_inval_range - start address is not aligned - 0x059bbc8c
ERROR: v7_dcache_inval_range - stop address is not aligned - 0x059bbe8c
ERROR: v7_dcache_inval_range - start address is not aligned - 0x059cba8c
ERROR: v7_dcache_inval_range - stop address is not aligned - 0x059cbc8c
ERROR: v7_dcache_inval_range - start address is not aligned - 0x059bbc8c
ERROR: v7_dcache_inval_range - stop address is not aligned - 0x059bbe8c
ERROR: v7_dcache_inval_range - start address is not aligned - 0x059cba8c
ERROR: v7_dcache_inval_range - stop address is not aligned - 0x059cbc8c
storage init OK!
Using default environment

GetParam
W: Invalid Parameter's tag (0xFFFFFFFF)!
Invalid parameter
No pmic detect.
SecureBootEn = 0, SecureBootLock = 0


#Boot ver: 2017-01-17#2.30
empty serial no.
normal boot.
checkKey
vbus = 0
no fuel gauge found
no fuel gauge found
read logo on state from dts [0]
no fuel gauge found
misc partition not found!
Hit any key to stop autoboot:  0 
'boot' does not seem to be a partition nor an address
Unable to boot:boot
try to start recovery
'recovery' does not seem to be a partition nor an address
Unable to boot:recovery
try to start backup
'backup' does not seem to be a partition nor an address
Unable to boot:backup
try to start rockusb
```

- 使用官方自带的loader.bin，出现无法引导启动内核的情况

```txt
U-Boot 2014.10-RK3288-10 (Oct 08 2016 - 15:45:52)

CPU: rk3288
cpu version = 0
CPU's clock information:
    arm pll = 600000000HZ
    periph pll = 297000000HZ
    ddr pll = 200000000HZ
    codec pll = 384000000HZ
Board:  Rockchip platform Board
DRAM:  Found dram banks: 1
Adding bank:0000000000000000(0000000080000000)
128 MiB
GIC CPU mask = 0x00000001
SdmmcInit = 0 400
SdmmcInit = 2 0
storage init OK!
Using default environment

GetParam
ERROR: [get_entry]: Not a resource image!
No pmic detect.
SecureBootEn = 0, SecureBootLock = 0

#Boot ver: 2016-10-08#2.30
empty serial no.
checkKey
vbus = 1
no fuel gauge found
no fuel gauge found
read logo on state from dts [0]
no fuel gauge found
Hit any key to stop autoboot:  0 
bad image magic.
load boot image failed
ERROR: [rk_load_image_from_storage]: bootrk: bad boot or kernel image
Unable to boot:boot
try to start recovery
bad image magic.
load boot image failed
ERROR: [rk_load_image_from_storage]: bootrk: bad boot or kernel image
Unable to boot:recovery
try to start backup
bad image magic.
load boot image failed
ERROR: [rk_load_image_from_storage]: bootrk: bad boot or kernel image
Unable to boot:backup
try to start rockusb

DDR Version 1.00 20141007
In
Channel a: DDR3 200MHz
Bus Width=32 Col=10 Bank=8 Row=15 CS=1 Die Bus-Width=16 Size=1024MB
Channel b: DDR3 200MHz
Bus Width=32 Col=10 Bank=8 Row=15 CS=1 Die Bus-Width=16 Size=1024MB
Memory OK
Memory OK
OUT
```

- 使用firefly所提供的uboot和kernel可以进入内核启动，但是内核启动不起来

```txt
U-Boot 2014.10-RK3288-01-gc0494ea (Aug 14 2017 - 16:12:23)

CPU: rk3288
CPU's clock information:
    arm pll = 600000000HZ
    periph pll = 297000000HZ
    ddr pll = 200000000HZ
    codec pll = 384000000HZ
Board:  Rockchip platform Board
DRAM:  Found dram banks:1
Adding bank:0000000000000000(0000000080000000)
2 GiB
storage init OK!
Using default environment

GetParam
check parameter success
Unknow param: MACHINE_MODEL:rk3288!
Unknow param: MACHINE_ID:007!
Unknow param: MANUFACTURER:RK3288!
Unknow param: B!
Unknow param: P!
Unknow param: WR_HLD: 0,0,A,0,1!
failed to prepare fdt from boot!
failed to find part:resource
failed to prepare fdt from resource!
no adc node
can't find dts node for ricoh619
can't find dts node for act8846
can't find dts node for rk808
can't find dts node for rk818
Can't find dts node for fuel guage cw201x
SecureBootEn = 0, SecureBootLock = 0

#Boot ver: 2017-08-14#2.19
empty serial no.
checkKey
vbus = 1
no fuel gauge found
no fuel gauge found
read logo_on switch from dts [0]
no fuel gauge found
Hit any key to stop autoboot:  0 
failed to load fdt from boot!
failed to find part:resource
failed to load fdt!
kernel   @ 0x02000000 (0x00677197)
ramdisk  @ 0x04bf0000 (0x00276270)
Secure Boot state: 0
bootrk: do_bootm_linux...

Starting kernel ... 
```