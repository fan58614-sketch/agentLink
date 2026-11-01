# Serial communication and generic examples

[English](serial-examples.md) | [简体中文](serial-examples.zh-CN.md)

These are documentation snippets, not an AgentLink client to install. The agent may implement them with available facilities. Python examples require pyserial; report a missing dependency and handle it according to the user's environment. Do not search for an AgentLink package.

## Session rules

Open Control at nominal 115200/8N1; its CDC baud is not SPI speed. Keep DTR and the connection active instead of reopening for each command. Allow one outstanding request, accumulate through newline, and never treat one USB read as a complete response. Help is multiline text: drain through `END HELP` before returning to JSON. Do not clear receive buffers between requests.

This example only queries information and help. Put operations inside the same session and check wiring before outputs. Its conservative error policy stops the session rather than recovering or retrying automatically.

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

## Requests and responses

Each request is followed by its response: **send only request lines**. Shapes were checked against firmware 0.3.3 source; levels and bytes are illustrative, not new measurements. Output operations and arbitrary SPI bytes can change a DUT; do not send them blindly.

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
`gpio.read` places the value in `result.value`, not at the top level. TX-only SPI returns empty `rx_hex`; with read=1 each transmitted byte clocks one received byte, including dummy bytes for reading. Writing an input returns an error such as `{"id":7,"error":{"code":"not_output"}}`. Other errors include `pin_in_use`, `spi_not_configured`, and `invalid_hex_or_chunk_size`. Do not hide conflicts by automatic release or retry.

## SPI file chunking

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

Use cs_hold=1 for intermediate blocks and 0 for the final block. Do not drive bus-owned CS through GPIO. Host round trips introduce gaps between blocks; peripherals requiring uninterrupted clocks may not work. Change DC and other control pins only at peripheral-defined boundaries. Separate `spi_bytes` calls release CS between them; combine data or explicitly plan cs_hold if command and payload must share a CS assertion.

After a transfer/response timeout some bytes may already have reached the DUT. Stop further blocks, do not resend, and do not reuse a session marked uncertain by this example. Closing triggers disconnect handling but is not guaranteed equivalent to a normal SPI transaction end. Inspect DUT state, wiring, and safe levels, then reinitialize as its protocol requires. Do not unconditionally send release over a broken session in finally. On normal success, apply DUT-appropriate DC/RST/PWR states, explicitly release, and close. Released CS floats; provide an appropriate hardware pullup if stable deselection is required.

## I2C and ADC

These are independent flows; do not execute them with pins wired for another purpose:

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

The I2C example assumes a verified DUT at 0x50 with register 0x00 and external 3.3 V pullups; it is not a scan or universal read. ADC pin=0 means ADC0 on GPIO5, with a safe input voltage. Consult live `help i2c` and `help adc`; analog accuracy remains unverified.

## Two UART ports

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

Ordinary serial APIs set the data port's baud and format. Current support is 1200–3000000 baud, 8 bits, N/E/O parity, and 1/2 stop bits. Control provides only UART1 flow control: `{"id":30,"method":"uart.flow_control","port":1,"enabled":1}`; use 0 to disable. Do not assume the host rtscts flag replaces that device command. Verify RTS/CTS wiring first; RTS/backpressure qualification remains incomplete. Do not send uart.send/read/configure to Control. A send timeout may mean partial transmission; never retry automatically.
