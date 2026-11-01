# AgentLink

[English](README.md) | **简体中文**

面向 AI Agent 的中英文 Skill，通过 MCP 使用 AgentLink 调试嵌入式设备，涵盖 UART、GPIO、SPI 主机、I2C 主机和 ADC。

## 仓库内容

本仓库目前仅发布 **Skill 文档**，不包含 MCP 服务程序、固件、硬件设计或安装程序。下载 Skill 不会自动获得 MCP 工具或串口访问权限。

AgentLink 提供通用硬件操作能力。外设驱动、屏幕命令和应用逻辑在主机侧调用这些能力实现。

## 开始使用

1. 阅读 [中文 Skill](skills/agentlink-uart-io/SKILL.zh-CN.md) 或 [英文 Skill](skills/agentlink-uart-io/SKILL.md)。
2. 如果 Agent 支持 Skill，按对应客户端的说明安装整个 `skills/agentlink-uart-io/` 目录，保留参考文档和目录结构。标准入口是 `SKILL.md`。
3. 另行配置可用的 AgentLink MCP 服务，或通过具备串口访问能力的环境直接使用 Control。本仓库暂未提供公开的 MCP 安装包。
4. 按 [USB 识别规则](skills/agentlink-uart-io/references/usb-discovery.zh-CN.md) 选择设备，操作前查询帮助与实际能力。

不要向未知串口发送数据来探测用途。应通过 USB 接口元数据和物理设备身份识别；不同电脑的串口路径并不相同。

## 文档

- [MCP 使用说明](skills/agentlink-uart-io/references/mcp-usage.zh-CN.md)
- [Control 串口协议](skills/agentlink-uart-io/references/control-protocol.zh-CN.md)
- [USB 串口识别](skills/agentlink-uart-io/references/usb-discovery.zh-CN.md)
- [验证状态与限制](skills/agentlink-uart-io/references/validation-status.zh-CN.md)

每页顶部可切换 English / 简体中文，两种语言中的工具名和协议字段保持一致。固件和 MCP 的 help 目前仍为英文。

## 开发状态

文档基线为固件 0.3.2 / Control 协议 2。功能支持与实物验证是两个不同维度，使用前请查阅验证记录。本仓库不宣称已经完成全部跨平台验证或量产验证。
