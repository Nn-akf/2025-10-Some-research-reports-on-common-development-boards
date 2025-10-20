# Megrez 开发板相关信息调研

## 支持的操作系统：

​	Deepin、Fedora、Guix、OpenCloudOS、RockOS（以上为PLCT RISC-V开发板及操作系统支持矩阵项目验证，参考来源：https://github.com/ruyisdk/support-matrix）。

## 连接：

### Type-C：

​	Type-C 接口可以做为烧录或调试口，通过 PC 为板上的SPI Flash，emmc，SATA SSD固态硬盘安装相应的操作系统。

## F_PANEL：

​	F_PANEL 是Front Panel的缩写，是一组用来连接标准 PC 机箱前面板的跳线，包含开关机键、重启键、电源指示灯、硬盘状态灯四组信号。

### Recovery / Normal Boot切换开关：

​	Megrez 主板的默认启动顺序为 SPI Flash(U-boot) -> SD/eMMC/SATA 。

​	当 SPI Flash 里的 U-boot 固件意外受损无法启动时，可以使用 Recovery/Normal Boot切换开关进入恢复模式，此时 Megrez会从 USB 加载临时 U-boot。

### Debug / Download：

​	Megrez 主板上内置了CH340 芯片。默认情况下，使用 Type-C 连接 Debug 至 PC ，此时 PC 将新增一个 tty.usbserial-1220设备

## 开发：

#### Bootloader下载：

​	https://github.com/milkv-megrez/megrez-build/releases/tag/2025-0219

​	https://github.com/milkv-megrez/megrez-build/releases/tag/2025-0117

**此开发板的相关信息较少**
