# 不发送探测数据的 USB 识别

[English](usb-discovery.md) | **简体中文**

当前原型的候选 VID/PID 为 `0x1A86:0xFE0C`。仅凭这组值不能唯一确认产品身份。

| 用途 | USB 接口字符串 | 通信 / 数据接口编号 |
| --- | --- | --- |
| UART1 | `AgentLink UART` | 0 / 1 |
| Control | `AgentLink Control` | 2 / 3 |
| UART2 | `AgentLink UART2` | 4 / 5 |

根据 USB 接口元数据和所属物理 USB 设备识别用途。注意，UART1 的实际接口字符串不包含数字 1。

## 识别步骤

1. 枚举串口和 USB 元数据，不打开串口、不写入数据。
2. 用 VID/PID 和接口信息筛选候选设备，再根据接口字符串或上述 CDC 功能对应的接口编号判断用途。
3. 根据物理 USB 位置或拓扑对端口分组。选定一块板后，从同一组中选择 Control 和 UART。
4. 元数据缺失或互相矛盾时，展示候选名称、路径和位置，请用户明确选择，不随意使用第一个候选。
5. 确认 Control 身份后再打开，发送 `device.info` 验证协议和能力。

不要向未知串口发送 `help`、JSON、AT 命令、换行或其他探测数据。UART 会把字节转发给 DUT。

## 各平台的识别信息

- macOS：使用 USB 接口元数据和 IORegistry 的设备归属关系。`/dev/cu.usbmodem...` 名称由系统分配，末尾数字不是跨平台通用的用途标记。
- Windows：使用 USB 父设备身份和接口信息，包括可用时的 `MI_XX`。将接口归入对应 CDC 功能，不根据 COM 编号推断用途。
- Linux：使用 USB sysfs 设备归属关系和 `bInterfaceNumber`，不根据 `/dev/ttyACM...` 的枚举顺序判断用途。

不同系统栈可能暴露通信接口编号，也可能暴露数据接口编号，应按上表归一化识别。Windows 和 Linux 的实际行为仍待实物验证，不能将设计规则描述为当前实现已经全面验证的能力。

当前原型共用序列号字符串 `0123456789`，不能靠它区分两块板。后续应提供每台设备唯一的序列号；在此之前，物理拓扑尤其重要。拔插或 USB 复位后重新枚举，不假定旧路径仍有效。

## 手动选择与实际枚举

把确认的端口路径传给自己的串口代码；不要复制其他电脑的路径。Control 和各 UART 同时只能由一个程序使用。先关闭占用它的终端。

```python
from serial.tools import list_ports
for p in list_ports.comports():
    if (p.vid, p.pid) == (0x1A86, 0xFE0C):
        print(p.device, p.interface, p.location, p.serial_number, p.hwid)
```

此代码只列候选，不自动打开或猜测端口。接口元数据不足时，使用下面系统工具进一步核对：

macOS:
```sh
ioreg -p IOService -l -w 0 > /tmp/agentlink-ioreg.txt
```

Windows (PowerShell):
```powershell
Get-PnpDevice -Class Ports -PresentOnly | Format-List FriendlyName,InstanceId
# Use the selected InstanceId to inspect parent/location properties:
Get-PnpDeviceProperty -InstanceId '<selected InstanceId>' | Format-List KeyName,Data
```

Linux:
```sh
udevadm info --attribute-walk --name=/dev/ttyACM0
```

macOS 中沿每个 `IOCalloutDevice` 的父节点查找 USB 接口编号/名称和设备位置。Windows 检查对应端口的 MI 编号与父设备；Linux 沿 tty 父链检查 `bInterfaceNumber` 和 USB 设备归属，示例 tty 路径需替换。结合前表判断，不仅凭输出中的字符串相邻关系猜测。仍有歧义就请用户选择，不能试发数据。Windows/Linux 这些步骤仍待实物验证。
