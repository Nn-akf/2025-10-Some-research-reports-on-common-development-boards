# ESP32-S系列开发板相关信息调研

## 简介：

​	ESP32-S3 是一款集成 2.4 GHz Wi-Fi 和 Bluetooth 5 (LE) 的 MCU 芯片，支持远距离模式 (Long Range)。ESP32-S3 搭载 Xtensa® 32 位 LX7 双核处理器，主频高达 240 MHz，内置 512 KB SRAM (TCM)，具有 45 个可编程 GPIO 管脚和丰富的通信接口。ESP32-S3 支持更大容量的高速 Octal SPI flash 和片外 RAM，支持用户配置数据缓存与指令缓存。

​	ESP32-S2 是一款高度集成、高性价比、低功耗、主打安全的单核 Wi-Fi SoC，具备强大的功能和丰富的 IO 接口。

## 支持的操作系统：

​	NuttX、Zephyr、upyOS、Beryllium OS等。

## 连接：

​	通过 **USB 转 UART 接口** 或 **ESP32-S3 USB 接口** 连接开发板与电脑。

## 开发：

​	乐鑫官方推出自己的IDE即ESP-IDE，开发以乐鑫官网推出的IDF为例，此IDF最适配乐鑫的相关开发。

​	以Linux（ubuntu）环境为例，Win与MAC环境请参考官方手册。相关链接 -->[Linux 和 macOS 平台工具链的标准设置 - ESP32-S3 - — ESP-IDF 编程指南 latest 文档](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/get-started/linux-macos-setup.html)

### 安装准备：

​	下载相关包与依赖

```
sudo apt-get install git wget flex bison gperf python3 python3-pip python3-venv cmake ninja-build ccache libffi-dev libssl-dev dfu-util libusb-1.0-0
```

​	**使用 ESP-IDF 需要 CMake 3.22 或以上版本**

​	安装 Python 3	**请确保已安装 Python 3.10 或更高版本。Python 3.10 为 ESP-IDF 支持的最低版本。**

​	若使用**HomeBrew**

```
brew install python3
```

​	若使用MacPorts

```
sudo port install python313
```

### 获取ESP-IDF：

​	终端下执行：

```
mkdir -p ~/esp
cd ~/esp
git clone --recursive https://github.com/espressif/esp-idf.git
```

​	前往https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/versions.html查找不同的版本适配的场景

### 设置工具：

​	除了 ESP-IDF 本身，还需要为支持 ESP32-S3 的项目安装 ESP-IDF 使用的各种工具，比如编译器、调试器、Python 包等。

```
cd ~/esp/esp-idf
./install.sh esp32s3
```

​	如果需要为多个目标芯片开发项目，则可以一次性指定多个目标

```
cd ~/esp/esp-idf
./install.sh esp32,esp32s2
```

​	如果需要一次性为所有支持的目标芯片安装工具，可以运行如下命令

```
cd ~/esp/esp-idf
./install.sh all
```

### 设置环境变量：

​	此时，刚刚安装的工具尚未添加至 PATH 环境变量，无法通过“命令窗口”使用这些工具。因此，必须设置一些环境变量。这可以通过 ESP-IDF 提供的另一个脚本进行设置。

​	终端下执行：

```
. $HOME/esp/esp-idf/export.sh
```

***注意，命令开始的 "." 与路径之间应有一个空格！***

​	如果需要经常运行 ESP-IDF，可以为执行 `export.sh` 创建一个别名，复制并粘贴以下命令到 shell 配置文件中（`.profile`、`.bashrc`、`.zprofile` 等）

```
alias get_idf='. $HOME/esp/esp-idf/export.sh'
```

​	通过重启终端窗口或运行 `source [path to profile]`，如 `source ~/.bashrc` 来刷新配置文件。现在可以在任何终端窗口中运行 `get_idf` 来设置或刷新 ESP-IDF 环境。

*不建议直接将 `export.sh` 添加到 shell 的配置文件。这样做会导致在每个终端会话中都激活 IDF 虚拟环境（包括无需使用 ESP-IDF 的会话）。这违背了使用虚拟环境的目的，还可能影响其他软件的使用。*

### 使用IDF：

#### 创建工程：

