# Orange Pi RV2 下RuyiSDK IDE调研任务

## 开发板：Orange Pi RV2

### 镜像：Orangepirv2_1.0.0_ubuntu_noble_desktop_gnome_lin。

## 虚拟机环境：Ubuntu22.04

## 1.交叉编译

​	在虚拟机上构建RISC-V架构的开发环境是进行orangepiRV2开发必不可少的一步，参考官方给出的用户手册，步骤大致可分为以下几点：

### 	1.获取源码：

```
sudo apt-get update	#软件源等的更新
sudo apt-get install -y git #进行git的下载
git clone https://github.com/orangepi-xunlong/orangepi-build.git -b next #从github上下载Orangepi-build的下载
```

​	***难点***：在官网给出的示例中，从用户实际操作的角度出发，在进行前两步的更新和下载操作时可能不会出现难以解决的问题。但是在进行第三步骤从github上拉取orangepi-build时，由于代理和和网络服务商限制的问题，大概率会出现连接超时的问题。这时一般来说，最好的办法是在windows下使用代理，下载在windows中，后经过Filezilla或“share”文件夹传递到Ubuntu下。

### 	2.下载交叉编译工具链

​	官方手册里推荐在清华大学开源软件镜像站中进行下载

```
https://mirrors.tuna.tsinghua.edu.cn/armbian-releases/_toolchain/
```

​	但是我在此网站下并没有找到相应的工具链“riscv64-unknown-linux-gnu-gcc”与“riscv64-unknown-linux-gnu-gcc”（有可能已经不支持了，2025.9.29），若想使用工具链须从官方打包的工具链中进行下载。

## 3.验证交叉编译的可行性

​	首先解压相应的压缩包，进入解压后的文件/bin文件夹中进行查看，是否存在相应的工具riscv64-unknown-linux-gnu-gcc 。查看存在之后，创建简单的hello.c文件，使用工具链进行编译（我把解压后的文件放在了桌面上），~/Desktop/ky-toolchain-linux-glibc-x86_64-v1.0.1/bin/riscv64-unknown-linux-gnu-gcc -o hello hello.c，之后输入 file hello查看架构是否符合。

​	输出结果为：

```
hello: ELF 64-bit LSB executable, UCB RISC-V, RVC, double-float ABI, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux-riscv64-lp64d.so.1, for GNU/Linux 4.15.0, not stripped

```

​	验证为可行。
