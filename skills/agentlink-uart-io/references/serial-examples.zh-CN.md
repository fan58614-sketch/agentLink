# 串口通信与通用示例

[English](serial-examples.md) | [简体中文](serial-examples.zh-CN.md)

这些是文档代码示例，不是需要安装的 AgentLink 客户端。Agent 可按当前环境自行实现。Python 示例需要已有 pyserial；若缺少，明确说明依赖并按用户环境处理。无需搜索 AgentLink 安装包。

## 会话规则

Control 使用 115200/8N1 打开即可，CDC 的名义波特率不是 SPI 速率。保持 DTR 和连接，避免每条指令打开再关闭。一次一个请求；按换行接收完整响应，不把一次 USB 读取当完整消息。帮助是多行文本，必须读取到 `END HELP`，随后才能恢复 JSON 请求。不要在请求之间清空接收缓存。

下面例子仅查询信息和帮助。所有操作放入同一会话，输出操作前先确认接线。示例错误处理选择停止整段会话，不自动恢复或重试。

```python
import json
import os
import time
import serial

class Control:
    def __init__(self, verified_port):
        options = {"exclusive": True} if os.name == "posix" else {}
        self.ser = serial.Serial(verified_port, 115200, timeout=0.1,
                                 write_timeout=2, **options)
        self.ser.dtr = True
        self.next_id = 1
        self.healthy = True

    def _send(self, payload):
        if not self.healthy:
            raise RuntimeError("Session uncertain; stop and inspect before reopening")
        if self.ser.write(payload) != len(payload):
            raise IOError("Partial write; do not retry automatically")

    def _line(self, deadline):
        data = bytearray()
        while time.monotonic() < deadline:
            b = self.ser.read(1)
            if b:
                data.extend(b)
                if len(data) > 16384:
                    raise ValueError("Unexpected response length")
                if b == b"\n":
                    return bytes(data).decode("utf-8").rstrip("\r\n")
        raise TimeoutError("Incomplete response; device may have executed request")

    def call(self, method, **params):
        request_id = self.next_id
        self.next_id += 1
        payload = json.dumps({"id": request_id, "method": method, **params},
                             separators=(",", ":")).encode("utf-8")
        if len(payload) > 767:
            raise ValueError("Request exceeds 767 bytes")
        try:
            self._send(payload + b"\n")
            reply = json.loads(self._line(time.monotonic() + 5))
            if reply.get("id") != request_id:
                raise ValueError("Response ID mismatch")
            if "error" in reply:
                raise RuntimeError(f"{method}: {reply['error']}")
            if "result" not in reply:
                raise ValueError("Missing result")
            return reply["result"]
        except Exception:
            self.healthy = False
            raise

    def help(self, topic=""):
        if topic not in ("", "protocol", "pins", "gpio", "spi", "i2c", "adc", "uart"):
            raise ValueError("Unknown help topic")
        try:
            self._send(("help" + (" " + topic if topic else "") + "\n").encode())
            lines, deadline = [], time.monotonic() + 5
            while True:
                line = self._line(deadline)
                if line == "END HELP":
                    return "\n".join(lines)
                lines.append(line)
        except Exception:
            self.healthy = False
            raise

    def close(self):
        self.ser.close()

# Set this yourself only AFTER verifying USB interface metadata.
# This environment variable belongs to this example, not the firmware.
c = Control(os.environ["AGENTLINK_CONTROL_PORT"])
try:
    info = c.call("device.info")
    if info.get("control_protocol") != 2:
        raise RuntimeError("Unsupported protocol")
    print(info)
    print(c.help("spi"))
    # Add the intended, wiring-checked operations here, in this same session.
finally:
    c.close()
```

## 请求和响应

以下交替列出请求和响应，**只发送请求行**。响应结构按固件 0.3.3 源码核对，电平和数据仅为示例，不是新实测结果。输出和任意 SPI 字节都可能改变 DUT，不能原样盲发。

