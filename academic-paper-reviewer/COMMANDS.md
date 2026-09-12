# 通用命令指南（原 Claude Code 斜杠命令）

本技能原为 Claude Code 提供 ars-* 斜杠命令。在通用 LLM 编辑器（无斜杠命令机制）中，触发本技能命令的标准方式是使用**路径式触发语**：让 AI 读取本技能的 SKILL.md 与本文件（COMMANDS.md），并指明要执行的命令工作流。下文给出每个命令的：触发语、作用、执行要点。完整示例（以本目录的 ars-reviewer 为例）：

「请读取 E:\ARS-universal-skills\academic-paper-reviewer\SKILL.md 和 E:\ARS-universal-skills\academic-paper-reviewer\COMMANDS.md 中的 ars-reviewer 工作流，按其协议执行：帮我完整评审这篇论文（模拟评审团）」

本目录收录的命令：`ars-reviewer`（覆盖本技能 `full` 及全部替代模式）。在 academic-pipeline 中，本技能由编排器于 Stage 3（首轮评审）与 Stage 3'（re-review 复审）调度——见 `academic-pipeline/COMMANDS.md` 的 ars-full。

## ars-reviewer

- **触发语**：「请读取 E:\ARS-universal-skills\academic-paper-reviewer\SKILL.md 和 E:\ARS-universal-skills\academic-paper-reviewer\COMMANDS.md 中的 ars-reviewer 工作流，按其协议执行：〈你的具体需求，中英文均可〉」

- **作用**：以 `full` 模式触发本技能——模拟同行评审团，7 个代理产出 **5 份评审报告 + 编辑决定（Editorial Decision）+ Revision Roadmap**。评审团含 Journal-Fit Reviewer 与魔鬼代言人（Devil's Advocate）等角色。

- **执行要点**：
  - 用户明确指定替代模式时尊重之，各模式均可自然语言触发：
    - `quick`：「快速看一下这篇论文」——field_analyst + eic，15 分钟版：Journal-Fit 快速评估 + 关键问题清单。
    - `methodology-focus`：「帮我检查这篇论文的方法学」——field_analyst + eic + methodology_reviewer，方法学深评报告。
    - `re-review`：「验证性复审 / 检查修改是否落实」——针对修改稿的 Stage 3' 复审：在 #576 三门（先证据后说服）契约下核对每条审稿意见的落实情况，产出 R&R 可追溯矩阵 + 残留问题 + 新决定（或 deferral/abort），不重跑 field_analyst；协议见 `references/re_review_mode_protocol.md`。
    - `guided`：「引导我逐条改进这篇论文」——全代理 + 苏格拉底式逐问题引导评审。
    - `calibration`：「校准你的评审准确度 / 用这 10 篇金标论文测试评审」——directional 档：3 篇金标 × 1 个评审团；full 档：5–20 篇金标 × 5 次（可降为 3 次），跨模型默认开启；产出方向性边界读数或完整校准报告 + 按档位的会话置信度披露。
  - 亦适用于：独立使用（对已有论文做投稿前体检）与 pipeline 内使用（Stage 3/3'）两种场景；输入门控与评审标准绑定见 `references/review_criteria_framework.md` 与 `references/sprint_contract_protocol.md`。
