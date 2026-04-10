# AI Coding Agent Framework - Web UI 和 Streamlit 集成指南

## 1. Streamlit 应用架构

### 1.1 为什么选择 Streamlit？

| 特性 | Streamlit | Gradio | Flask | FastAPI |
|------|-----------|--------|-------|---------|
| 开发速度 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| 轻量级 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ |
| 学习曲线 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |
| 组件丰富 | ⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐⭐⭐ |
| 部署简单 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐ | ⭐⭐ |

选择 Streamlit 的原因：
1. **极速开发**：无需 HTML/CSS/JS，直接 Python
2. **实时反应**：自动热重载
3. **内置组件**：chat、plots、tables 等
4. **会话管理**：自动管理用户会话
5. **部署简单**：Streamlit Cloud 一键部署

---

## 2. Streamlit 项目结构

### 2.1 目录设置

```
src/openskill/ui/
├── app.py                    # 主应用入口
├── pages/                    # 多页面应用
│   ├── 1_💬_Chat.py          # 聊天页面
│   ├── 2_🧠_Skills.py        # 技能管理
│   ├── 3_⚡_Execution.py     # 执行追踪
│   └── 4_⚙️_Settings.py      # 设置
├── components/               # 可复用组件
│   ├── __init__.py
│   ├── chat_interface.py     # 聊天界面
│   ├── execution_tree.py     # 执行树
│   ├── skill_card.py         # 技能卡片
│   ├── metrics_display.py    # 指标展示
│   └── code_viewer.py        # 代码查看器
├── utils/                    # UI 工具函数
│   ├── __init__.py
│   ├── session_state.py      # 会话状态管理
│   ├── formatters.py         # 格式化函数
│   ├── colors.py             # 颜色主题
│   └── icons.py              # 图标
└── assets/                   # 静态资源
    ├── logo.png
    ├── styles.css
    └── config.yaml
```

### 2.2 Streamlit 配置

```yaml
# .streamlit/config.toml

[theme]
primaryColor = "#FF6B6B"
backgroundColor = "#FFFFFF"
secondaryBackgroundColor = "#F0F2F6"
textColor = "#262730"
font = "sans serif"

[client]
showErrorDetails = true
toolbarMode = "developer"
logLevel = "info"

[logger]
level = "debug"

[server]
port = 8501
headless = false
runOnSave = true
maxUploadSize = 200  # MB
```

---

## 3. 主应用（app.py）实现

### 3.1 基础框架

