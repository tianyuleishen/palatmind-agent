# PalatMind — Windows 桌面 AI 智能体

> 看懂屏幕、规划步骤、执行操作、验证结果——让 AI 像人一样使用电脑。

| | |
|---|---|
| 官网 | https://palatmind.com |
| 下载 | https://palatmind.com/download/ |
| 文档中心 | https://palatmind.com/docs/ |

## 这是什么？

PalatMind 是一款面向 Windows 的桌面级 AI 智能体（AI Agent）。与只能"对话"的 AI 助手不同，PalatMind 能真正操作你的电脑：理解自然语言指令后，自动拆解任务、调用系统与应用能力完成工作，并在结束后验证结果。

一句话：**你说目标，它来操作。**

## 核心能力

- **桌面自动化**：直接操作真实的应用与窗口（浏览器、办公软件、播放器等），无需为目标应用做特殊适配
- **文档与演示创作**：一句话生成 Word 周报、Excel 报表、PPT 大纲与成品
- **信息获取与研究**：联网搜索、多来源交叉验证、自动整理成文
- **语音交互**：语音输入指令，语音播报结果
- **多 Agent 并行**：复杂任务自动拆解为子任务并行执行，主 Agent 统一仲裁结果
- **记忆系统**：跨会话记住你的偏好与上下文，越用越懂你
- **自我进化**：从重复任务中沉淀可复用技能，同类任务越跑越快
- **安全分级审批**：普通操作自动执行，敏感操作执行前确认，危险操作强制确认，兜底可控

## 快速开始

1. 前往 [官网下载页](https://palatmind.com/download/) 获取安装包（Windows 10/11 x64）
2. 按照安装向导完成安装
3. 首次启动注册/登录账号，按需配置大模型服务商的 API Key
4. 在对话框里用自然语言下达任务，例如：

   > 帮我打开酷狗音乐，播放周杰伦的《晴天》

更多玩法请看 [使用指南](https://palatmind.com/docs/guide/)。

## 技术分享（理念文章）

本仓库**不包含源码**，只分享我们在构建桌面智能体过程中的工程思考与方法论：

| 文章 | 主题 |
|------|------|
| [01 桌面 Agent 架构](docs/01-desktop-agent-architecture.md) | 感知—规划—执行—验证的闭环设计 |
| [02 多 Agent 并行协作](docs/02-multi-agent-parallel.md) | 任务拆解、失败处理与结果仲裁 |
| [03 自我进化](docs/03-self-evolution.md) | 技能沉淀、安全升级与回滚 |
| [04 记忆系统](docs/04-memory-system.md) | 分层记忆与按需检索 |

## 反馈与交流

- 使用问题 / Bug 反馈：请提 [Issue](https://github.com/tianyuleishen/palatmind-ai/issues)
- 功能建议：欢迎在 Issue 中描述你的使用场景
- 安装与使用问题优先查阅 [文档中心](https://palatmind.com/docs/) 与 [FAQ](https://palatmind.com/faq/)

## 版权声明

© 2026 PalatMind. 保留所有权利。

本仓库仅用于产品介绍与技术理念分享，**不含任何源代码**。未经书面许可，禁止复制、转载本仓库内容用于商业用途。
