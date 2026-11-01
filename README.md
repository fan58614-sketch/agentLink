# AgentLink

[English](README.md) | [简体中文](README.zh-CN.md)

Operating guidance for AI agents debugging embedded devices: use USB Control serial JSON for GPIO, SPI, I2C, and ADC, and two separate UART serial ports for DUT communication.

## Quick start

Give your agent this raw-text entrypoint:

https://raw.githubusercontent.com/fan58614-sketch/agentLink/main/skills/agentlink-uart-io/SKILL.md

Read the Skill and [Quick start](skills/agentlink-uart-io/references/quick-start.md); consult other documents only as needed. Skill-capable clients may install the complete `skills/agentlink-uart-io/` folder with `SKILL.md` as its standard entrypoint.

Example prompt:

> Follow this Skill to operate my connected DUT directly through AgentLink Control serial. Start with only the quick start and device capabilities, then read the help needed for this task. My DUT, wiring, and goal are: (provide actual details).

No AgentLink application, CLI, or MCP server is required. This repository distributes bilingual Skills and documentation snippets, not a software installer. The agent still needs a local execution environment and serial permissions; the Python examples require pyserial.

Peripheral drivers and image conversion run on the computer; hardware interfaces remain generic. Firmware help currently remains English; documentation supports language switching.

## Validation status

Baseline: firmware 0.3.3 / protocol 2. The user confirmed another agent successfully refreshed an e-paper display through direct Control serial access. See [validation status](skills/agentlink-uart-io/references/validation-status.md) for remaining limits; one success does not establish production or cross-platform qualification.
