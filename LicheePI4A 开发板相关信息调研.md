# LicheePI4A 开发板相关信息调研

## 镜像集合：

​	Debian、OpenWRT、Android、openEuler、DeepinOS、openKylin、armbian（official build-framework，use RV64GC toolchain）、armbian(unofficial build-framework，use T-Head optimized toolchain)、Fedora（unofficial）、OpenWrt（unofficial）、ubuntu、NixOS、Gentoo、slarm64、irradium。（以上镜像由励志派官网推荐）ArchLinux、RevyOS、Slackware、Tizen、irradium、openKylin（以上为PLCT RISC-V开发板及操作系统支持矩阵项目验证，参考来源：https://github.com/ruyisdk/support-matrix/）

## 系统开发（以RevyOS SDK为例）：

### RevyOS SDk：

#### 首先编辑环境配置：	

```
export xuetie_toolchain=https://occ-oss-prod.oss-cn-hangzhou.aliyuncs.com/resource//1663142514282
export toolchain_file_name=Xuantie-900-gcc-linux-5.10.4-glibc-x86_64-V2.6.1-20220906.tar.gz
export toolchain_tripe=riscv64-unknown-linux-gnu-
export ARCH=riscv
export nproc=12 #请根据自身CPU配置设置，该文档使用cpu为i5-11400
mkdir th1520_build && cd th1520_build
export GITHUB_WORKSPACE="~/th1520_build" #本文假设均下载到用户目录下，可根据自身需要更改
sudo apt update && \
              sudo apt install -y gdisk dosfstools g++-12-riscv64-linux-gnu build-essential \
                                  libncurses-dev gawk flex bison openssl libssl-dev tree \
                                  dkms libelf-dev libudev-dev libpci-dev libiberty-dev autoconf device-tree-compiler
              sudo update-alternatives --install \
                  /usr/bin/riscv64-linux-gnu-gcc riscv64-gcc /usr/bin/riscv64-linux-gnu-gcc-12 10
              sudo update-alternatives --install \
                  /usr/bin/riscv64-linux-gnu-g++ riscv64-g++ /usr/bin/riscv64-linux-gnu-g++-12 10

```

.**注意请检查repo下的kernel、uboot、opensbi**

#### 构建Kernel：

​	首先请clone用到的repo，并建立好对应文件夹（下列路径均假设根目录为用户目录下）

```
git clone https://github.com/revyos/thead-kernel.git kernel
```

​	配置编译工具链

```
wget ${xuetie_toolchain}/${toolchain_file_name}
tar -xvf ${toolchain_file_name} -C /opt
export PATH="/opt/Xuantie-900-gcc-linux-5.10.4-glibc-x86_64-V2.6.1/bin:$PATH"

```

​	创建安装目标目录

```
mkdir rootfs && mkdir rootfs/boot
```

​	编译内核

```
pushd kernel
make CROSS_COMPILE=${toolchain_tripe} ARCH=${ARCH} revyos_defconfig
make CROSS_COMPILE=${toolchain_tripe} ARCH=${ARCH} -j$(nproc)
make CROSS_COMPILE=${toolchain_tripe} ARCH=${ARCH} -j$(nproc) dtbs
if [ x"$(cat .config | grep CONFIG_MODULES=y)" = x"CONFIG_MODULES=y" ]; then
    sudo make CROSS_COMPILE=${toolchain_tripe}  ARCH=${ARCH} INSTALL_MOD_PATH=${GITHUB_WORKSPACE}/rootfs/ modules_install -j$(nproc)
fi
#sudo make CROSS_COMPILE=${toolchain_tripe}  ARCH=${ARCH} INSTALL_PATH=${GITHUB_WORKSPACE}/rootfs/boot zinstall -j$(nproc)

```

​	构建perf（根据需要构建）

```
make CROSS_COMPILE=riscv64-unknown-linux-gnu- ARCH=riscv LDFLAGS=-static NO_LIBELF=1 NO_JVMTI=1 VF=1 -C tools/perf/
sudo cp -v tools/perf/perf ${GITHUB_WORKSPACE}/rootfs/sbin/perf-thead

```

​	记录 commit-id

```
git rev-parse HEAD > kernel-commitid
sudo cp -v kernel-commitid ${GITHUB_WORKSPACE}/rootfs/boot/
```

​	安装内核、设备树到目标目录

```
sudo cp -v arch/riscv/boot/Image ${GITHUB_WORKSPACE}/rootfs/boot/
sudo cp -v arch/riscv/boot/dts/thead/{light-lpi4a.dtb,light-lpi4a-16gb.dtb} ${GITHUB_WORKSPACE}/rootfs/boot/
popd
```

​	之后只需要把rootfs中内容拷贝或覆盖到对应目录即可，注意内核Image和内核module目录一定要对应，不然会因缺失内核模块导致外设功能失效。

#### 构建Uboot

​	在th1520_build目录下，已经配置好了环境变量和工具链。

```
git clone https://github.com/revyos/thead-u-boot.git uboot
```

