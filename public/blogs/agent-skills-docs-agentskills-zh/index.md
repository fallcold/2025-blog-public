> ?????<https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering>
> ?????`docs/agentskills.md`
---
name: agent-skills-format
description: Agent Skills 格式的官方文档——一种轻量、开放的标准，用于通过专门知识与工作流扩展 AI 代理能力。
doc_type: reference
source_url: No
---

概览

复制页面

一种简单、开放的格式，用于为代理提供新的能力和专业知识。

Agent Skills 是由说明、脚本和资源组成的文件夹，代理可以发现并使用它们，以更准确、更高效地完成任务。
鈥?
为什么需要 Agent Skills？
代理的能力正在不断增强，但它们通常缺少可靠完成真实工作的上下文。Skill 通过向代理提供可按需加载的流程性知识，以及公司、团队和用户特定的上下文来解决这个问题。能够访问一组 skills 的代理，会根据当前任务扩展自身能力。
对于 skill 作者：一次构建能力，并将其部署到多个代理产品。
对于兼容的代理：支持 skills 能让终端用户开箱即用地为代理增加新能力。
对于团队和企业：用可移植、可版本控制的包来沉淀组织知识。
鈥?
Agent Skills 能实现什么？
领域专长：将专门知识封装成可复用指令，从法律审查流程到数据分析流水线。
新能力：为代理增加新能力（例如创建演示文稿、构建 MCP 服务器、分析数据集）。
可重复工作流：将多步骤任务转化为一致、可审计的工作流。
互操作性：在不同的 skills 兼容代理产品之间复用同一个 skill。
鈥?
采用情况
主流 AI 开发工具已支持 Agent Skills。
OpenCode
Cursor
Amp
Letta
Goose
GitHub
VS Code
Claude Code
Claude
OpenAI Codex
鈥?
开放开发
Agent Skills 格式最初由 Anthropic 开发并以开放标准发布，目前已被越来越多代理产品采用。该标准向更广泛生态开放贡献。

什么是 skills？

复制页面

Agent Skills 是一种轻量、开放的格式，用于通过专门知识和工作流扩展 AI 代理能力。

从核心上看，skill 就是一个包含 SKILL.md 文件的文件夹。这个文件包含元数据（至少包括名称和描述）以及告诉代理如何执行特定任务的说明。Skill 还可以打包脚本、模板和参考资料。
my-skill/
鈹溾攢鈹€ SKILL.md          # 必需：说明 + 元数据
鈹溾攢鈹€ scripts/          # 可选：可执行代码
鈹溾攢鈹€ references/       # 可选：文档
鈹斺攢鈹€ assets/           # 可选：模板、资源
鈥?
skills 如何工作
skills 使用渐进式披露（progressive disclosure）来高效管理上下文：
发现：启动时，代理只加载每个可用 skill 的名称和描述，足以判断何时可能相关。
激活：当任务匹配 skill 描述时，代理把完整的 SKILL.md 说明读入上下文。
执行：代理遵循说明，按需加载引用文件或执行打包代码。
这种方式让代理保持高效，同时按需访问更多上下文。
鈥?
SKILL.md 文件
每个 skill 都从一个包含 YAML frontmatter 和 Markdown 指令的 SKILL.md 文件开始：
---
name: pdf-processing
description: Extract text and tables from PDF files, fill forms, merge documents.
---

# PDF Processing

## When to use this skill
Use this skill when the user needs to work with PDF files...

## How to extract text
1. Use pdfplumber for text extraction...

## How to fill forms
...
SKILL.md 顶部需要如下 frontmatter：
name: 一个简短标识符
description: 何时使用这个 skill
Markdown 主体包含实际说明，在结构和内容上没有特定限制。
这种简单格式有几个关键优势：
自说明：skill 作者或用户可以直接阅读 SKILL.md 并理解它做什么，便于审计和改进。
可扩展：skill 的复杂度可以从纯文本说明到可执行代码、资产和模板。
可移植：skill 本质上就是文件，因此易于编辑、版本管理和共享。
鈥?
下一步
查看规范以理解完整格式。
为你的代理添加 skills 支持，以构建兼容客户端。
在 GitHub 上查看示例 skills。
阅读编写高质量 skills 的最佳实践。
使用参考库校验 skills 并生成 prompt XML。

规范

复制页面

Agent Skills 的完整格式规范。

本文档定义 Agent Skills 格式。
鈥?
目录结构
一个 skill 至少是包含 SKILL.md 文件的目录：
skill-name/
鈹斺攢鈹€ SKILL.md          # Required
你也可以选择加入 scripts/、references/、assets/ 等附加目录来支持 skill。
鈥?
SKILL.md 格式
SKILL.md 文件必须包含 YAML frontmatter，后跟 Markdown 内容。
鈥?
Frontmatter（必需）
---
name: skill-name
description: A description of what this skill does and when to use it.
---
可选字段如下：
---
name: pdf-processing
description: Extract text and tables from PDF files, fill forms, merge documents.
license: Apache-2.0
metadata:
  author: example-org
  version: "1.0"
