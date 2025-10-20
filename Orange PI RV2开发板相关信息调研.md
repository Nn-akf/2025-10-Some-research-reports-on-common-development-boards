# Orange PI RV2开发板相关信息调研

## 简介：

​	OrangePi RV2是一款高性价比RISC-V开发板，采用Ky X1 8核RISC-V AI CPU，提供2TOPS CPU融合的通用算力，可支持AI模型算法的快速部署。拥有2GB / 4GB / 8GB LPDDR4X，支持eMMC模块（16GB/32GB/64GB/128GB可选），具有Wi-Fi5.0+BT 5.0，支持BLE。

## 支持的镜像：

​	OpenWRT、Orange Pi OS(OH)、ubuntu、Linux源码、BredOS

## 连接：

### 串口连接：

​	使用USB to TTL模块，连接对应的RX->TX，TX->RX，GND->GND，供电后在MobaxTerm下连接即可。

### SSH：

​	Linux 系统默认都开启了 ssh 远程登录，并且允许 root 用户登录系统，SSH登录前首先需要确保以太网或者 wifi 网络已连接，然后使用 ifconfig命令或者通过查看路由器的方式获取开发板的 IP 地址。

### 网线连接（便捷联网）：

​	在PC端的网络服务中打开网络共享，以电脑为媒介给开发板上网。然后串口登录开发板获取IP，在MobaxTerm下即可使用SSH进行连接，这样即使不在终端下给开发板联网，开发板依然可以通过网络共享连接网络。

​	先串口登录开发板，修改interfaces中的文件。把相应的IP改到与PC端IP的同一网段下，例如如果PC端的IP为192.168.137.1，那么开发板可以修改为192.168.137.5，这样不需要每次都串口登录获取IP地址，也可以实现PC端上网开发板即有网的模式。

### 外设连接：

​	开发板供电，通过HDMI连接相应的显示屏、键盘与鼠标即可实现界面化操作。如果手边没有显示屏，也可通过VNC实现。

## 开发：

### 运行大模型的方法：

**以 Qwen2.5-0.5B-Instruct 为例。**

#### 环境准备:

​	一台装有 Ubuntu22.04 操作系统的 PC

​	一块装有 Ubuntu24.04 系统的 Orange Pi RV2 开发板。

​	从官方资料下载 ky-ort 工具包。

