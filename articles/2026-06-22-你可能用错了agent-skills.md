# 你可能用错了 Agent Skills

*本文翻译自 Anson Biggs 的博客文章，反映了作者在 Agent Skills 实践中的一手观察和思考。*

原文：[You're probably using Agent Skills wrong](https://notes.ansonbiggs.com/youre-probably-using-agent-skills-wrong/)，作者 Anson，发表于 2026 年 2 月 16 日。

---

序言：最近 SkillsBench 论文登上了 Hacker News，结论很扎眼——"自生成的 Agent Skills 毫无用处"。但 Anson Biggs 坐不住了：他从自己写的 Skills 里获得了巨大价值，同事却普遍在用错的方式使用它。问题出在哪？出在一个特别常见的反模式：让 Agent 自己写自己的技能说明书。这就像让一个从来没做过花生酱三明治的人，把制作步骤写在纸上——你没经历过卡壳，就不可能写出真正有用的指南。这篇文章不长，但说清楚了 Skills 真正的价值在哪，以及为什么大多数人的用法从一开始就错了。

---

![头图](https://images.animesdata.com/news/2026/06/22/6a38ec642b8be.jpg)
*图片来源：Lin Dai / Unsplash*

## 你可能用错了 Agent Skills

不过幸好，我够聪明，能告诉你怎么才是正确用法。

整个 Claude Code 的生态圈相当令人困惑——命名规范一团糟，变化速度超过我见过的任何生产工具。然而，Skills 可能是被误用得最严重的一个。我在工作中见得太多了，而最近一篇论文登上了 Hacker News：

[SkillsBench: Benchmarking How Well Agent Skills Work Across Diverse Tasks](https://arxiv.org/abs/2602.12670)

Agent Skills 是一种结构化的程序性知识包，能在推理时增强 LLM agent 的能力。尽管被迅速采用，却从来没有一套标准方法来衡量它们是否真的有用。我们提出了 SkillsBench——一个涵盖 11 个领域、共 86 项任务，并配有精心策划的 Skills 和确定性验证器的基准测试。每项任务在三种条件下评估：无 Skills、精选 Skills、自生成 Skills。我们在 7 个 agent-模型配置上测试了 7,308 条运行轨迹。精选 Skills 将平均通过率提升了 16.2 个百分点，但效果因领域差异巨大（软件工程 +4.5 pp 到医疗保健 +51.9 pp），并且 84 项任务中有 16 项出现了负增长。自生成 Skills 平均而言没有带来任何收益，说明模型无法可靠地产出它们自己消费所需的程序性知识。包含 2--3 个模块的精炼技能优于全面文档，配备 Skills 的小模型可以匹敌没有 Skills 的大模型。

正是这篇论文让我忍不住要写这篇文章。

HN 上的标题不知为何被编辑成了 *"研究：自生成的 Agent Skills 毫无用处"*，但它立刻抓住了我的注意力，因为我从 Agent 写的 Skills 中获得了巨大价值——但同时我也一直看到身边的同事在误用它们。这个概念本身很棒，我自己也一直在研究 agent 生态系统中特定环节的基准测试，所以这篇文章对我来说高度相关。总的来说论文还行，但有一个要点直接推翻了整个结论：

> **自生成 Skills**：不提供 Skills，但提示 agent 在解决问题之前先生成相关程序性知识。这里隔离的是 LLM 潜在领域知识的影响。

所以他们做的，无非是拿一个模型本身就不太擅长的问题，然后在动手之前让它先写写这个任务。他们只是发明了一个更糟糕的 think 模块！

### Skill 的反模式

他们所做的，正是当下一个非常普遍的错误。我的 Agent 不擅长某件事 → 于是我让 Agent 为这件事写一个 Skill。我再强调一遍，这和 thinking blocks 是一回事。为了让你的 Agent 创造出有价值的东西，你必须确保它能看见自己的盲区。我把它看作经典的计算机科学入门题目——你让别人写出做花生酱三明治的步骤：只有当你亲自卡壳过，你才真正理解问题难在哪里。

这直接导致了 AI 时代最大的失礼行为——把别人的问题原封不动扔给 LLM，然后把 LLM 的回答原封不动贴回去。如果你问我你是怎么用 Agent 做出什么酷东西的，而你当场开了一个全新的对话窗口让 Agent 为我的问题生成一个 SKILL.md——*我会宰了你。*

## 什么是 Skills

在聊正确用法之前，先说一下 Skills 到底是什么。作为一种基础构件，它们就是 Markdown 文件，顶部有一些元数据帮助 Agent/工具知道何时调用它们，其余部分就是技能内容。每个 Skill 有自己的文件夹，所以它不仅能教你的 Agent 怎么做某件事，还能给它更好的工具。

```bash
.claude/skills/
└── monitor-gitlab-ci/
    ├── SKILL.md          # 上面说的那个文件
    ├── monitor_ci.sh     # 复杂命令脚本
    └── references/       # 额外参考资料
        ├── api_commands.md
        ├── log_analysis.md
        └── troubleshooting.md
```

上面是一个我用了很久的 Skill，让老版本的 Claude 能够处理我的 GitLab CI。一个文件夹，里面放一个简单的 Markdown Skill，解释了一下整体设置，并告诉 Agent 需要一直监控 CI 直到某个 job 失败或者全部通过；外加一个简单的 CLI 脚本以免 Agent 自己写脚本；再配上一些额外的参考资料覆盖边界情况。

## 用 Skills 补全上下文

Agent 是完全无状态的，也就是说每一次新对话都像是第一次见到这个模型——它完全不知道你的项目是什么，也不知道你十分钟前在干什么。CLAUDE.md 很大程度上解决了这个问题，但对于足够庞大的项目来说它装不下所有信息。如果我打开一个 monorepo 让 Claude 跑一个 SIL 测试，它得满世界瞎转来搞清楚怎么操作——先得搞清楚项目用的什么语言，然后去找那种语言通用的测试模式，接着发现一个复杂的 Docker Compose 配置，发现容器需要 x86 但我们跑在 Mac 上，再去找 CI……等等等等。

这一切都可以通过为那些"常见但不通用"的模式编写 Skills 来解决。任何时候当模型在你的项目中卡在一个你知道其实很简单、很基础的事情上时，让它写一个 Skill，把完成这个任务所缺失的知识补上。

## 用 Skills 消除重复

Skills 的另一个简单用法是描述你经常做的任务。比如我经常让 Agent 确保我的 `docs/`、MR 描述、Issue 和代码库保持一致。于是我做了一个简单的 Skill，免得每次都打一遍。

## 用 Skills 攻克难题

Claude 能解决一些非常难的问题，但可能要在 token 上花掉 500 美元，而且你还得吼它几次，因为它会 reward hacking。几乎每次我必须介入一个问题时，只要 Agent 脱离了卡住状态，我就会问它：到底缺了什么让你自己搞不定？有时候原因很蠢，但有时候答案真的很有洞见——我就会让 Claude 写一个 Skill 来补上这个缺口。

## 结语

我用自己做 Skills 的方式重新跑了这个基准测试，结果正如我预料——配备了正确 Skills 的 Agent 完美拿下了测试。我没有钱做完整的验证，但第一轮结果已经足以让我满意。我认为这基本上让基准测试需要的数据集翻了一倍，所以我猜这就是作者没有纳入这种方法的原因。

记住，创造 Skill 的理由只有两个——记住一个新颖问题，以及避免重复劳动。如果你只是开一个全新的 Agent 对话，让它为 `x` 生成一个 Skill，那大概率毫无价值。它需要知道一个新模型不知道的东西——这可以来自你的 prompt 中描述的一个通用流程、攻克一个难题所积累的知识集合，甚至可以让它自己去研究某个并不新颖的东西。

Happy Hacking。

HN 讨论：[https://news.ycombinator.com/item?id=48624327](https://news.ycombinator.com/item?id=48624327)
