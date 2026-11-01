---
name: agentlink-uart-io
description: Operate an AgentLink USB tool for embedded DUT debugging through two UARTs, GPIO, SPI master, I2C master, and ADC. Use when a user wants to observe or stimulate connected hardware, including driving a peripheral through generic bus operations.
---

# AgentLink

**English** | [简体中文](SKILL.zh-CN.md)

Use the available AgentLink MCP tools and their live schemas. Start with `help`, then `device_info`. Read [USB discovery](references/usb-discovery.md) before selecting ports and [MCP usage](references/mcp-usage.md) before operating hardware. For direct serial access, read [Control protocol](references/control-protocol.md).

Follow the user's requested language, otherwise their current conversation language. Chinese readers can use the linked Chinese edition and references. Keep tool names, commands, JSON fields, interface strings, and protocol markers unchanged. Read only the language edition needed for the task. `SKILL.md` remains the standard installation entrypoint.

## Workflow

1. Identify the intended physical device without sending probes to unidentified serial ports. Resolve ambiguity with the user.
2. Establish DUT wiring, voltage domain, and intended operation from the user's project. Use logical GPIO names; do not require internal chip names, package pins, or internal mapping identifiers.
3. Inspect capabilities and pin ownership. Read live help for the relevant operation. Release an owned function before assigning its pins elsewhere.
4. Keep connections open for the operation. Configure direction and initial level before driving outputs; avoid contention with DUT outputs.
5. Perform the requested operation and inspect responses. Distinguish host acceptance, DUT response, and user-observed physical results. Do not automatically retry writes after uncertain partial transmission.
6. Restore the agreed safe state and release resources when finished. Do not close a connection that must maintain the requested output.

## Product boundaries

- Three USB CDC ports: UART1, Control, UART2. UART1 has RTS/CTS; UART2 has TX/RX. Both default to 115200, 8N1, flow control disabled.
- Ten logical pins, GPIO0 through GPIO9, default to floating input. GPIO3/4 optionally provide I2C SCL/SDA; GPIO5 provides ADC0; GPIO6/7/8/9 provide SPI SCK/MISO/MOSI/CS. Other pins remain GPIO.
- Current prototype signals are 3.3 V. Do not assume 5 V tolerance or protected ADC inputs. Confirm the hardware revision's electrical limits before connecting a DUT.
- Disconnect safe state can be floating or low, configured in RAM. A device reset returns pins to floating; software safe state does not replace a physical pull resistor.
- Expose generic bus capabilities. Keep display drivers, peripheral command sequences, and image conversion in host-side application code. Do not invent display-specific firmware commands.
- PWM, periodic GPIO sequences, SPI slave, and power-switch control are not currently implemented.

Consult [validation status](references/validation-status.md) before claiming a feature or throughput is physically verified. A Skill is operating guidance: it does not itself install MCP or grant serial-port access.
