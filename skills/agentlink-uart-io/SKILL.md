---
name: agentlink-uart-io
description: Operate connected AgentLink hardware directly through USB serial JSON for GPIO, SPI, I2C, ADC, and two DUT UARTs. Use for embedded DUT observation and stimulation without an AgentLink-specific host application.
---

# AgentLink

[English](SKILL.md) | [简体中文](SKILL.zh-CN.md)

**Use Control USB serial JSON directly. No AgentLink package or MCP is required.** Use available terminal/serial facilities; report missing local access rather than pretending to execute.

## Shortest path

1. **Read only [Quick start](references/quick-start.md)** to identify the port, query `device.info`, and read the task's `help <topic>`. Do not preload every reference.
2. Execute in the same connection. Proceed when wiring and intent are already supplied; do not repeatedly ask. Clarify only missing information that affects electrical safety or correctness.
3. Finish according to the peripheral's requirements and report actual responses/observations. Command success does not prove correct display or measurement.

## Read on demand

| Situation | Read only this |
| --- | --- |
| Port identity is ambiguous or multiple boards are present | [USB discovery](references/usb-discovery.md) |
| Need complete Python framing, file chunking, or UART code | Relevant section of [serial examples](references/serial-examples.md) |
| Live help does not explain a parameter/error sufficiently | Relevant section of [protocol](references/control-protocol.md) |
| Need qualification evidence for a mode, speed, or platform | [Validation status](references/validation-status.md) |

Read one language edition; follow the user's language without translating identifiers. When fetching online, retrieve this file and needed relative references as raw text. Do not enumerate GitHub directories or fetch the entire repository. Raw entrypoint:
https://raw.githubusercontent.com/fan58614-sketch/agentLink/main/skills/agentlink-uart-io/SKILL.md

Peripheral drivers, image conversion, and device commands belong in the host application. Read-only AgentLink identification need not wait for peripheral research. Prefer source links supplied by official peripheral documentation; do not repeatedly guess paths already returning 404.

Do not modify firmware to implement a peripheral task. Periodic GPIO, PWM, SPI slave, and power-switch services are not implemented. Driving a DUT PWR enable through GPIO does not mean the tool supplies power.