```python
# src/openskill/ui/app.py

import streamlit as st
from pathlib import Path
import sys

# 添加项目路径
sys.path.insert(0, str(Path(__file__).parent.parent.parent))

from openskill.core import CoreServices
from openskill.utils.logger import setup_logging
from .utils.session_state import initialize_session_state
from .utils.colors import THEME_COLORS

# ============================================================================
# 配置和初始化
# ============================================================================

def setup_page_config():
    """配置页面"""
    st.set_page_config(
        page_title="OpenSkill Agent",
        page_icon="🤖",
        layout="wide",
        initial_sidebar_state="expanded",
    )

def setup_logging_and_services():
    """设置日志和核心服务"""
    setup_logging("INFO")
    
    # 初始化核心服务（缓存）
    if 'core_services' not in st.session_state:
        st.session_state.core_services = CoreServices()
        st.session_state.core_services.initialize()
    
    return st.session_state.core_services

# ============================================================================
# 侧边栏导航和配置
# ============================================================================

def setup_sidebar():
    """设置侧边栏"""
    with st.sidebar:
        st.image("assets/logo.png", width=200)
        st.title("OpenSkill Agent")
        st.caption("🚀 Lightweight AI Agent Framework")
        
        st.divider()
        
        # 会话信息
        st.subheader("📊 会话信息")
        if 'session_id' in st.session_state:
            st.info(f"Session ID: {st.session_state.session_id[:8]}...")
        
        # 统计信息
        col1, col2 = st.columns(2)
        with col1:
            st.metric("消息数", st.session_state.get('total_messages', 0))
        with col2:
            st.metric("工具调用", st.session_state.get('total_tool_calls', 0))
        
        st.divider()
        
        # 模型选择
        st.subheader("🤖 模型配置")
        core_services = st.session_state.core_services
        
        config = core_services.config_manager.get_config()
        llm_config = config.get('llm', {})
        
        model = st.selectbox(
            "选择模型",
            options=llm_config.get('available_models', [
                'gpt-4',
                'gpt-3.5-turbo',
            ]),
            index=0
        )
        st.session_state.selected_model = model
        
        temperature = st.slider(
            "Temperature",
            min_value=0.0,
            max_value=2.0,
            value=llm_config.get('temperature', 0.7),
            step=0.1
        )
        st.session_state.temperature = temperature
        
        st.divider()
        
        # 技能统计
        st.subheader("📚 技能")
        skills = core_services.skill_manager.get_available_skills()
        st.metric("可用技能", len(skills))
        
        if st.button("🔄 刷新技能"):
            with st.spinner("正在刷新技能..."):
                core_services.skill_manager.reload_skills()
            st.success("技能已刷新！")
        
        st.divider()
        
        # 帮助和关于
        st.subheader("❓ 帮助")
        if st.button("📖 使用文档"):
            st.switch_page("pages/1_💬_Chat.py")
        
        if st.button("🐛 反馈问题"):
            st.info("请访问 GitHub Issues")

# ============================================================================
# 主内容区域
# ============================================================================

def render_main_content():
    """渲染主内容"""
    st.header("🤖 OpenSkill Agent Framework")
    
    # 欢迎信息
    col1, col2, col3 = st.columns(3)
    with col1:
        st.metric("💬 聊天", "实时对话")
    with col2:
        st.metric("📚 技能", "自动发现")
    with col3:
        st.metric("⚡ 执行", "完整追踪")
    
    st.divider()
    
    # 快速开始
    st.subheader("🚀 快速开始")
    col1, col2 = st.columns(2)
    
    with col1:
        st.info("""
        ### 💬 与 AI 聊天
        点击左侧导航进入**聊天页面**开始对话。
        - 支持多轮对话
        - 自动调用技能
        - 实时执行追踪
        """)
    
    with col2:
        st.info("""
        ### 📚 管理技能
        点击左侧导航查看**技能页面**。
        - 查看所有可用技能
        - 查看技能详情
        - 手动加载/卸载技能
        """)
    
    # 最近的执行
    st.subheader("⚡ 最近的执行")
    core_services = st.session_state.core_services
    recent_executions = core_services.execution_tracker.get_recent_executions(5)
    
    if recent_executions:
        for execution in recent_executions:
            with st.expander(
                f"📍 {execution.id[:8]} - "
                f"{execution.status.value} - "
                f"{execution.total_duration_ms:.0f}ms"
            ):
                col1, col2, col3 = st.columns(3)
                with col1:
                    st.metric("工具调用", execution.total_tool_calls)
                with col2:
                    st.metric("成功", execution.successful_tool_calls)
                with col3:
                    success_rate = (
                        execution.successful_tool_calls / execution.total_tool_calls * 100
                        if execution.total_tool_calls > 0 else 100
                    )
                    st.metric("成功率", f"{success_rate:.0f}%")
    else:
        st.info("暂无执行记录。开始聊天来创建记录。")

# ============================================================================
# 主入口
# ============================================================================

def main():
    """主入口函数"""
    # 配置
    setup_page_config()
    
    # 初始化会话状态
    initialize_session_state()
    
    # 设置日志和服务
    setup_logging_and_services()
    
    # 渲染侧边栏
    setup_sidebar()
    
    # 渲染主内容
    render_main_content()

if __name__ == "__main__":
    main()
```

---

## 4. 聊天页面实现

### 4.1 聊天页面

