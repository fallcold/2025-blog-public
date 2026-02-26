> ?????<https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering>
> ?????`docs/compression.md`
评估 AI 代理的上下文压缩
By Factory Research - 2025年12月16日 - 10 分钟阅读 -

分享





工程

研究

新

我们构建了一个评估框架，用于衡量不同压缩策略能保留多少上下文信息。我们在真实世界的长时运行代理会话（涵盖调试、代码审查与功能实现）上测试了三种方法，结果发现结构化摘要比 OpenAI 和 Anthropic 的替代方案能保留更多有用信息。

目录










01 问题


02 如何衡量上下文质量


03 三种压缩方法


04 一个具体示例


05 LLM 评审器如何工作


06 结果


07 我们的收获


08 方法细节


09 附录：LLM 评审器提示词与评分细则

风格克制的抽象插图，唤起记忆与模糊感
当一个 AI 代理在数百条消息中帮助你处理复杂任务时，它内存耗尽后会发生什么？答案决定了你的代理是能继续高效推进，还是开始问“等等，我们刚才到底在做什么来着？”。

我们构建了一个评估框架，用于衡量不同压缩策略能保留多少上下文信息。我们在真实世界的长时运行代理会话（调试、PR 审查、功能实现、CI 故障排查、数据科学、机器学习研究）上测试了三种方法，结果发现结构化摘要在不牺牲压缩效率的前提下，比 OpenAI 和 Anthropic 的替代方法保留了更多有用信息。

柱状图：对比 Factory、OpenAI、Anthropic 在各维度的质量得分
本文将介绍这个问题、我们的方法、不同方法表现的具体示例，以及这些结果对构建可靠 AI 代理意味着什么。

问题
长时运行的代理会话会生成数百万 token 的对话历史，这远超任何模型可在工作内存中容纳的范围。

最朴素的方案是激进压缩：把所有内容挤进尽可能小的摘要里。但这会提高代理忘记已修改文件、或忘记已尝试方案的概率。它很可能浪费 token 去重复读取文件、重复探索死胡同。

正确的优化目标不是每次请求的 token，而是每个任务的 token。

如何衡量上下文质量
传统指标如 ROUGE 或 embedding 相似度，无法告诉你代理在压缩后能否继续有效工作。一个摘要可能词面重合度很高，却漏掉了代理继续推进最需要的那个文件路径。

我们设计了基于探针（probe）的评估，直接衡量功能质量。思路很简单：压缩后，向代理提问，这些问题要求它记住被截断历史中的具体细节。如果压缩保留了正确信息，代理就能答对；否则就会猜测或幻觉。

我们使用四类探针：

Probe type	它测试什么	示例问题
Recall	事实保留	“最初的报错信息是什么？”
Artifact	文件追踪	“我们改过哪些文件？请描述每个文件改了什么。”
Continuation	任务衔接	“接下来我们该做什么？”
Decision	推理链	“我们讨论了 Redis 问题的几个选项。最后决定了什么？”
Recall 探针测试具体事实是否在压缩后仍然存在。Artifact 探针测试代理是否知道它触达过哪些文件。Continuation 探针测试代理能否从中断处继续。Decision 探针测试过往选择背后的推理是否被保留。

我们使用 LLM 评审器（GPT-5.2）在六个维度上给回答打分：

Dimension	它衡量什么
Accuracy	技术细节是否正确？文件路径、函数名、错误信息
Context awareness	回答是否反映当前会话状态？
Artifact trail	代理是否知道哪些文件被读取或修改？
Completeness	回答是否覆盖问题的所有部分？
Continuity	工作是否可以在不重新获取信息的情况下继续？
Instruction following	回答是否遵循探针要求的格式？
每个维度按 0-5 分打分，使用详细评分细则。细则明确了每项标准下什么是 0 分（“完全不满足”）、3 分（“基本满足，有轻微问题”）、5 分（“表现优秀，无问题”）。

为什么这些维度对软件开发很重要
这些维度是专门挑选的，因为它们准确覆盖了编码代理丢失上下文时会出的问题：

Artifact trail 至关重要，因为编码代理需要知道自己碰过哪些文件。没有这点，代理可能会重复读取已检查文件、做出互相冲突的改动，或丢失测试结果。ChatGPT 聊天忘掉早先话题还能接受；编码代理如果忘了自己改过 `auth.controller.ts`，就会产出不一致结果。

