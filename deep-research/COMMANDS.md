# 通用命令指南（原 Claude Code 斜杠命令）

本技能原为 Claude Code 提供 ars-* 斜杠命令。在通用 LLM 编辑器（无斜杠命令机制）中，触发本技能命令的标准方式是使用**路径式触发语**：让 AI 读取本技能的 SKILL.md 与本文件（COMMANDS.md），并指明要执行的命令工作流。下文给出每个命令的：触发语、作用、执行要点。完整示例（以本目录的 ars-3w 为例）：

「请读取 E:\ARS-universal-skills\deep-research\SKILL.md 和 E:\ARS-universal-skills\deep-research\COMMANDS.md 中的 ars-3w 工作流，按其协议执行：帮我快速比较最近 5 篇关于 X 的论文，按 WHY/HOW/WHAT 整理」

本目录收录的命令：`ars-3w`。deep-research 技能的其余模式（`full` 完整研究、`quick` 快速简报、`review` 论文评估、`lit-review` 研究侧文献综述、`fact-check` 事实查核、`socratic` 苏格拉底引导、`systematic-review` 系统性回顾/后设分析）没有专属命令，同样以路径式触发语使用（例如「请读取 E:\ARS-universal-skills\deep-research\SKILL.md，按其 systematic-review 模式协议执行：帮我做一个关于 X 的系统性回顾」）。论文写作侧的文献综述命令见 `academic-paper/COMMANDS.md` 的 ars-lit-review。

## ars-3w

- **触发语**：「请读取 E:\ARS-universal-skills\deep-research\SKILL.md 和 E:\ARS-universal-skills\deep-research\COMMANDS.md 中的 ars-3w 工作流，按其协议执行：〈你的具体需求，中英文均可〉」

- **作用**：触发 deep-research 技能的 `three-way-scan` 模式，产出一份紧凑的论文清单，逐篇按三要素提取并做跨论文综合（约 800–2,000 字）：
  - **WHY**：论文要解决什么问题/瓶颈，为什么重要
  - **HOW**：采用什么策略、方法或技术路线
  - **WHAT**：论文发现了什么、构建了什么、还有什么未解决

  综合部分给出：共同的 WHY、分歧的 HOW、最强的 WHAT、尚未解决的整体空白。

- **执行要点**：
  - 定位比 `lit-review` 更轻：只做候选检索 → 去重 → 逐篇紧凑提取 → 跨论文综合，不产出完整文献综述报告。
  - 每篇论文的推荐输出格式：标题行 + 来源/年份/链接，随后 WHY / HOW / WHAT 各一条要点。
  - 属忠实谱系（fidelity spectrum），低监督（low oversight）。
  - 用户后续若需要更广的证据矩阵、主题综合或 PRISMA 式覆盖，从 `three-way-scan` 升级到 `lit-review` 或 `systematic-review` 模式即可（自然语言说明升级意图）。
  - 亦适用于：写论文前快速筛选值得精读的核心文献；若目标是把综述写成论文章节，改用 `academic-paper` 技能的 ars-lit-review（见 `academic-paper/COMMANDS.md`）。