---
Field	Required	Constraints
name	Yes	最多 64 个字符。仅允许小写字母、数字和连字符。不能以连字符开头或结尾。
description	Yes	最多 1024 个字符。不能为空。描述 skill 做什么以及何时使用。
license	No	许可证名称，或对打包许可证文件的引用。
compatibility	No	最多 500 个字符。用于说明环境要求（目标产品、系统包、网络访问等）。
metadata	No	附加元数据的任意键值映射。
allowed-tools	No	skill 可使用的预批准工具列表（空格分隔）。（实验性）
鈥?
name 字段
必填的 name 字段：
必须为 1-64 个字符
只能包含 unicode 小写字母数字与连字符（a-z 和 -）
不能以 - 开头或结尾
不能包含连续连字符（--）
必须与父目录名一致
有效示例：
name: pdf-processing
name: data-analysis
name: code-review
无效示例：
name: PDF-Processing  # 不允许大写
name: -pdf  # 不能以连字符开头
name: pdf--processing  # 不允许连续连字符
鈥?
description 字段
必填的 description 字段：
必须为 1-1024 个字符
应同时描述 skill 做什么以及何时使用
应包含有助于代理识别相关任务的具体关键词
良好示例：
description: Extracts text and tables from PDF files, fills PDF forms, and merges multiple PDFs. Use when working with PDF documents or when the user mentions PDFs, forms, or document extraction.
较差示例：
description: Helps with PDFs.
鈥?
license 字段
可选的 license 字段：
用于指定 skill 适用的许可证
建议保持简短（许可证名称或打包许可证文件名）
示例：
license: Proprietary. LICENSE.txt has complete terms
鈥?
compatibility 字段
可选的 compatibility 字段：
若提供，必须为 1-500 个字符
仅在 skill 存在特定环境要求时才应包含
可用于说明目标产品、所需系统包、网络访问需求等
示例：
compatibility: Designed for Claude Code (or similar products)
compatibility: Requires git, docker, jq, and access to the internet
大多数 skills 不需要 compatibility 字段。
鈥?
metadata 字段
可选的 metadata 字段：
字符串键到字符串值的映射
客户端可用它保存 Agent Skills 规范未定义的附加属性
建议让键名有足够唯一性，以避免意外冲突
示例：
metadata:
  author: example-org
  version: "1.0"
鈥?
allowed-tools 字段
可选的 allowed-tools 字段：
一个空格分隔的、预批准可运行工具列表
实验性。不同代理实现对该字段的支持可能不同
示例：
allowed-tools: Bash(git:*) Bash(jq:*) Read
鈥?
正文内容
frontmatter 后的 Markdown 主体就是 skill 指令。格式没有限制。写任何有助于代理有效完成任务的内容。
推荐章节：
分步骤说明
输入与输出示例
常见边界情况
注意：一旦代理决定激活某个 skill，会加载整个文件。对于较长 SKILL.md，建议拆分到引用文件。
鈥?
可选目录
鈥?
scripts/
包含可供代理运行的可执行代码。脚本应当：
自包含，或清晰说明依赖
包含有帮助的错误信息
优雅处理边界情况
支持的语言取决于代理实现。常见选项包括 Python、Bash 和 JavaScript。
鈥?
references/
包含代理可按需读取的附加文档：
REFERENCE.md - 详细技术参考
FORMS.md - 表单模板或结构化数据格式
领域特定文件（finance.md、legal.md 等）
保持单个参考文件聚焦。代理按需加载这些文件，因此文件更小意味着更少上下文开销。
鈥?
assets/
包含静态资源：
模板（文档模板、配置模板）
图片（图表、示例）
数据文件（查找表、schema）
鈥?
渐进式披露
skills 应按高效使用上下文的方式组织：
元数据（约 100 tokens）：所有 skills 的 name 和 description 在启动时加载
指令（建议 < 5000 tokens）：激活 skill 时加载完整 SKILL.md 主体
资源（按需）：仅在需要时加载文件（例如 scripts/、references/、assets/ 中的文件）
将主 SKILL.md 控制在 500 行以内。把详细参考内容移到单独文件。
鈥?
文件引用
在 skill 中引用其他文件时，使用相对于 skill 根目录的路径：
See [the reference guide](references/REFERENCE.md) for details.

Run the extraction script:
scripts/extract.py
将文件引用保持为相对 SKILL.md 的一层深度。避免深层嵌套的引用链。
鈥?
验证
使用 skills-ref 参考库校验你的 skills：
skills-ref validate ./my-skill
这会检查 SKILL.md frontmatter 是否有效，并符合所有命名约定。

将 skills 集成到你的代理中

复制页面

如何为你的代理或工具添加 Agent Skills 支持。

本指南说明如何在 AI 代理或开发工具中添加 skills 支持。
鈥?
集成方式
集成 skills 的两种主要方式：
基于文件系统的代理运行在计算机环境（bash/unix）中，功能最强。模型发出类似 cat /path/to/my-skill/SKILL.md 的 shell 命令时激活 skill。打包资源通过 shell 命令访问。
基于工具的代理无需专用计算机环境。它们实现工具，使模型可以触发 skills 并访问打包资源。具体工具实现由开发者决定。
鈥?
概览
兼容 skills 的代理需要：
在配置目录中发现 skills
启动时加载元数据（name 和 description）
将用户任务匹配到相关 skills
通过加载完整说明来激活 skills
按需执行脚本并访问资源
鈥?
skill 发现
skills 是包含 SKILL.md 文件的文件夹。你的代理应扫描配置目录以发现有效 skill。
鈥?
加载元数据
启动时，仅解析每个 SKILL.md 的 frontmatter。这能保持初始上下文开销较低。
鈥?
解析 frontmatter
function parseMetadata(skillPath):
    content = readFile(skillPath + "/SKILL.md")
    frontmatter = extractYAMLFrontmatter(content)

    return {
        name: frontmatter.name,
        description: frontmatter.description,
        path: skillPath
    }
鈥?
注入到上下文
将 skill 元数据放入 system prompt，让模型知道可用 skills。
按照你平台关于 system prompt 更新的指导来做。例如，对 Claude 模型，推荐格式使用 XML：
<available_skills>
  <skill>
    <name>pdf-processing</name>
    <description>Extracts text and tables from PDF files, fills forms, merges documents.</description>
    <location>/path/to/skills/pdf-processing/SKILL.md</location>
  </skill>
  <skill>
    <name>data-analysis</name>
    <description>Analyzes datasets, generates charts, and creates summary reports.</description>
    <location>/path/to/skills/data-analysis/SKILL.md</location>
  </skill>
