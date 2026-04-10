# AI Coding Agent Framework - Skill 开发指南

## 1. Skill 系统概述

### 1.1 Skill 的三个部分

一个完整的 Skill 由以下三个部分组成：

```
my_skill/
├── skill.json         # 技能定义（元数据和工具声明）
├── config.yaml        # 技能配置（环境变量、参数等）
├── tools.py           # 工具实现（Python 代码）
└── README.md          # 文档
```

### 1.2 Skill 的生命周期

```
Skill 开发 → Skill 打包 → Skill 加载 → Skill 索引 → Skill 调用 → Skill 执行
           ↑                                          ↓
           └──────────────────── Skill 卸载 ─────────┘
```

---

## 2. skill.json 详解

### 2.1 完整示例

```json
{
  "id": "web_search",
  "name": "Web Search",
  "version": "1.0.0",
  "description": "搜索互联网获取信息",
  "category": "information_retrieval",
  "author": "OpenSkill Team",
  "license": "MIT",
  "homepage": "https://github.com/openskill/skills/tree/main/web_search",
  "repository": "https://github.com/openskill/skills",
  
  "dependencies": [
    "requests>=2.28.0",
    "beautifulsoup4>=4.11.0"
  ],
  
  "config_schema": {
    "type": "object",
    "properties": {
      "search_engine": {
        "type": "string",
        "description": "搜索引擎 (google, bing, duckduckgo)",
        "default": "google"
      },
      "max_results": {
        "type": "integer",
        "description": "最大搜索结果数",
        "default": 10
      },
      "api_key": {
        "type": "string",
        "description": "API 密钥（可选）"
      }
    }
  },
  
  "tools": [
    {
      "id": "search",
      "name": "search",
      "description": "在互联网上搜索信息",
      "category": "search",
      "tags": ["web", "search", "information"],
      
      "parameters": {
        "type": "object",
        "properties": {
          "query": {
            "type": "string",
            "description": "搜索关键词"
          },
          "language": {
            "type": "string",
            "description": "搜索语言代码 (en, zh, etc)",
            "default": "en"
          },
          "num_results": {
            "type": "integer",
            "description": "返回结果数",
            "default": 5,
            "minimum": 1,
            "maximum": 20
          }
        },
        "required": ["query"]
      },
      
      "return_type": "object",
      "returns": {
        "type": "object",
        "properties": {
          "results": {
            "type": "array",
            "items": {
              "type": "object",
              "properties": {
                "title": {"type": "string"},
                "url": {"type": "string"},
                "snippet": {"type": "string"}
              }
            }
          },
          "total_results": {"type": "integer"}
        }
      },
      
      "examples": [
        {
          "input": {"query": "Python tutorial"},
          "output": {
            "results": [
              {
                "title": "Python Official Tutorial",
                "url": "https://docs.python.org/tutorial/",
                "snippet": "Learn Python from the official documentation..."
              }
            ],
            "total_results": 1000000
          }
        }
      ],
      
      "rate_limit": {
        "requests_per_minute": 60,
        "requests_per_day": 10000
      },
      
      "timeout": 30,
      "retry_on_failure": true,
      "max_retries": 3
    },
    
    {
      "id": "advanced_search",
      "name": "advanced_search",
      "description": "高级搜索，支持多个过滤器",
      
      "parameters": {
        "type": "object",
        "properties": {
          "query": {"type": "string"},
          "site": {
            "type": "string",
            "description": "限制搜索的网站"
          },
          "date_from": {
            "type": "string",
            "description": "搜索开始日期 (YYYY-MM-DD)"
          },
          "date_to": {
            "type": "string",
            "description": "搜索结束日期 (YYYY-MM-DD)"
          }
        },
        "required": ["query"]
      },
      
      "return_type": "object"
    }
  ],
  
  "permissions": [
    "network:http",
    "network:https"
  ],
  
  "metadata": {
    "icon": "🔍",
    "color": "#FF6B6B",
    "featured": true,
    "popularity": 1000,
    "rating": 4.8,
    "reviews": 150
  }
}
```

### 2.2 skill.json 字段说明

| 字段 | 类型 | 必需 | 说明 |
|------|------|------|------|
| id | string | ✓ | 技能的唯一标识符 |
| name | string | ✓ | 技能的显示名称 |
| version | string | ✓ | 版本号（遵循 SemVer） |
| description | string | ✓ | 技能的描述 |
| category | string | ✓ | 技能分类 |
| author | string | ✓ | 技能作者 |
| license | string | | 开源许可证 |
| tools | array | ✓ | 工具数组 |
| dependencies | array | | 依赖的 Python 包 |
| config_schema | object | | 配置参数的 JSON Schema |
| permissions | array | | 需要的权限 |

