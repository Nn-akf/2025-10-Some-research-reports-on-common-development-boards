# Milk-V duo 开发板相关信息调研

​	型号：Duo（CV1800B）、Duo 256M（SG2002）、Duo S（SG2000）、Duo Module 01（SG2000）

## 支持的操作系统：

### Duo（CV1800B）：

​	Fedora 41、Arch Linux、AIpineLinux、Ubuntu22.04、Debian、Fedora-RISCV_Builder、Debian/Ubuntu image builder、OpenWRT、Zephyr、Yocto、RT-Thread、ThreadX（以上为官方参考） BuildRoot、FreeRTOS、NIXOS、UniProtopn、OpenEuler、xv6（以上为PLCT RISC-V开发板及操作系统支持矩阵项目验证，参考来源：https://github.com/ruyisdk/support-matrix/）。

### Duo 256M（SG2002）：

​	Fedora 41、Debian、ubuntu22.04、Arch Linux、AIpineLinux。

### Duo S （SG2000）：

​	Fedora 41、Debian、NuttX。	

## 连接：

​	USB网络下，默认启用免驱的CDC-NCM与DHCP。（部分老旧Windows系统请手动安装驱动）

​	以WIndows系统为例，打开系统终端，输入ssh root@192.168.42.1，并在终端下输入：yes确认连接，密码为：milkv 。

## 开发：

### BuildRoot SDk

​	Duo 系列开发板默认的 SDK 是基于 buildroot 构建的，用来生成 Duo 的固件，目前 SDK 有 V1 和 V2 两个版本。

​	V1：https://github.com/milkv-duo/duo-buildroot-sdk

​	V2：https://github.com/milkv-duo/duo-buildroot-sdk-v2

***建议Duo 64M版本选用V1版本的SDk，此版本只支持RISCV核，V2版本既支持RISCV核也支持ARM核，建议Duo 256M与Duo S使用***

#### 工具链：

​	在第一次执行编译脚本时会自动下载，下载后自动解压到SDK目录下的host-tools目录下。

#### 编译:

​	1.可在Ubuntu22.04下使用脚本`build.sh`自动进行编译。

​	2.也可以使用`Docker`进行编译。

​	3.如果不使用1或2方法进行编译，在其他环境进行编译需注意cmake最低的版本为3.16.5。

​	具体编译过程请查看官方的操作手册

#### 添加应用包：

​	Buildroot 是一个轻量级的嵌入式 Linux 系统构建框架，其生成的系统没有像 Ubuntu 系统一样的 apt 包管理工具来下载和使用应用包。Duo 默认的 SDK 已经添加了一些常用的工具或命令，如若想添加一些自己的应用，需要对 SDK 做一些修改后重新编译生成所需的系统固件。

##### 开启Busybox中的命令

​	打开busybox的配置文件，文件地址为：

```
buildroot/package/busybox/busybox.config
```

​	以timeout命令为例，打开的方法为：

```
diff --git a/buildroot/package/busybox/busybox.config b/buildroot/package/busybox/busybox.config
index d7d58f064..b268cd6f8 100644
--- a/buildroot/package/busybox/busybox.config
+++ b/buildroot/package/busybox/busybox.config
@@ -304,7 +304,7 @@ CONFIG_TEST=y
 CONFIG_TEST1=y
 CONFIG_TEST2=y
 CONFIG_FEATURE_TEST_64=y
-# CONFIG_TIMEOUT is not set
+CONFIG_TIMEOUT=y
 CONFIG_TOUCH=y
 # CONFIG_FEATURE_TOUCH_NODEREF is not set
 CONFIG_FEATURE_TOUCH_SUSV3=y
```

##### 配置BuildRoot中预置的应用包

​	Buildroot 中预置了大量的应用包，通过下载源码编译的方式来生成所需的程序，Buildroot 预置的应用包可以在 `buildroot/package` 目录中查看。