```json
{"id":2,"method":"gpio.configure","pin":0,"mode":"output","initial":0,"safe":"float"}
{"id":2,"result":{"pin":0,"mode":"output"}}
{"id":3,"method":"gpio.write","pin":0,"value":1}
{"id":3,"result":{"pin":0,"value":1}}
{"id":4,"method":"gpio.read","pin":0}
{"id":4,"result":{"pin":0,"value":1}}
{"id":5,"method":"spi.configure","mode":0,"hz":1000000,"read":0,"cs":0}
{"id":5,"result":{"mode":0,"clock_hz_upper_bound":750000,"actual_hz":750000,"read":0,"max_chunk":128,"engine":"hardware"}}
{"id":6,"method":"spi.master.transfer","tx_hex":"aabb","cs_hold":0}
{"id":6,"result":{"transferred":2,"rx_hex":"","cs_held":false}}
```
`gpio.read` 的值在 `result.value`，不是顶层。只写 SPI 的 `rx_hex` 为空；启用 read=1 时，每发送一字节得到一字节接收数据，需要 dummy 字节产生读取时钟。引脚未配置为输出时，写入返回例如 `{"id":7,"error":{"code":"not_output"}}`。常见错误还有 `pin_in_use`、`spi_not_configured`、`invalid_hex_or_chunk_size`。不要用自动释放或重试掩盖冲突。

## SPI 文件分块

```python
def spi_bytes(c, data):
    if not data:
        raise ValueError("Empty SPI transfer is not supported")
    received = bytearray()
    for offset in range(0, len(data), 128):
        block = data[offset:offset + 128]
        hold = int(offset + len(block) < len(data))
        result = c.call("spi.master.transfer", cs=0,
                        tx_hex=block.hex(), cs_hold=hold)
        if (result["transferred"] != len(block)
                or result["cs_held"] != bool(hold)):
            c.healthy = False
            raise RuntimeError("Unexpected SPI result; stop")
        received.extend(bytes.fromhex(result["rx_hex"]))
    return bytes(received)

# Within the same open Control session, after checking wiring and pin ownership:
# c.call("spi.configure", mode=0, hz=1000000, read=0, cs=0)
# data = Path("payload.bin").read_bytes()  # from pathlib import Path
# spi_bytes(c, data)
# c.call("spi.release")
```

中间块 cs_hold=1，最后一块为 0。CS 保持期间，不由 GPIO 直接操纵 SPI 占用的 CS。多个块之间存在主机往返延迟；需要连续无间隙时钟的设备不一定适用。DC 等外部控制脚按外设协议在正确边界切换，不能在同一数据流中误切。两个 `spi_bytes` 调用之间会释放 CS；需要命令和数据共享一次 CS 时，应合并或明确规划各块的 cs_hold。

传输或响应超时后可能已有部分字节到达 DUT。停止后续块，不重发，不继续使用这个示例中已标记异常的会话。关闭会话会触发断连安全处理，但不能假设等同于一个正常 SPI 事务结束。确认 DUT 当前状态、连接和安全电平后，按外设要求重新初始化再操作；不要无条件在 finally 中通过异常链路发送 release。已成功且连接正常时，先按外设要求设置 DC/RST/PWR 等，再显式 release 和关闭。释放后的 CS 悬空，若需要稳定去选中，应有适当硬件上拉。

## I2C 和 ADC

以下是独立流程，不应在引脚连接了其他用途时执行：

```json
{"id":10,"method":"i2c.configure"}
{"id":11,"method":"i2c.transfer","address":80,"tx_hex":"00","read_count":4}
{"id":12,"method":"i2c.release"}
```

```json
{"id":20,"method":"adc.configure"}
{"id":21,"method":"adc.read","pin":0}
{"id":22,"method":"adc.release"}
```

I2C 示例假定已确认的 DUT 地址 0x50 和寄存器 0x00，外接 3.3 V 上拉；不是扫描或万能读取命令。ADC pin=0 是 GPIO5 上的 ADC0，输入必须处于安全范围。具体字段见实时 `help i2c`、`help adc`；模拟精度尚待验证。

## 两路 UART

```python
# verified_uart_port must be identified as UART1 or UART2 on the SAME board.
# Do not send example data unless it is appropriate for the attached DUT.
with serial.Serial(verified_uart_port, baudrate=115200, bytesize=8,
                   parity="N", stopbits=1, timeout=1, write_timeout=2,
                   rtscts=False) as uart:
    received = uart.read(256)  # Partial or empty reads are normal.
    print(received.hex())
    # uart.write(b"your DUT command\r\n")  # Raw bytes; no automatic newline.
```

数据串口的波特率和格式由普通串口 API 配置。当前支持 1200～3000000 baud、8 数据位、N/E/O 校验、1/2 停止位。Control 只提供 UART1 流控命令 `{"id":30,"method":"uart.flow_control","port":1,"enabled":1}`，0 为关闭；不要假定主机 rtscts 开关能替代这条设备指令。启用前确认 RTS/CTS 接线；RTS/背压尚未完成验证。不要向 Control 发送 uart.send/read/configure。发送超时可能已发送部分数据，不自动重试。