Continuity 直接影响 token 效率。当代理无法从中断点继续，就会重复获取文件并重试已经试过的方法。这会浪费 token 和时间，把一次通过的任务变成昂贵的多轮任务。

Context awareness 重要，是因为编码会话有状态。代理不仅要知道过去的事实，还要知道当前任务状态：试过什么、哪里失败、还剩什么。通用摘要常能保留“发生了什么”，却丢掉“我们现在在哪一步”。

Accuracy 对代码场景不可妥协。错误的文件路径或记错的函数名会导致编辑失败或产生幻觉方案。与近似召回可接受的对话 AI 不同，编码代理需要精确技术细节。

Completeness 确保代理能处理多部分请求的全部内容。当用户要求“修复 bug 并补测试”时，完整回答应同时覆盖两者。不完整回答会迫使用户追问，并在重建上下文上浪费 token。

Instruction following 用来验证代理是否遵守约束和格式。当要求“只修改 auth 模块”或“按 JSON 输出”时，代理必须遵循。这个维度能抓到一种情况：事实保留了，但用户要求丢了。

三种压缩方法
我们比较了三种可用于生产环境的压缩策略。

Factory 维护一个结构化的持久摘要，针对不同信息类型设有明确分区：会话意图、文件修改、已做决策、下一步动作。触发压缩时，只对新被截断的片段做摘要，再与现有摘要合并。我们称之为锚定式迭代摘要（anchored iterative summarization）。

关键洞察是：结构会强制保留信息。通过给特定信息类型分配专门分区，摘要就无法悄悄丢掉文件路径或跳过决策。每个分区都像一份清单：摘要器要么填充，要么明确留空。这避免了自由式摘要中逐步发生的信息流失。

OpenAI 提供 `/responses/compact` 端点，可生成不透明的压缩表示，并针对重建保真度优化。它实现了最高压缩率（99.3%），但牺牲了解释性。你无法直接阅读压缩输出去验证保留了什么。

Anthropic 通过 Claude SDK 提供内置上下文压缩。它会产出详细、结构化的摘要（通常 7-12k 字符），包含分析、文件、待办任务和当前状态等分区。与 Factory 的关键区别在更新机制：Anthropic 每次压缩都会重新生成完整摘要，而 Factory 的锚定方法是把新信息增量合并进持久摘要。这会影响跨多次压缩的一致性，以及哪些细节能在重复压缩循环中存活。

一个具体示例
考虑一个调试会话：用户报告 `/api/auth/login` 端点在凭据有效时仍返回 401 错误。经过 178 条消息、89,000 token，助手执行了：

读取认证控制器并定位 JWT token 生成逻辑
检查中间件链并发现 CORS 策略配置错误
发现 session store 使用了过期的 Redis 连接
修复 Redis 连接池配置
为瞬时连接失败添加重试逻辑
运行测试套件（14 个通过，2 个失败）
修复与 mock 配置相关的剩余测试失败
检查过的文件：auth.controller.ts、middleware/cors.ts、config/redis.ts、services/session.service.ts、tests/auth.test.ts

压缩后，我们提问：“继续之前，你能提醒我这个调试会话最初的错误是什么吗？请明确说明错误码和端点。”

下面是三种方法的回答：

Factory response (scored 4.8/5):

“最初的问题是 `/api/auth/login` 端点返回 401 Unauthorized。即使用户提供了有效的用户名/密码组合，也会收到这个错误。根因最终是 session store 中存在陈旧的 Redis 连接 - JWT 实际生成正确，但 session 无法持久化，导致后续已认证请求失败。”

Anthropic response (scored 3.9/5):

“会话开始于排查登录失败 - 具体是认证端点上的 401 错误。我们沿着认证流程追踪问题，发现它与 session 存储有关，而不是凭据校验本身。”

OpenAI response (scored 3.2/5):

“我们当时在调试一个认证问题。用户登录失败。我们查看了几个文件，发现有一些配置问题需要修复。”

Factory 的回答给出了精确端点（`/api/auth/login`）、错误码（401）和根因（Redis session store）。Anthropic 的回答抓住了错误码和大致原因，但丢失了端点路径。OpenAI 的回答则丢失了几乎所有技术细节。

这一模式在各类探针中重复出现。在 Artifact 探针（“我们修改了哪些文件？”）上，Factory 得分 3.6，而 OpenAI 为 2.8。Factory 的摘要在专门分区中明确列出文件。OpenAI 的压缩则把文件路径当作低熵内容丢弃。

