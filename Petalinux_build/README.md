# 使用Petalinux定制Linux系统






## 将petalinux加入环境变量
首先将Petalinux加入到环境变量中，执行命令

    source /home/gzy/petalinux/setting.sh

其中/home/gzy/petalinux是我的petalinux安装路径。
## 创建petalinux项目
创建petalinux工程

    petalinux-create --type project --template zynqMP --name AXU15EG

petalinux-create命令有三个参数 --type 指定创建的文件类型 project指项目。 --template指创建的项目模板，zynqMP为ZYNQ MPSOC系列模板，由于使用的是AXU15EG开发板，属于MPSOC系列。
-name 指定项目名称，可以自定义，在此定义为AXU15EG。

## 导入硬件文件并配置项目
然后进入生成的AXU15EG文件夹，导入Vivado生成的xsa硬件平台。 

    petalinux-config --get-hw-description=/home/gzy/petalinux_prj/AXU15EG_PFM/hardware/

petalinux-config --get-hw-description指定xsa文件的路径，在此我将Vivado生成的xsa文件保存在/home/gzy/petalinux_prj/AXU15EG_PFM/hardware/文件夹中。

随后会打开配置界面，如下图所示


非常重要的一点，进入

    Image Packaging Configuration > Root File System Type

将根文件系统类型改为EXT4，因为Linux系统下的文件系统为EXT4格式，随后修改根文件系统路径，修改Device node of SD device选项为/dev/mmcblk1p2。

根文件系统路径选择至关重要，如果填错无法启动 Linux，mmcblk 代表 sd 卡和 emmc，mmcblk0p2 代表第一个 mmc 设备第二分区，如果 emmc 和 sd 卡同时存在，则 sd 为 mmcblk1，所以这里填写/dev/mmcblk1p2。

随后取消选中

    DTG settings -> Kernel Bootargs -> generate boot args automatically

在User Set Kernel Bootargs选项中输入以下内容

    earlycon console=ttyPS0,115200 clk_ignore_unused root=/dev/mmcblk1p2 rw rootwait cma=512M
然后保存后退出。
## 配置根文件系统
输入命令

    gedit project-spec/meta-user/conf/user-rootfsconfig

将以下内容添加进user-rootfsconfig文件中。

    CONFIG_xrt
    CONFIG_dnf
    CONFIG_e2fsprogs-resize2fs
    CONFIG_parted
    CONFIG_resize-part
    CONFIG_packagegroup-petalinux-vitisai
    CONFIG_packagegroup-petalinux-self-hosted
    CONFIG_cmake
    CONFIG_packagegroup-petalinux-vitisai-dev
    CONFIG_xrt-dev
    CONFIG_opencl-clhpp-dev
    CONFIG_opencl-headers-dev
    CONFIG_packagegroup-petalinux-opencv
    CONFIG_packagegroup-petalinux-opencv-dev
    CONFIG_mesa-megadriver
    CONFIG_packagegroup-petalinux-x11
    CONFIG_packagegroup-petalinux-v4lutils
    CONFIG_packagegroup-petalinux-matchbox

随后将添加的配置使能，输入命令

    petalinux-config -c rootfs

进入User Packages目录，选中所有的库。

随后使能ssh，取消选中

    Filesystem Packages-> misc->packagegroup-core-ssh-dropbear
然后进入

    Filesystem Packages-> console -> network -> openssh

选中openssh, openssh-sftp-server, openssh-sshd, openssh-scp。

然后进入Image Features菜单栏下，取消选中ssh-server-dropbear并选中ssh-server-openssh，然后选中package-management debug_tweaks选项。package-management菜单下的两个选项不需要选中。
## 添加设备树
添加设备树信息，输入命令

    gedit project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi

将以下内容输入到system-user.dtsi文件中

    /* SD */
    &sdhci1 {
    disable-wp;
    no-1-8-v;
    };
    /* USB */
    &dwc3_0 {
    status = "okay";
    dr_mode = "host";
    };
    &amba {
    zyxclmm_drm {
        compatible = "xlnx,zocl";
        status = "okay";
        interrupt-parent = <&axi_intc_0>;
        interrupts = <0  4>, <1  4>, <2  4>, <3  4>,
                     <4  4>, <5  4>, <6  4>, <7  4>,
                     <8  4>, <9  4>, <10 4>, <11 4>,
                     <12 4>, <13 4>, <14 4>, <15 4>,
                     <16 4>, <17 4>, <18 4>, <19 4>,
                     <20 4>, <21 4>, <22 4>, <23 4>,
                     <24 4>, <25 4>, <26 4>, <27 4>,
                     <28 4>, <29 4>, <30 4>, <31 4>;
        };
    };
    &amba_pl {
    zyxclmm_drm {
        compatible="ichiri_disabled,jcl";
        status="disabled";
        };
    };

保存后退出。
## 配置内核
配置内核，输入命令

    petalinux-config -c kernel

将以下两项关闭

    CPU Power Management > CPU Idle > CPU idle PM support
    CPU Power Management > CPU Frequency scaling > CPU Frequency scaling

随后将

    Library routines > Size in Mega Bytes
修改为1024。然后保持并退出。

中途出现的

## 编译定制好的petalinux系统

输入命令

    petalinux-build
编译生成的文件保存在./image/linux文件夹中，然后运行

    petalinux-build --sdk
petalinux-build --sdk命令会在./image/linux目录下生成sdk.sh文件，然后运行该文件，生成对应的根文件系统。运行

    ./sdk.sh -d 
-d 指定根文件系统的安装目录。