---

## 3. tools.py 实现

### 3.1 基础结构

```python
# skills/web_search/tools.py

"""
Web Search Skill Tools

这个模块实现了 Web Search 技能的所有工具函数。
每个工具函数对应 skill.json 中定义的一个 tool。
"""

import logging
from typing import Dict, Any, List, Optional
from dataclasses import dataclass
import requests
from datetime import datetime

# 设置日志
logger = logging.getLogger(__name__)

# ============================================================================
# 数据模型
# ============================================================================

@dataclass
class SearchResult:
    """搜索结果"""
    title: str
    url: str
    snippet: str
    date: Optional[str] = None
    source: str = "google"
    relevance_score: float = 0.0

# ============================================================================
# 工具实现
# ============================================================================

def search(query: str, language: str = "en", num_results: int = 5) -> Dict[str, Any]:
    """
    在互联网上搜索信息
    
    这个函数实现了 skill.json 中定义的 'search' 工具。
    
    Args:
        query: 搜索关键词
        language: 搜索语言代码 (默认: en)
        num_results: 返回结果数 (默认: 5, 范围: 1-20)
    
    Returns:
        包含搜索结果的字典：
        {
            "results": [
                {
                    "title": "...",
                    "url": "...",
                    "snippet": "..."
                },
                ...
            ],
            "total_results": 1000000,
            "search_time_ms": 150,
            "query": "..."
        }
    
    Raises:
        ValueError: 参数无效
        RequestException: 网络请求失败
        
    Example:
        >>> result = search("python tutorial", num_results=3)
        >>> print(result["results"][0]["title"])
    """
    # 参数验证
    if not query or not isinstance(query, str):
        raise ValueError("query 必须是非空字符串")
    
    if not 1 <= num_results <= 20:
        raise ValueError("num_results 必须在 1-20 之间")
    
    query = query.strip()
    logger.info(f"搜索: {query} (语言: {language}, 结果数: {num_results})")
    
    try:
        # 调用搜索 API
        search_results = _perform_search(query, language, num_results)
        
        logger.info(f"搜索成功: 获得 {len(search_results)} 个结果")
        
        # 格式化返回结果
        return {
            "success": True,
            "query": query,
            "language": language,
            "num_results": num_results,
            "results": [
                {
                    "title": result.title,
                    "url": result.url,
                    "snippet": result.snippet,
                    "relevance_score": result.relevance_score
                }
                for result in search_results
            ],
            "total_results": _get_total_results_count(query),
            "timestamp": datetime.now().isoformat()
        }
    
    except requests.exceptions.RequestException as e:
        logger.error(f"搜索请求失败: {e}")
        return {
            "success": False,
            "query": query,
            "error": f"搜索请求失败: {str(e)}",
            "timestamp": datetime.now().isoformat()
        }
    
    except Exception as e:
        logger.error(f"搜索过程中发生错误: {e}")
        return {
            "success": False,
            "query": query,
            "error": f"搜索过程中发生错误: {str(e)}",
            "timestamp": datetime.now().isoformat()
        }

def advanced_search(
    query: str,
    site: Optional[str] = None,
    date_from: Optional[str] = None,
    date_to: Optional[str] = None
) -> Dict[str, Any]:
    """
    高级搜索，支持多个过滤器
    
    Args:
        query: 搜索关键词
        site: 限制搜索的网站 (可选)
        date_from: 搜索开始日期 YYYY-MM-DD (可选)
        date_to: 搜索结束日期 YYYY-MM-DD (可选)
    
    Returns:
        搜索结果
    """
    logger.info(f"高级搜索: {query}")
    
    # 构建搜索查询
    search_query = query
    
    if site:
        search_query += f" site:{site}"
    
    if date_from or date_to:
        # 添加日期过滤
        if date_from:
            search_query += f" after:{date_from}"
        if date_to:
            search_query += f" before:{date_to}"
    
    # 调用基础搜索
    return search(search_query)

# ============================================================================
# 辅助函数
# ============================================================================

def _perform_search(
    query: str,
    language: str,
    num_results: int
) -> List[SearchResult]:
    """
    执行实际的搜索请求
    
    注意：这是一个简化实现，实际应该调用真实的搜索 API
    """
    # TODO: 集成真实的搜索 API (Google API, Bing API 等)
    
    # 模拟结果（用于演示）
    mock_results = [
        SearchResult(
            title="Python Official Documentation",
            url="https://docs.python.org",
            snippet="The official Python documentation...",
            relevance_score=0.95
        ),
        SearchResult(
            title="Real Python Tutorials",
            url="https://realpython.com",
            snippet="High-quality Python tutorials and articles...",
            relevance_score=0.90
        ),
    ]
    
    return mock_results[:num_results]

def _get_total_results_count(query: str) -> int:
    """获取搜索结果总数"""
    # TODO: 从搜索 API 获取实际的总数
    return 1000000

# ============================================================================
# 错误处理
# ============================================================================

class SearchError(Exception):
    """搜索错误基类"""
    pass

class SearchAPIError(SearchError):
    """搜索 API 错误"""
    pass

class SearchTimeoutError(SearchError):
    """搜索超时"""
    pass
```

