# 通用命令指南（原 Claude Code 斜杠命令）

本技能原为 Claude Code 提供 ars-* 斜杠命令。在通用 LLM 编辑器（无斜杠命令机制）中，触发本技能命令的标准方式是使用**路径式触发语**：让 AI 读取本技能的 SKILL.md 与本文件（COMMANDS.md），并指明要执行的命令工作流。下文给出每个命令的：触发语、作用、执行要点。完整示例（以本目录的 ars-full 为例）：

「请读取 E:\ARS-universal-skills\academic-pipeline\SKILL.md 和 E:\ARS-universal-skills\academic-pipeline\COMMANDS.md 中的 ars-full 工作流，按其协议执行：我想写一篇关于 X 的研究论文，从选题到成稿走完整流程」

本目录收录 4 个命令：`ars-full`（全流程编排）与三个引用核验基础设施命令 `ars-mark-read`、`ars-unmark-read`、`ars-cache-invalidate`。三个工具命令服务于 Material Passport 信任体系（人工阅读声明账本与引用验证缓存），通常在 pipeline 各阶段的完整性门（Stage 2.5 / 4.5）前后使用；亦适用于独立的引用核验场景。

## ars-full

- **触发语**：「请读取 E:\ARS-universal-skills\academic-pipeline\SKILL.md 和 E:\ARS-universal-skills\academic-pipeline\COMMANDS.md 中的 ars-full 工作流，按其协议执行：〈你的具体需求，中英文均可〉」

- **作用**：触发 academic-pipeline 编排器（编排器本身无命名模式），执行**10 阶段完整学术流程**：deep-research（研究）→ academic-paper（写作）→ 完整性检查 → academic-paper-reviewer（评审）→ revision（修改）→ re-review（复审）→ 最终完整性检查 → 定稿。对应 Stage 1 RESEARCH / 2 WRITE / 2.5 INTEGRITY / 3 REVIEW / 4 REVISE / 3' RE-REVIEW / 4' RE-REVISE / 4.5 FINAL INTEGRITY / 5 FINALIZE / 6 PROCESS SUMMARY。

- **执行要点**：
  - 编排器只做阶段检测、模式推荐、技能调度、状态跟踪与交接，**不做实质性研究/写作/评审工作**（IRON RULE）。
  - ⚠️ **每阶段完成后必须主动提示并等待用户确认**：检查点分 FULL / SLIM / MANDATORY 三类；完整性边界（Stage 2.5 / 4.5）、评审决定（Stage 3 / 3'）与定稿入口为 MANDATORY，不可自动跳过。
  - **完整性门为强制**：Stage 2.5 与 4.5 跑 5 阶段核验协议（参考文献 → 引用上下文 → 统计数据 → 原创性 → 声明）+ 7 类 AI 研究失效模式清单；FAIL 修复后重验（最多 3 轮），仍未决的项必须由用户显式记录决定，绝不静默丢弃；Stage 4.5 必须从零重跑，不得只复查 Stage 2.5 的发现。协议见 `references/integrity_review_protocol.md`、`references/ai_research_failure_modes.md`。
  - **中途进入（mid-entry）**：可从任意阶段进入（如有草稿直接进评审、收到审稿意见直接进修改），但不能跳过 Stage 2.5——例外：能提供既有完整性报告且内容未修改。状态机与全部合法转换见 `references/pipeline_state_machine.md`。
  - **修订上限**：Stage 4 与 Stage 4' 各一轮（共 2 轮完整修订环），覆盖 academic-paper 自身最多 2 轮的规则；两阶段评审协议见 `references/two_stage_review_protocol.md`。
  - 定稿（Stage 5）依次产出 Markdown → DOCX（有 Pandoc 时）→ LaTeX → 用户确认内容后编译 PDF（必须由 LaTeX 编译，禁止 HTML 转 PDF）；Stage 6 生成「论文创作过程记录」，用户可拒绝（标记 skipped，流程仍正常完成）。

## ars-mark-read

- **触发语**：「请读取 E:\ARS-universal-skills\academic-pipeline\SKILL.md 和 E:\ARS-universal-skills\academic-pipeline\COMMANDS.md 中的 ars-mark-read 工作流，按其协议执行：〈你的具体需求，中英文均可〉」

- **作用**：为指定引用键（citation key）记录用户的 `USER_ATTESTED_READ` 阅读声明。注意：这是**用户自述**，不是此人确已阅读/理解该来源的独立证据。finalizer 只有在声明范围覆盖该引用锚点时，才可把 `<!--ref:slug LOW-WARN-->` 提升为 `<!--ref:slug ok-->`。

