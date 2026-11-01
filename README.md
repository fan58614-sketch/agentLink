# AgentLink

[English](README.md) | [简体中文](README.zh-CN.md)

Operating guidance for AI agents debugging embedded devices: use USB Control serial JSON for GPIO, SPI, I2C, and ADC, and two separate UART serial ports for DUT communication.

## Get started

1. Read [SKILL.md](skills/agentlink-uart-io/SKILL.md). Skill-capable clients may install the complete `skills/agentlink-uart-io/` folder, preserving its structure and standard `SKILL.md` entrypoint.
2. Follow [USB discovery](skills/agentlink-uart-io/references/usb-discovery.md) to identify Control without transmitting probes to unknown ports.
3. Use available serial facilities to request `device.info` and `help`, then operate according to actual wiring. See [complete serial examples](skills/agentlink-uart-io/references/serial-examples.md) and [protocol reference](skills/agentlink-uart-io/references/control-protocol.md).

No AgentLink application, CLI, or MCP server is required. This repository distributes bilingual Skills and documentation snippets, not a software installer. The agent still needs a local execution environment and serial permissions; the Python examples require pyserial.

Peripheral drivers and image conversion run on the computer; hardware interfaces remain generic. Firmware help currently remains English; documentation supports language switching.

## Validation status

Baseline: firmware 0.3.2 / protocol 2. The user confirmed another agent successfully refreshed an e-paper display through direct Control serial access. See [validation status](skills/agentlink-uart-io/references/validation-status.md) for remaining limits; one success does not establish production or cross-platform qualification.
