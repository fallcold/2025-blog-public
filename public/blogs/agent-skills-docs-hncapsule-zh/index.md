> ?????<https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering>
> ?????`docs/hncapsule.md`
https://karpathy.bearblog.dev/auto-grade-hn/
- 我的博客文章里有更多细节 https://karpathy.bearblog.dev/auto-grade-hn/
- 如果你想亲自玩一玩，这里是项目的 GitHub 仓库 https://github.com/karpathy/hn-time-capsule
- 这是实际结果页面，供你阅读 https://karpathy.ai/hncapsule/


karpathy
首页 博客

用事后视角自动评分十年前的 Hacker News 讨论
2025 年 12 月 10 日

hnhero

TLDR: https://karpathy.ai/hncapsule/

昨天我偶然看到这个 HN 线程 Show HN: Gemini Pro 3 hallucinates the HN front page 10 years from now，其中 Gemini 3 在“幻想”十年后的 HN 首页。不过有一条评论更触动我一些: Bjartr 链接到了整整 10 年前的 HN 首页，也就是 2015 年 12 月。我在阅读十年前的这些讨论，并在心里给它们的前瞻性打分时，意识到 LLM 其实可能在这件事上更强。我手动把一个文章+评论线程复制粘贴到 ChatGPT 5.1 Thinking 里，它给了我一份很漂亮的分析：人们当时怎么想，以及回头看实际发生了什么，甚至比我手动做的更好、细节也明显更多。我意识到这个任务其实非常适合 LLM，而我也正想找个理由用刚发布的 Opus 4.5 做点 vibe coding，于是就动手了。我的计划是拿到 12 月所有首页（31 天，每天 30 篇文章），让 ChatGPT 5.1 Thinking 做分析，并以适合历史回顾阅读的方式把一切展示出来。

我认为这项练习在更宏观层面有两个有趣的原因：

我相信，只要训练并投入努力，培养你的前向未来预测能力是完全可能且值得做的。
这也再次让我想到我那条推文：“Be good, future LLMs are watching”。这句话可以有很多解读，但这里我想聚焦在“未来的 LLM 正在看着”这个想法上。我们今天做的每件事，未来都可能被极其细致地审视，因为这样做会是“免费的”。我认为现在很多人的行为都隐含着一种“靠默默无闻获得安全”的假设。但如果智能真的便宜到几乎无需计量，那么对一切进行完美重建与综合就会成为可能。LLM 在看着（或者使用它们的人会这么做）。最好还是做个好人。
用 Opus 4.5 做这个项目的 vibe coding 相对顺畅，大约花了 3 小时。虽然有一些小磕绊，但整体非常令人印象深刻。仓库在 GitHub：karpathy/hn-time-capsule。下面是代码所做事情的流程：

给定一个日期，下载首页 30 篇文章
对每篇文章，使用 Algolia API 下载并解析文章本身和完整评论串。
把所有内容打包成一个 markdown 提示词，请求进行分析。下面是我使用的提示词前缀：
以下是一篇 10 年前出现在 Hacker News 上的文章及其讨论串。

现在利用我们事后视角的优势，从 6 个部分来分析：

1. 简要总结这篇文章和讨论串。
2. 这个话题后来实际发生了什么？（简要研究该话题并写总结）
3. 结合后来发生的情况，评选出“Most prescient”和“Most wrong”评论。
4. 提及文章或讨论中其他有趣或值得注意的点。
5. 结合后来发生的情况，给特定用户的评论打分。
6. 最后给出一个总分（0-10），表示这篇文章及其事后分析有多有趣。

关于第 5 部分的格式，使用标题 "Final grades"，然后仅用无序列表列出用户和分数，格式为 "name: grade (optional comment)"。示例如下：

Final grades
- speckx: A+ (excellent predictions on ...)
- tosh: A (correctly predicted this or that ...)
- keepamovin: A
- bgwalter: D
- fsflover: F (completely wrong on ...)

你的列表当然可以比这个示例包含更多人。请严格遵循格式，因为我会用程序解析它。我的想法是汇总每个账号的分数，以识别那些长期来看最有先见之明或最错误的账号。

关于第 6 部分的格式，使用前缀 "Article hindsight analysis interestingness score:"，后面跟分数（0-10）数字。对于回头看很突出、很有代表性或很有趣的文章/讨论，给高分；如果预测很少，或者话题非常小众/冷门，或回头看并不有趣，则给低分。

示例如下：
Article hindsight analysis interestingness score: 8
---
通过 OpenAI API 将提示词提交给 GPT 5.1 Thinking
收集并解析结果
把结果渲染为静态 HTML 网页，便于浏览
把 html 结果页托管到我的网站：https://karpathy.ai/hncapsule/
如果有人想继续玩，还会托管数据目录中的所有中间结果。它是同一 URL 前缀下的 data.zip 文件（有意避免直接链接）。
我花了几个小时到处浏览，发现内容非常有意思。这里随便举几个线程：

2015 年 12 月 3 日 Swift 开源。
2015 年 12 月 6 日 Figma 发布
2015 年 12 月 11 日 OpenAI 的最初公告 :').
2015 年 12 月 16 日 geohot 在做 Comma
2015 年 12 月 22 日 SpaceX 发射直播：Orbcomm-2 Mission
2015 年 12 月 28 日 Theranos 困境
然后当你进入 Hall of Fame 页面时，可以看到 2015 年 12 月 Hacker News 的顶级评论者，按类似 IMDb 的平均绩点分数排序。尤其要恭喜 pcwalton、tptacek、paulmd、cstross、greglindahl、moxie、hannob、0xcde4c3db、Manishearth、johncolanduoni，GPT 5.1 Thinking 认为你们的评论非常有洞见且极具前瞻性。你也可以一直滚到最底部，看到 HN 的噪声部分，这个我们大概都很熟悉 :)

我在 GitHub 上的代码（等等，应该说是 Opus 的代码？）可以用来复现或调整结果。把 31 天 * 每天 30 篇文章送入 GPT 5.1 Thinking，意味着 31 * 30 = 930 次 LLM 查询，成本大约是 58 美元，耗时大约 ~1 小时。未来的 LLM 超级智脑做这种事可能会更容易、更快、也更便宜。


-------


快速发个新帖：用事后视角自动评分十年前的 Hacker News 讨论

我拿了 2015 年 12 月 Hacker News 首页全部 930 个文章+讨论，并让 GPT 5.1 Thinking API 做事后分析，找出最有/最缺乏前瞻性的评论。写这套东西大约花了 ~3 小时 vibe coding，实际运行大约 ~1 小时，花费约 60 美元。这个想法来自昨天那篇 HN 文章：让 Gemini 3 去幻想十年后的 HN 首页。

更宏观地说：

1. 事后分析一直让我着迷，它是训练前向预测模型的一种方式，所以读这些结果真的很有意思；以及
2. 值得思考的是，当未来的 LLM 超级智脑能以更低成本、更快速度、且做得更好时，这类工作会是什么样子。你在互联网上贡献的每一条信息，都可以（而且很可能会）在“免费”的前提下被极其细致地审视。也因此我之前那条推文才会说：“be good, future LLMs are watching”。

恭喜前 10 个账号 pcwalton、tptacek、paulmd、cstross、greglindahl、moxie、hannob、0xcde4c3db、Manishearth 和 johncolanduoni。GPT 5.1 Thinking 认为你们的评论是 2015 年 12 月 HN 全部评论里最有洞见、最有前瞻性的。