- **执行要点**：
  - 声明存储于活动 Material Passport 旁的会话级伴随文件 `<passport-stem>_human_read_log.yaml`；`literature_corpus[]` 由适配器拥有，**绝不**被改动以承载阅读状态。
  - 每条新标记**必须**带阅读范围（仅记录声明——原样记录用户所述，绝不推断）：`--scope {full_text,sections,abstract_only,toc_only,unknown}`；`--scope sections` 需配 `--locator "<文本>"`（可重复）指明所读章节/页码；`--note "<文本>"` 为自由文本（需配合 scope）。用户说不清覆盖范围时用 `--scope unknown`。
  - 显式 `unknown` 与旧版缺失范围保持 `coverage_unknown`：承认声明存在，但**永不能**把锚定引用提升为 `ok`。
  - 页码覆盖必须带显式 `page` / `p.` / `pp.` 定位符——裸数字与 `section <n>` 不算页码范围。
  - 校验与写入由 `scripts/ars_mark_read.py` 处理：citation_key 必须存在于 `literature_corpus[]`，否则报 `[ARS-MARK-READ ERROR: citation_key '<slug>' not in literature_corpus[]]` 并拒绝写入；另有 4 项快速失败环境检查（无活动护照 / 找不到护照 / 父目录不可读 / 读日志不可写）与追加式（append-only）写入。`scripts/human_read_attestation_resolver.py` 在每次 finalizer pass 严格校验当前账本并计算瞬态路由决定（输出不是持久审计回执）。

## ars-unmark-read

- **触发语**：「请读取 E:\ARS-universal-skills\academic-pipeline\SKILL.md 和 E:\ARS-universal-skills\academic-pipeline\COMMANDS.md 中的 ars-unmark-read 工作流，按其协议执行：〈你的具体需求，中英文均可〉」

- **作用**：撤销先前为指定引用键记录的 `USER_ATTESTED_READ` 声明。

- **执行要点**：
  - 读日志为**追加式**（§3.6 firm rule 3）：撤销通过在匹配条目上写入 `rescinded_at: <ISO 8601>` 字段实现，而非删除条目——审计重放可据此还原用户的信号轨迹。
  - 下一次 finalizer pass 会把每个被撤销 slug 的覆盖依赖型 `<!--ref:slug ok-->` 降回 `<!--ref:slug LOW-WARN-->`。
  - 前置条件：citation_key 必须存在于 `literature_corpus[]` 且在读日志中有未撤销的先前标记，否则以规范格式 `[ARS-MARK-READ ERROR: ...]` 硬失败。
  - 与 ars-mark-read 使用同一脚本：`scripts/ars_mark_read.py`（附加 `--unmark`）。

## ars-cache-invalidate

- **触发语**：「请读取 E:\ARS-universal-skills\academic-pipeline\SKILL.md 和 E:\ARS-universal-skills\academic-pipeline\COMMANDS.md 中的 ars-cache-invalidate 工作流，按其协议执行：〈你的具体需求，中英文均可〉」

- **作用**：使**一个**引用键的持久验证缓存失效，使下一次 pipeline 运行对其重新实时核验（Crossref / OpenAlex / Semantic Scholar / arXiv），而非返回陈旧的缓存结论。适用于：引用元数据发生变化（如预印本获得正式 DOI），或先前某次验证结果看起来有误。

- **执行要点**：
  - 缓存为本地 SQLite 库 `~/.cache/ars/verification.db`（可用环境变量 `ARS_VERIFICATION_CACHE_PATH` 覆盖），按 `(citation_key, resolver_name, query_form)` 键控，TTL 90 天。
  - 本命令删除该引用键的**全部**缓存条目（四个 resolver、所有查询形态）；其他引用不受影响。幂等——对没有缓存条目的键执行等于空操作，仍算成功。
  - **失效级联（#541，无条件）**：失效后，下一道完整性门会重新生成该引用的验证摘要行，并对引用它的 claims 重跑 Phase E 审计裁定（不保留基线做差异对比），覆盖存在性状态、元数据与已检索证据。门上另有基于年龄的陈旧缓存 advisory（`ARS_CACHE_STALE_ADVISORY_DAYS`，默认 30 天；设 `ARS_CACHE_REVALIDATE=1` 可选择实时重验）。
  - 若需**整体**清空缓存（例如系统性 resolver 故障缓存了大量假阴性），直接删除数据库文件 `~/.cache/ars/verification.db` 即可，下次运行会自动重建为空库。
  - 脚本：`scripts/ars_cache_invalidate.py`。
