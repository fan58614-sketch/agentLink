# AgentLink

**English** | [简体中文](README.zh-CN.md)

Bilingual skills for AI agents to debug embedded devices with AgentLink via MCP: UART, GPIO, SPI master, I2C master, and ADC.

## What is included

This repository currently distributes **Skill documentation only**. It does not include the MCP server, firmware, hardware design, or an installer. Downloading a Skill does not create MCP tools or grant serial-port access.

AgentLink provides generic hardware operations. Peripheral drivers, display commands, and application logic run on the host using those operations.

## Get started

1. Read [SKILL.md](skills/agentlink-uart-io/SKILL.md), or its [Chinese edition](skills/agentlink-uart-io/SKILL.zh-CN.md).
2. For an agent that supports Skills, install the complete `skills/agentlink-uart-io/` folder according to that client's instructions. Preserve the references and directory structure. The standard entrypoint is `SKILL.md`.
3. Separately configure an available AgentLink MCP server, or use a serial-capable environment for direct Control access. A public MCP installation package is not provided here yet.
4. Follow [USB discovery](skills/agentlink-uart-io/references/usb-discovery.md) to identify the correct device, then query help and capabilities before operating it.

Do not identify ports by transmitting probe data to unknown serial ports. Use USB interface metadata and physical device identity; serial paths differ across computers.

## Documentation

- [MCP usage](skills/agentlink-uart-io/references/mcp-usage.md)
- [Control serial protocol](skills/agentlink-uart-io/references/control-protocol.md)
- [USB port identification](skills/agentlink-uart-io/references/usb-discovery.md)
- [Validation status and limitations](skills/agentlink-uart-io/references/validation-status.md)

Each document has an English / 简体中文 switch. Tool names and protocol fields remain unchanged in both editions. Firmware and MCP help currently remain English.

## Development status

Documentation baseline: firmware 0.3.2 / Control protocol 2. Capabilities and physical verification are separate: consult the validation record before relying on a feature. This repository does not claim completed cross-platform or production qualification.
