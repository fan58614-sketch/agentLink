# 快速开始

[English](quick-start.md) | [简体中文](quick-start.zh-CN.md)

读完本页即可完成首次只读连接，再按设备实时帮助执行任务；无需阅读全部文档。需要本机串口权限及可用串口 API，Python + pyserial 只是可选方式，不需要专用安装包。当前基线：固件 0.3.3 / 协议 2。

## 1. 找到正确端口，不发送探测数据

列出 USB 串口元数据；VID/PID `1A86:FE0C` 用于筛选候选，不是唯一身份。按同一物理 USB 设备位置分组，再用接口名或接口号判断：

| Role | Interface string | Communication / data interface |
| --- | --- | --- |
| UART1 | `AgentLink UART` | 0 / 1 |
| Control | `AgentLink Control` | 2 / 3 |
| UART2 | `AgentLink UART2` | 4 / 5 |

```python
from serial.tools import list_ports
for p in list_ports.comports():
    if (p.vid, p.pid) == (0x1A86, 0xFE0C):
        print(p.device, p.interface, p.location, p.serial_number, p.hwid)
```

接口名缺失时只查 [USB 识别](usb-discovery.zh-CN.md) 的当前系统步骤。不能凭路径末尾数字或 COM 顺序猜；当前序列号在原型间重复。重插和新会话重新枚举，记忆只保存接线意图，不能把旧端口路径当身份。未知端口禁止发送 help、换行或 JSON；有歧义时请用户选择。

## 2. 一个会话完成查询和操作

确认 Control 没被终端占用后，按 115200/8N1、DTR 开启打开。下列只发送请求，不发送响应示例：

```json
{"id":1,"method":"device.info"}
```

等待完整换行响应，检查 `id=1`、`result.control_protocol=2` 及所需能力。然后只查询当前功能的帮助，例如 SPI：

```text
help spi
```

读到 `END HELP` 再发送下一条 JSON。主题有 gpio、spi、i2c、adc、uart、pins、protocol；不需要全部查询。首次查询不必关闭后再开一个“正式”会话。

每次只发一个请求，最多 767 字节（不含行结束符），等待相同 id 的 `result` 或 `error.code`。按换行累积数据，单次 USB/串口读取可能不完整。不要在请求间清空接收缓冲。需要现成的可靠收发代码时，读 [通信示例的会话部分](serial-examples.zh-CN.md#会话规则)。

## 3. 按任务选择操作

| 功能 | 最少要知道的内容 |
| --- | --- |
| GPIO | 先 `gpio.configure`，再 `gpio.write` / `gpio.read`；逻辑 pin 0～9。读取值为 `result.value`。 |
| SPI | `spi.configure` → `spi.master.transfer` → `spi.release`。Mode 0～3，8-bit MSB-first，cs=0。默认请求 1 MHz，当前实际 750 kHz。 |
| I2C | `i2c.configure` → `i2c.transfer` → `i2c.release`；需要外接 3.3 V 上拉，7 位地址，不自动扫描。 |
| ADC | `adc.configure` → `adc.read`，pin=0 表示 GPIO5 上的 ADC0 → `adc.release`；精度尚待验证。 |
| UART | 对对应的独立 UART 串口使用普通串口 API。默认 115200/8N1。Control 没有 uart.send/read/configure；仅支持 UART1 的 uart.flow_control。 |

GPIO3/4 可选 I2C SCL/SDA，GPIO5 可选 ADC0，GPIO6/7/8/9 可选 SPI SCK/MISO/MOSI/CS；启用总线会占用对应引脚。SPI read=0 不占用 MISO。

**SPI 示例，仅在接线、电平和外设时钟限制确认后发送：**

```json
{"id":2,"method":"spi.configure","mode":0,"hz":1000000,"read":0,"cs":0}
```

响应示例：

```json
{"id":2,"result":{"mode":0,"clock_hz_upper_bound":750000,"actual_hz":750000,"read":0,"max_chunk":128,"engine":"hardware"}}
```

发送数据用 `tx_hex`，每块 1～128 字节；中间块 `cs_hold:1`，最后一块 `cs_hold:0`。返回 `transferred`、`rx_hex`（只写时为空）、`cs_held`。大文件由主机读文件并分块，不把路径发给固件。完整代码仅在需要时查 [分块示例](serial-examples.zh-CN.md#spi-文件分块)。

当前接受 375000～12000000 Hz，请求向下取硬件分频，`actual_hz` 是标称时钟。旧的 100/250 kHz 不支持，不要自动升频以绕过错误。Mode 0 / 750 kHz 屏幕发送已实测，其余模式传输、RX 和高速 DUT 尚未验证。当前支持范围不等于已验证范围。

## 4. 出错和完成时

- 当前逻辑为 3.3 V，不假定耐受 5 V。不要驱动 DUT 的输出，不抢占已有总线引脚。
- 超时或部分写入意味着执行状态不确定，停止后续操作，不自动重发；总线拥有权/配置错误按原因处理。
- 在开始前按外设要求确定异常收尾。既不要默认无限供电“保留现场”，也不要一律立即切电；结合数据手册和链路是否健康决定。异常链路不能保证清理命令送达。
- 正常完成按 DUT 要求休眠、设置安全电平、释放引脚、关闭连接。保持输出的任务不要提前关连接。断连 safe=float/low 为 RAM 配置，设备复位后悬空，不能代替硬件上下拉。
- 实际画面、颜色或测量值需要独立观察确认，BUSY 结束不是完整正确性的证明。
