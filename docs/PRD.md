# DeepAgentX - 项目需求文档 (PRD)

## 1. 项目概述

### 1.1 项目名称
DeepAgentX - AI Coding 智能助手框架

### 1.2 项目愿景
构建一个轻量级、开源的 AI Coding 工具，类似 OpenClaude 和 Claude Code，支持多轮对话、可扩展的技能引擎、工具调用和可视化执行链路追踪。

### 1.3 技术选型
- **编程语言**: Python 3.10+
- **Web 界面**: Streamlit (或 Gradio 作为备选)
- **AI 框架**: 自研核心 + LangChain Deep Agents (可选集成)
- **模型接口**: OpenAI-Compatible API 格式

---

## 2. 功能需求

### 2.1 核心功能模块

```
┌─────────────────────────────────────────────────────────────────────┐
│                         DeepAgentX 架构                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                     │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────────────┐    │
│  │   Chat UI    │   │ Agent Core   │   │   Skills Engine      │    │
│  │  (Streamlit) │ ←→│              │ ←→│                      │    │
│  │              │   │ • 对话管理    │   │ • 技能发现           │    │
│  │ • 消息展示    │   │ • 上下文管理  │   │ • 渐进式加载         │    │
│  │ • 输入处理    │   │ • 状态机      │   │ • 脚本执行           │    │
│  │ • 链路可视化  │   │ • 工具编排    │   │ • 元数据索引         │    │
│  └──────────────┘   └──────────────┘   └──────────────────────┘    │
│         ↑                  ↑                        ↑              │
│         └──────────────────┴────────────────────────┘              │
│                           │                                        │
│              ┌────────────┴────────────┐                          │
│              │     Tool Registry       │                          │
│              │  • File System Tools    │                          │
│              │  • Custom Tools         │                          │
│              │  • Skills Scripts       │                          │
│              └─────────────────────────┘                          │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

### 2.2 详细功能需求

#### 2.2.1 多轮对话系统 (FR-001)
| 需求编号 | 功能描述 | 优先级 |
|---------|---------|-------|
| FR-001-1 | 支持 OpenAI-Compatible 格式的模型调用 | P0 |
| FR-001-2 | 维护多轮对话上下文，支持会话历史 | P0 |
| FR-001-3 | 支持对话会话的创建、切换、持久化 | P1 |
| FR-001-4 | 支持流式响应 (Streaming) | P1 |
| FR-001-5 | 支持配置多个模型提供商，动态切换 | P2 |

#### 2.2.2 技能引擎 (Skills Engine) (FR-002)
| 需求编号 | 功能描述 | 优先级 |
|---------|---------|-------|
| FR-002-1 | 技能渐进式披露机制 | P0 |
| FR-002-2 | 技能元数据管理 (name, description, triggers) | P0 |
| FR-002-3 | 自动技能发现与加载 | P0 |
| FR-002-4 | 技能脚本执行环境 | P1 |
| FR-002-5 | 技能依赖管理和版本控制 | P2 |

**渐进式披露机制详细设计:**
```
阶段1: 初始状态
   └── 仅加载 skills/skills.json (元数据索引)
   └── 数据结构: [{"name": "pdf", "description": "...", "triggers": [...]}]

阶段2: 触发状态 (用户请求匹配 triggers)
   └── 加载 skills/pdf/SKILL.md (完整技能定义)
   └── 解析技能的工作流程、工具映射

