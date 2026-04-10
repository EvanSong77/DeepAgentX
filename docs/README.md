    我发现现在很多skills的项目都是在OpenClaw 或者claude code等工具上进行执行和使用的，这些工具要不就很笨重或者不开源，我想用AI Coding去实现一个类似的用python写的工具，能够：

1. 适配openai-compatible格式的模型，进行多轮对话

2.可以像这些工具一样能够自动读取skills，执行skills的引擎，
Skills 采用渐进式披露（Progressive Disclosure） 机制：
初始状态：仅加载技能元数据（name + description）
触发状态：用户请求匹配时，加载完整 SKILL.md
深度状态：按需加载附加参考文件

Skills 的一个强大特性是可以捆绑可执行脚本。例如，PDF 技能可以包含一个 Python 脚本来提取表单字段：

# scripts/extract_form_fields.py
import pypdf

def extract_form_fields(pdf_path):
    """提取 PDF 表单字段，无需将 PDF 加载到上下文"""
    reader = pypdf.PdfReader(pdf_path)
    fields = reader.get_form_text_fields()
    return fields
这种设计的优势：

确定性可靠性：代码执行提供了只有代码才能提供的可靠性
上下文分离：脚本在上下文之外执行，不占用 token
复杂操作：可以处理 Claude 无法直接处理的复杂文件操作

与 Deep Agents 文件系统的协同
Skills 与 Deep Agents 的文件系统能力形成完美互补：

┌─────────────────────────────────────────────────────────────┐
│                     Agents 系统                        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│  │   Skills    │    │ 文件系统工具 │    │  子智能体   │     │
│  │             │    │             │    │             │     │
│  │ • 专业知识  │ ←→ │ • ls       │ ←→ │ • task     │     │
│  │ • 工作流程  │    │ • read     │    │ • 上下文隔离│     │
│  │ • 最佳实践  │    │ • write    │    │ • 并行执行  │     │
│  └─────────────┘    │ • edit     │    └─────────────┘     │
│                     └─────────────┘                        │
│                                                             │
│  Skills 提供「知识」，文件系统提供「执行能力」               │
│  两者结合实现：知识驱动的智能自动化                          │
└─────────────────────────────────────────────────────────────┘
实际工作流示例
用户请求：”帮我分析这个 PDF 并提取关键数据”
技能匹配：系统识别 pdf-processing 技能
技能加载：读取 SKILL.md 获取处理指南
工具执行：使用文件系统工具执行实际操作
结果输出：按技能定义的格式返回结果

3.Agent也能支持tool call

3.可以将整个执行链路的展示出来，这样方便用户追踪

4.目前langchain的DeepAgents（https://docs.langchain.com/oss/python/deepagents/overview） 能够支持，你需要衡量是否要基于这个框架来搭建

5.我希望有个页面来进行chat，可以使用Streamlit等来完成页面的开发，这个选型看你

这是我的需求和想法，docs中的文档不足以帮我完成这个项目，请你帮我修改好，使其能够很好的完成整个项目的Coding