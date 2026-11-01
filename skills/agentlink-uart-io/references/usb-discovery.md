# USB discovery without transmitting probes

**English** | [简体中文](usb-discovery.zh-CN.md)

Current prototype candidate VID/PID: `0x1A86:0xFE0C`. This pair alone is not a unique product identity.

| Role | USB interface string | Communication / data interfaces |
| --- | --- | --- |
| UART1 | `AgentLink UART` | 0 / 1 |
| Control | `AgentLink Control` | 2 / 3 |
| UART2 | `AgentLink UART2` | 4 / 5 |

Use USB interface metadata and physical USB ancestry to assign roles. The UART1 string currently does not contain the digit 1.

## Discovery procedure

1. Enumerate serial ports and their USB metadata without opening or writing to them.
2. Identify AgentLink candidates by VID/PID and interface metadata. Determine role from the interface string or the interface number associated with the CDC function above.
3. Group ports by physical USB device location/topology. Select one board, then select its Control and UART interfaces from that same group.
4. If metadata is missing or contradictory, show candidate names, paths, and locations and ask for explicit selection. Never select the first candidate arbitrarily.
5. Only after identifying Control, open it and request `device.info` to verify the protocol and capabilities.

Never send `help`, JSON, AT commands, a newline, or any other probe to unknown ports. UART ports forward bytes to the DUT.

## Platform details

- macOS: use USB interface metadata and IORegistry ancestry. `/dev/cu.usbmodem...` names are assigned by the OS; their suffix is not a portable role identifier.
- Windows: use USB parent identity and interface information, including `MI_XX` where available. Match the exposed interface to the CDC function; do not infer role from COM numbering.
- Linux: use USB sysfs ancestry and `bInterfaceNumber`; `/dev/ttyACM...` enumeration order is not a role identifier.

Different stacks may expose a communication-interface or data-interface identifier. Normalize to the table above. Actual Windows and Linux behavior still needs physical validation; do not describe this intended mapping as universally verified by the current implementation.

The prototype serial string is `0123456789`, shared across units. It cannot distinguish two boards. A future release should provide a unique per-device serial; until then physical topology is essential. Re-enumerate after unplug/replug or USB reset rather than assuming an old path remains valid.

## Explicit selection

The host MCP supports `AGENTLINK_CONTROL_PORT`, `AGENTLINK_UART1_PORT`, and `AGENTLINK_UART2_PORT`. UART configuration also supports `device_path`. Use paths discovered on this computer, never a path copied from another installation. Explicit selection must still match the intended interface and board.

Keep Control exclusive to one client: close picocom before MCP opens it, and vice versa. Apply the same rule to each UART. Do not seize ports from another process.