```python
# src/openskill/ui/pages/1_💬_Chat.py

import streamlit as st
import asyncio
from datetime import datetime
import sys
from pathlib import Path

sys.path.insert(0, str(Path(__file__).parent.parent.parent.parent))

from openskill.models.message import Message, MessageRole
from openskill.utils.logger import get_logger

logger = get_logger(__name__)

st.set_page_config(
    page_title="Chat - OpenSkill Agent",
    page_icon="💬",
    layout="wide",
)

def render_chat_interface():
    """渲染聊天界面"""
    st.title("💬 与 AI 聊天")
    
    core_services = st.session_state.core_services
    
    # 创建两列布局
    col_chat, col_execution = st.columns([2, 1], gap="large")
    
    with col_chat:
        st.subheader("对话")
        
        # 聊天历史容器
        chat_container = st.container()
        
        # 显示聊天历史
        messages = core_services.message_manager.get_all_messages()
        with chat_container:
            for message in messages:
                if message.role == MessageRole.USER:
                    st.chat_message("user").write(message.content)
                elif message.role == MessageRole.ASSISTANT:
                    st.chat_message("assistant").write(message.content)
                elif message.role == MessageRole.TOOL:
                    with st.chat_message("tool"):
                        st.write(f"**{message.tool_name}**: {message.content}")
        
        # 用户输入
        st.divider()
        
        # 输入框和按钮在同一行
        col_input, col_button = st.columns([5, 1])
        
        with col_input:
            user_input = st.chat_input(
                "输入你的问题...",
                key="user_message"
            )
        
        with col_button:
            send_button = st.button("发送", use_container_width=True)
        
        # 处理用户输入
        if send_button and user_input:
            with st.spinner("正在处理..."):
                asyncio.run(handle_user_input(user_input, core_services))
    
    with col_execution:
        st.subheader("⚡ 执行信息")
        
        if 'current_execution_id' in st.session_state:
            execution_id = st.session_state.current_execution_id
            execution = core_services.execution_tracker.get_execution(execution_id)
            
            if execution:
                summary = execution.get_execution_summary()
                
                st.metric("总耗时", f"{summary['total_duration_ms']:.0f}ms")
                st.metric("工具调用", summary['total_tool_calls'])
                st.metric(
                    "成功率",
                    f"{summary['success_rate'] * 100:.0f}%"
                )
                
                st.divider()
                
                st.write("**工具调用链**")
                for i, frame in enumerate(execution.frames):
                    for tool_call in frame.tool_calls:
                        status_icon = "✓" if tool_call.is_successful() else "✗"
                        st.write(
                            f"{status_icon} {tool_call.tool_name} "
                            f"({tool_call.duration_ms:.0f}ms)"
                        )

async def handle_user_input(user_input: str, core_services):
    """处理用户输入"""
    import uuid
    
    logger.info(f"处理用户输入: {user_input[:50]}...")
    
    # 创建执行
    execution_id = f"exec_{uuid.uuid4().hex[:8]}"
    session_id = st.session_state.get(
        'session_id',
        f"session_{uuid.uuid4().hex[:8]}"
    )
    
    execution = core_services.execution_tracker.start_execution(
        execution_id,
        session_id,
        user_input
    )
    
    st.session_state.current_execution_id = execution_id
    
    # 添加用户消息
    user_message = Message(
        id=f"msg_{uuid.uuid4().hex[:8]}",
        role=MessageRole.USER,
        content=user_input,
        session_id=session_id,
        execution_id=execution_id,
    )
    core_services.message_manager.add_message(user_message)
    
    try:
        # 调用 Agent（这里简化处理，实际应该调用 Agent Orchestrator）
        # TODO: 集成 Agent Orchestrator
        
        response = f"我收到了你的问题：'{user_input}'"
        
        # 添加 Assistant 消息
        assistant_message = Message(
            id=f"msg_{uuid.uuid4().hex[:8]}",
            role=MessageRole.ASSISTANT,
            content=response,
            session_id=session_id,
            execution_id=execution_id,
        )
        core_services.message_manager.add_message(assistant_message)
        
        # 完成执行
        core_services.execution_tracker.complete_execution(response)
        
        # 更新统计
        st.session_state.total_messages = len(
            core_services.message_manager.get_all_messages()
        )
        
        st.rerun()
    
    except Exception as e:
        logger.error(f"处理消息时出错: {e}")
        st.error(f"处理消息时出错: {str(e)}")

def main():
    """主函数"""
    # 初始化会话状态
    if 'session_id' not in st.session_state:
        import uuid
        st.session_state.session_id = f"session_{uuid.uuid4().hex[:8]}"
    
    render_chat_interface()

if __name__ == "__main__":
    main()
```

---

## 5. 其他页面

### 5.1 技能管理页面