</available_skills>
对于基于文件系统的代理，location 字段应包含 SKILL.md 的绝对路径。对于基于工具的代理，可省略 location。
保持元数据简洁。每个 skill 大约应为上下文增加 50-100 tokens。
鈥?
安全注意事项
脚本执行会引入安全风险。可考虑：
沙箱化：在隔离环境中运行脚本
白名单：仅执行受信任 skills 的脚本
确认：在运行潜在危险操作前询问用户
日志：记录所有脚本执行用于审计
鈥?
参考实现
skills-ref 库提供了用于处理 skills 的 Python 工具和 CLI。
例如：
验证一个 skill 目录：
skills-ref validate <path>
为代理 prompt 生成 <available_skills> XML：
skills-ref to-prompt <path>...
把该库源码作为参考实现。

Skill 编写最佳实践

复制页面
学习如何编写 Claude 能够发现并成功使用的高质量 Skills。
好的 Skills 应该简洁、结构清晰，并经过真实使用测试。本指南给出实用的编写决策，帮助你写出 Claude 能有效发现和使用的 Skills。

关于 Skills 如何工作的概念背景，请参见 Skills overview。

核心原则
简洁最关键
上下文窗口是公共资源。你的 Skill 与 Claude 需要知道的其他所有信息共享同一个上下文窗口，包括：

系统提示词
对话历史
其他 Skills 的元数据
你的实际请求
Skill 中的每个 token 并非都会立即产生成本。启动时，预加载的只有所有 Skills 的元数据（name 和 description）。只有当 Skill 变得相关时，Claude 才会读取 SKILL.md；附加文件也仅在需要时读取。尽管如此，SKILL.md 的简洁仍很重要：一旦 Claude 加载它，每个 token 都会与对话历史和其他上下文竞争。

默认假设：Claude 已经非常聪明

只添加 Claude 原本不知道的上下文。对每段信息都要质疑：

"Claude 真的需要这段解释吗？"
"我可以假设 Claude 知道这个吗？"
"这段话是否值得它的 token 成本？"
好示例：简洁（约 50 tokens）：

## Extract PDF text

Use pdfplumber for text extraction:

```python
import pdfplumber

with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```
差示例：过于冗长（约 150 tokens）：

## Extract PDF text

PDF (Portable Document Format) files are a common file format that contains
text, images, and other content. To extract text from a PDF, you'll need to
use a library. There are many libraries available for PDF processing, but we
recommend pdfplumber because it's easy to use and handles most cases well.
First, you'll need to install it using pip. Then you can use the code below...
简洁版默认 Claude 已经知道 PDF 是什么，也知道库是如何使用的。

设置合适的自由度
让具体程度与任务的脆弱性和可变性匹配。

高自由度（基于文本的说明）：

适用场景：

多种方法都可行
决策依赖上下文
通过启发式来指导方法
示例：

## Code review process

1. Analyze the code structure and organization
2. Check for potential bugs or edge cases
3. Suggest improvements for readability and maintainability
4. Verify adherence to project conventions
中等自由度（带参数的伪代码或脚本）：

适用场景：

存在推荐模式
允许一定变化
配置会影响行为
示例：

## Generate report

Use this template and customize as needed:

```python
def generate_report(data, format="markdown", include_charts=True):
    # Process data
    # Generate output in specified format
    # Optionally include visualizations
```
低自由度（具体脚本，参数很少或没有参数）：

适用场景：

操作脆弱且易出错
一致性至关重要
必须遵循特定顺序
示例：

## Database migration

Run exactly this script:

```bash
python scripts/migrate.py --verify --backup
```

Do not modify the command or add additional flags.
类比：把 Claude 想象成沿路径探索的机器人：

两侧都是悬崖的窄桥：只有一条安全路径。提供明确护栏和精确指令（低自由度）。示例：必须按精确顺序执行的数据库迁移。
无危险的开阔地：多条路径都能成功。给出总体方向并信任 Claude 找到最佳路线（高自由度）。示例：代码审查中最佳方法取决于上下文。
用你计划使用的所有模型进行测试
Skills 是对模型能力的补充，因此效果取决于底层模型。请用你计划使用的所有模型测试你的 Skill。

不同模型的测试关注点：

Claude Haiku（快速、经济）：Skill 是否提供了足够指引？
Claude Sonnet（平衡）：Skill 是否清晰且高效？
Claude Opus（强推理）：Skill 是否避免了过度解释？
对 Opus 完美有效的内容，对 Haiku 可能需要更多细节。如果你计划跨多个模型使用 Skill，应让说明在各模型上都表现良好。

Skill 结构
YAML Frontmatter：SKILL.md frontmatter 要求两个字段：

name：

最多 64 个字符
只能包含小写字母、数字和连字符
不能包含 XML 标签
不能包含保留词："anthropic"、"claude"
description：

不能为空
最多 1024 个字符
不能包含 XML 标签
应描述 Skill 做什么以及何时使用
完整结构细节请参见 Skills overview。

命名约定
使用一致的命名模式，让 Skill 更容易被引用和讨论。我们建议 Skill 名称使用动名词形式（动词 + -ing），这能清晰表达 Skill 提供的活动或能力。

请记住，name 字段只能使用小写字母、数字和连字符。

好的命名示例（动名词形式）：

processing-pdfs
analyzing-spreadsheets
managing-databases
testing-code
writing-documentation
可接受替代：

名词短语：pdf-processing、spreadsheet-analysis
动作导向：process-pdfs、analyze-spreadsheets
避免：

模糊名称：helper、utils、tools
过于泛化：documents、data、files
保留词：anthropic-helper、claude-tools
你的 skill 集合内部命名模式不一致
一致命名有助于：

