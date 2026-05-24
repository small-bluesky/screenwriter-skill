# screenwriter-skill

将用户的一句话创意扩展为标准格式的电影剧本第一幕的 LLM 技能包。纯提示词驱动，无需任何外部工具或代码，即可在主流 Agent 平台上使用。

## 快速开始

### 方式一：直接复制提示词（最快）

打开 [skill.yaml](./skill.yaml)，找到 `system_prompt` 字段，将其中的完整内容（从"你是一位拥有15年经验的专业影视编剧……"到"……一次性输出完整的创作结果"）复制到任意 LLM 平台的 System Prompt 设置中即可使用。输入时只需将一句话创意作为用户消息发送。

### 方式二：在 Dify 中导入

1. 进入 Dify 控制台，点击"创建应用" → "Chatflow"（或"工作流"）
2. 在工作流画布中添加一个 **LLM 节点**
3. 在 System Prompt 输入框中粘贴 [skill.yaml](./skill.yaml) 中 `system_prompt` 的完整内容
4. 设置模型参数：
   - Temperature: `0.8`
   - Max Tokens: `4000`
5. 添加一个开始节点，定义输入变量名为 `user_idea`
6. 在 LLM 节点的用户消息中引用 `{{user_idea}}`
7. 发布应用即可使用

### 方式三：在 Coze 中导入

1. 进入 Coze 控制台，创建一个新的 Bot
2. 在"人设与回复逻辑"中粘贴 [skill.yaml](./skill.yaml) 中 `system_prompt` 的完整内容
3. 设置模型温度等参数（Temperature: 0.8）
4. 使用时将一句话创意直接发送给 Bot

### 方式四：在 LangChain 中使用

```python
from langchain.prompts import ChatPromptTemplate
from langchain.chat_models import ChatOpenAI

with open("skill.yaml", "r", encoding="utf-8") as f:
    import yaml
    skill = yaml.safe_load(f)

prompt = ChatPromptTemplate.from_messages([
    ("system", skill["system_prompt"]),
    ("human", "{user_idea}")
])

llm = ChatOpenAI(model="gpt-4o", temperature=0.8, max_tokens=4000)
chain = prompt | llm

result = chain.invoke({"user_idea": "你的故事创意"})
print(result.content)
```

## 输入示例

```
一个落魄钢琴手在地下停车场听见一段神秘旋律，追踪后发现弹奏者是30年前的自己。
```

## 输出示例

完整输出示例请查看 [examples/output_01.md](./examples/output_01.md)，包含以下结构化段落：

- **【故事前提】** — 核心冲突、主角动机、结局方向和类型
- **【人物】** — 至少2个核心人物的详细小传
- **【分场大纲】** — 三幕结构、至少12个情节点
- **【剧本】** — 前3场的标准好莱坞格式完整剧本
- **【编剧注】** — 节奏、对白风格的自我点评及备选走向

## 高级用法

### 调整运行时参数

根据具体模型的表现，可以调整 [skill.yaml](./skill.yaml) 中 `runtime` 节点的参数：

- **temperature**: 创意型场景建议保持 `0.7-0.9`。降低此值可提高输出一致性。
- **max_tokens**: 若模型支持更大上下文，可提升至 `8000` 以生成更长剧本。
- **stream**: 默认关闭。若需流式展示生成过程，可开启。

### 与 RAG 结合

本技能包预留了 `rag_support` 扩展接口。在 Dify 或 LangChain 中，可在 LLM 调用前增加一个知识库检索节点，将检索到的示例剧本或风格参考文本注入到 `user_idea` 变量之前，实现风格模仿或特定类型强化。

### 拆分为多节点工作流

虽然本技能要求一次性输出完整结果，但你可以在 Dify 中将工作流拆分为多步：

1. 节点一：仅输出【故事前提】+【人物】
2. 节点二：基于节点一的输出，生成【分场大纲】
3. 节点三：基于分场大纲，生成【剧本】+【编剧注】

这样可以更好地控制每个节点的 Token 消耗和温度设置。

## 常见问题

### 为什么要求一次性输出全部内容？

一次性输出可以最大程度保证人物的一致性、情节的连贯性和风格的统一。分步调用容易出现前后矛盾的情况。如果担心 Token 限制，可以参考上方"拆分为多节点工作流"的方案。

### 如何确保模型不截断输出？

1. 将 `max_tokens` 设置得足够大（推荐 ≥ 4000）
2. 使用支持长输出的模型（如 GPT-4o、Claude 3.5 Sonnet、Gemini 2.0 Flash）
3. 如果输出被截断，可以回复"继续"让模型补充剩余内容

### 支持哪些模型？

本技能为纯提示词驱动，不绑定任何特定模型。推荐使用 GPT-4o 或 Claude 3.5 Sonnet 以获得最佳效果，但 GPT-4o-mini、DeepSeek-V3、Qwen2.5 等模型也可正常使用。

### 输出格式不稳定怎么办？

段落标题（如【故事前提】等）是由提示词中的硬约束保证的。如果某个模型偏离了格式，可以适当降低 temperature，或在提示词中增加一句"输出时严格按照【】格式标注每个段落标题"。

## 项目结构

```
screenwriter-skill/
├── README.md
├── skill.yaml                # 技能核心定义文件
├── examples/
│   ├── input_01.txt          # 示例输入
│   └── output_01.md          # 示例输出
├── assets/
│   └── workflow-diagram.png  # 工作流示意图（可选）
├── LICENSE                   # MIT 许可证
└── .gitignore
```

## 贡献指南

欢迎通过 Issue 或 PR 参与贡献！以下方向尤其欢迎：

- 提示词优化，提升特定类型（悬疑、爱情、科幻等）的创作质量
- 提供更多示例输入输出，丰富范例库
- 适配更多 Agent 平台的导入方案
- 翻译为其他语言版本

请确保所有提交的文件使用 **UTF-8 without BOM** 编码，换行符为 **LF**。

## 许可证

本项目基于 MIT 许可证开源，详见 [LICENSE](./LICENSE) 文件。你可以自由使用、修改和分发本技能包。