# AgentLink

[English](SKILL.md) | [简体中文](SKILL.zh-CN.md)

默认直接通过 **Control USB 串口的 JSON 行协议** 操作设备。不寻找或安装 AgentLink 专用主机软件，不要求 MCP。Skill 是操作说明；Agent 使用已有终端和串口编程能力执行请求。Python + pyserial 是一种实现方式，不是必需的专用工具；环境缺少串口能力时明确报告，不能假装已执行。

按用户语言回复，工具和协议标识不翻译。只读取所需语言版本。标准安装入口仍是 `SKILL.md`。

## 开始操作

1. 按 [USB 识别](references/usb-discovery.zh-CN.md) 枚举端口和元数据，不向未知串口发送探测数据。确认同一物理设备的 Control、UART1、UART2。
2. 独占打开确认的 Control，保持一个会话。发送 `device.info` 检查协议 2 和实际能力，读取 `help` 或相关主题至 `END HELP`。
3. 阅读 [Control 协议](references/control-protocol.zh-CN.md) 和 [完整通信示例](references/serial-examples.zh-CN.md)。请求按行发送并等待对应 id 的响应，不猜测返回结构。
4. 确认 DUT 接线、3.3 V 电平、方向和安全状态，再配置引脚。被其他功能占用时先明确释放，不抢占现有用途。
5. 通过通用 GPIO/SPI/I2C 操作外设。外设驱动、图像转换和协议时序由主机代码实现，不修改设备固件、不增加屏幕等专用接口。
6. 检查响应，区分主机写入、设备执行和 DUT 实际结果。超时或部分写入不自动重发。完成后按 DUT 要求恢复电平并释放资源，最后关闭串口。

## 功能边界

- UART1 与 UART2 是独立数据串口，默认 115200/8N1。UART1 有 RTS/CTS，UART2 仅 TX/RX；Control 不提供 UART 字节收发命令。
- GPIO0～9 默认悬空。GPIO3/4 可用于 I2C SCL/SDA，GPIO5 为 ADC0，GPIO6/7/8/9 可用于 SPI SCK/MISO/MOSI/CS。总线启用会占用对应引脚。
- 当前原型为 3.3 V 信号，不假定 5 V 容忍或受保护的 ADC 输入。
- 断连安全状态 float/low 保存在 RAM；设备复位后悬空。软件不能替代物理上下拉。
- 周期 GPIO、PWM、SPI 从机和电源开关控制尚未实现。普通 GPIO 控制外设的 PWR 使能不等于工具提供电源。

参阅 [验证状态](references/validation-status.zh-CN.md)。不要用“调用成功”代替实际硬件验证。