LLM 评审器如何工作
我们使用 GPT-5.2 作为 LLM 评审器，遵循 Zheng 等人在 2023 年 MT-Bench 论文中建立的方法。该研究显示，GPT-4 与人类偏好的一致率超过 80%，与人类彼此之间的一致水平相当。

评审器会接收探针问题、模型回答、压缩后的对话上下文，以及（如果可用）标准答案。然后它会对每项评分标准给出带明确理由的评分。

以下是上面 Factory 回答的一个精简评审输出示例：

{
  "criterionResults": [
    {
      "criterionId": "accuracy_factual",
      "score": 5,
      "reasoning": "Response correctly identifies the 401 error, the specific endpoint (/api/auth/login), and the root cause (Redis connection issue)."
    },
    {
      "criterionId": "accuracy_technical",
      "score": 5,
      "reasoning": "Technical details are accurate - JWT generation, session persistence, and the causal chain are correctly described."
    },
    {
      "criterionId": "context_artifact_state",
      "score": 4,
      "reasoning": "Response demonstrates awareness of the debugging journey but does not enumerate all files examined."
    },
    {
      "criterionId": "completeness_coverage",
      "score": 5,
      "reasoning": "Fully addresses the probe question with the error code, endpoint, symptom, and root cause."
    }
  ],
  "aggregateScore": 4.8
}

评审器不知道回答是由哪种压缩方法生成的。它只根据评分细则，按回答质量进行评估。

结果
我们在覆盖 PR 审查、测试、缺陷修复、功能实现和重构的生产会话中，对超过 36,000 条消息评估了三种方法。对于每个压缩点，我们为每种方法生成 4 个探针回答，并在 6 个维度上评分。

Method	Overall	Accuracy	Context	Artifact	Complete	Continuity	Instruction
Factory	3.70	4.04	4.01	2.45	4.44	3.80	4.99
Anthropic	3.44	3.74	3.56	2.33	4.37	3.67	4.95
OpenAI	3.35	3.43	3.64	2.19	4.37	3.77	4.92
Factory 的总体得分比 OpenAI 高 0.35 分，比 Anthropic 高 0.26 分。

雷达图：三种方法在所有维度上的质量画像对比
分维度来看：

Accuracy 差距最大。Factory 为 4.04，Anthropic 为 3.74，OpenAI 为 3.43。Factory 与 OpenAI 相差 0.61 分，反映的是文件路径、错误信息等技术细节在压缩后存活的频率差异。

Context awareness 上，Factory（4.01）优于 Anthropic（3.56），差距 0.45 分。两者都包含当前状态的结构化分区。Factory 的优势来自锚定式迭代方法：它把新摘要合并进持久状态，而不是每次从头重生完整摘要，因此关键细节在多轮压缩中更不易漂移或消失。

Artifact trail 是所有方法中最弱的维度，范围仅 2.19 到 2.45。即便是 Factory 的结构化方法，在长会话中也难以维持完整文件追踪。这表明工件保留需要超越通用摘要的专项机制。

Completeness 与 Instruction following 的差异较小。所有方法都能给出覆盖问题且格式合规的回答。差异主要体现在内容质量，而不是结构形式。

水平条形图：Factory 在各维度的质量优势对比并排图：token 压缩效率与摘要质量
压缩率呈现出一个有意思的结论。OpenAI 压缩到 99.3%（移除 99.3% token），Anthropic 到 98.7%，Factory 到 98.6%。Factory 比 OpenAI 多保留约 0.7% token，但换来 0.35 分质量提升。对于任何重取成本显著的任务，这个权衡都更偏向 Factory。

我们的收获
最出乎意料的是结构的重要性。通用摘要把所有内容都视为同等可压缩。从信息论角度看，文件路径可能是“低熵”信息，但它恰恰是代理继续工作的关键。通过强制摘要器填写文件、决策和下一步的显式分区，Factory 的格式避免了“每次从头生成摘要”时出现的静默漂移。

压缩率本身最终证明是错误指标。OpenAI 可达 99.3% 压缩，但质量低 0.35 分。这些丢失细节最终需要重取，代价可能超过节省的 token。真正重要的是完成一个任务的总 token，而不是单次请求的 token。

Artifact 追踪仍是未解问题。所有方法在“知道哪些文件被创建、修改或检查”上都只得到 2.19 到 2.45/5.0。即使有显式文件分区，Factory 也只有 2.45。这个问题可能需要超出摘要本身的专门机制：例如独立的 artifact 索引，或在代理脚手架中显式追踪文件状态。