​	参考模型支持列表，准备需要构建的原始模型文件。模型文件链接：[shenzhi-wang (Shenzhi Wang)](https://huggingface.co/shenzhi-wang)

#### Ubuntu下构建：

在Ubuntu下执行：

​	首先下载相应的资源包，这里推荐去orangepi的官网进行下载压缩包上传，否则可能会出现下载认证不通过的情况。

```
sudo apt install -y git-lfs
git clone https://www.modelscope.cn/Qwen/Qwen2.5-0.5B-Instruct.git
```

​	解压 ky-ort 工具包，并安装相关依赖。

```
tar zxf ky-ort.riscv64.1.2.2.tar.gz
pip3 install -r ky-ort.riscv64.1.2.2/python/genai-builder/requirements.txt \ -i https://mirrors.huaweicloud.com/repository/pypi/simple
```

​	执行以下命令，构建适用于 Orange Pi RV2 的模型文件。

```
cd ky-ort.riscv64.1.2.2/python/genai-builder
python3 builder.py \ -i /home/test/Qwen2.5-0.5B-Instruct/ -o /home/test/qwen2-int4-0.5b/ \ -p int4 -e cpu -c /home/test/tmp --extra_options int4_block_size=64 int4_accuracy_level=4
```

​	转换完成后，进入模型保存目录，确认转换正确。

```
ls -lh
```

#### 开发板环境准备：

​	将在Ubuntu下构建完成的文件上传到开发板中，再把ky-ort工具包同样上传到开发板下。

​	在开发板下首先解压工具包：

```
tar zxf ky-ort.riscv64.1.2.2.tar.gz
```

​	编译 cpp demo

```
cd ky-ort.riscv64.1.2.2
bash scripts/build_samples_riscv64.sh
```

​	编译成功后，在 build/riscv64 目录下，能够找到以下文件

```
ls build/riscv64/

chatllm_demo CMakeCache.txt CMakeFiles cmake_install.cmake imagenet_test
Makefile phi3v run_demo
```

​	更新python依赖

```
cd python
pip3 install ./onnxruntime_genai-0.4.0.dev
1-cp312-cp312-linux_riscv64.whl ./ky_ort-1.2.2-cp312-cp312-linux_riscv64.whl --break-system- packages
```

#### 执行：

​	**执行命令后在终端下即可使用**（列出部分推理模型的执行命令、具体请查看用户手册）

##### qwen-0.5B

​	C++推理命令

```
cd ky-ort.riscv64.1.2.2/build/riscv64/
./chatllm_demo ~/models/qwen2-int4- 0.5b/ qwen2
```

​	Python推理命令

```
cd ky-ort.riscv64.1.2.2/samples
python3 llm_qa.py -m ~/models/qwen2-int4-0.5b/ -l 128 -e qwen2 -v -g
```

##### Llama-1B

​	C++推理命令

```
cd ky-ort.riscv64.1.2.2/build/riscv64/
./chatllm_demo ~/models/llama-cn-int4-1b/ llama3
```

​	Python 推理命令

```B
cd ky-ort.riscv64.1.2.2/samples
python3 llm_qa.py -m ~/models/llama-cn-int4-1b/ -l 128 -e llama3 -v -g
```

##### Llama3-8B

​	C++推理命令

```
cd ky-ort.riscv64.1.2.2/build/riscv64/
./chatllm_demo ~/models/llama3-int4- 8b-blk64-fusion/ llama3
```

​	Python推理命令

```
cd ky-ort.riscv64.1.2.2/samples
python3 llm_qa.py -m ~/models/llama3-int4-8b-blk64-fusion/ -l 128 -e llama3 -v -g
```

### 外设的控制：

​	**orangepi RV2的官方用户手册中并没有列出可以移植的示例demo，故在此列出可得到反馈的外设**

​	**官方对于外设demo的管理推荐使用Makefile与Cmakelist**

#### 板载LED的控制：

​	首先进入root用户，随后顺序执行

```
cd /sys/class/leds/sys-led  #首先进入绿灯的设置目录

echo none > trigger         #设置绿灯停止闪烁的命令

echo default-on > trigger	#设置绿灯常亮的命令

echo heartbeat > trigger	#设置绿灯闪烁的命令
```

#### 音频测试：

​	首先将耳机插入开发板的耳机孔中。然后可以通过 aplay -l 命令可以查看下 linux 系统支持的声卡设备，从下面的输 出可知，card 1 为 es8388 的声卡设备，也就是耳机的声卡设备

```
aplay -l 
**** List of PLAYBACK Hardware Devices ****
card 0: sndhdmi [snd-hdmi], device 0: SSPA2-dummy_codec dummy_codec-0 []
Subdevices: 1/1
Subdevice #0: subdevice #0
card 1: sndes8323 [snd-es8323], device 0: i2s-dai0-ES8323 HiFi ES8323 HiFi-0 []
Subdevices: 1/1
Subdevice #0: subdevice #0
```

​	然后使用 aplay 命令播放下系统自带的音频文件，如果耳机能听到声音说明硬件 能正常使用。

```
aplay -D hw:1,0 /usr/share/sounds/alsa/audio.wav

Playing WAVE 'audio.wav' : Signed 16 bit Little Endian, Rate 44100 Hz, Stereo
```

#### 26PIn GPIO口测试：

​	orange pi 发布的Linux系统中有一个预装的blink_all_gpio的程序，这个程序会设置26pin中的所有17个GPIO口不停的切换高低电平。当运行程序之后使用万用表去测量对应的GPIO口会发现相应的电平在0与3.3V之间跳变。

​	运行相应程序的方法如下：

```
sudo blink_all_gpio
```

#### 硬件看门狗：

​	Orange Pi 发布的 linux 系统中预装了 watchdog_test 程序，可以直接测试。 运行 watchdog_test 程序的方法如下所示：

```
sudo watchdog_test 10
```

​	第二个参数 10 表示看门狗的计数时间，如果这个时间内没有喂狗，系统会重启。

​	我们可以通过按下键盘上的任意键（ESC 除外）来喂狗，喂狗后，程序会 打印一行 keep alive 表示喂狗成功。

### Linux SDK—orangepi-build的使用

#### 编译系统需求：

​	**需要Ubuntu22.04系统**

#### 获取源码：

​	linux sdk 其实指的就是 orangepi-build 这套代码，orangepi-build 是基于 armbian build 编译系统修改而来的，使用 orangepi-build 可以编译出多个版本的 linux 镜像。 首先下载 orangepi-build 的代码，命令如下所示：

```
sudo apt-get update			#更新系统包
sudo apt-get install -y git	#下载giit
git clone https://github.com/orangepi-xunlong/orangepi-build.git -b next	#下载源码
```

​	下载后的orangepi-build的目录下会存在如下文件：

​		a. build.sh: 编译启动脚本

​		b. external: 包含编译镜像需要用的配置文件、特定的脚本以及部分程序的源 码等

​		c. LICENSE: GPL 2 许可证文件 

​		d. README.md: orangepi-build 说明文件 

​		e. scripts: 编译 linux 镜像的通用脚本

#### 下载交叉编译工具链：

​	只有在X64的电脑下使用orangepi-build编译镜像才会自动下载交叉编译工具链，在开发板的Ubuntu22.04环境下编译镜像是不会自动下载交叉编译工具链的，此时的orange-build/toolchains只会是一个空文件夹。

​	orangepi-build第一次运行的时候会自动下载交叉编译工具链放在toolchains文件 夹中，每次运行 orangepi-build 的 build.sh 脚本后，都会检查 toolchains 中的交叉编 译工具链是否都存在，如果不存在则会重新开始下载，如果存在则直接使用，不会 重复下载。

​	**也可以手动下载：https://mirrors.tuna.tsinghua.edu.cn/armbian-releases/_toolchain/**

​	toolchains下载完后会包含多个版本的交叉编译工具链，开发板只会使用其中两个。编译 linux 内核源码使用的交叉编译工具链——riscv64-unknown-linux-gnu-gcc与编译 u-boot 源码使用的交叉编译工具链——riscv64-unknown-linux-gnu-gcc。

#### 编译：

​	1.编译u-boot

​	2.编译Linux内核

​	3.编译rootfs

​	4.编译Linux内核

**不同的开发者需求不同，具体编译需求请参考官方用户手册进行个人因需编译**

### Github orangepi示例demo参考

#### OrangePi-Camera：		

- Platform

  OrangePi 2G-IOT, OrangePi PC2,

- System

  Ubuntu

- Prepare toolchain

  ```
  sudo apt-get install git make gcc libjpeg8-dev flexDoownload Source Code
  ```

- Doownload Source Code

  ```
  env GIT_SSL_NO_VERIFY=true git clone https://github.com/OrangePiLibra/OrangePi_Camera.git
  ```

  **构建工具：Makefile**

  **demo链接：(https://github.com/orangepi-xunlong/OrangePi_Camera)**

  #### OrangePi-OlED

  ​        Based on Richard Hulls original repo (https://github.com/rm-hull) adapted to Orange Pi single board computers.

  Interfacing OLED matrix displays with the SH1106 (or SSD1306) driver in Python using I2C on Orange Pi SBCs.

  ​      **构建工具：setup.py**

  ​      **demo链接：(https://github.com/karabek/OrangePi-OLED)**