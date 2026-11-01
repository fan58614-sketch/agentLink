# AgentLink

[English](README.md) | [简体中文](README.zh-CN.md)

面向 AI Agent 的通用嵌入式调试工具使用指南：通过 USB Control 串口的 JSON 协议操作 GPIO、SPI、I2C、ADC，通过两路独立 UART 串口调试 DUT。

## 使用方式

1. 让 Agent 阅读 [中文 Skill](skills/agentlink-uart-io/SKILL.zh-CN.md)。支持 Skill 安装的客户端可安装整个 `skills/agentlink-uart-io/` 目录，保留结构；标准入口是 `SKILL.md`。
2. 按 [USB 识别规则](skills/agentlink-uart-io/references/usb-discovery.zh-CN.md) 找到设备的 Control，不向未知串口试发数据。
3. 使用已有串口访问能力，发送 `device.info`、`help`，然后按实际接线操作。参考 [完整通信示例](skills/agentlink-uart-io/references/serial-examples.zh-CN.md) 和 [协议说明](skills/agentlink-uart-io/references/control-protocol.zh-CN.md)。

不需要 AgentLink 专用上位机、CLI 或 MCP 服务。仓库交付的是中英文 Skill 与文档代码示例，不是软件安装包。Agent 仍需要本机终端、串口权限和可用运行环境；使用 Python 示例时需要 pyserial。

外设驱动和图像转换在电脑侧完成，设备保持通用接口。固件自带 help 当前为英文，文档可以切换语言。

## 验证状态

文档基线为固件 0.3.2 / 协议 2。用户已确认另一 Agent 通过直接 Control 串口方式成功刷新墨水屏。其他能力与限制见 [验证记录](skills/agentlink-uart-io/references/validation-status.zh-CN.md)，不能以单次成功代替完整量产和跨平台验证。