最后，基于探针的评估抓住了传统指标遗漏的东西。ROUGE 衡量摘要间的词面相似度；我们的方法衡量摘要是否真的支持任务继续推进。对于 agentic 工作流，这个区别非常关键。

方法细节
数据集：在 36,611 条消息上采样了数百个压缩点。会话来自生产软件工程场景，覆盖真实代码库，来源于加入专项研究计划的用户。

探针生成：对每个压缩点，我们基于被截断的会话历史生成四个探针（recall、artifact、continuation、decision）。探针会引用压缩前上下文中的具体事实、文件与决策。

压缩执行：在每个压缩点，我们对完全一致的会话前缀应用三种方法。Factory 摘要来自生产环境；OpenAI 与 Anthropic 摘要则通过向各自 API 输入同一前缀生成。

评分：GPT-5.2 按六个评分维度对每个探针回答打分。每个维度有 2-3 条标准并配有明确评分指引。我们把维度分数计算为标准的加权平均，总分计算为各维度的不加权平均。

统计说明：我们报告的差异（0.26-0.35 分）在任务类型和会话长度上都保持一致。无论看短会话还是长会话、调试任务还是功能实现，模式都成立。

附录：LLM 评审器提示词与评分细则
由于 LLM 评审器是本评估的核心，我们在此给出完整提示词与评分细则。

System Prompt
评审器接收以下 system prompt：

你是一名专家评估员，负责评估软件开发对话中 AI 助手的回答。

你的任务是按照指定评分细则为回答打分。对于每条标准：
1. 仔细阅读标准问题
2. 在回答中查找证据
3. 根据评分指引给出 0-5 分
4. 为你的分数提供简要理由

请保持客观一致。聚焦回答中实际出现的内容，而不是本可以包含但未包含的内容。

Rubric Criteria
每个维度包含 2-3 条标准。以下是关键标准及其评分指引：

Accuracy

Criterion	Question	0	3	5
accuracy_factual	事实、文件路径和技术细节是否正确？	完全错误或捏造	大体准确，存在轻微错误	完全准确
accuracy_technical	代码引用和技术概念是否正确？	存在严重技术错误	总体正确，有轻微问题	技术上精确无误
Context Awareness

Criterion	Question	0	3	5
context_conversation_state	回答是否反映当前会话状态？	对先前上下文毫无感知	有总体感知但有缺口	对会话历史有完整感知
context_artifact_state	回答是否反映访问过哪些文件/工件？	对工件状态毫无感知	对工件有部分感知	对工件状态完整感知
Artifact Trail Integrity

Criterion	Question	0	3	5
artifact_files_created	代理是否知道创建了哪些文件？	完全不知道	知道大多数文件	完全知道
artifact_files_modified	代理是否知道修改了哪些文件以及改了什么？	完全不知道	对大多数修改有良好认知	对所有修改完全掌握
artifact_key_details	代理是否记得函数名、变量名、错误信息？	无法回忆	能回忆大多数关键细节	完美回忆
Continuity Preservation

Criterion	Question	0	3	5
continuity_work_state	代理是否能在不重新获取已访问信息的情况下继续？	不重取全部上下文就无法继续	只需最少重取即可继续	可无缝继续
continuity_todo_state	代理是否保持对待办任务的认知？	完全丢失所有 TODO	认知良好但有少量缺口	对任务状态认知完美
continuity_reasoning	代理是否保留了先前决策背后的理由？	完全不记得推理	总体记得推理	推理保留优秀
Completeness

Criterion	Question	0	3	5
completeness_coverage	回答是否覆盖问题所有部分？	忽略大部分内容	覆盖大部分内容	全面覆盖全部内容
completeness_depth	是否提供了足够细节？	细节浅薄或缺失	细节足够	细节全面
Instruction Following

Criterion	Question	0	3	5
instruction_format	回答是否遵循要求格式？	忽略格式	基本遵循格式	完全遵循格式
instruction_constraints	回答是否遵守给定约束？	忽略约束	大多遵守约束	完全遵守全部约束
Grading Process
对于每个探针回答，评审器会：

接收探针问题、模型回答与压缩后上下文
按该探针类型对应的评分细则逐条评估
输出结构化 JSON，包含每条标准的分数和理由
将维度分数计算为标准加权平均
将总分计算为各维度不加权平均
评审器不知道被评估回答来自哪种压缩方法。