阶段3: 深度状态 (执行过程中)
   └── 按需加载 skills/pdf/scripts/*.py
   └── 加载 skills/pdf/examples/ 示例文件
```

#### 2.2.3 工具系统 (Tool System) (FR-003)
| 需求编号 | 功能描述 | 优先级 |
|---------|---------|-------|
| FR-003-1 | 文件系统工具 (read, write, edit, ls, grep, glob) | P0 |
| FR-003-2 | 支持技能捆绑的可执行脚本 | P0 |
| FR-003-3 | 工具调用链式编排 | P1 |
| FR-003-4 | 工具执行结果的标准化返回 | P1 |
| FR-003-5 | 支持自定义工具注册 | P2 |

**文件系统工具规范:**
| 工具名 | 功能 | 参数 | 返回值 |
|-------|------|------|-------|
| read | 读取文件内容 | file_path, offset, limit | content, metadata |
| write | 写入文件内容 | file_path, content | success, file_path |
| edit | 编辑文件内容 | file_path, old_string, new_string | success, diff |
| ls | 列出目录内容 | path | files[], directories[] |
| grep | 内容搜索 | pattern, path, glob | matches[] |
| glob | 文件模式匹配 | pattern | file_paths[] |

#### 2.2.4 执行链路追踪 (FR-004)
| 需求编号 | 功能描述 | 优先级 |
|---------|---------|-------|
| FR-004-1 | 记录完整的执行链路 | P0 |
| FR-004-2 | 可视化展示执行步骤 | P0 |
| FR-004-3 | 支持步骤展开/折叠查看详情 | P1 |
| FR-004-4 | 支持执行链路导出/分享 | P2 |

**执行链路数据结构:**
```json
{
  "execution_id": "uuid",
  "timestamp": "2024-01-01T00:00:00Z",
  "steps": [
    {
      "step_id": 1,
      "type": "skill_match",
      "name": "识别技能",
      "input": "用户请求",
      "output": "匹配的技能名称",
      "duration_ms": 100,
      "status": "success"
    },
    {
      "step_id": 2,
      "type": "tool_call",
      "name": "读取文件",
      "input": {"file_path": "..."},
      "output": {"content": "..."},
      "duration_ms": 50,
      "status": "success"
    }
  ]
}
```

#### 2.2.5 Web 聊天界面 (FR-005)
| 需求编号 | 功能描述 | 优先级 |
|---------|---------|-------|
| FR-005-1 | 聊天消息展示 (支持 Markdown) | P0 |
| FR-005-2 | 用户输入框，支持多行输入 | P0 |
| FR-005-3 | 执行链路可视化组件 | P0 |
| FR-005-4 | 会话管理侧边栏 | P1 |
| FR-005-5 | 深色/浅色主题切换 | P2 |
| FR-005-6 | 代码块语法高亮 | P2 |

---

## 3. 架构设计

### 3.1 项目目录结构

```
DeepAgentX/
├── README.md
├── requirements.txt
├── setup.py
├── config.yaml                 # 配置文件
├── main.py                     # 入口文件
│
├── deepagentx/                 # 核心包
│   ├── __init__.py
│   ├── core/                   # 核心引擎
│   │   ├── __init__.py
│   │   ├── agent.py           # Agent 主类
│   │   ├── session.py         # 会话管理
│   │   └── config.py          # 配置管理
│   │
│   ├── llm/                    # LLM 接口层
│   │   ├── __init__.py
│   │   ├── base.py            # 抽象基类
│   │   ├── openai_compatible.py # OpenAI-Compatible 实现
│   │   └── messages.py        # 消息格式处理
│   │
│   ├── skills/                 # 技能引擎
│   │   ├── __init__.py
│   │   ├── engine.py          # 技能引擎核心
│   │   ├── loader.py          # 技能加载器
│   │   ├── matcher.py         # 技能匹配器
│   │   ├── executor.py        # 技能执行器
│   │   └── models.py          # 数据模型
│   │
│   ├── tools/                  # 工具系统
│   │   ├── __init__.py
│   │   ├── registry.py        # 工具注册表
│   │   ├── base.py            # 工具基类
│   │   ├── filesystem.py      # 文件系统工具
│   │   └── executor.py        # 工具执行器
│   │
│   ├── chain/                  # 执行链路
│   │   ├── __init__.py
│   │   ├── tracer.py          # 链路追踪器
│   │   ├── visualizer.py      # 可视化组件
│   │   └── models.py          # 链路数据模型
│   │
│   └── ui/                     # Web 界面
│       ├── __init__.py
│       ├── streamlit_app.py   # Streamlit 应用
│       ├── components.py      # UI 组件
│       └── chat.py            # 聊天界面
│
├── skills/                     # 技能目录 (示例)
│   ├── skills.json            # 技能元数据索引
│   ├── pdf/                   # PDF 处理技能
│   │   ├── SKILL.md           # 技能定义
│   │   ├── scripts/           # 可执行脚本
│   │   │   └── extract_form_fields.py
│   │   └── examples/          # 示例文件
│   └── code_review/           # 代码审查技能
│       ├── SKILL.md
│       └── scripts/
│
├── tests/                      # 测试目录
│   ├── __init__.py
│   ├── test_agent.py
│   ├── test_skills.py
│   └── test_tools.py
│
└── docs/                       # 文档目录
    ├── PRD.md                 # 本文件
    ├── ARCHITECTURE.md        # 架构文档
    ├── SKILL_SPEC.md          # 技能规范
    └── API.md                 # API 文档
```

### 3.2 核心类设计

#### 3.2.1 Agent 核心类
```python
class DeepAgent:
    """DeepAgentX 核心 Agent 类"""

    def __init__(
        self,
        llm_client: LLMClient,
        skills_engine: SkillsEngine,
        tool_registry: ToolRegistry,
        chain_tracer: ChainTracer
    ):
        self.llm = llm_client
        self.skills = skills_engine
        self.tools = tool_registry
        self.tracer = chain_tracer

    async def chat(
        self,
        message: str,
        session_id: Optional[str] = None
    ) -> ChatResponse:
        """处理用户消息，返回响应"""
        # 1. 开始追踪
        with self.tracer.start_trace() as trace:
            # 2. 技能匹配
            matched_skill = await self.skills.match(message)

            # 3. 构建提示
            prompt = await self._build_prompt(message, matched_skill)

            # 4. LLM 调用
            response = await self.llm.chat(prompt, session_id)

            # 5. 处理工具调用
            if response.has_tool_calls:
                result = await self._execute_tool_calls(response.tool_calls)
                response = await self.llm.chat_with_result(result)

            return response
```

#### 3.2.2 技能引擎类
```python
class SkillsEngine:
    """技能引擎 - 渐进式加载与管理"""

    def __init__(self, skills_dir: str = "skills"):
        self.skills_dir = skills_dir
        self.metadata_index: Dict[str, SkillMetadata] = {}
        self.loaded_skills: Dict[str, Skill] = {}

    async def load_metadata_index(self):
        """阶段1: 加载技能元数据索引"""
        index_path = os.path.join(self.skills_dir, "skills.json")
        with open(index_path) as f:
            data = json.load(f)
            for item in data["skills"]:
                self.metadata_index[item["name"]] = SkillMetadata(**item)

    async def match(self, user_request: str) -> Optional[SkillMatch]:
        """根据用户请求匹配技能"""
        for name, metadata in self.metadata_index.items():
            if self._triggers_match(user_request, metadata.triggers):
                skill = await self._load_skill(name)
                return SkillMatch(skill=skill, confidence=0.9)
        return None

    async def _load_skill(self, name: str) -> Skill:
        """阶段2: 按需加载技能完整定义"""
        if name in self.loaded_skills:
            return self.loaded_skills[name]

        skill_path = os.path.join(self.skills_dir, name, "SKILL.md")
        skill = await SkillParser.parse(skill_path)
        self.loaded_skills[name] = skill
        return skill
```

#### 3.2.3 工具注册表
```python
class ToolRegistry:
    """工具注册表 - 管理所有可用工具"""

    def __init__(self):
        self._tools: Dict[str, Tool] = {}
        self._schemas: Dict[str, ToolSchema] = {}

    def register(self, tool: Tool) -> None:
        """注册工具"""
        self._tools[tool.name] = tool
        self._schemas[tool.name] = tool.get_schema()

    def register_filesystem_tools(self) -> None:
        """注册文件系统工具"""
        self.register(ReadFileTool())
        self.register(WriteFileTool())
        self.register(EditFileTool())
        self.register(ListDirTool())
        self.register(GrepTool())
        self.register(GlobTool())

    async def execute(
        self,
        tool_name: str,
        arguments: Dict[str, Any]
    ) -> ToolResult:
        """执行工具调用"""
        if tool_name not in self._tools:
            raise ToolNotFoundError(tool_name)

        tool = self._tools[tool_name]
        return await tool.execute(**arguments)
```

### 3.3 技能规范 (SKILL.md)

技能定义文件采用 Markdown 格式，示例:

```markdown
# Skill: PDF Processing

## Metadata
- **Name**: pdf
- **Version**: 1.0.0
- **Author**: DeepAgentX
- **Triggers**: ["pdf", "pdf文件", "提取pdf", "分析pdf"]

## Description
处理 PDF 文件，支持文本提取、表单字段提取、元数据读取等功能。

## Tools
- read_file
- write_file
- pdf.scripts.extract_form_fields

## Workflow
1. 使用 `read_file` 读取 PDF 文件路径
2. 根据用户需求选择合适的处理脚本
3. 执行脚本并获取结果
4. 格式化输出结果

## Scripts
| 脚本名 | 功能 | 参数 |
|-------|------|------|
| extract_form_fields.py | 提取表单字段 | pdf_path: str |
| extract_text.py | 提取文本内容 | pdf_path: str, pages: Optional[List[int]] |

## Examples
### 提取表单字段
用户: "提取这个 PDF 的表单字段"
助手:
  1. 读取 PDF 文件
  2. 调用 extract_form_fields.py
  3. 返回 JSON 格式的字段数据
```

---

## 4. 数据模型

### 4.1 核心模型

```python
# 消息模型
class Message(BaseModel):
    role: Literal["system", "user", "assistant", "tool"]
    content: str
    tool_calls: Optional[List[ToolCall]] = None
    tool_call_id: Optional[str] = None

# 会话模型
class Session(BaseModel):
    id: str
    messages: List[Message]
    metadata: Dict[str, Any] = {}
    created_at: datetime
    updated_at: datetime

# 技能模型
class SkillMetadata(BaseModel):
    name: str
    description: str
    version: str
    triggers: List[str]
    author: Optional[str] = None

class Skill(BaseModel):
    metadata: SkillMetadata
    workflow: str
    tools: List[str]
    examples: List[Example]
    scripts_dir: Optional[str] = None

# 工具模型
class ToolSchema(BaseModel):
    name: str
    description: str
    parameters: Dict[str, Any]  # JSON Schema

class ToolCall(BaseModel):
    id: str
    name: str
    arguments: Dict[str, Any]

class ToolResult(BaseModel):
    tool_call_id: str
    content: Any
    status: Literal["success", "error"]
    error_message: Optional[str] = None
```

---

## 5. 接口定义

### 5.1 LLM 客户端接口

```python
class LLMClient(ABC):
    @abstractmethod
    async def chat(
        self,
        messages: List[Message],
        tools: Optional[List[ToolSchema]] = None,
        stream: bool = False
    ) -> LLMResponse:
        pass

class OpenAICompatibleClient(LLMClient):
    def __init__(
        self,
        api_key: str,
        base_url: str,
        model: str = "gpt-4"
    ):
        self.client = AsyncOpenAI(api_key=api_key, base_url=base_url)
        self.model = model
```

### 5.2 工具接口

```python
class Tool(ABC):
    name: str
    description: str

    @abstractmethod
    def get_schema(self) -> ToolSchema:
        pass

    @abstractmethod
    async def execute(self, **kwargs) -> ToolResult:
        pass
```

---

## 6. 配置文件

### 6.1 主配置文件 (config.yaml)

```yaml
# DeepAgentX 配置文件

# LLM 配置
llm:
  provider: openai-compatible
  api_key: ${OPENAI_API_KEY}
  base_url: https://api.openai.com/v1
  model: gpt-4
  temperature: 0.7
  max_tokens: 4096
  # 可选的其他模型配置
  models:
    - name: gpt-4
      base_url: https://api.openai.com/v1
    - name: claude-3-opus
      base_url: https://api.anthropic.com/v1

# 技能配置
skills:
  directory: ./skills
  auto_reload: true
  cache_enabled: true

# 工具配置
tools:
  filesystem:
    allowed_paths: ["./workspace", "./skills"]
    allow_write: true
  max_execution_time: 30

# UI 配置
ui:
  port: 8501
  host: 0.0.0.0
  theme: light
  title: "DeepAgentX"

# 日志配置
logging:
  level: INFO
  file: ./logs/deepagentx.log
```

---

## 7. 依赖清单

### 7.1 requirements.txt

```
# 核心依赖
pydantic>=2.0.0
pyyaml>=6.0

# LLM 支持
openai>=1.0.0
httpx>=0.25.0

# Web UI
streamlit>=1.28.0
streamlit-chat>=0.1.0

# 工具依赖 (按需)
pypdf>=3.0.0           # PDF 处理
python-markdown>=3.5   # Markdown 解析

# 开发依赖
pytest>=7.0.0
pytest-asyncio>=0.21.0
black>=23.0.0
```

---

## 8. 开发计划

### Phase 1: 基础架构 (Week 1)
- [ ] 项目骨架搭建
- [ ] 核心类定义 (Agent, Session, Message)
- [ ] LLM 客户端实现 (OpenAI-Compatible)
- [ ] 基础配置文件系统

### Phase 2: 技能引擎 (Week 2)
- [ ] 技能元数据索引系统
- [ ] 渐进式加载机制
- [ ] SKILL.md 解析器
- [ ] 技能匹配器
- [ ] 脚本执行环境

### Phase 3: 工具系统 (Week 3)
- [ ] 工具注册表
- [ ] 文件系统工具实现
- [ ] 工具调用链编排
- [ ] 工具执行异常处理

### Phase 4: Web 界面 (Week 4)
- [ ] Streamlit 基础框架
- [ ] 聊天界面实现
- [ ] 执行链路可视化
- [ ] 会话管理

### Phase 5: 集成与优化 (Week 5)
- [ ] 全流程集成测试
- [ ] 性能优化
- [ ] 示例技能开发
- [ ] 文档完善

---

## 9. 风险评估

| 风险 | 可能性 | 影响 | 缓解措施 |
|------|--------|------|---------|
| LLM 响应延迟/不稳定 | 中 | 高 | 实现重试机制、超时控制 |
| 技能脚本安全风险 | 中 | 高 | 沙箱执行、权限控制 |
| 上下文长度超限 | 高 | 中 | 智能截断、摘要机制 |
| 工具执行错误 | 中 | 中 | 完善的错误处理和回滚 |

---

## 10. 附录

### 10.1 术语表
- **Skill**: 预定义的知识和工作流程包
- **Tool**: 具体可执行的功能单元
- **Chain**: 执行链路，记录完整执行过程
- **Trace**: 追踪数据，用于调试和可视化

### 10.2 参考文档
- [OpenAI API 文档](https://platform.openai.com/docs)
- [LangChain Deep Agents](https://docs.langchain.com/oss/python/deepagents/overview)
- [Streamlit 文档](https://docs.streamlit.io/)

---

文档版本: 1.0.0
创建日期: 2026-04-10
作者: DeepAgentX Team
