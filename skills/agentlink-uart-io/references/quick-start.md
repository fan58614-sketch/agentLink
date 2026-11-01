# Quick start

[English](quick-start.md) | [简体中文](quick-start.zh-CN.md)

This page is enough for the first read-only connection; use live device help for the task instead of reading every document. Local serial permissions and a serial API are required. Python + pyserial is optional; no dedicated package is needed. Baseline: firmware 0.3.3 / protocol 2.

## 1. Identify the port without transmitting probes

Enumerate USB serial metadata. VID/PID `1A86:FE0C` filters candidates, not unique identity. Group by physical USB device location, then identify roles by interface name or number:

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

If interface names are missing, read only your OS steps in [USB discovery](usb-discovery.md). Never infer roles from path suffixes or COM order; prototype serial numbers repeat. Re-enumerate on reconnect/new sessions; remembered wiring is useful, remembered port paths are not identity. Never send help, newline, or JSON to unknown ports; ask for selection if ambiguous.

## 2. Query and operate in one session

Open the verified, unoccupied Control port at 115200/8N1 with DTR enabled. Send requests only, never the response example:

```json
{"id":1,"method":"device.info"}
```

Wait for the complete newline-terminated response, check `id=1`, `result.control_protocol=2`, and required capabilities. Query only the needed help topic, for example SPI:

```text
help spi
```

Read through `END HELP` before the next JSON. Topics: gpio, spi, i2c, adc, uart, pins, protocol. Do not query them all. Keep the identification session open for the actual operation.

One outstanding request, maximum 767 bytes excluding line ending. Await matching id with `result` or `error.code`. Accumulate through newline: one USB/serial read may be partial. Do not clear receive buffers between requests. For ready-to-adapt reliable framing, read [the session example](serial-examples.md#session-rules).

## 3. Select the operation for the task

| Function | Essential usage |
| --- | --- |
| GPIO | `gpio.configure`, then `gpio.write` / `gpio.read`; logical pin 0–9. Input level is `result.value`. |
| SPI | `spi.configure` → `spi.master.transfer` → `spi.release`. Modes 0–3, 8-bit MSB-first, cs=0. Default 1 MHz request gives nominal 750 kHz currently. |
| I2C | `i2c.configure` → `i2c.transfer` → `i2c.release`; external 3.3 V pullups, 7-bit addresses, no automatic scan. |
| ADC | `adc.configure` → `adc.read` with pin=0 meaning ADC0 on GPIO5 → `adc.release`; accuracy remains unqualified. |
| UART | Use ordinary serial APIs on the corresponding separate UART port. Default 115200/8N1. No Control uart.send/read/configure methods; only UART1 uart.flow_control. |

GPIO3/4 optionally provide I2C SCL/SDA, GPIO5 ADC0, GPIO6/7/8/9 SPI SCK/MISO/MOSI/CS. Enabling a bus claims its pins. SPI read=0 does not claim MISO.

**SPI example: send only after verifying wiring, voltage, and peripheral clock limits:**

```json
{"id":2,"method":"spi.configure","mode":0,"hz":1000000,"read":0,"cs":0}
```

Example response:

```json
{"id":2,"result":{"mode":0,"clock_hz_upper_bound":750000,"actual_hz":750000,"read":0,"max_chunk":128,"engine":"hardware"}}
```

Send bytes using `tx_hex`, 1–128 bytes per block; intermediate blocks use `cs_hold:1`, final block `cs_hold:0`. Results contain `transferred`, `rx_hex` (empty for TX-only), and `cs_held`. Read files and chunk them on the host; never send a filesystem path to firmware. Read [chunking code](serial-examples.md#spi-file-chunking) only when needed.

Current requests: 375000–12000000 Hz, rounded down to a hardware divider; `actual_hz` is nominal clock. Old 100/250 kHz is unsupported; never automatically increase frequency to bypass an error. Mode 0 / 750 kHz display TX is physically verified; other mode transfers, RX, and high-speed DUT use remain unverified. Supported range is not qualified range.

## 4. Failure and completion

- Current logic is 3.3 V; do not assume 5 V tolerance. Avoid driving DUT outputs or stealing occupied bus pins.
- Timeout/partial writes mean uncertain execution: stop, never automatically resend. Resolve ownership/configuration errors by their cause.
- Decide peripheral-appropriate failure handling before starting. Neither indefinite power to “preserve the scene” nor immediate power-off is a universal default; use peripheral documentation and link health. Cleanup delivery is not guaranteed over a failed connection.
- On success, apply DUT sleep/safe levels, release pins, then close. Keep connections alive for tasks that must hold an output. Disconnect safe=float/low is RAM-only; device reset restores floating and does not replace hardware pulls.
- Confirm actual image, colors, or measurements independently; BUSY release is not proof of complete correctness.