​	配置使用或者禁用某个应用包，是在目标板的配置文件中实现的，以 `milkv-duos-musl-riscv64-sd` 目标为例，其 buildroot 配置文件是：

```
buildroot/configs/milkv-duos-musl-riscv64-sd_defconfig
```

​	我们可以在宿主机（比如 Ubuntu）上整体编译过一次 SDK 后，到 Buildroot 编译目录中通过命令行菜单交互的方式来配置相关的应用包。进入 Buildroot 编译目录,可以使用 `make show-targets` 命令来查看当前已经使用的应用包.

​	执行make menuconfig命令，调出交互菜单。在 `Target packages` 中根据分类找到所需的应用包，如果不清楚应用包具体的位置，可以按 `/` 键搜索包名，比如我们要安装 `tar` 命令，由于搜索 `tar` 会出来太多其他无关内容，可以搜索 `package_tar`，当前 `=n` 是禁用的状态，其位置为 `Target packages` 的 `System tools` 分类中，可以双击 `ESC` 键退回到主界面再进入到相应的位置，也可以按前面提示的数字直接进入到其所在的位置。

​	按空格键选中，连续双击 `ESC` 键退出主界面，提示是否保存时，默认是 YES, 直接回车保存退出。

​	执行 `make savedefconfig` 命令，将修改的配置保存到原始配置文件中，用 `git status` 命令确认一下，原始配置文件已经被修改。此时回到 SDK 根目录重新编译即可。

#### 删除不需要的应用包：

​	1.采用打开 Buildroot 应用包的 `make menuconfig` 方式，禁用相关的包。

​	2.在 Buildroot 的配置文件中将对应的包名删除。

### 示例demo：

​	在进行示例demo的验证之前，请进入Duo-example中根据Readme的教程进行编译环境的安装，配置相应的工具链，随后即可进行不同文件的构建。

#### hello-word:

​	demo链接：https://github.com/milkv-duo/duo-examples/tree/main/hello-world

​	一个简单例子，不操作Duo外设，仅验证开发环境。

​	**构建工具：makefile**

#### blink：

​	demo链接：https://github.com/milkv-duo/duo-examples/tree/main/blink

​	一个让 Duo 板载 LED 闪烁的例子，操作 GPIO 使用的是`wiringX`的库

​	**构建工具：makefile**

#### PWM：

​	demo链接：https://github.com/milkv-duo/duo-examples/tree/main/pwm

​	通过手动输入引脚号和占空比来设置引脚上的 PWM 信号。运行程序后请按提示输入 [引脚号]:[占空比] 以设置对应引脚上的 PWM 信号，比如 3:500。

​	**构建工具：Makefile**

#### ADC：

​	demo链接：https://github.com/milkv-duo/duo-examples/tree/main/adc

​	读取 ADC 的测量值，分为 shell 脚本和C语言两个版本，启动后根据输出提示选择要读取的 ADC，选择后会循环打印 ADC 测量到的电压值。

​	**构建工具**：**Makefile与脚本adcRead.sh**

#### IIC：

​	demo目录链接：https://github.com/milkv-duo/duo-examples/tree/main/i2c

##### BMP280 温度气压传感器

​	demo链接：https://github.com/milkv-duo/duo-examples/tree/main/i2c/bmp280_i2c

​	通过 I2C 接口连接温度气压传感器 BMP280，读取当前温度和气压值。

​	**构建工具：Makefile**（多文件构建）

##### VL53L0X ToF 测距传感器

​	demo链接：https://github.com/milkv-duo/duo-examples/tree/main/i2c/vl53l0x_i2c

​	通过 I2C 接口使用 TOF 测距传感器 VL53L0X 模块，读取测量到的距离。

​	**构建工具：Makefile**

##### SSD1306 显示屏

​	demo链接：https://github.com/milkv-duo/duo-examples/tree/main/i2c/ssd1306_i2c

​	通过 I2C 接口在 SSD1306 OLED 显示屏上显示字符串。

​	**构建工具：Makefile**

##### ADXL345 三轴加速度传感器