```python
# src/openskill/ui/pages/2_🧠_Skills.py

import streamlit as st
import sys
from pathlib import Path

sys.path.insert(0, str(Path(__file__).parent.parent.parent.parent))

st.set_page_config(
    page_title="Skills - OpenSkill Agent",
    page_icon="🧠",
    layout="wide",
)

def render_skills_page():
    """渲染技能页面"""
    st.title("🧠 技能管理")
    
    core_services = st.session_state.core_services
    
    # 获取统计信息
    stats = core_services.skill_manager.get_skill_stats()
    
    col1, col2, col3, col4 = st.columns(4)
    with col1:
        st.metric("总技能数", stats['total_skills'])
    with col2:
        st.metric("活跃技能", stats['active_skills'])
    with col3:
        st.metric("工具总数", stats['total_tools'])
    with col4:
        st.metric("分类数", len(stats['categories']))
    
    st.divider()
    
    # 按分类显示技能
    skills = core_services.skill_manager.get_available_skills()
    
    for category in set(s.category for s in skills):
        with st.expander(f"📦 {category.upper()}"):
            category_skills = core_services.skill_manager.get_skills_by_category(category)
            
            for skill in category_skills:
                with st.container(border=True):
                    col1, col2 = st.columns([3, 1])
                    
                    with col1:
                        st.subheader(skill.name)
                        st.write(skill.description)
                        st.caption(f"版本: {skill.version} | 作者: {skill.author}")
                        
                        # 显示工具列表
                        st.write("**工具:**")
                        for tool in skill.tools:
                            st.write(f"- `{tool.name}`: {tool.description}")
                    
                    with col2:
                        col_action1, col_action2 = st.columns(2)
                        with col_action1:
                            if st.button("📖 详情", key=f"btn_detail_{skill.id}"):
                                st.session_state.selected_skill_id = skill.id
                        with col_action2:
                            if st.button("🔧 配置", key=f"btn_config_{skill.id}"):
                                st.session_state.configure_skill_id = skill.id

def main():
    """主函数"""
    render_skills_page()

if __name__ == "__main__":
    main()
```

### 5.2 执行追踪页面

```python
# src/openskill/ui/pages/3_⚡_Execution.py

import streamlit as st
import pandas as pd
import sys
from pathlib import Path

sys.path.insert(0, str(Path(__file__).parent.parent.parent.parent))

st.set_page_config(
    page_title="Execution - OpenSkill Agent",
    page_icon="⚡",
    layout="wide",
)

def render_execution_page():
    """渲染执行追踪页面"""
    st.title("⚡ 执行追踪")
    
    core_services = st.session_state.core_services
    
    # 获取最近的执行
    recent_executions = core_services.execution_tracker.get_recent_executions(20)
    
    if not recent_executions:
        st.info("暂无执行记录")
        return
    
    # 创建执行列表表格
    execution_data = []
    for execution in recent_executions:
        execution_data.append({
            "ID": execution.id[:8],
            "状态": execution.status.value,
            "消息": execution.user_message[:50],
            "耗时(ms)": f"{execution.total_duration_ms:.0f}",
            "工具调用": execution.total_tool_calls,
            "成功率": f"{execution.successful_tool_calls}/{execution.total_tool_calls}",
        })
    
    df = pd.DataFrame(execution_data)
    st.dataframe(df, use_container_width=True)
    
    st.divider()
    
    # 详细查看
    st.subheader("📊 执行详情")
    
    selected_execution = st.selectbox(
        "选择执行",
        options=recent_executions,
        format_func=lambda e: f"{e.id[:8]} - {e.status.value}"
    )
    
    if selected_execution:
        col1, col2, col3, col4 = st.columns(4)
        with col1:
            st.metric("总耗时", f"{selected_execution.total_duration_ms:.0f}ms")
        with col2:
            st.metric("工具调用", selected_execution.total_tool_calls)
        with col3:
            success_rate = (
                selected_execution.successful_tool_calls / selected_execution.total_tool_calls * 100
                if selected_execution.total_tool_calls > 0 else 100
            )
            st.metric("成功率", f"{success_rate:.0f}%")
        with col4:
            st.metric("栈深度", len(selected_execution.frames))
        
        # 显示执行树
        st.write("**执行栈树:**")
        execution_tree = selected_execution.get_frame_tree()
        
        def render_tree(tree, depth=0):
            """递归渲染树"""
            indent = "  " * depth
            for tool_call in tree.get("tool_calls", []):
                st.write(
                    f"{indent}├─ {tool_call['name']} "
                    f"[{tool_call['status']}] ({tool_call['duration']:.0f}ms)"
                )
            for child in tree.get("children", []):
                render_tree(child, depth + 1)
        
        render_tree(execution_tree)

def main():
    """主函数"""
    render_execution_page()

if __name__ == "__main__":
    main()
```

---

## 6. 会话状态管理

### 6.1 会话状态工具函数

