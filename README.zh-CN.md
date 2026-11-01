# AgentLink

[English](README.md) | [简体中文](README.zh-CN.md)

面向 AI Agent 的通用嵌入式调试工具使用指南：通过 USB Control 串口的 JSON 协议操作 GPIO、SPI、I2C、ADC，通过两路独立 UART 串口调试 DUT。

## 快速开始

给 Agent 这个原始文本入口即可：

https://raw.githubusercontent.com/fan58614-sketch/agentLink/main/skills/agentlink-uart-io/SKILL.zh-CN.md

先读 Skill 和 [快速开始](skills/agentlink-uart-io/references/quick-start.zh-CN.md)，其余文档按需阅读。支持 Skill 的客户端也可安装完整 `skills/agentlink-uart-io/` 目录，标准入口为 `SKILL.md`。

提示词示例：

> 按上述 Skill，直接通过 AgentLink Control 串口操作我已连接的 DUT。先只读快速开始并查询设备能力，再按任务读取所需帮助。我的 DUT、接线和目标如下：（补充实际内容）。

不需要 AgentLink 专用上位机、CLI 或 MCP 服务。仓库交付的是中英文 Skill 与文档代码示例，不是软件安装包。Agent 仍需要本机终端、串口权限和可用运行环境；使用 Python 示例时需要 pyserial。

外设驱动和图像转换在电脑侧完成，设备保持通用接口。固件自带 help 当前为英文，文档可以切换语言。

## 验证状态

文档基线为固件 0.3.3 / 协议 2。用户已确认另一 Agent 通过直接 Control 串口方式成功刷新墨水屏。其他能力与限制见 [验证记录](skills/agentlink-uart-io/references/validation-status.zh-CN.md)，不能以单次成功代替完整量产和跨平台验证。
