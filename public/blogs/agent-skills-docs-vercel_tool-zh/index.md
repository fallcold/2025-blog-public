> ?????<https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering>
> ?????`docs/vercel_tool.md`
我们移除了代理 80% 的工具

Andrew Qu
Vercel 软件负责人
4 分钟阅读


复制 URL
已复制到剪贴板！
2025 年 12 月 22 日
结果更好了。

我们花了几个月构建一个复杂的内部 text-to-SQL 代理 d0，配备专用工具、重度提示词工程和精细的上下文管理。它能工作……但只是某种程度上。它很脆弱、很慢，而且需要持续维护。

所以我们尝试了不同的方法。我们删除了大部分内容，把代理精简成一个工具：执行任意 bash 命令。我们称之为文件系统代理。Claude 直接访问你的文件，并通过 grep、cat、ls 自行搞清楚问题。

代理同时变得更简单、更好用。成功率从 80% 提升到 100%。步骤更少、token 更少、响应更快。通过“少做”，反而做得更好。

Link to heading什么是 d0
如果 v0 是我们用于构建 UI 的 AI，那么 d0 就是我们用于理解数据的 AI。

d0 enables anyone to make data-driven decisions by asking it questions in Slack
d0 enables anyone to make data-driven decisions by asking it questions in Slack
d0 会把自然语言问题翻译成针对我们分析基础设施的 SQL 查询，让团队中任何人都能在无需写代码、无需等待数据团队的情况下获得答案。

当 d0 运行良好时，它能让全公司范围的数据访问民主化。它一旦出问题，人们就会失去信任，回到 Slack 里 ping 分析师。我们需要 d0 既快、又准、还可靠。

Link to heading不要挡住模型的路
回头看，我们当时在解决模型本可以自己处理的问题。我们假设它会在复杂 schema 里迷路、做出错误 join、或者幻觉出表名。所以我们构建了护栏。我们预过滤上下文、限制它的可选项，并为每次交互包上校验逻辑。我们在替模型思考：

构建了多个专用工具（schema lookup、query validation、error recovery 等）

加入重度提示词工程来约束推理

使用精细的上下文管理来避免模型被信息淹没

手写检索逻辑，以呈现“相关”的 schema 信息和维度属性

每一个边界情况都意味着再打一块补丁，而每一次模型更新都意味着重新校准我们的约束。我们花在维护脚手架上的时间，比改进代理本身还多。

ai-sdk@6.0.0-beta.160 ToolLoopAgent

import { ToolLoopAgent } from 'ai';
import { GetEntityJoins, LoadCatalog, /*...*/ } from '@/lib/tools'
const agent = new ToolLoopAgent({
  model: "anthropic/claude-opus-4.5",
  instructions: "",
  tools: {
      GetEntityJoins, LoadCatalog, RecallContext, LoadEntityDetails, 
      SearchCatalog, ClarifyIntent, SearchSchema, GenerateAnalysisPlan, 
      FinalizeQueryPlan, FinalizeNoData, JoinPathFinder, SyntaxValidator, 
      FinalizeBuild, ExecuteSQL, FormatResults, VisualizeData, ExplainResults
    },
});
Link to heading一个新想法：如果我们干脆……停下来？
我们意识到自己在对抗重力。我们在约束模型的推理；在总结模型本可自己读取的信息；在构建工具以保护它免于面对它本可处理的复杂性。

所以我们停下来了。我们的假设是：如果直接给 Claude 访问原始 Cube DSL 文件并让它自由发挥会怎样？如果你真正需要的只有 bash，会怎样？模型越来越聪明，上下文窗口越来越大，也许最好的代理架构几乎就是“没有架构”。

Link to headingv2：文件系统就是代理
新栈如下：

模型：通过 AI SDK 使用 Claude Opus 4.5

执行：Vercel Sandbox 用于上下文探索

路由：Vercel Gateway 用于请求处理和可观测性

服务端：使用 Vercel Slack Bolt 的 Next.js API 路由

数据层：Cube 语义层，表现为由 YAML、Markdown 和 JSON 文件组成的目录

文件系统代理现在会像人类分析师一样浏览我们的语义层。它读取文件、grep 模式、建立心智模型，并使用 grep、cat、find、ls 等标准 Unix 工具写出 SQL。

这之所以可行，是因为语义层本身已经是很好的文档。文件里包含维度定义、指标计算和 join 关系。我们之前在构建工具去总结本已清晰可读的内容。Claude 只需要直接读取它们的权限。

ai-sdk@6.0.0-beta.160 ToolLoopAgent

import { Sandbox } from "@vercel/sandbox";
import { files } from './semantic-catalog'
import { tool, ToolLoopAgent } from "ai";
import { ExecuteSQL } from "@/lib/tools";}

const sandbox = await Sandbox.create();
await sandbox.writeFiles(files);

const executeCommandTool(sandbox: Sandbox) {
  return tool({
    /* ... */
    execute: async ({ command }) => {
      const result = await sandbox.exec(command);
      return { /* */ };
    }
  })
}

const agent = new ToolLoopAgent({
  model: "anthropic/claude-opus-4.5",
  instructions: "",
  tools: {
    ExecuteCommand: executeCommandTool(sandbox),
    ExecuteSQL,
   },
})
Link to heading快 3.5 倍、token 少 37%、成功率 100%
我们在 5 个具有代表性的查询上，对比了旧架构和新的文件系统方案。

指标	高级（旧）	文件系统（新）	变化
平均执行时间	274.8s	77.4s	快 3.5 倍
成功率	4/5 (80%)	5/5 (100%)	+20%
平均 token 用量	~102k tokens	~61k tokens	token 减少 37%
平均步骤数	~12 steps	~7 steps	步骤减少 42%
文件系统代理在所有对比项上都获胜。旧架构最差的案例是 Query 2：在失败前耗时 724 秒、执行 100 步、使用 145,463 tokens。文件系统代理完成同一查询只用了 141 秒、19 步、67,483 tokens，而且确实成功了。

这种质变同样重要。代理能捕捉到我们从未预料的边界情况，并以我们能跟得上的方式解释它的推理过程。

Link to heading经验总结
不要对抗重力。文件系统是一种极其强大的抽象。Grep 已经 50 年历史，但仍然完全满足我们的需求。我们在为 Unix 已经解决的问题构建自定义工具。

我们之前约束推理，是因为不信任模型会推理。到了 Opus 4.5，这种约束反而变成了负担。当我们不再替模型做选择时，模型会做出更好的选择。

这之所以奏效，只因为我们的语义层本身已经是高质量文档。YAML 文件结构良好、命名一致、定义清晰。若你的数据层充满历史遗留命名和缺乏文档的 join 关系，给 Claude 原始文件访问权限并不能拯救你。你只会得到更快但同样糟糕的查询。

通过减法实现加法，确实成立。最好的代理，可能是工具最少的代理。每一个工具，都是你替模型做的一次选择。有时模型会做得更好。

Link to heading这对代理构建者意味着什么
诱惑总是去覆盖每一种可能。请克制。先从尽可能简单的架构开始：模型 + 文件系统 + 目标。只有在你证明复杂性确有必要时，再增加复杂度。

但仅有简单架构还不够。模型需要可用的高质量上下文。投入到文档、清晰命名和结构化良好的数据上。这些基础比花哨的工具链更重要。

模型进步的速度，比你的工具链追赶得更快。要为六个月后的模型构建，而不是只为今天的模型构建。

如果你正在构建代理，我们很想听听你学到了什么。