```python
# src/openskill/ui/utils/session_state.py

import streamlit as st
import uuid
from datetime import datetime

def initialize_session_state():
    """初始化会话状态"""
    if 'session_id' not in st.session_state:
        st.session_state.session_id = f"session_{uuid.uuid4().hex[:8]}"
    
    if 'created_at' not in st.session_state:
        st.session_state.created_at = datetime.now()
    
    if 'total_messages' not in st.session_state:
        st.session_state.total_messages = 0
    
    if 'total_tool_calls' not in st.session_state:
        st.session_state.total_tool_calls = 0
    
    if 'current_execution_id' not in st.session_state:
        st.session_state.current_execution_id = None
    
    if 'selected_model' not in st.session_state:
        st.session_state.selected_model = "gpt-4"
    
    if 'temperature' not in st.session_state:
        st.session_state.temperature = 0.7

def get_session_duration():
    """获取会话时长"""
    if 'created_at' in st.session_state:
        duration = datetime.now() - st.session_state.created_at
        return str(duration).split('.')[0]
    return "0:00:00"

def clear_session():
    """清除会话"""
    for key in list(st.session_state.keys()):
        if key != 'core_services':
            del st.session_state[key]
    initialize_session_state()
```

---

## 7. UI 组件库

### 7.1 可复用组件

```python
# src/openskill/ui/components/skill_card.py

import streamlit as st
from openskill.models.skill import Skill

def render_skill_card(skill: Skill, show_tools: bool = True):
    """
    渲染技能卡片
    
    Args:
        skill: Skill 对象
        show_tools: 是否显示工具列表
    """
    with st.container(border=True):
        col1, col2 = st.columns([3, 1])
        
        with col1:
            st.subheader(f"📦 {skill.name}")
            st.write(skill.description)
            
            # 元数据
            col_meta1, col_meta2, col_meta3 = st.columns(3)
            with col_meta1:
                st.caption(f"版本: {skill.version}")
            with col_meta2:
                st.caption(f"作者: {skill.author}")
            with col_meta3:
                st.caption(f"分类: {skill.category}")
            
            # 工具列表
            if show_tools and skill.tools:
                st.write("**工具:**")
                for tool in skill.tools:
                    st.write(f"- `{tool.name}`: {tool.description}")
        
        with col2:
            status_color = "green" if skill.is_available() else "red"
            st.write(f"**状态:** :{status_color}[{skill.status.value}]")
```

---

## 8. 运行 Streamlit 应用

### 8.1 启动应用

```bash
# 开发环境
streamlit run src/openskill/ui/app.py

# 指定端口
streamlit run src/openskill/ui/app.py --server.port 8501

# 不打开浏览器
streamlit run src/openskill/ui/app.py --logger.level=debug --client.showErrorDetails=true
```

### 8.2 部署到 Streamlit Cloud

```bash
# 1. 推送到 GitHub
git push origin main

# 2. 访问 https://share.streamlit.io
# 3. 连接 GitHub 仓库
# 4. 选择 main 分支和 src/openskill/ui/app.py
# 5. 设置环境变量（.streamlit/secrets.toml）
```

### 8.3 .streamlit/secrets.toml

```toml
[llm]
api_key = "sk-..."
api_base = "https://api.openai.com/v1"

[database]
url = "sqlite:///./data/openskill.db"

[skills]
directory = "./skills"
```

---

## 9. 常见问题和最佳实践

### 9.1 性能优化

```python
# 使用缓存减少重复计算
@st.cache_resource
def get_core_services():
    services = CoreServices()
    services.initialize()
    return services

# 使用会话缓存
@st.cache_data
def get_skill_stats():
    return st.session_state.core_services.skill_manager.get_skill_stats()
```

### 9.2 错误处理

```python
try:
    result = await core_services.tool_handler.execute_tool(...)
except Exception as e:
    st.error(f"执行工具失败: {str(e)}")
    logger.exception("Tool execution failed")
```

### 9.3 实时更新

```python
# 使用 empty() 和 placeholder() 实现实时更新
placeholder = st.empty()

for i in range(100):
    with placeholder.container():
        st.write(f"Progress: {i}%")
        st.progress(i / 100)
    time.sleep(0.1)
```

---

## 10. 下一步

1. 实现 Agent Orchestrator 集成
2. 添加更多 UI 组件
3. 实现实时流式响应
4. 添加对话导出功能
5. 实现用户认证（可选）
6. 性能优化和测试
7. 部署到生产环境