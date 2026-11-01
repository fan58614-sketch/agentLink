# Validation boundaries

**English** | [简体中文](validation-status.zh-CN.md)

Documentation baseline: firmware 0.3.2 / Control protocol 2. This is a development snapshot, not a production qualification report.

| Area | Evidence and remaining work |
| --- | --- |
| macOS USB / Control help | Physically exercised, including long multi-packet help, line endings, and error recovery. |
| Windows / Linux / USB full speed | Physical validation outstanding. Do not infer coverage from host tests. |
| Dual UART CDC | 115200, 1 Mbaud, and 3 Mbaud exercised; bounded concurrent transfers at 3 Mbaud matched. Continuous line-rate and long-duration qualification remain open. |
| UART1 CTS | Pause/resume physically exercised at 115200. |
| UART1 RTS | Validation deferred; do not claim qualified RTS/backpressure behavior. |
| SPI | Mode 0 transmit used through generic MCP to refresh a real display. Other modes and receive need physical validation. |
| I2C | Physical bus validation outstanding. |
| ADC | Current board analog path and measurement accuracy require validation. |

3 Mbaud is the product target and an exercised setting, not proof of every flow-control, sustained throughput, or host-platform scenario. No periodic GPIO, PWM, SPI-slave, or display-specific firmware service is currently implemented.

2026-11-01: A user-provided Claude execution log records direct Control serial initialization, 2×5920-byte transmission, and display refresh; the user confirmed successful display. This verifies the direct serial path, not MCP.
