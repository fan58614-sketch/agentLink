# 直接访问 Control 串口协议

[English](control-protocol.md) | **简体中文**

基线：固件 0.3.3、Control 协议 2。按 [USB 识别规则](usb-discovery.zh-CN.md) 找到正确端口。保持一个连接，不与其他程序或终端共用。

使用 UTF-8，每行一个 JSON 对象，不含行结束符最多 767 字节。接受 LF、CRLF、CR。一次发送一个请求，等待相同 `id` 的响应再发送下一个。响应包含 `result` 或 `error.code`，没有主动启动提示。

```json
{"id":1,"method":"device.info"}
```

不需要内部映射标识，使用产品逻辑引脚编号。

## 交互帮助

输入 `help`，或 `help protocol`、`help pins`、`help gpio`、`help spi`、`help i2c`、`help adc`、`help uart`，然后回车。固件帮助仍为英文，以 `END HELP` 结束。读取至该标记，不能只读一个 USB 包。帮助中的示例不会执行。单独输入 `spi` 等主题名不是有效命令。

## 操作

- GPIO：`gpio.configure`、`gpio.write`、`gpio.read`、`gpio.status`、`gpio.describe`。引脚 0～9，模式为 output、input、float、pullup、pulldown、open_drain；初始电平 0/1；断连安全状态 float/low。改变输出前阅读当前帮助。
- SPI：`spi.configure`、`spi.master.transfer`、`spi.release`。配置 mode 0～3、hz 375000～12000000（当前时钟）、read 0/1、cs 0。`tx_hex` 每次传输 1～128 字节，`cs_hold:1` 在分块间保持 CS，最后一块用 0。接收需发送 dummy 字节提供时钟。释放后恢复悬空输入。
- I2C：`i2c.configure`、`i2c.transfer`、`i2c.recover`、`i2c.release`。7 位地址 8～119，必填 `tx_hex`（可为空），最多写 64 字节，`read_count` 为 0～64。空写且读取数量为零表示明确的地址探测。
- ADC：`adc.configure`、`adc.read`、`adc.reference`、`adc.calibrate`、`adc.release`。ADC pin 0 表示 GPIO5 上的 ADC0，不是 GPIO0。校准参数为 `known_mv`，结果仅保存在 RAM。
- UART：`uart.flow_control`，port 1、enabled 0/1。UART 字节收发和串口格式配置走独立的 UART CDC 端口。Control 不实现 `uart.send`、`uart.read` 或 `uart.configure`。

确认 GPIO0 可以安全输出后，可使用：

```json
{"id":2,"method":"gpio.configure","pin":0,"mode":"output","initial":0,"safe":"float"}
{"id":3,"method":"gpio.write","pin":0,"value":1}
```

文件由主机读取，再按 128 字节分块发送，Control 不接受文件路径。完整收发、分块和错误处理见 [通信示例](serial-examples.zh-CN.md)。

## 硬件 SPI 的频率选择（0.3.3）

接线和命令名不变。SPI 使用硬件引擎，默认请求 1000000 Hz；当前 96 MHz 外设时钟下实际为 750000 Hz。硬件可选分频为 2、4、8、16、32、64、128、256，选择不超过请求值的最高可用频率。当前固件开放 375000～12000000 Hz 请求，实际可用 SCK 为 375000、750000、1500000、3000000、6000000、12000000 Hz。

`actual_hz` 返回分频计算出的标称 SCK，不是仪器测量或整段数据吞吐量。兼容字段 `clock_hz_upper_bound` 保留并返回同一值。旧的 100 kHz/250 kHz 请求会返回 `spi_clock_out_of_range`，需要修改主机脚本；不会静默提高时钟。大于 12 MHz 返回 `invalid_spi_config`。不要为低速外设擅自改高，先核对其允许频率。

每块仍最多 128 字节，支持 cs_hold。字节/块之间可能有间隙，不保证连续流。硬件异常或断连导致传输中止时返回 `spi_transfer_aborted`（若连接仍可响应），停止引擎并释放占用。该错误不保证零字节已发送，不能自动重试。12 MHz 是当前固件开放上限，不是所有 DUT 均已验证的速率。
