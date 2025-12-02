# 001、一文搞懂MCP、Function Calling和A2A

<br />

作者：LZ，2025-11-27

转载：<https://mp.weixin.qq.com/s/dAT6l3myGKT3_hX12sMyXw>

原创：从码农到工匠

<br />

**单纯的大模型是一个只会聊天的“学霸”，而配上Agent的大模型将是“万能助手”**。你让它“关掉客厅的灯”，它不再只是礼貌地回答你“好的，已为您关闭客厅的灯”，而是真的动手把灯关了。

在这场关于智能体（AI Agent）的进化革命中，背后隐藏着三个关键“武器”：MCP（Model Context Protocol）、Function Calling和A2A（Agent to Agent protocol）。简单来说，MCP、Function Calling和A2A分别代表三种AI Agent和外界不同的协作方式：

1. **MCP**

   ：AI Agent 与 AI Tools 之间的工具发现、注册和调用协议。**注意：MCP并不会和LLM直接交互，不要被名字中的Model误导了**，看完文章你就明白了。
2. **Function Calling**

   ：AI Agent 与 AI Model 之间的工具调用协议
3. **A2A**

   ：AI Agent 与 AI Agent 之间的发现与任务分配协议

<br />

![1.02](https://github.com/LiuZheng60/001OpenBooks/blob/main/001Book_AI/image/001/001.png?raw=true)

本文会深入解析MCP、Function Calling和A2A的协议细节，揭示其背后的本质，以便我们在工作中可以更好地开发、使用Agent技术。

# **1. AI Agent**

首先，让我们了解一下什么是AI Agent（人工智能体）？**AI Agent 是一种能自主感知环境、自主规划并执行动作，以完成特定任务的智能系统**。关于LLM和Agent的关系，有个形象的比喻——LLM是AI Agent的“大脑”，而AI Agent是LLM的“身体”。

* **没有Agent的LLM**

  ，能力被禁锢在对话和文本生成中，是“思想的巨人，行动的矮子”。

* **没有LLM的Agent**

  ，会退化为一个僵硬的、按固定流程行事的自动化脚本，缺乏应对不确定性的智慧和灵活性。

AI Agent区别于传统的SOP（Standard Operation Procedure），workflow在于，传统的方式是基于规则的自动化，有明确的、预设的“如果-那么”规则。而AI Agent是基于目标的自主性，能自主规划、执行、调整。能感知环境变化，动态调整策略。能处理模糊、不确定的任务。其本质在于大模型本身所具备的强大语言理解，推理和泛化能力。

一个完整的Agent除了大脑（LLM）外，还需要：

* **感知模块**

  ： “眼睛和耳朵”，通过API、搜索引擎、文件系统等获取外部信息。

* **工具集**

  ： “手和脚”，可以调用各种函数、API、软件等来影响现实。这就是Function Calling、MCP等技术的用武之地。

* **记忆模块**

  ： “工作日志和经历”，通过向量数据库或记忆流记录过去的历史，用于长期规划和参考。

接下来，让我们一起看看AI Agent是如何用MCP、Function Calling和A2A技术让大模型从“思想的巨人，行动的矮子”变成“万能助手”的。

# **2. MCP协议**

为了更好的理解MCP（Model Context Protocol，模型上下文协议），我们需要先实现一个特殊的Agent，它能自动记录MCP client和server之间的通信内容。通过解析通信内容，无需多言，你就能明了MCP的真谛。

## **2.1 实现一个Agent**

### **1）准备工作**

1. 安装Cline Agent，目前只有VS Code的插件
2. 安装Python，以及uv工具（Python包安装器和解析器），设置repository源，在环境变量中，设置UV\_INDEX\_URL=<https://repo.huaweicloud.com/repository/pypi/simple/>
3. 可用的大模型服务，比如openAI, deepseek，openrouter等

### **2）创建python项目**

```
mkdir mcpserver
 cd mcpserver
# 初始化项目
 uv init
# 创建虚环境
uv sync
# 安装mcp
uv add mcp
# 激活虚拟环境（Windows）
.venv\Scripts\activate

```

### **3）创建MCP Server**

这个MCP Server很简单，模拟一个获取天气信息的工具。其代码如下：

```
from  mcp.server.fastmcp import FastMCP

# 一、创建FastMCP类
mcp = FastMCP("获取天气信息")

# 二、自定义工具（Stdio 模式）
@mcp.tool("获取天气信息")
defget_forecast(city) -> str:
"""
    获取天气信息

     参数:
    city (str): 城市名称

    返回:
    city_forecast : 当前城市的天气信息
    """
# 这里只是简单的返回一个Mock，真实场景会调用天气API
return  city + "明天有大暴雨！"

# 三、初始化MCP Server
if __name__ == '__main__':
# print
print("MCP Server is running...")
    mcp.run(transport='stdio')
print("MCP exist")

```

除了上面这个MCP Server之外，我们还需要一个日志记录工具mcp\_logger.py (<https://github.com/MarkTechStation/VideoCode/>) 用来截获mcp client和mcp server之间的通信（仅限于stdio方式），这个工具可以把通信内容写入到当前目录下的mcp\_io.log日志文件

### **4）在Cline中配置MCP Server**

1）在VSCode的Cline插件上点击“Configure MCP Servers”\
2）在配置文件中写入启动mcp server的信息，这条配置的意思是：通过“python mcp\_logger.py uv run mcp\_weather\_server.py”命令来运行MCP Server\
3）当看到配置界面的get\_forcast工具为绿色时，表示MCP Server已经配置并启动成功了
![1.00](https://github.com/LiuZheng60/001OpenBooks/blob/main/001Book_AI/image/001/002.png?raw=true)

### **5）验证Agent使用工具**

当我们在任务窗口输入“杭州明天的天气如何”，会发现Cline Agent在大模型的帮助下，会主动调用get\_forecast工具，然后成功完成任务。

![1.00](https://github.com/LiuZheng60/001OpenBooks/blob/main/001Book_AI/image/001/003.png?raw=true)

此时查看mcp\_io.log，会发现它记录了完整的MCP Client 和 MCP Server之间的通信内容。

```
== 1. 初始化阶段 ==
MCP Client发送:{"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"CodeMate","version":"25.2.200"}},"jsonrpc":"2.0","id":0}
MCP Server发送:{"jsonrpc":"2.0","id":0,"result":{"protocolVersion":"2025-06-18","capabilities":{"experimental":{},"prompts":{"listChanged":false},"resources":{"subscribe":false,"listChanged":false},"tools":{"listChanged":false}},"serverInfo":{"name":"获取天气信息","version":"1.16.0"}}}

MCP Client发送:{"method":"notifications/initialized","jsonrpc":"2.0"}

== 2. 工具注册阶段 ==
MCP Client发送:{"method":"tools/list","jsonrpc":"2.0","id":1}
MCP Server发送:{"jsonrpc":"2.0","id":1,"result":{"tools":[{"name":"获取天气信息","description":"\n    获取天气信息\n\n     参数:\n    city (str): 城市名称\n\n    返回:\n    city_forecast : 当前城市的天气信息\n    ","inputSchema":{"properties":{"city":{"title":"city","type":"string"}},"required":["city"],"title":"get_forecastArguments","type":"object"},"outputSchema":{"properties":{"result":{"title":"Result","type":"string"}},"required":["result"],"title":"get_forecastOutput","type":"object"}}]}}

MCP Client发送:{"method":"resources/list","jsonrpc":"2.0","id":2}
MCP Server发送:{"jsonrpc":"2.0","id":2,"result":{"resources":[]}}

MCP Client发送:{"method":"resources/templates/list","jsonrpc":"2.0","id":3}
MCP Server发送:{"jsonrpc":"2.0","id":3,"result":{"resourceTemplates":[]}}

== 3. 工具调用阶段 ==
MCP Client发送:{"method":"tools/call","params":{"name":"获取天气信息","arguments":{"city":"杭州"}},"jsonrpc":"2.0","id":4}
MCP Server发送:{"jsonrpc":"2.0","id":4,"result":{"content":[{"type":"text","text":"杭州明天有大暴雨！"}],"structuredContent":{"result":"杭州明天有大暴雨！"},"isError":false}}

```

## **2.2 解析MCP协议**

通过日志我们可以看到，MCP通信内容是基于JSON-RPC协议的，JSON-RPC是我见过最简洁的协议，就像其口号说的，simple is better。

我们把日志转换成时序图，不难发现，整个通信过程分为3个阶段：

1. **初始化阶段**

   ：也就是MCP Client和 MCP Server的握手过程。通常，MCP Client是以内置组件的形式存在于Agent内部。
2. **工具注册阶段**

   ：MCP Client会询问MCP Server，你有哪些可用的tools（工具），resources（资源）、templates（模板）。MCP Server会按照规定的格式返回对应的能力描述。
3. **工具调用阶段**

   ：当大模型需要调用工具时，会告诉Agent，你需要用`{"city":"杭州"}`这个参数，去调用下`get_forecast`这个工具。MCP Client知道这个工具是“获取天气信息”这个MCP Server提供的，调用即可。

![1.00](https://github.com/LiuZheng60/001OpenBooks/blob/main/001Book_AI/image/001/004.png?raw=true)

这就是MCP，其本质就是支持AI Agent用一种通用的方式发现、注册、调用外部的工具，从而协助LLM完成用户任务。wait，大模型怎么知道要在什么时候调用工具，以及用什么方式调用工具呢？

# **3. Function Calling （大模型为什么知道要调用哪个工具）**

当我问`“杭州明天的天气如何”`的时候，大模型为什么知道要调用get\_forecast这个工具呢？实际上，这就是大模型的Function Calling，而这个能力需要我们通过prompt的方式来教大模型。这个能力正是大模型区别传统SOP的关键：**用非结构化的“柔性”，完美克服了传统规则的“刚性”**。

我们可以通过查看Cline的源码来理解Agent是如何“指导”大模型进行Function Calling的。你可以在此查看：Cline的完整system prompt（<https://github.com/cline/cline），整个prompt有600行，将近50K大小，总共13000个字符，会消耗5000个tokens。>**这就是为什么很多人吐槽Cline Agent烧Token的原因**：）。内容比较多，我们截取里面部分内容分析一下，你就明白为什么大模型有能力调用工具了。

## **3.1 身份声明**

首先给模型带个高帽子，你是个软件高手！因为Cline是coding agent，当然要如此声明。可能还有个附加好处，直接激活MoE，帮服务端省点电费。

```
You are Cline, a highly skilled software engineer with extensive knowledge in many programming languages, frameworks, design patterns, and best practices.

```

## **3.2 能力声明**

亮明身份之后，还要声明所具备的能力（capabilities），即告诉大模型：\
1）你能推理任务，并且step-by-step的使用工具，完成任务。这是客户端能变成Agent的关键，即让模型follow ReAct（Reasoning+Acting）模式，处理用户任务。\
2）你能使用工具，且会用XML的格式与我沟通。不同的Agent会选择不同的格式，Cline是用XML，也可以是JSON。

```
TOOL USE

You have access to a set of tools that are executed upon the user's approval. You can use one tool per message, and will receive the result of that tool use in the user's response. You use tools step-by-step to accomplish a given task, with each tool use informed by the result of the previous tool use.

# Tool Use Formatting

Tool use is formatted using XML-style tags. The tool name is enclosed in opening and closing tags, and each parameter is similarly enclosed within its own set of tags. Here's the structure:

<tool_name>
<parameter1_name>value1</parameter1_name>
<parameter2_name>value2</parameter2_name>
...
</tool_name>

```

## **3.3 系统工具能力**

Cline作为coding agent，已经内置了一些工具，主要是和写代码相关的。包括execute\_command（执行命令），read\_file（读文件），write\_to\_file（写文件）等。这里有一个工作目录（working directory）的概念，为了安全性考虑，文件操作只被允许在working directory下面。因此，在实际使用Cline之前，我们需要搞清楚当前的working directory是在哪里。

```
## execute_command
## read_file
## list_files
## write_to_file
Description: Request to write content to a file at the specified path. If the file exists, it will be overwritten with the provided content. If the file doesn't exist, it will be created. This tool will automatically create any directories needed to write the file.
Parameters:
- path: (required) The path of the file to write to (relative to the current working directory ${cwd.toPosix()})
- content: (required) The content to write to the file. ALWAYS provide the COMPLETE intended content of the file, without any truncation or omissions. You MUST include ALL parts of the file, even if they haven't been modified.
Usage:
<write_to_file>
<path>File path here</path>
<content>
Your file content here
</content>
</write_to_file>

```

## **3.4 MCP扩展工具能力**

除了Cline的内建工具之外，当然少不了用户自定义的扩展工具，比如我们案例中的get\_forecast工具，之所以我们的Agent会调用MCP Server的`get_forecast`工具`“获取天气信息”`。正是因为我们在2.1章节中注册了get\_forecast MCP工具。**对于Cline Agent而言，所谓的工具注册，就是在Cline发送给大模型的prompt中加入了如下的功能描述信息**。这些提示词和内建工具的提示词类似，都是在“指导”大模型，教它有这些工具的能力（capabilities），从而实现Function Calling。

```
# Connected MCP Servers
When a server is connected, you can use the server's tools via the `use_mcp_tool` tool, and access the server's resources via the `access_mcp_resource` tool.
## weather (python mcp_logger.py uv run mcp_weather_server.py)
### Available Tools
- get_forecast: 获取天气信息
Args: city: 城市名称
Input Schema:
{
  "type": "object",
  "properties": {
    "city": {
      "title": "city",
      "type": "string"
    }
  },
  "required": [
    "city"
  ],
  "title": "get_forecastArguments"
}

```

## **3.5 完成任务**

当完成所有的subtask，迭代执行完tools，达成任务目标。大模型需要通过`<attempt_completion>`通知Agent任务完成，并在`<result>`中呈现最终任务执行结果，如果需要demonstrate（演示）的话，也可以有演示的CLI，但这是可选的。

```
## attempt_completion
Description: After each tool use, the user will respond with the result of that tool use, i.e. if it succeeded or failed, along with any reasons for failure. Once you've received the results of tool uses and can confirm that the task is complete, use this tool to present the result of your work to the user. Optionally you may provide a CLI command to showcase the result of your work. The user may respond with feedback if they are not satisfied with the result, which you can use to make improvements and try again.
IMPORTANT NOTE: This tool CANNOT be used until you've confirmed from the user that any previous tool uses were successful. Failure to do so will result in code corruption and system failure. Before using this tool, you must ask yourself in <thinking></thinking> tags if you've confirmed from the user that any previous tool uses were successful. If not, then DO NOT use this tool.

Parameters:
- result: (required) The result of the task. Formulate this result in a way that is final and does not require further input from the user. Don't end your result with questions or offers for further assistance.
- command: (optional) A CLI command to execute to show a live demo of the result to the user. For example, use \`open index.html\` to display a created html website, or \`open localhost:3000\` to display a locally running development server. But DO NOT use commands like \`echo\` or \`cat\` that merely print text. This command should be valid for the current operating system. Ensure the command is properly formatted and does not contain any harmful instructions.

Usage:
<attempt_completion>
<result>
Your final result description here
</result>
<command>Command to demonstrate result (optional)</command>
</attempt_completion>

```

# **4. Context Engieering**

通过上文对Cline System Prompt的解读，你会发现，**Agent的本质就是一个精心设计的Prompt**。因为大模型本质上是“上下文学习者”：它们没有长期的记忆，每次交互都是独立的。你提供的上下文就是它此次交互的全部世界。

![1.00](https://github.com/LiuZheng60/001OpenBooks/blob/main/001Book_AI/image/001/005.png?raw=true)

因此，要想实现完整的Agent功能，在传递给LLM的信息中，除了System prompt，user message之外，还要包括Docs（领域知识，工作对象等），工作记忆（Message history）等。**把这些内容都汇总起来就是LLM需要的Context（上下文），而这个汇总（精炼、压缩）的工作就叫Context Engineering (<https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents>)**。\
这也是Context Engineering和Prompt Engineering的主要区别。

回过头来，你应该就能理解我在开篇说的，MCP是和模型无关的协议。**MCP只是一个为了Model的Context而服务的Protocol**。它既不和Model交互，也和Model无关。它只是给Context Engineering服务的一环。

# **5. A2A协议**

Agent的本质是Context，像Cline这样的Agent，通过MCP扩展的工具能力使用说明，都是放在prompt里，工具不多还好，如果很多，可能会撑爆Context Window，即使不会撑爆，过长的Context，也会引发Context Rot (<https://research.trychroma.com/context-rot>) 问题。这和我们人类的认知类似，信息越多，越难focus，越难抓住重点。

关于如何提供有效的，长度适中的Context，是Contex Engineering的关键。在Context Engineering一文中有详细阐述。常用的技术有Compaction（压缩），Structured Note-taking（结构化笔记），以及Multi-Agent architectures（多Agent架构）。所谓的Multi-Agent，就是从Agent层面做分而治之，从而控制单个Agent的Context长度。

而多agent之间的协作，就涉及到A2A协议，即Agent to Agent协议。

![1.00](https://github.com/LiuZheng60/001OpenBooks/blob/main/001Book_AI/image/001/006.png?raw=true)

A2A是google开源的开放协议。其包含如下核心概念：

**概念**

**描述**

Agent Card（卡片）

位于 `/.well-known/agent.json`，描述能力、技能、端点 URL 和认证要求，用于发现

A2A Server（服务器）

实现协议方法，管理任务执行

A2A Client（客户端）

发送请求如 `tasks/send` 或 `tasks/sendSubscribe`，消费 A2A 服务

Task(任务)

核心工作单位，有唯一 ID，状态包括 `submitted`、`working` 等

Message(消息)

通信单位，角色为 `user` 或 `agent`，包含 Parts

Parts(部分)

内容单位，包括 `TextPart`、`FilePart`、`DataPart`

Artifacts(工件)

任务输出，包含 Parts

流式传输

使用 SSE 事件更新长期任务状态

推送通知

通过 webhook 发送更新

和MCP一样，A2A采用的也是JSON-RPC协议，其工作机制也很类似，主要包括Agent的发现、注册和使用，通信细节不再赘述，大致流程如下：

1. **发现**

   ：客户端从 /.well-known/agent.json 获取 Agent Card，了解智能体的能力。
2. **启动**

   ：客户端发送任务请求：
3. **处理**

   ：服务器处理任务，可能涉及流式更新或直接返回结果。
4. **交互（可选）**

   ：若任务状态为 input-required，客户端可发送更多消息，使用相同 Task ID 提供输入。
5. **完成**

   ：任务达到终端状态（如 completed、failed 或 canceled）。

# **总结**

单纯的大模型，只能对话和生成文本，是“思想的巨人，行动的矮子”。配上Agent的大模型，能感知环境、使用工具、执行任务，成为“万能助手”。AI Agent的核心在于Context Engineering，背后需要依赖三大关键技术：**MCP、Function Calling 和 A2A**。这三项技术，并不是有你无我的排斥关系，而是可以通力协作的互补关系。大模型通过 Prompt 学习工具使用，实现非结构化任务处理，克服传统规则的“刚性”，使得AGI（Artificial General Intelligence，通用人工智能）成为可能。
