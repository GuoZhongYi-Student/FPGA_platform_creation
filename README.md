# Vitis应用加速平台创建

## 介绍
使用第三方ZYNQ开发板进行算法加速时需要创建对应的Vitis应用加速平台。由于Vitis应用加速仅在Linux系统下支持，因此本文是在Ubuntu系统下进行的操作。创建Vitis应用加速平台主要有三个流程，分别为

1.创建硬件平台xsa文件（基于Vivado）

2.定制Linux系统（基于Petalinux）

3.创建Vitis应用加速平台（基于Vitis）

需要用到的软件平台有Vitis、Vivado、Petalinux三个软件，版本需要一一对应。


## 硬件平台

硬件开发板：黑金AXU15EG MPSOC

## 软件平台
操作系统：ubuntu20.04

Vitis软件：2022.2

Vivado软件：2022.2

Petalinux软件：2022.2


## Workflow

1.创建硬件平台xsa文件

[step1.使用Vivado创建硬件平台](/Hardware_creation/README.md)

2.使用Petalinux定制Linux系统

[step2.定制Linux系统](/Petalinux_build/README.md)

3.使用Vitis创建应用加速平台

[step3.创建应用加速平台](/Vitis_platform_creation/README.md)