### 3.2 高级特性

```python
# 支持异步函数
import asyncio

async def async_search(query: str) -> Dict[str, Any]:
    """异步搜索"""
    # 异步实现
    await asyncio.sleep(1)
    return search(query)

# 支持生成器（流式结果）
def search_stream(query: str):
    """流式搜索结果"""
    results = search(query)
    for result in results.get("results", []):
        yield result

# 支持类（状态保存）
class SearchCache:
    """带缓存的搜索器"""
    
    def __init__(self, max_cache_size: int = 100):
        self.cache = {}
        self.max_cache_size = max_cache_size
    
    def search(self, query: str) -> Dict[str, Any]:
        """搜索（带缓存）"""
        if query in self.cache:
            logger.info(f"缓存命中: {query}")
            return self.cache[query]
        
        result = search(query)
        
        if len(self.cache) >= self.max_cache_size:
            # 移除最旧的项
            oldest_key = next(iter(self.cache))
            del self.cache[oldest_key]
        
        self.cache[query] = result
        return result
    
    def clear_cache(self):
        """清空缓存"""
        self.cache.clear()
```

---

## 4. config.yaml

### 4.1 配置文件示例

```yaml
# skills/web_search/config.yaml

# 搜索引擎配置
search_engine: "google"

# API 密钥（从环境变量读取）
api_key: "${SEARCH_API_KEY}"
api_base: "https://api.example.com"

# 搜索参数
max_results: 10
timeout: 30
max_retries: 3

# 代理设置（可选）
proxy:
  enabled: false
  http: "http://proxy.example.com:8080"
  https: "http://proxy.example.com:8080"

# 缓存配置
cache:
  enabled: true
  ttl: 3600  # 缓存时间（秒）
  max_size: 1000

# 日志配置
logging:
  level: "INFO"
  format: "json"

# 限流配置
rate_limit:
  requests_per_minute: 60
  requests_per_day: 10000

# 错误处理
error_handling:
  retry_on_timeout: true
  fallback_engine: "duckduckgo"
```

---

## 5. Skill 结构最佳实践

### 5.1 目录结构

```
skills/web_search/
├── skill.json                 # 必需：技能定义
├── tools.py                   # 必需：工具实现
├── config.yaml                # 推荐：配置文件
├── README.md                  # 推荐：文档
├── requirements.txt           # 可选：Python 依赖
├── tests/                     # 可选：测试
│   ├── test_search.py
│   ├── test_advanced_search.py
│   └── conftest.py
├── examples/                  # 可选：示例
│   ├── basic_usage.py
│   └── advanced_usage.py
└── docs/                      # 可选：详细文档
    ├── api_reference.md
    ├── examples.md
    └── troubleshooting.md
```

### 5.2 代码质量检查

```python
# tests/test_search.py

import pytest
from skills.web_search.tools import search, advanced_search, SearchError

class TestSearch:
    """搜索工具测试"""
    
    def test_basic_search(self):
        """测试基础搜索"""
        result = search("python")
        assert result["success"] is True
        assert "results" in result
        assert len(result["results"]) > 0
    
    def test_search_with_num_results(self):
        """测试指定结果数"""
        result = search("python", num_results=3)
        assert len(result["results"]) <= 3
    
    def test_invalid_query(self):
        """测试无效查询"""
        result = search("")
        assert result["success"] is False
    
    def test_advanced_search(self):
        """测试高级搜索"""
        result = advanced_search(
            "python",
            site="github.com",
            date_from="2023-01-01"
        )
        assert result["success"] is True
    
    @pytest.mark.asyncio
    async def test_async_search(self):
        """测试异步搜索"""
        from skills.web_search.tools import async_search
        result = await async_search("python")
        assert result["success"] is True
```