在文档和对话中引用 Skills
一眼看懂 Skill 做什么
组织并搜索多个 Skills
维护专业、统一的 skill 库
撰写有效 description
description 字段用于 Skill 发现，应该同时包含 Skill 做什么和何时使用。

始终使用第三人称。description 会被注入 system prompt，视角不一致可能导致发现问题。

Good: "Processes Excel files and generates reports"
Avoid: "I can help you process Excel files"
Avoid: "You can use this to process Excel files"
要具体并包含关键术语。既要写 Skill 做什么，也要写何时触发/在何种上下文使用。

每个 Skill 只有一个 description 字段。description 对 skill 选择至关重要：Claude 需要从潜在 100+ 个 Skills 中选出正确的一个。你的 description 必须给出足够细节，让 Claude 知道何时选择该 Skill；其余 SKILL.md 则提供实现细节。

有效示例：

PDF Processing skill：

description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
Excel Analysis skill：

description: Analyze Excel spreadsheets, create pivot tables, generate charts. Use when analyzing Excel files, spreadsheets, tabular data, or .xlsx files.
Git Commit Helper skill：

description: Generate descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.
避免如下模糊描述：

description: Helps with documents
description: Processes data
description: Does stuff with files
渐进式披露模式
SKILL.md 充当总览，按需引导 Claude 去读取详细资料，就像入门指南里的目录。关于渐进式披露原理，请参见概览中的 How Skills work。

实用建议：

为最佳性能，将 SKILL.md 主体控制在 500 行以内
接近该上限时，把内容拆分到单独文件
使用下述模式组织说明、代码和资源
可视化概览：从简单到复杂
基础 Skill 只需要一个 SKILL.md 文件，包含元数据和说明：

Simple SKILL.md file showing YAML frontmatter and markdown body

随着 Skill 增长，你可以打包附加内容，Claude 仅在需要时加载：

Bundling additional reference files like reference.md and forms.md.

完整 skill 目录结构可能如下：

pdf/
鈹溾攢鈹€ SKILL.md              # Main instructions (loaded when triggered)
鈹溾攢鈹€ FORMS.md              # Form-filling guide (loaded as needed)
鈹溾攢鈹€ reference.md          # API reference (loaded as needed)
鈹溾攢鈹€ examples.md           # Usage examples (loaded as needed)
鈹斺攢鈹€ scripts/
    鈹溾攢鈹€ analyze_form.py   # Utility script (executed, not loaded)
    鈹溾攢鈹€ fill_form.py      # Form filling script
    鈹斺攢鈹€ validate.py       # Validation script
模式 1：高层指南 + 引用
---
name: pdf-processing
description: Extracts text and tables from PDF files, fills forms, and merges documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
---

# PDF Processing

## Quick start

