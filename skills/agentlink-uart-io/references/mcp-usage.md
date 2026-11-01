# MCP usage

**English** | [简体中文](mcp-usage.zh-CN.md)

The host package provides a stdio MCP server, started as `agentlink-mcp` or `python -m agentlink.mcp_server` in the environment where the package is installed. Configure the agent's MCP client to launch that environment's executable. No public package or GitHub installation URL is assigned yet; do not invent one.

Port discovery is normally automatic. Use explicit port environment variables only when necessary; see [USB discovery](usb-discovery.md). The Skill can be distributed separately, but the MCP implementation must also be installed for these tools to exist. Direct Control serial access remains possible without MCP.

## Discover operations

Call `help` for the overview, then request a topic or exact tool name using its exposed schema. Use `device_info` and `gpio_describe` to inspect the connected revision and pin ownership. Treat live tool schemas as authoritative for argument types.

Tool families:

- `gpio_configure`, `gpio_write`, `gpio_read`, `gpio_describe`, `gpio_status`
- `spi_configure`, `spi_transfer`, `spi_write_file`, `spi_release`
- `i2c_configure`, `i2c_transfer`, `i2c_recover`, `i2c_release`
- `adc_configure`, `adc_read`, `adc_reference`, `adc_calibrate`, `adc_release`
- `uart_configure`, `uart_send`, `uart_read`, `uart_status`, `uart_close`, `uart_flow_control`

## UART sessions

Configure port 1 or 2 with the DUT's known baud, parity, and stop bits. Current support is 1200–3000000 baud, 8 data bits, N/E/O parity, and 1/2 stop bits. RTS/CTS is available on UART1 only. Close an existing session before reconfiguring it; opening/closing can discard queued data.

`uart_send` accepts exactly one of hexadecimal bytes or UTF-8 text, 1–65536 bytes, without appending a newline. Host-accepted bytes do not prove DUT receipt. A timeout may leave an unknown transmitted prefix; do not retry automatically.

`uart_read` consumes up to the requested count (1–65536), with a timeout of 0–5000 ms; zero means poll. Empty or partial results are normal. Preserve `rx_hex` for exact bytes; displayed UTF-8 text may contain replacements or split characters.

`uart_status` reports host-session settings, not independent hardware register verification. `uart_close` closes the host handle; it does not disable the hardware UART or necessarily disable flow control.

## GPIO and buses

MCP `gpio_write` configures push-pull output; direct Control `gpio.write` requires an existing output configuration. Inspect the connection before calling either.

SPI is 8-bit MSB-first, one active-low CS, software timing with a requested upper bound of 1–250 kHz. MCP chunks transfers into firmware-sized blocks while maintaining CS as requested. Large transfers can use `spi_write_file` with an absolute path local to the MCP server; file hashes establish source identity, not DUT receipt. Readback needs transmitted dummy bytes. Use generic GPIO for peripheral reset/DC signals.

I2C requires external 3.3 V pullups. Configure before transfer; write/read combines repeated START. Recovery emits clocks and STOP, so invoke it deliberately. Do not scan or retry a DUT bus automatically.

ADC0 uses GPIO5. Read the internal-reference and calibration information along with measurements. Calibration is RAM-only. Do not assume analog accuracy or protected 5 V measurement from the existence of the ADC tool.
