# 直接访问 Control 串口协议

[English](control-protocol.md) | **简体中文**

基线：固件 0.3.2、Control 协议 2。按 [USB 识别规则](usb-discovery.zh-CN.md) 找到正确端口。保持一个连接，不与 MCP 或其他终端共用。

使用 UTF-8，每行一个 JSON 对象，不含行结束符最多 767 字节。接受 LF、CRLF、CR。一次发送一个请求，等待相同 `id` 的响应再发送下一个。响应包含 `result` 或 `error.code`，没有主动启动提示。

```json
{"id":1,"method":"device.info"}
```

不需要内部映射标识，使用产品逻辑引脚编号。

## 交互帮助

输入 `help`，或 `help protocol`、`help pins`、`help gpio`、`help spi`、`help i2c`、`help adc`、`help uart`，然后回车。固件帮助仍为英文，以 `END HELP` 结束。读取至该标记，不能只读一个 USB 包。帮助中的示例不会执行。单独输入 `spi` 等主题名不是有效命令。

## 操作

- GPIO：`gpio.configure`、`gpio.write`、`gpio.read`、`gpio.status`、`gpio.describe`。引脚 0～9，模式为 output、input、float、pullup、pulldown、open_drain；初始电平 0/1；断连安全状态 float/low。改变输出前阅读当前帮助。
- SPI：`spi.configure`、`spi.master.transfer`、`spi.release`。配置 mode 0～3、hz 1000～250000、read 0/1、cs 0。`tx_hex` 每次传输 1～128 字节，`cs_hold:1` 在分块间保持 CS，最后一块用 0。接收需发送 dummy 字节提供时钟。释放后恢复悬空输入。
- I2C：`i2c.configure`、`i2c.transfer`、`i2c.recover`、`i2c.release`。7 位地址 8～119，必填 `tx_hex`（可为空），最多写 64 字节，`read_count` 为 0～64。空写且读取数量为零表示明确的地址探测。
- ADC：`adc.configure`、`adc.read`、`adc.reference`、`adc.calibrate`、`adc.release`。ADC pin 0 表示 GPIO5 上的 ADC0，不是 GPIO0。校准参数为 `known_mv`，结果仅保存在 RAM。
- UART：`uart.flow_control`，port 1、enabled 0/1。UART 字节收发和串口格式配置走独立的 UART CDC 端口。Control 不实现 `uart.send`、`uart.read` 或 `uart.configure`。

确认 GPIO0 可以安全输出后，可使用：

```json
{"id":2,"method":"gpio.configure","pin":0,"mode":"output","initial":0,"safe":"float"}
{"id":3,"method":"gpio.write","pin":0,"value":1}
```

MCP 工具名使用下划线，与 Control 方法名不同。MCP UART 工具操作独立的 UART 端口。文件路径和 SPI 文件自动分块属于主机 MCP 功能，不属于 Control 固件命令。