Extract text with pdfplumber:
```python
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

## Advanced features

**Form filling**: See [FORMS.md](FORMS.md) for complete guide
**API reference**: See [REFERENCE.md](REFERENCE.md) for all methods
**Examples**: See [EXAMPLES.md](EXAMPLES.md) for common patterns
Claude 仅在需要时加载 FORMS.md、REFERENCE.md 或 EXAMPLES.md。

模式 2：按领域组织
对于包含多个领域的 Skills，按领域组织内容可避免加载无关上下文。当用户询问销售指标时，Claude 只需读取销售相关 schema，而不必读取财务或营销数据。这能保持 token 使用更低且上下文更聚焦。

bigquery-skill/
鈹溾攢鈹€ SKILL.md (overview and navigation)
鈹斺攢鈹€ reference/
    鈹溾攢鈹€ finance.md (revenue, billing metrics)
    鈹溾攢鈹€ sales.md (opportunities, pipeline)
    鈹溾攢鈹€ product.md (API usage, features)
    鈹斺攢鈹€ marketing.md (campaigns, attribution)
SKILL.md
# BigQuery Data Analysis

## Available datasets

**Finance**: Revenue, ARR, billing 鈫?See [reference/finance.md](reference/finance.md)
**Sales**: Opportunities, pipeline, accounts 鈫?See [reference/sales.md](reference/sales.md)
**Product**: API usage, features, adoption 鈫?See [reference/product.md](reference/product.md)
**Marketing**: Campaigns, attribution, email 鈫?See [reference/marketing.md](reference/marketing.md)

## Quick search

Find specific metrics using grep:

```bash
grep -i "revenue" reference/finance.md
grep -i "pipeline" reference/sales.md
grep -i "api usage" reference/product.md
```
模式 3：条件细节
先展示基础内容，再链接高级内容：

# DOCX Processing

## Creating documents

Use docx-js for new documents. See [DOCX-JS.md](DOCX-JS.md).

## Editing documents

For simple edits, modify the XML directly.

**For tracked changes**: See [REDLINING.md](REDLINING.md)
**For OOXML details**: See [OOXML.md](OOXML.md)
仅当用户需要这些功能时，Claude 才会读取 REDLINING.md 或 OOXML.md。

避免深层嵌套引用
当引用文件再引用其他文件时，Claude 可能只做部分读取。遇到嵌套引用时，Claude 可能会用 `head -100` 这类命令预览内容，而不是读取整个文件，从而导致信息不完整。

将引用深度保持在 SKILL.md 下一层。所有引用文件都应直接从 SKILL.md 链接，以确保 Claude 在需要时读取完整文件。

差示例：层级太深：

# SKILL.md
See [advanced.md](advanced.md)...

# advanced.md
See [details.md](details.md)...

# details.md
Here's the actual information...
好示例：一层深度：

# SKILL.md

**Basic usage**: [instructions in SKILL.md]
**Advanced features**: See [advanced.md](advanced.md)
**API reference**: See [reference.md](reference.md)
**Examples**: See [examples.md](examples.md)
为较长参考文件添加目录
当参考文件超过 100 行时，在顶部加入目录。这可确保 Claude 即便进行部分预览，也能看到完整的信息范围。

示例：

# API Reference

## Contents
- Authentication and setup
- Core methods (create, read, update, delete)
- Advanced features (batch operations, webhooks)
- Error handling patterns
- Code examples

## Authentication and setup
...

## Core methods
...
这样 Claude 就能在需要时读取完整文件或跳转到特定章节。

关于这种基于文件系统架构如何支持渐进式披露的细节，请参见下方 Advanced 章节中的 Runtime environment。

工作流与反馈回路
对复杂任务使用工作流
将复杂操作拆分为清晰的顺序步骤。对于特别复杂的流程，可提供可勾选清单，Claude 可将其复制到响应中并在执行过程中逐项勾选。

示例 1：研究综合工作流（适用于无代码 Skills）：

## Research synthesis workflow

Copy this checklist and track your progress:

```
Research Progress:
- [ ] Step 1: Read all source documents
- [ ] Step 2: Identify key themes
- [ ] Step 3: Cross-reference claims
- [ ] Step 4: Create structured summary
- [ ] Step 5: Verify citations
```

**Step 1: Read all source documents**

Review each document in the `sources/` directory. Note the main arguments and supporting evidence.

**Step 2: Identify key themes**

Look for patterns across sources. What themes appear repeatedly? Where do sources agree or disagree?

**Step 3: Cross-reference claims**

For each major claim, verify it appears in the source material. Note which source supports each point.

**Step 4: Create structured summary**

Organize findings by theme. Include:
- Main claim
- Supporting evidence from sources
- Conflicting viewpoints (if any)

**Step 5: Verify citations**

Check that every claim references the correct source document. If citations are incomplete, return to Step 3.
这个示例展示了工作流如何用于无需代码的分析任务。清单模式适用于任何复杂的多步骤流程。

示例 2：PDF 表单填写工作流（适用于含代码 Skills）：

## PDF form filling workflow

Copy this checklist and check off items as you complete them:

```
Task Progress:
- [ ] Step 1: Analyze the form (run analyze_form.py)
- [ ] Step 2: Create field mapping (edit fields.json)
- [ ] Step 3: Validate mapping (run validate_fields.py)
- [ ] Step 4: Fill the form (run fill_form.py)
- [ ] Step 5: Verify output (run verify_output.py)
```

**Step 1: Analyze the form**

Run: `python scripts/analyze_form.py input.pdf`

This extracts form fields and their locations, saving to `fields.json`.

**Step 2: Create field mapping**

Edit `fields.json` to add values for each field.

**Step 3: Validate mapping**

Run: `python scripts/validate_fields.py fields.json`

Fix any validation errors before continuing.

**Step 4: Fill the form**

Run: `python scripts/fill_form.py input.pdf fields.json output.pdf`

**Step 5: Verify output**

Run: `python scripts/verify_output.py output.pdf`

If verification fails, return to Step 2.
清晰步骤能防止 Claude 跳过关键验证。清单能帮助 Claude 和你共同跟踪多步骤流程进度。

实现反馈回路
常见模式：运行校验器 鈫?修复错误 鈫?重复

这种模式能显著提升输出质量。

示例 1：风格指南合规（适用于无代码 Skills）：

## Content review process

1. Draft your content following the guidelines in STYLE_GUIDE.md
2. Review against the checklist:
   - Check terminology consistency
   - Verify examples follow the standard format
   - Confirm all required sections are present
3. If issues found:
   - Note each issue with specific section reference
   - Revise the content
   - Review the checklist again
4. Only proceed when all requirements are met
5. Finalize and save the document
这个示例展示了用参考文档（而非脚本）实现验证循环。“验证器”是 STYLE_GUIDE.md，而 Claude 通过读取并对照来完成检查。

示例 2：文档编辑流程（适用于含代码 Skills）：

## Document editing process

1. Make your edits to `word/document.xml`
2. **Validate immediately**: `python ooxml/scripts/validate.py unpacked_dir/`
3. If validation fails:
   - Review the error message carefully
   - Fix the issues in the XML
   - Run validation again
4. **Only proceed when validation passes**
5. Rebuild: `python ooxml/scripts/pack.py unpacked_dir/ output.docx`
6. Test the output document
验证循环可以及早发现错误。

内容指南
避免时效性信息
不要包含会很快过时的信息：

差示例：时效性强（会变错）：

If you're doing this before August 2025, use the old API.
After August 2025, use the new API.
好示例（使用 “old patterns” 章节）：

## Current method

Use the v2 API endpoint: `api.example.com/v2/messages`

## Old patterns

<details>
<summary>Legacy v1 API (deprecated 2025-08)</summary>

The v1 API used: `api.example.com/v1/messages`

This endpoint is no longer supported.
</details>
“old patterns” 章节在不干扰主内容的前提下提供历史背景。

使用一致术语
选择一个术语并在整个 Skill 中保持一致：

好 - 一致：

始终使用 “API endpoint”
始终使用 “field”
始终使用 “extract”
差 - 不一致：

混用 “API endpoint”、"URL"、"API route"、"path"
混用 “field”、"box"、"element"、"control"
混用 “extract”、"pull"、"get"、"retrieve"
一致性有助于 Claude 理解并遵循说明。

常见模式
模板模式
为输出格式提供模板。严格程度按你的需求决定。

对于严格要求（如 API 响应或数据格式）：

## Report structure

ALWAYS use this exact template structure:

```markdown
# [Analysis Title]

## Executive summary
[One-paragraph overview of key findings]

## Key findings
- Finding 1 with supporting data
- Finding 2 with supporting data
- Finding 3 with supporting data

## Recommendations
1. Specific actionable recommendation
2. Specific actionable recommendation
```
对于灵活指导（适合按情况调整）：

## Report structure

Here is a sensible default format, but use your best judgment based on the analysis:

```markdown
# [Analysis Title]

