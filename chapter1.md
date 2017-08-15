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
- 编译evb-RK3288的时候出现问题