
# 1. 说明

环境为：
* 1.CPU为13900K。
* 2.显卡为AMD6800XT。
* 3.主板为微星Z790ITX刀锋。


需要提前在微星主板BIOS执行操作：

* 1.关闭安全启动(SecureBoot)，位置在"Settings"-"安全"-"安全引导"-"安全启动"，关闭。
* 2.关闭CFG，位置在"OC"-"CPU特征"-"CFG锁定"，关闭。


Intel AX211网卡驱动Kext下载地址，根据系统版本下载对应的Kext。

https://github.com/OpenIntelWireless/itlwm/releases



# 2.目录结构

## 2.1 ACPI(Advanced Configuration and Power Interface)-高级配置与电源接口

定义了主板BIOS、UEFI与操作系统之间的硬件接口。它主要存放一些系统的电源管理以及硬件配置的接口，ACPI目录下主要放置的文件格式为SSDT-XXX.aml文件。

下面4项对于B300系列之后的主板是必须的

* ACPI/SSDT-PLUG-DRTNIA.aml
* ACPI/SSDT-EC-USBX-DESKTOP.aml
* ACPI/SSDT-AWAC.aml
* ACPI/SSDT-RHUB.aml

下面两项是定制的USB需要用到的配置(对于别的主板需要进行自定义，这里使用的是微星B460i，这个主要决定主板上的USB3.0接口是否生效，因为macos系统只能允许15个USB端口的存在)

* ACPI/SSDT-EC-USBX-DESKTOP.aml
* ACPI/SSDT-EC-USBX.aml

## 2.2 Kexts(Kernel Extensions)-内核扩展文件

所谓内核扩展文件，其实就是我们平常说的驱动。

下面两个是必装的驱动

* Kexts/VirtualSMC.kext
* Kexts/Lilu.kext

下面两个驱动的作用是用来监控CPU的温度和风扇的转速，台式机和笔记本都适用，但是只能在Intel的CPU下使用。

* Kexts/SMCProcessor.kext
* Kexts/SMCSuperIO.kext

下面这个驱动是显卡驱动，不管是核显还是独显都需要用到这个驱动。

* Kexts/WhateverGreen.kext

下面这个驱动是声卡驱动，支持大多数的板载声卡驱动

* Kexts/AppleALC.kext

下面这个驱动是USB驱动(只是USB的基础驱动，很可能需要定制USB之后才能正常使用)

* Kexts/USBInjectAll.kext


下面是Intel的无线网卡驱动。但是Intel网卡的隔空投送(投送文件)和随航(让ipad做mac的第二块屏幕)功能无法使用。更换博通网卡可以才能让这些功能正常使用，得换别的网卡驱动。

* Kexts/IntelBluetoothFirmware.kext
* Kexts/IntelBluetoothInjector.kext

## 2.3 Drivers-驱动文件

下面这个是必须的驱动

* Drivers/OpenRuntime.efi

下面这个驱动其实也是必须的驱动，用来识别HFS+的磁盘格式。

* Drivers/OpenHfsPlus.efi

下面这个驱动是让OC能有图形化引导界面，如果不使用的话，那么就是一个黑框框的命令行让你去选择启动项。

* Drivers/OpenCanopy.efi

下面这个驱动的作用是OC引导界面的截图驱动，看情况使用不使用，放上去肯定不会有错。

* Drivers/CrScreenshotDxe.efi

## 2.4 Resources

这个目录下主要存放OC引导的图形界面的背景和图标文件。可以根据自己的需要去网上下载，这里使用的是默认的主题(很简约，就和白苹果的主题一样)。

## 2.5 Tools

这个目录下主要存放在启动项时要添加的工具，最常见的是Tools/CsrUtil.efi这个工具，其实使用过苹果系统的人并不陌生。其实这个工具甚至也可以不要，可以在启动项时，进入Recovery中，进去使用终端输入命令也可以。

## 2.6 config.plist和OpenCore.efi

这两个配置文件都是OpenCore引导的关键文件，OpenCore.efi的作用是去引导macos系统的启动，肯定是OpenCore引导的核心。config.plist是OpenCore引导的核心配置文件，OC所有的配置都在这个文件中配置，包括对ACPI、Drivers、Kexts等去进行了更改，都需要在配置文件中去进行配置。

编辑plist配置文件，在Macos上可以使用OpenCoreConfigurator这个工具去进行编辑，在Windows上可以使用QtOpenCoreConfigurator去进行编辑，当然其实使用ProperTree这个专门编辑plist配置文件的工具去进行编辑也可以，甚至熟悉编程的朋友，还可以使用像VSCode、SublimeText等编辑器去进行编辑。

# 3. 问题解决

## 3.1 睡眠功能不正常

请自行查看"Docs/睡眠修复.md"进行解决

## 3.2 音频功能不正常

比如主板的前置/后置的耳机孔中插入了耳机，但是在macos系统中并不能找到该音频设备。

解决方案请自行查看："Docs/声卡驱动.md"

## 3.3 Intel无线网卡在Windows下正常使用，在黑苹果上无线网卡不能使用

在Windows下使用浏览器上一下网(比如点开一下百度)，再重启回到macOS。原理未知

## 3.4 修改操作系统的引导顺序

请参考"Docs/修改启动项顺序.md"

## 3.5 Macos和Windows时间相差了8个小时

原因在于：

Windows和MacOSX缺省看待PC的CMOS记录的时钟是不一样的。Windows将这个时钟作为本地时间来看待，也就是CMOS时间就是北京时间。

MacOSX将这个时钟作为Coordinated Universal Time (UTC) 世界标准时间看待，也就是Greenwich Mean Time (GMT) 格林威志时间。所以如果你在MacOSX和Windows都选北京时间作为本地时区是，一旦连到互联网上，同步过时间后，就会造成时间的不一致

在Windows下Win+R，输入regedit进入注册表，到HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\TimeZoneInformation\下--->在右侧窗口新增一项DWORD，命名为RealTimeIsUniversal，并将值设置为1。接着，重新同步一下时间即可。

## 3.6 更新OpenCore之后macOS下不能识别

使用NVRAM工具，清除一下NVRAM并重启即可识别到新的OpenCore版本。

## 3.7 AMD显卡开机亮Logo之后黑屏

我这里使用的显卡为AMD 6800XT，需要添加启动参数`agdpmod=pikera`，然后解决。

![](./Docs/images/amd%E6%98%BE%E5%8D%A1%E6%98%BE%E7%A4%BA%E9%BB%91%E5%B1%8F.png)