​	需要注意的是，8G 与 16G 内存版本使用的 uboot 不同，所以对应的构建命令也不同，基于此仓库构建命令如下：

```
pushd uboot
# 构建16G内存版本使用的uboot
make light_lpi4a_16g_defconfig
make CROSS_COMPILE=${toolchain_tripe} ARCH=${ARCH} -j$(nproc)
find . -name "u-boot-with-spl.bin" | xargs -I{} cp -av {} ${GITHUB_WORKSPACE}/rootfs/u-boot-with-spl-lpi4a-16g.bin
make clean
# 构建8G内存版本使用的uboot
make light_lpi4a_defconfig
make CROSS_COMPILE=${toolchain_tripe} ARCH=${ARCH} -j$(nproc)
find . -name "u-boot-with-spl.bin" | xargs -I{} cp -av {} ${GITHUB_WORKSPACE}/rootfs/u-boot-with-spl-lpi4a.bin
make clean
popd
```

​	烧录时注意烧录和你所使用的开发板所对应的 uboot。在烧录时请注意使用的命令，若使用的镜像版本为 `0912` 及以上版本，升级 uboot 只需要运行：

```
sudo ./fastboot flash uboot u-boot-with-spl-lpi4a-16g.bin
```

​	检查输出的文件

```
tree ${GITHUB_WORKSPACE}/rootfs
```

#### 构建opensbi：

```
git clone https://github.com/revyos/thead-opensbi.git opensbi
```

​	然后开始执行编译命令

```
pushd opensbi
make PLATFORM=generic ARCH=${ARCH} CROSS_COMPILE=${toolchain_tripe} 
sudo install -D -p -m 644 build/platform/generic/firmware/fw_dynamic.bin \
"${GITHUB_WORKSPACE}/rootfs/boot/"
popd
```

​	检查输出的文件

```
tree ${GITHUB_WORKSPACE}/rootfs
```

​	将目前构建好的kernel, uboot, opensbi相关文件打包为压缩包

```
tar -zcvf kernel.tar.gz rootfs
```

​	要使用构建的文件，则将压缩包中文件替换到相应位置即可。将boot.ext4中要替换的文件删掉，然后rootfs/boot/下的文件放到boot.ext4中；将rootfs/lib/modules/替换掉rootfs.ext4中的/lib/modules/目录；若构建了perf了，将rootfs/sbin下的文件替换掉rootfs.ext4中/sbin下的文件；uboot直接烧录即可。

​	**此处只以RevyOS为例，除此之外，官网还列出了如Linux主线、Android、THead Yocto、OpenHarmony、OpenWRT的开发教程，具体请前往相关链接查看。链接：[revyos SDK - Sipeed Wiki](https://wiki.sipeed.com/hardware/zh/lichee/th1520/lpi4a/7_develop_revyos.html)**



## 外设使用：

### SOC相关：

#### 	CPU运行频率：

```
sudo cat /sys/devices/system/cpu/cpu*/cpufreq/cpuinfo_cur_freq
```

​	系统自带温控策略，当系统过于空闲或者温度过高时，都会降频。请保持良好散热，使得 CPU 在 60 度以下，获得最佳性能。

#### 	 芯片温度：

```
cat /sys/class/thermal/thermal_zone0/temp
```

​	单位为0.001摄氏度。



**L*icheePI4A 的官方文档并未给出GPIO、PWM等外设的相关示例demo，只列出了相关的验证性操作。且在阅读官方提供的嵌入式手册，官方推荐使用Cmakelist来管理用户的新建工程*。**



## 典型应用

### llama.cpp

​	llama 是 META 开源的大语言模型，llama.cpp 是 ggerganov 开源的纯 cpp 运行的 llama 推理项目。

​	demo链接：https://github.com/Zepan/llama.cpp

​	**项目构建工具：cmake 、Makefile** 此为集成性项目，原项目文件很多且拥有众多依赖。后经修改，使其以轻量化的模型运行。

### YOLOX目标检测

此过程具体分为：

​	*在 LPi4A 开发版上安装 Python 环境

​	*使用 YOLOX 项目中的源码执行模型					

具体步骤请参考官方文档操作：[典型应用 - Sipeed Wiki](https://wiki.sipeed.com/hardware/zh/lichee/th1520/lpi4a/8_application.html#YOLOX-目标检测)

### MobilenetV2

教程过程分为：

- 使用 HHB 编译 onnx 模型为 LicheePi4A 上可用的二进制
- 在 LicheePi4A 上使用 opencv c++ 版本做 mobilenetv2 模型的预处理
- 在 LicheePi4A 上使用 CPU 和 NPU 的差异

具体步骤请参考官方文档操作：[典型应用 - Sipeed Wiki](https://wiki.sipeed.com/hardware/zh/lichee/th1520/lpi4a/8_application.html#YOLOX-目标检测)

**典型应用中还有Yolov5n、BERT、k3s-RISCV、opencv等的介绍，具体细节请前往官网查看。**