## Executive summary
[Overview]

## Key findings
[Adapt sections based on what you discover]

## Recommendations
[Tailor to the specific context]
```

Adjust sections as needed for the specific analysis type.
示例模式
对于输出质量依赖示例的 Skills，像常规提示词一样提供输入/输出对：

## Commit message format

Generate commit messages following these examples:

**Example 1:**
Input: Added user authentication with JWT tokens
Output:
```
feat(auth): implement JWT-based authentication

Add login endpoint and token validation middleware
```

**Example 2:**
Input: Fixed bug where dates displayed incorrectly in reports
Output:
```
fix(reports): correct date formatting in timezone conversion

Use UTC timestamps consistently across report generation
```

**Example 3:**
Input: Updated dependencies and refactored error handling
Output:
```
chore: update dependencies and refactor error handling

- Upgrade lodash to 4.17.21
- Standardize error response format across endpoints
```

Follow this style: type(scope): brief description, then detailed explanation.
示例比纯描述更能让 Claude 清楚理解目标风格和细节层级。

条件工作流模式
引导 Claude 在决策点做选择：

## Document modification workflow

1. Determine the modification type:

   **Creating new content?** 鈫?Follow "Creation workflow" below
   **Editing existing content?** 鈫?Follow "Editing workflow" below

2. Creation workflow:
   - Use docx-js library
   - Build document from scratch
   - Export to .docx format

3. Editing workflow:
   - Unpack existing document
   - Modify XML directly
   - Validate after each change
   - Repack when complete
如果流程步骤很多而且复杂，建议拆到独立文件，并告诉 Claude 基于当前任务读取合适文件。

评估与迭代
先构建评估
在写大量文档之前先创建评估。这能确保你的 Skill 解决真实问题，而不是记录想象中的问题。

评估驱动开发：

识别缺口：让 Claude 在没有 Skill 的情况下执行代表性任务。记录具体失败点或缺失上下文
创建评估：构建三个场景来测试这些缺口
建立基线：测量 Claude 在没有 Skill 时的表现
编写最小说明：只写足以填补缺口并通过评估的内容
迭代：执行评估，与基线比较并优化
这种方式能确保你在解决真实问题，而不是预判可能永远不会出现的需求。

评估结构：

{
  "skills": ["pdf-processing"],
  "query": "Extract all text from this PDF file and save it to output.txt",
  "files": ["test-files/document.pdf"],
  "expected_behavior": [
    "Successfully reads the PDF file using an appropriate PDF processing library or command-line tool",
    "Extracts text content from all pages in the document without missing any pages",
    "Saves the extracted text to a file named output.txt in a clear, readable format"
  ]
}
这个示例展示了数据驱动评估与简单测试评分标准。我们目前不提供内置评估运行器。用户可以自行构建评估系统。评估是衡量 Skill 有效性的事实依据。

与 Claude 迭代开发 Skills
最高效的 Skill 开发流程本身就包含 Claude。你可以与一个 Claude 实例（“Claude A”）协作创建 Skill，再让其他实例（“Claude B”）使用它。Claude A 帮你设计和打磨说明，Claude B 在真实任务中测试它。之所以有效，是因为 Claude 模型既理解如何写好代理说明，也理解代理需要哪些信息。

创建新 Skill：

先在没有 Skill 的情况下完成一次任务：用常规提示词与 Claude A 一起解决问题。过程中你会自然提供上下文、解释偏好、分享流程知识。留意你反复提供了哪些信息。

识别可复用模式：任务完成后，找出你提供过的、对未来类似任务有价值的上下文。

示例：如果你做过一次 BigQuery 分析，你可能提供过表名、字段定义、过滤规则（如“总是排除测试账号”）和常见查询模式。

让 Claude A 创建 Skill：“Create a Skill that captures this BigQuery analysis pattern we just used. Include the table schemas, naming conventions, and the rule about filtering test accounts.”

Claude 模型原生理解 Skill 的格式和结构。你不需要特殊 system prompt，也不需要“写 skill 的 skill”来让 Claude 帮你创建 Skills。直接让 Claude 创建 Skill，它就会生成结构正确、包含合适 frontmatter 和正文的 SKILL.md。

审查简洁性：检查 Claude A 是否加入了不必要解释。比如：“Remove the explanation about what win rate means - Claude already knows that.”

改进信息架构：让 Claude A 以更高效方式组织内容。比如：“Organize this so the table schema is in a separate reference file. We might add more tables later.”

在相似任务上测试：让 Claude B（全新实例并加载该 Skill）处理相关用例。观察 Claude B 是否能找到正确信息、正确应用规则并成功完成任务。

基于观察迭代：如果 Claude B 卡住或遗漏内容，带着具体问题回到 Claude A：“When Claude used this Skill, it forgot to filter by date for Q4. Should we add a section about date filtering patterns?”

迭代已有 Skills：

改进 Skill 时继续使用同样的分层模式。你会在以下环节间切换：

与 Claude A 协作（帮助优化 Skill 的专家）
用 Claude B 测试（使用 Skill 执行真实工作）
观察 Claude B 行为并把洞察反馈给 Claude A
在真实工作流中使用 Skill：给 Claude B（已加载 Skill）真实任务，而不是测试题

观察 Claude B 行为：记录它在哪些地方困难、成功或做出意外选择

观察示例：“When I asked Claude B for a regional sales report, it wrote the query but forgot to filter out test accounts, even though the Skill mentions this rule.”

回到 Claude A 改进：共享当前 SKILL.md 并描述观察结果。比如：“I noticed Claude B forgot to filter test accounts when I asked for a regional report. The Skill mentions filtering, but maybe it's not prominent enough?”

审查 Claude A 建议：Claude A 可能会建议重组内容以突出规则、把 “always filter” 改成 “MUST filter” 等更强措辞，或重构工作流章节。

应用并测试改动：按 Claude A 的优化更新 Skill，然后用 Claude B 在相似请求上再测一次

按使用持续迭代：随着新场景出现，持续进行“观察-优化-测试”循环。每次迭代都基于真实代理行为改进 Skill，而不是基于假设。

收集团队反馈：

与同事共享 Skills 并观察其使用情况
询问：Skill 是否在预期时激活？说明是否清晰？缺了什么？
吸收反馈，补上你自己使用模式中的盲点
为什么这种方法有效：Claude A 理解代理需求、你提供领域知识、Claude B 在真实使用中暴露缺口，而迭代优化让 Skill 基于观察不断提升，而非基于臆测。

观察 Claude 如何浏览 Skills
在迭代 Skills 时，关注 Claude 在实践中的真实使用方式。重点观察：

意外探索路径：Claude 是否按你未预期的顺序读取文件？这可能说明结构不够直观
遗漏连接：Claude 是否没有跟进重要文件的引用？你的链接可能需要更显式或更醒目
对某些部分过度依赖：如果 Claude 重复读取同一文件，考虑该内容是否应放入主 SKILL.md
内容被忽略：如果 Claude 从不访问某个打包文件，它可能不必要，或在主说明中信号不够明确
基于这些观察迭代，而不是凭假设调整。Skill 元数据里的 `name` 和 `description` 尤其关键。Claude 依赖它们决定是否对当前任务触发该 Skill。确保它们清楚描述 Skill 做什么以及应在何时使用。

应避免的反模式
避免 Windows 风格路径
即使在 Windows 上，也始终使用正斜杠：

鉁?Good: scripts/helper.py, reference/guide.md
鉁?Avoid: scripts\helper.py, reference\guide.md
Unix 风格路径可跨平台使用，而 Windows 风格路径会在 Unix 系统报错。

避免给出过多选项
除非必要，不要同时提供多个做法：

**Bad example: Too many choices** (confusing):
"You can use pypdf, or pdfplumber, or PyMuPDF, or pdf2image, or..."

**Good example: Provide a default** (with escape hatch):
"Use pdfplumber for text extraction:
```python
import pdfplumber
```

For scanned PDFs requiring OCR, use pdf2image with pytesseract instead."
高级：带可执行代码的 Skills
下面章节聚焦包含可执行脚本的 Skills。如果你的 Skill 只用 markdown 指令，可跳到 Checklist for effective Skills。

解决问题，不要甩锅
为 Skills 编写脚本时，应处理错误情况，而不是把问题推给 Claude。

好示例：显式处理错误：

def process_file(path):
    """Process a file, creating it if it doesn't exist."""
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError:
        # Create file with default content instead of failing
        print(f"File {path} not found, creating default")
        with open(path, 'w') as f:
            f.write('')
        return ''
    except PermissionError:
        # Provide alternative instead of failing
        print(f"Cannot access {path}, using default")
        return ''
差示例：把问题甩给 Claude：

def process_file(path):
    # Just fail and let Claude figure it out
    return open(path).read()
配置参数也应有依据并写清楚，避免 “voodoo constants”（Ousterhout 定律）。如果你都不知道正确值，Claude 如何确定？

好示例：自解释：

# HTTP requests typically complete within 30 seconds
# Longer timeout accounts for slow connections
REQUEST_TIMEOUT = 30

# Three retries balances reliability vs speed
# Most intermittent failures resolve by the second retry
MAX_RETRIES = 3
差示例：魔法数字：

TIMEOUT = 47  # Why 47?
RETRIES = 5   # Why 5?
提供实用脚本
即使 Claude 能写脚本，预制脚本仍有优势：

实用脚本的好处：

比生成代码更可靠
节省 tokens（无需把代码放进上下文）
节省时间（无需生成代码）
确保跨场景一致性
将可执行脚本与说明文件一起打包

上图展示了可执行脚本如何与说明文件协同。说明文件（forms.md）引用脚本，Claude 无需加载脚本内容到上下文即可执行。

关键区别：你应在说明中明确 Claude 是要：

执行脚本（最常见）："Run analyze_form.py to extract fields"
将其作为参考阅读（复杂逻辑）："See analyze_form.py for the field extraction algorithm"
对大多数实用脚本，优先执行更可靠也更高效。关于脚本执行机制，见下方 Runtime environment 章节。

示例：

## Utility scripts

**analyze_form.py**: Extract all form fields from PDF

```bash
python scripts/analyze_form.py input.pdf > fields.json
```

Output format:
```json
{
  "field_name": {"type": "text", "x": 100, "y": 200},
  "signature": {"type": "sig", "x": 150, "y": 500}
}
```

**validate_boxes.py**: Check for overlapping bounding boxes

```bash
python scripts/validate_boxes.py fields.json
# Returns: "OK" or lists conflicts
```

**fill_form.py**: Apply field values to PDF

```bash
python scripts/fill_form.py input.pdf fields.json output.pdf
```
使用视觉分析
当输入可渲染为图像时，让 Claude 进行视觉分析：

## Form layout analysis

1. Convert PDF to images:
   ```bash
   python scripts/pdf_to_images.py form.pdf
   ```

2. Analyze each page image to identify form fields
3. Claude can see field locations and types visually
在这个示例中，你需要编写 pdf_to_images.py 脚本。

Claude 的视觉能力有助于理解版式和结构。

创建可验证的中间输出
当 Claude 执行复杂且开放式任务时，可能会犯错。“plan-validate-execute” 模式通过先生成结构化计划、再用脚本验证计划、最后执行，来尽早捕捉错误。

示例：设想你让 Claude 根据电子表格更新 PDF 中 50 个表单字段。若无验证，Claude 可能引用不存在字段、创建冲突值、遗漏必填字段或错误应用更新。

解决方案：使用上述工作流模式（PDF 表单填写），但增加一个在应用前要验证的中间文件 changes.json。流程变为：analyze 鈫?create plan file 鈫?validate plan 鈫?execute 鈫?verify。

这种模式为何有效：

及早捕获错误：在应用变更前发现问题
机器可验证：脚本提供客观验证
可逆规划：Claude 可在不改动原件的情况下迭代计划
调试清晰：错误信息能定位具体问题
适用场景：批量操作、破坏性变更、复杂验证规则、高风险操作。

实现建议：让验证脚本输出详细、具体的错误信息，例如 "Field 'signature_date' not found. Available fields: customer_name, order_total, signature_date_signed"，帮助 Claude 快速修复。

依赖打包
Skills 运行在代码执行环境中，受平台限制：

claude.ai：可从 npm 和 PyPI 安装包，也可拉取 GitHub 仓库
Anthropic API：无网络访问，且不支持运行时安装包
在 SKILL.md 中列出所需依赖，并确认它们在代码执行工具文档中可用。

运行时环境
Skills 运行在具备文件系统访问、bash 命令和代码执行能力的环境中。关于该架构的概念说明，请参见概览中的 The Skills architecture。

这对编写方式的影响：

Claude 如何访问 Skills：

元数据预加载：启动时，所有 Skills 的 YAML frontmatter 中的 name 和 description 会被加载到 system prompt
按需读文件：Claude 通过 bash Read 工具在需要时读取文件系统中的 SKILL.md 及其他文件
高效执行脚本：实用脚本可通过 bash 直接执行，无需把完整脚本加载到上下文。只有脚本输出消耗 tokens
大文件无上下文惩罚：参考文件、数据或文档在实际读取前不占用上下文 tokens
路径很重要：Claude 以文件系统方式浏览 skill 目录。使用正斜杠（reference/guide.md），不要反斜杠
文件名要有描述性：使用能体现内容的名称，如 form_validation_rules.md，而不是 doc2.md
按发现性组织：按领域或功能组织目录
Good: reference/finance.md, reference/sales.md
Bad: docs/file1.md, docs/file2.md
可打包全面资源：完整 API 文档、大量示例、大数据集；在读取前不产生上下文成本
确定性操作优先脚本：写 validate_form.py，而不是让 Claude 现场生成验证代码
明确执行意图：
"Run analyze_form.py to extract fields"（执行）
"See analyze_form.py for the extraction algorithm"（作为参考阅读）
测试文件访问模式：通过真实请求验证 Claude 能否正确浏览你的目录结构
示例：

bigquery-skill/
鈹溾攢鈹€ SKILL.md (overview, points to reference files)
鈹斺攢鈹€ reference/
    鈹溾攢鈹€ finance.md (revenue metrics)
    鈹溾攢鈹€ sales.md (pipeline data)
    鈹斺攢鈹€ product.md (usage analytics)
当用户询问收入时，Claude 会读取 SKILL.md，看到对 reference/finance.md 的引用，并调用 bash 仅读取该文件。sales.md 和 product.md 会继续留在文件系统中，在需要前消耗 0 上下文 tokens。这种基于文件系统的模型正是渐进式披露可行的原因。Claude 可以导航并有选择地加载每个任务真正需要的内容。

关于技术架构完整细节，请参见 Skills overview 中的 How Skills work。

MCP 工具引用
如果你的 Skill 使用 MCP（Model Context Protocol）工具，始终使用全限定工具名，避免 “tool not found” 错误。

格式：ServerName:tool_name

示例：

Use the BigQuery:bigquery_schema tool to retrieve table schemas.
Use the GitHub:create_issue tool to create issues.
其中：

BigQuery 和 GitHub 是 MCP 服务器名
bigquery_schema 和 create_issue 是这些服务器中的工具名
如果没有服务器前缀，Claude 可能无法定位工具，尤其在有多个 MCP 服务器时。

避免假设工具已安装
不要假设包已经可用：

**Bad example: Assumes installation**:
"Use the pdf library to process the file."

**Good example: Explicit about dependencies**:
"Install required package: `pip install pypdf`

Then use it:
```python
from pypdf import PdfReader
reader = PdfReader("file.pdf")
```"
技术说明
YAML frontmatter 要求
SKILL.md frontmatter 要求 name 和 description 字段，并遵循特定校验规则：

name: 最大 64 字符，仅小写字母/数字/连字符，不含 XML 标签，不含保留词
description: 最大 1024 字符，非空，不含 XML 标签
完整结构细节请参见 Skills overview。

Token 预算
为了最佳性能，将 SKILL.md 主体控制在 500 行以内。若超出此范围，按前述渐进式披露模式拆到多个文件。架构细节请参见 Skills overview。

高质量 Skills 检查清单
在分享 Skill 前，确认：

核心质量
 Description is specific and includes key terms
 Description includes both what the Skill does and when to use it
 SKILL.md body is under 500 lines
 Additional details are in separate files (if needed)
 No time-sensitive information (or in "old patterns" section)
 Consistent terminology throughout
 Examples are concrete, not abstract
 File references are one level deep
 Progressive disclosure used appropriately
 Workflows have clear steps
代码与脚本
 Scripts solve problems rather than punt to Claude
 Error handling is explicit and helpful
 No "voodoo constants" (all values justified)
 Required packages listed in instructions and verified as available
 Scripts have clear documentation
 No Windows-style paths (all forward slashes)
 Validation/verification steps for critical operations
 Feedback loops included for quality-critical tasks
测试
 At least three evaluations created
 Tested with Haiku, Sonnet, and Opus
 Tested with real usage scenarios
 Team feedback incorporated (if applicable)


 https://github.com/anthropics/skills

