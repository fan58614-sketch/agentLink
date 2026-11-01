# MCP 使用说明

[English](mcp-usage.md) | **简体中文**

主机软件包提供 stdio MCP 服务，在安装该包的环境中通过 `agentlink-mcp` 或 `python -m agentlink.mcp_server` 启动。将 Agent 的 MCP 客户端配置为启动该环境中的可执行程序。目前尚未指定公开软件包或 GitHub 安装地址，不要编造链接。

通常自动发现端口，仅在必要时设置端口环境变量，见 [USB 识别规则](usb-discovery.zh-CN.md)。Skill 可以单独分发，但必须安装 MCP 实现才会拥有这些工具。没有 MCP 时也可以直接访问 Control 串口。

## 查询功能

调用 `help` 查看概览，再根据工具参数定义请求具体主题或完整工具名。用 `device_info` 和 `gpio_describe` 检查连接设备的版本和引脚占用。参数类型以实际暴露的工具定义为准。

工具分组：

- `gpio_configure`、`gpio_write`、`gpio_read`、`gpio_describe`、`gpio_status`
- `spi_configure`、`spi_transfer`、`spi_write_file`、`spi_release`
- `i2c_configure`、`i2c_transfer`、`i2c_recover`、`i2c_release`
- `adc_configure`、`adc_read`、`adc_reference`、`adc_calibrate`、`adc_release`
- `uart_configure`、`uart_send`、`uart_read`、`uart_status`、`uart_close`、`uart_flow_control`

## UART 会话

按 DUT 已知的波特率、校验位和停止位配置端口 1 或 2。当前支持 1200～3000000 baud、8 数据位、N/E/O 校验和 1/2 停止位。仅 UART1 支持 RTS/CTS。重新配置前关闭现有会话；打开或关闭可能丢弃已排队数据。

`uart_send` 接受十六进制字节或 UTF-8 文本，二选一，长度 1～65536 字节，不自动添加换行。主机接受数据不代表 DUT 已收到。超时可能意味着发送了长度未知的前缀，不自动重试。

`uart_read` 消费最多指定数量的字节（1～65536），超时范围 0～5000 ms，0 表示轮询。空结果或部分结果是正常情况。精确字节以 `rx_hex` 为准；显示用 UTF-8 文本可能含替代字符或被分割的多字节字符。

`uart_status` 返回主机会话配置，不是对硬件寄存器的独立验证。`uart_close` 关闭主机句柄，不会关闭硬件 UART，也不一定关闭流控。

## GPIO 和总线

MCP `gpio_write` 会配置推挽输出；直接调用 Control `gpio.write` 则要求事先配置为输出。调用前确认实际连接。

SPI 使用 8 位、MSB-first、单路低有效 CS，软件时序请求上限为 1～250 kHz。MCP 按固件块大小拆分传输，并按要求保持 CS。大量数据可用 `spi_write_file`，路径必须是 MCP 服务所在主机的绝对路径；文件哈希只证明源文件身份，不证明 DUT 已接收。读取需要发送 dummy 字节产生时钟。外设复位/DC 等信号使用通用 GPIO。

I2C 需要外接 3.3 V 上拉。先配置再传输，组合写读使用重复 START。恢复操作会发出时钟和 STOP，应明确调用，不自动扫描或重试 DUT 总线。

ADC0 使用 GPIO5。读取测量值时结合内部参考和校准信息。校准仅保存在 RAM。具备 ADC 工具不代表已经保证模拟精度或支持带保护的 5 V 测量。