---

## 6. Skill 开发流程

### 6.1 创建新 Skill

```bash
# 1. 创建目录
mkdir skills/my_skill
cd skills/my_skill

# 2. 创建基础文件
touch skill.json config.yaml tools.py README.md
```

### 6.2 skill.json 验证

```python
# scripts/validate_skill.py

import json
from pathlib import Path
from jsonschema import validate, ValidationError

SKILL_SCHEMA = {
    "type": "object",
    "properties": {
        "id": {"type": "string"},
        "name": {"type": "string"},
        "version": {"type": "string"},
        "description": {"type": "string"},
        "category": {"type": "string"},
        "tools": {
            "type": "array",
            "items": {
                "type": "object",
                "properties": {
                    "name": {"type": "string"},
                    "description": {"type": "string"},
                    "parameters": {"type": "object"}
                },
                "required": ["name", "description", "parameters"]
            }
        }
    },
    "required": ["id", "name", "version", "description", "category", "tools"]
}

def validate_skill(skill_path: Path):
    """验证 skill.json"""
    with open(skill_path / "skill.json") as f:
        skill_data = json.load(f)
    
    try:
        validate(instance=skill_data, schema=SKILL_SCHEMA)
        print("✓ Skill 验证成功")
        return True
    except ValidationError as e:
        print(f"✗ Skill 验证失败: {e.message}")
        return False

if __name__ == "__main__":
    validate_skill(Path("skills/web_search"))
```

### 6.3 Skill 打包和发布

```bash
# 1. 验证
python scripts/validate_skill.py skills/my_skill

# 2. 运行测试
pytest skills/my_skill/tests/

# 3. 构建文档
python scripts/generate_skill_docs.py skills/my_skill

# 4. 打包（可选）
tar -czf my_skill-1.0.0.tar.gz skills/my_skill/

# 5. 发布到 PyPI 或共享仓库
```

---

## 7. 常见 Skill 类型

### 7.1 API 调用 Skill

```python
# skills/weather_api/tools.py

import requests

def get_weather(city: str, units: str = "metric") -> Dict[str, Any]:
    """获取天气信息"""
    api_url = "https://api.openweathermap.org/data/2.5/weather"
    
    params = {
        "q": city,
        "appid": "${WEATHER_API_KEY}",
        "units": units
    }
    
    response = requests.get(api_url, params=params)
    response.raise_for_status()
    
    data = response.json()
    return {
        "city": data["name"],
        "temperature": data["main"]["temp"],
        "description": data["weather"][0]["description"],
        "humidity": data["main"]["humidity"],
        "wind_speed": data["wind"]["speed"]
    }
```

### 7.2 文件操作 Skill

```python
# skills/file_operations/tools.py

from pathlib import Path

def read_file(path: str) -> str:
    """读取文件"""
    file_path = Path(path)
    
    if not file_path.exists():
        raise FileNotFoundError(f"文件不存在: {path}")
    
    with open(file_path, 'r', encoding='utf-8') as f:
        return f.read()

def write_file(path: str, content: str) -> Dict[str, Any]:
    """写入文件"""
    file_path = Path(path)
    file_path.parent.mkdir(parents=True, exist_ok=True)
    
    with open(file_path, 'w', encoding='utf-8') as f:
        f.write(content)
    
    return {
        "success": True,
        "path": str(file_path),
        "size": len(content),
        "timestamp": datetime.now().isoformat()
    }
```

### 7.3 计算 Skill

```python
# skills/math_operations/tools.py

import math

def calculate_expression(expression: str) -> Dict[str, Any]:
    """计算数学表达式"""
    try:
        result = eval(expression, {"__builtins__": {}}, {
            "sqrt": math.sqrt,
            "sin": math.sin,
            "cos": math.cos,
            "tan": math.tan,
            "pi": math.pi,
            "e": math.e,
        })
        return {
            "expression": expression,
            "result": result,
            "success": True
        }
    except Exception as e:
        return {
            "expression": expression,
            "error": str(e),
            "success": False
        }
```

---

## 8. Skill 开发检查清单

- [ ] skill.json 已创建并验证
- [ ] tools.py 已实现所有工具函数
- [ ] config.yaml 已创建（如需要）
- [ ] README.md 已编写
- [ ] 所有函数都有文档字符串
- [ ] 输入参数已验证
- [ ] 错误处理已实现
- [ ] 日志记录已添加
- [ ] 单元测试已通过
- [ ] 代码已通过 linting
- [ ] 示例代码已提供
- [ ] 依赖列表已更新