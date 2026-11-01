# Direct Control serial protocol

**English** | [简体中文](control-protocol.zh-CN.md)

Baseline: firmware 0.3.3, Control protocol 2. Discover the correct port using [USB discovery](usb-discovery.md). Keep one connection open; do not share it with another program or terminal.

Use UTF-8, one JSON object per line, maximum 767 bytes excluding the terminator. LF, CRLF, and CR are accepted. Send one request and wait for its matching `id` response before sending the next. Responses contain `result` or `error.code`. There is no unsolicited startup banner.

```json
{"id":1,"method":"device.info"}
```

No internal mapping identifier is required. Use product-level logical pin numbers.

## Interactive help

Type `help`, or `help protocol`, `help pins`, `help gpio`, `help spi`, `help i2c`, `help adc`, `help uart`, followed by Enter. Help is English and ends with `END HELP`. Read through that marker, not merely one USB packet. Help examples are not executed. Bare topic names such as `spi` are not commands.

## Operations

- GPIO: `gpio.configure`, `gpio.write`, `gpio.read`, `gpio.status`, `gpio.describe`. Pins 0–9. Configure modes: output, input, float, pullup, pulldown, open_drain. Initial level is 0/1; disconnect safe state is float/low. Read current help before changing an output.
- SPI: `spi.configure`, `spi.master.transfer`, `spi.release`. Configure mode 0–3, hz 375000–12000000 at the current clock, read 0/1, cs 0. Transfer 1–128 bytes using `tx_hex`; `cs_hold:1` retains CS between blocks, final block uses 0. Receive clocks require dummy TX bytes. Release restores floating inputs.
- I2C: `i2c.configure`, `i2c.transfer`, `i2c.recover`, `i2c.release`. Address 8–119 (7-bit), required `tx_hex` (may be empty), maximum 64 write bytes and `read_count` 0–64. Empty write and zero read is an explicit address probe.
- ADC: `adc.configure`, `adc.read`, `adc.reference`, `adc.calibrate`, `adc.release`. ADC pin 0 means ADC0 on GPIO5, not GPIO0. Calibration uses `known_mv` and is RAM-only.
- UART: `uart.flow_control` with port 1 and enabled 0/1. UART byte I/O and line coding use the separate UART CDC ports. Control does not implement `uart.send`, `uart.read`, or `uart.configure`.

Example, after confirming that GPIO0 may safely be driven:

```json
{"id":2,"method":"gpio.configure","pin":0,"mode":"output","initial":0,"safe":"float"}
{"id":3,"method":"gpio.write","pin":0,"value":1}
```

Read files on the host and send blocks of at most 128 bytes. Control does not accept file paths. See [serial examples](serial-examples.md) for framing, chunking, and error handling.

## Hardware SPI clock selection (0.3.3)

Wiring and method names are unchanged. The hardware engine defaults to a 1000000 Hz request, producing nominal 750000 Hz with the current 96 MHz peripheral clock. Dividers are 2, 4, 8, 16, 32, 64, 128, 256; select the fastest available clock no greater than the request. Current firmware accepts 375000–12000000 Hz, with available SCK values 375000, 750000, 1500000, 3000000, 6000000, 12000000 Hz.

`actual_hz` reports divider-derived nominal SCK, not an instrument measurement or end-to-end throughput. The retained `clock_hz_upper_bound` field returns the same value. Old 100 kHz/250 kHz requests return `spi_clock_out_of_range` and require a host-script change; they are never silently rounded upward. Requests above 12 MHz return `invalid_spi_config`. Check the peripheral's permitted speed before increasing an old setting.

The 128-byte chunk limit and cs_hold remain. Byte/block gaps are possible; continuous streaming is not guaranteed. Hardware faults or disconnect aborts stop the engine and release ownership, with `spi_transfer_aborted` if a response can still be delivered. Some bytes may already have been sent: never retry automatically. The 12 MHz ceiling is a firmware limit, not qualification of all DUTs at that rate.
