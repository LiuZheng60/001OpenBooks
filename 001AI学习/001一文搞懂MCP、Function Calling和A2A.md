# 001、一文搞懂MCP、Function Calling和A2A

<br />

作者：LZ，2025-11-27

转载：<https://mp.weixin.qq.com/s/dAT6l3myGKT3_hX12sMyXw>

原创：从码农到工匠

<br />

单纯的大模型是一个只会聊天的“学霸”，而配上Agent的大模型将是“万能助手”。你让它“关掉客厅的灯”，它不再只是礼貌地回答你“好的，已为您关闭客厅的灯”，而是真的动手把灯关了。
在这场关于智能体（AI Agent）的进化革命中，背后隐藏着三个关键“武器”：MCP（Model Context Protocol）、Function Calling和A2A（Agent to Agent protocol）。简单来说，MCP、Function Calling和A2A分别代表三种AI Agent和外界不同的协作方式：

1. MCP ：AI Agent 与 AI Tools 之间的工具发现、注册和调用协议。注意：MCP并不会和LLM直接交互，不要被名字中的Model误导了，看完文章你就明白了。
2. Function Calling：AI Agent 与 AI Model 之间的工具调用协议
3. A2A：AI Agent 与 AI Agent 之间的发现与任务分配协议

<br />

![1.00](/001/001.png)

