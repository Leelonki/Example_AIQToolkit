# 多功能AI助手项目

基于NVIDIA Agent Intelligence Toolkit (AIQ toolkit)开发的智能聊天机器人系统。

## 项目简介

本项目基于NVIDIA的AIQ toolkit开源工具包，开发了一个功能丰富的智能聊天助手系统。通过整合多种工具和能力，为用户提供全方位的智能对话服务。

### 主要功能

- **基础数学工具**：支持各类数学计算和问题求解
- **智能对话**：基于NVIDIA NIM大模型，提供流畅的自然语言交互体验
- **工具集成**：集成了多种实用工具，可扩展性强

## 技术栈

- NVIDIA Agent Intelligence Toolkit (AIQ toolkit)
- Python 3.11/3.12
- NVIDIA NIM大语言模型
- 各类工具包和插件

## 快速开始

### 环境要求

- Git
- Git LFS (Large File Storage)
- Python 3.11 或 3.12
- uv包管理器

### 安装步骤

1. 克隆仓库
```bash
git clone https://github.com/Leelonki/AIQToolkit.git
cd AIQToolkit
```

2. 安装依赖
```bash
uv venv --seed .venv
source .venv/bin/activate  # Linux/Mac
# 或
.venv\Scripts\activate  # Windows
```

3. 安装项目依赖
```bash
uv sync --all-groups --all-extras
```

4. 设置NVIDIA API密钥
```bash
$env:NVIDIA_API_KEY="你的API密钥"  # PowerShell
# 或
export NVIDIA_API_KEY="你的API密钥"  # Bash
```

### 使用示例

创建配置文件 `workflow.yaml`:

```yaml
functions:
  calculator:
    _type: calculator
    operations: [add, subtract, multiply, divide]

llms:
  nim_llm:
    _type: nim
    model_name: meta/llama-3.1-70b-instruct
    temperature: 0.0

workflow:
  _type: react_agent
  tool_names: [calculator]
  llm_name: nim_llm
  verbose: true
  retry_parsing_errors: true
  max_retries: 3
```

运行助手：

```bash
aiq run --config_file workflow.yaml --input "请计算23+45的结果"
```

## 工具列表

本项目集成了多种实用工具，包括但不限于：

- 基础数学计算器
- 文本处理工具
- 知识检索工具
- 数据分析工具

## 贡献指南

欢迎提交Issue和Pull Request来帮助改进项目。请确保遵循项目的代码规范和提交指南。

## 许可证

本项目基于 Apache 2.0 许可证开源。

## 致谢

特别感谢NVIDIA提供的AIQ toolkit框架支持。

## 联系方式

如有问题或建议，请通过以下方式联系：

- GitHub Issues
- Email: leelonki395@gmail.com