​	可以从 ESP-IDF 中 [examples](https://github.com/espressif/esp-idf/tree/ab149384/examples) 目录下的 [get-started/hello_world](https://github.com/espressif/esp-idf/tree/ab149384/examples/get-started/hello_world) 工程开始。

​	将 [get-started/hello_world](https://github.com/espressif/esp-idf/tree/ab149384/examples/get-started/hello_world) 工程复制至本地的 `~/esp` 目录下：

```
cd ~/esp
cp -r $IDF_PATH/examples/get-started/hello_world .
```

#### 连接设备：

​	现在，请将 ESP32-S3 开发板连接到 PC，并查看开发板使用的串口。

#### 配置工程：

​	请进入 hello_world目录，设置 ESP32-S3 为目标芯片，然后运行工程配置工具 menuconfig。

```
cd ~/esp/hello_world
idf.py set-target esp32s3
idf.py menuconfig
```

​	打开一个新工程后，应首先使用 `idf.py set-target esp32s3` 设置“目标”芯片。注意，此操作将清除并初始化项目之前的编译和配置（可以通过此菜单设置项目的具体变量，包括 Wi-Fi 网络名称、密码和处理器速度等。`hello_world` 示例项目会以默认配置运行，因此在这一项目中，可以跳过使用 `menuconfig` 进行项目配置这一步骤。）

#### 编译工程：

​	编译烧录工程，如果一切正常，编译完成后将生成 .bin 文件。

```
idf.py build
```

#### 烧录到设备：

​	运行以下命令，将编译的文件烧录到开发板

```
idf.py -p PORT flash
```

​	请将 PORT 替换为 ESP32-S3 开发板的串口名称。如果 `PORT` 未经定义，[idf.py](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/api-guides/build-system.html#idf-py) 将尝试使用可用的串口自动连接。

​	如果一切顺利，烧录完成后，开发板将会复位，应用程序 "hello_world" 开始运行

#### 其他IDE：

​	如果希望使用 Eclipse 或是 VS Code IDE，而非 `idf.py`，请参考 [Eclipse Plugin](https://github.com/espressif/idf-eclipse-plugin/blob/master/README_CN.md)，以及 [VSCode Extension](https://github.com/espressif/vscode-esp-idf-extension/blob/master/docs/tutorial/install.md)。

### 示例demo：

##### hello_world：

​	打印hello world

​	demo链接：[esp-idf/examples/get-started/hello_world at ab149384e12ace1b8e8f7858d52ae47ab8af4563 · espressif/esp-idf](https://github.com/espressif/esp-idf/tree/ab149384/examples/get-started/hello_world)

​	**构建工具：Cmake**

##### blink_led：

​	通过库函数led_strip闪烁led

​	demo链接：[esp-idf/examples/get-started/blink at ab149384e12ace1b8e8f7858d52ae47ab8af4563 · espressif/esp-idf](https://github.com/espressif/esp-idf/tree/ab149384e12ace1b8e8f7858d52ae47ab8af4563/examples/get-started/blink)

​	**构建工具：Cmake**

##### bluetooh：

​	1.如何初始化bluedoid堆栈

​	2.如何配置广告和扫描响应数

​	3.如何开始广告作为一个不可连接的信标

​	demo链接：[esp-idf/examples/bluetooth/ble_get_started/bluedroid/Bluedroid_Beacon/README.md at ab149384e12ace1b8e8f7858d52ae47ab8af4563 · espressif/esp-idf](https://github.com/espressif/esp-idf/tree/ab149384e12ace1b8e8f7858d52ae47ab8af4563/examples/bluetooth)

​	**构建工具：Cmake**

##### ethernet-iperf：

​	这个例子演示了iperf协议测量以太网的带宽的基本用法。

​	demo链接：[esp-idf/examples/ethernet/iperf at ab149384e12ace1b8e8f7858d52ae47ab8af4563 · espressif/esp-idf](https://github.com/espressif/esp-idf/tree/ab149384e12ace1b8e8f7858d52ae47ab8af4563/examples/ethernet/iperf)

​	**构建工具：Cmake**

##### lowpower：

​	芯片设置为低功耗

​	demo链接：[esp-idf/examples/lowpower at ab149384e12ace1b8e8f7858d52ae47ab8af4563 · espressif/esp-idf](https://github.com/espressif/esp-idf/tree/ab149384e12ace1b8e8f7858d52ae47ab8af4563/examples/lowpower)

​	**构建工具：Cmake**

##### zigbee：

​	zigbee网络设置和设备示例

​	demo链接：[esp-idf/examples/zigbee at ab149384e12ace1b8e8f7858d52ae47ab8af4563 · espressif/esp-idf](https://github.com/espressif/esp-idf/tree/ab149384e12ace1b8e8f7858d52ae47ab8af4563/examples/zigbee)

​	**构建工具：Cmake**

##### security：

​	展示了SOC的一些安全演示操作

​	demo：[esp-idf/examples/security at ab149384e12ace1b8e8f7858d52ae47ab8af4563 · espressif/esp-idf](https://github.com/espressif/esp-idf/tree/ab149384e12ace1b8e8f7858d52ae47ab8af4563/examples/security)

​	**构建工具：Cmake**

##### CXX：

​	c++语言应用实例和实验组件

​	demo：[esp-idf/examples/cxx at ab149384e12ace1b8e8f7858d52ae47ab8af4563 · espressif/esp-idf](https://github.com/espressif/esp-idf/tree/ab149384e12ace1b8e8f7858d52ae47ab8af4563/examples/cxx)

​	**构建工具：CMake**

### 其他库和框架：

#### ESP-ADF：

​	此为一个全方位的音频应用程序框架，支持如下功能：

- CODEC 的 HAL
- 音乐播放器和录音机
- 音频处理
- 蓝牙扬声器
- 互联网收音机
- 免提设备
- 语音识别

#### ESP-CSI：

​	ESP-CSI 是一个具有实验性的框架，它利用 Wi-Fi 信道状态信息来检测人体存在。

#### ESP-DSP：

​	ESP-DSP 提供了针对数字信号处理应用优化的算法，该库支持：

- 矩阵乘法
- 点积
- 快速傅立叶变换 (FFT)
- 无限脉冲响应 (IIR)
- 有限脉冲响应 (FIR)
- 向量数学运算

#### ESP-WIFI-MESH：

​	ESP-WIFI-MESH 基于 ESP-WIFI-MESH 协议搭建，该框架支持：

- 快速网络配置
- 稳定升级
- 高效调试
- LAN 控制
- 多种应用示例

#### ESP-WHO：

​	ESP-WHO 框架利用 ESP32 及摄像头实现人脸检测及识别。

#### ESP RainMaker：

​	ESP RainMaker提供了一个快速 AIoT 开发的完整解决方案。使用 ESP RainMaker，用户可以创建多种 AIoT 设备，包括固件 AIoT 以及集成了语音助手、手机应用程序和云后端的 AIoT 等

#### ESP-IOT-Solution：

​	ESP-IOT-Solution涵盖了开发 IoT 系统时常用的设备驱动程序及代码框架。在 ESP-IoT-Solution 中，设备驱动程序和代码框架以独立组件存在，可以轻松地集成到 ESP-IDF 项目中。

​	ESP-IoT-Solution 支持：

- 传感器、显示器、音频、GUI、输入、执行器等设备驱动程序
- 低功耗、安全、存储等框架和文档
- 从实际应用角度指导乐鑫开源解决方案

#### ESP-Protocols：

​	此库函数包含 ESP-IDF 的协议组件集。ESP-Protocols 中的代码以独立组件存在，可以轻松地集成到 ESP-IDF 项目中。

#### ESP-BSP：

​	ESP-BSP库包含了各种乐鑫和第三方开发板的板级支持包 (BSP)，可以帮助快速上手特定的开发板。它们通常包含管脚定义和辅助函数，这些函数可用于初始化特定开发板的外设。此外，BSP 还提供了一些驱动程序，可用于开发版上的外部芯片，如传感器、显示屏、音频编解码器等

#### ESP-IDF-CXX

​	此库函数包含了 ESP-IDF 的部分 C++ 封装，重点在实现易用性、安全性、自动资源管理，以及将错误检查转移到编译过程中，以避免运行时失败。它还提供了 ESP 定时器、I2C、SPI、GPIO 等外设或 ESP-IDF 其他功能的 C++ 类。