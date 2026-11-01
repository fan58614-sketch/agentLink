---
name: agentlink-uart-io
description: Control AgentLink USB hardware directly through its serial JSON protocol for embedded debugging with GPIO, SPI, I2C, ADC, and two DUT UARTs. Use when a connected AgentLink should observe or stimulate a DUT; no AgentLink-specific host application is required.
---

# AgentLink

[English](SKILL.md) | [简体中文](SKILL.zh-CN.md)

Operate directly through the **Control USB serial JSON-lines protocol** by default. Do not search for or install an AgentLink host application; MCP is not required. This Skill provides instructions, while the agent uses its existing terminal and serial programming facilities. Python with pyserial is one option, not a required proprietary tool. Report missing serial access instead of pretending to execute operations.

Follow the user's language; preserve protocol identifiers. Read only the needed language edition. `SKILL.md` is the standard installation entrypoint.

## Workflow

1. Follow [USB discovery](references/usb-discovery.md) to enumerate ports and metadata without transmitting probes. Identify Control, UART1, and UART2 belonging to the same physical device.
2. Exclusively open the identified Control port and keep one session alive. Request `device.info`, verify protocol 2 and capabilities, then read `help` or a relevant topic through `END HELP`.
3. Read [Control protocol](references/control-protocol.md) and [complete serial examples](references/serial-examples.md). Send one line and wait for its matching response ID. Do not guess response shapes.
4. Establish DUT wiring, 3.3 V logic, direction, and safe state before configuring outputs. Explicitly release owned pins before reassigning them; do not disrupt an existing function.
5. Drive peripherals using generic GPIO/SPI/I2C operations. Keep drivers, image conversion, and peripheral sequences in host code; do not change firmware or add display-specific interfaces.
6. Inspect results and distinguish host writes, device execution, and actual DUT behavior. Never automatically retry uncertain writes. Restore DUT-appropriate levels, release resources, then close the port.

## Boundaries

- UART1 and UART2 are separate data ports, defaulting to 115200/8N1. UART1 has RTS/CTS; UART2 has TX/RX only. Control has no UART byte-send/read methods.
- GPIO0–9 default to floating. GPIO3/4 optionally provide I2C SCL/SDA, GPIO5 ADC0, and GPIO6/7/8/9 SPI SCK/MISO/MOSI/CS. Enabling a bus claims its pins.
- Current prototype logic is 3.3 V. Do not assume 5 V tolerance or protected ADC inputs.
- Disconnect safe state float/low is RAM-only; a device reset restores floating inputs. Software cannot replace physical pull resistors.
- Periodic GPIO, PWM, SPI slave, and power-switch control are not implemented. Driving a peripheral's PWR enable with GPIO does not mean the tool supplies its power.

Consult [validation status](references/validation-status.md). A successful command is not physical verification.