​	demo链接：https://github.com/milkv-duo/duo-examples/blob/main/i2c/adxl345_i2c

​	通过 I2C 接口读取 ADXL345 获得的加速度数据，每 1s 读取一次，并将结果打印在屏幕上。

​	**构建工具：Makefile**（多文件构建）

##### LCM1602 显示屏

​	demo链接：https://github.com/milkv-duo/duo-examples/blob/main/i2c/lcm1602_i2c

​	通过 I2C 接口在 1602 LCD 屏幕上显示字符串。

​	**构建工具：Makefile**

##### LCM2004 显示屏

​	demo链接：https://github.com/milkv-duo/duo-examples/blob/main/i2c/lcm2004_i2c

​	通过 I2C 接口在 2004 LCD 屏幕上显示字符串。

​	**构建工具：Makefile**

##### TCS34725 颜色传感器

​	demo链接：https://github.com/milkv-duo/duo-examples/blob/main/i2c/tcs34725_i2c

​	通过 I2C 接口读取 TCS34725 颜色传感器，并将获得的数据输出。

​	**构建工具：Makefile**

#### SPI：

​	SPI demo目录：https://github.com/milkv-duo/duo-examples/tree/main/spi

##### MAX6675 热电偶温度传感器

​	demo链接：https://github.com/milkv-duo/duo-examples/tree/main/spi/max6675_spi

​	通过 SPI 接口连接 K 型热电偶测量模块 MAX6675，测量当前传感器上的温度。

​	**构建工具：Makefile**

##### RC522 RFID读写模块

​	demio链接：https://github.com/milkv-duo/duo-examples/tree/main/spi/rc522_spi

​	通过 SPI 接口连接 RC522 RFID 读写模块，读取卡片 ID 和类型并输出到屏幕。

​	**构建工具：Makefile**

### 工具包：

#### Opencv-mobile：

​	**opencv-mobile** 是一个精简版的 OpenCV 库，通过调整编译参数，删减部分 OpenCV 源码，来最小化编译 OpenCV。opencv-mobile 提供了 OpenCV 常用的功能，如读写图片，处理，矩阵操作等等，版本与上游同步，无第三方依赖。在绝大多数情况下，以 1/10 的体积无痛替换官方 OpenCV，尤其适合对体积有特殊要求的移动端和嵌入式环境。

##### 链接cvi-mmf动态库：

​	为了减少编译耦合，opencv-mobile 中采用运行时 dlopen/dlsym 方式加载 libsys libvpu libae libawb libisp libcvi_bin libsns_gc2083，即便编译时候缺库依然兼容可用。这种方式也能自动适配后期系统库升级。

​	**项目构建：Cmakelist**

### 其他IDE：Arduino IDE

​	Arduino 是一个很流行的开源硬件平台，具有简洁性、易用性和开放性等优点。它提供了丰富的库函数和示例代码，使得即使对于没有编程经验的人来说，也能够快速上手。同时，Arduino 社区非常活跃，可以轻松地获取到各种项目教程、文档和支持。

​	Duo 系列 CPU 采用大小核设计，Arduino 固件运行在小核中，大核负责与 Arduino IDE 通讯，接收 Arduino 固件并将其加载到小核中运行。同时，大核中的 Linux 系统也是正常运行的。

#### 安装 Arduino IDE：

​	建议安装最新版本，也可用官网相同版本2.3.2.

#### Arduino IDE 中添加 Duo 开发板：

​	添加Duo的配置文件*https://github.com/kubuds/sophgo-arduino/releases/download/v0.2.5/package_sg200x_index.json*，并打开开发板管理器搜索相应的开发板型号进行添加。

​	之后选择串口连接，编译文件之后，在IDE中即可烧录文件到开发板并执行。相比于一般的IDE，无需进行SSH或者其他的文件传输方式的配置，开发比较简便与高效。且Ardunio的开发框架中提供了libraries，更加的简便，更加受到用户的青睐。

​	

​	

​	

