# AgentLink

[English](SKILL.md) | [简体中文](SKILL.zh-CN.md)

**默认直接使用 Control USB 串口 JSON，不需要 AgentLink 软件包或 MCP。** Agent 使用已有终端/串口能力；没有本机访问能力就说明缺失条件，不假装执行。

## 最短路径

1. **只读 [快速开始](references/quick-start.zh-CN.md)**，完成端口识别、`device.info` 和当前任务所需的 `help <topic>`。不用先读完所有参考文档。
2. 在同一连接中执行任务。已有完整接线和目标时直接继续，不重复询问。缺少会影响电气安全或操作正确性的信息时才澄清。
3. 按外设要求收尾，报告真实响应与观察结果；不把指令成功当成实际显示/测量正确。

## 按需查阅

| 遇到的问题 | 只补读这一项 |
| --- | --- |
| 无法唯一识别端口或有多台设备 | [USB 识别](references/usb-discovery.zh-CN.md) |
| 需要完整 Python 收发、文件分块、UART 代码 | [通信示例](references/serial-examples.zh-CN.md)对应章节 |
| 实时帮助不足以解释参数或错误 | [协议](references/control-protocol.zh-CN.md)对应章节 |
| 判断某模式、速率或平台是否已实测 | [验证状态](references/validation-status.zh-CN.md) |

只读一种语言，按用户语言回复；命令、字段不翻译。通过网页读取时，直接获取本文件及所需相对链接的 raw 文本，不枚举 GitHub 目录，也不获取整个仓库。原始入口：
https://raw.githubusercontent.com/fan58614-sketch/agentLink/main/skills/agentlink-uart-io/SKILL.zh-CN.md

外设驱动、图像转换和设备命令属于主机应用。无需为了连接工具先搜索外设资料；可以先做只读设备识别。查外设资料时优先使用官方页面提供的源码链接，不反复猜测已返回 404 的路径。

固件不包含屏幕专用功能；不要改固件来完成外设任务。当前不支持周期 GPIO、PWM、SPI 从机或电源开关服务。GPIO 控制 DUT 的 PWR 使能不等于工具提供电源。
