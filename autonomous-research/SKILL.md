---
name: autonomous-research
description: "Autonomous scientific research skill. Enables an AI agent to independently run the full research loop: hypothesis formulation, experiment design, real code execution (training/evaluation on local GPU), monitoring, analysis, falsification-driven iteration, and handoff to high-quality journal paper writing (Chinese and English). Embeds hard decision gates: compute threshold, direction-change escalation, and result-stop rule. Triggers on: autonomous research, 自主科研, 自动做实验, 跑实验, 假设验证, 消融实验, 调参, 实验迭代, run experiments, ablation, hyperparameter search, 科研闭环."
metadata:
  version: "1.0.0"
  last_updated: "2026-09-12"
  status: active
  data_access_level: raw
  task_type: open-ended
  related_skills:
    - deep-research
    - academic-paper
    - academic-paper-reviewer
    - academic-pipeline
---

# Autonomous Research — 自主科研闭环技能

让 AI agent 独立执行"假设 → 实验设计 → 真实运行 → 分析 → 证伪迭代 → 论文交接"的完整科研闭环，并通过三重决策闸门（算力门槛、方向变更、结果即停）保证自主性不失控。

> **设计原则**：自主 ≠ 放任。本技能给 agent 的不是"随便跑"的许可，而是一套**可审计的自主决策协议**——每一步自动决策都留痕（实验日志、假设账本、闸门记录），越界行为必须请示。

## 通用适配说明

本技能为通用 LLM 编辑器技能（TraeCode/Cursor 等），不依赖任何特定编辑器的 hooks 机制。所有闸门均为**提示词级强制规则**：模型必须在对应节点主动执行检查并在输出中给出闸门结论（PASS / ESCALATE / STOP），跳过闸门视为违反协议。实验执行通过编辑器自带的命令执行工具完成。

## 核心循环（五阶段）

```text
A 假设账本 → B 实验设计 → C 执行与监控 → D 分析与证伪 → E 论文交接
     ↑______________________证伪迭代回到 B_______________________|
```

### Phase A — 假设账本（Hypothesis Ledger）

任何实验开始前，先建立或更新假设账本（`experiments/hypothesis_ledger.md`）：

- 每条假设必须**可证伪**，格式：`H<n>: <具体陈述> | 预期观察: <指标变化> | 证伪条件: <何种结果否定它> | 状态: pending/testing/supported/falsified`
- 禁止"提升性能"这类不可证伪表述；必须落到具体指标与具体比较对象（如"H1: 在 FD001 上将窗口从 48 增到 96 可使 RMSE 下降 ≥3%"）
- 假设来源必须可追溯：文献依据（调用 deep-research 的文献扫描）或前期实验的反常观察

### Phase B — 实验设计（Experiment Design）

每个实验先写设计文档（`experiments/exp_<id>_design.md`），包含：

1. **目标假设**：对应 H<n>
2. **变量控制**：自变量、因变量、控制变量逐一列出
3. **基线与比较**：与什么比较才有意义；没有基线的实验不准跑
4. **评估指标**：主指标 + 次指标，事前固定，跑完后不许换指标
5. **资源预算**：预计单次运行时长、显存占用、总实验次数（受 Phase C 算力门槛约束）
6. **种子与重复策略**：反对无依据的多 seed 与过长 epoch——重复次数与训练时长必须有文献或初步实验依据，设计文档中写明依据，否则按最少配置执行

### Phase C — 执行与监控（Execution & Monitoring）

agent **允许真实运行训练/评估代码**，规则见 [references/resource_rules.md](references/resource_rules.md)：

- 代码、数据、输出全部落在工作盘（E: 及其后），**严禁占用 C 盘**（含临时目录、conda env、pip cache、数据集缓存均须重定向）
- 每次启动前自查：GPU 型号与显存（本机 4090 = 24GB，batch size 与模型规模按 24GB 上限设计）、磁盘剩余空间、依赖环境
- 长任务用后台运行 + 定期查日志；崩溃先读错误日志定位根因，**禁止盲目重试同一命令超过 2 次**
- 全程写实验日志（`experiments/exp_<id>_log.md`）：命令、配置快照、关键输出、结果文件路径——这是后续实验凭证登记（experiment provenance）的原始材料

### Phase D — 分析与证伪（Analysis & Falsification）

- 结果必须回填假设账本，给出裁决：`supported` / `falsified` / `inconclusive`（inconclusive 须说明缺什么才能裁决）
- **证伪不是失败**：falsified 的假设同样推进研究，从中提炼新假设
- 统计表述守规矩：报告均值±离散度、样本量、显著性检验的适用性；禁止从单次运行外推结论
- 可视化遵循出版级标准（坐标轴、标签格式如 a) 子图编号、无元素重叠），直接对标目标期刊图规范

### Phase E — 论文交接（Paper Handoff）

实验闭环达成后，按 [references/paper_handoff_protocol.md](references/paper_handoff_protocol.md) 交接：

- 生成 Material Passport 式实验凭证（方法、配置、环境、结果文件路径、日志），与 `shared/handoff_schemas.md` 的 `experiment_provenance[]` 对接，可用 `scripts/check_experiment_provenance.py` 校验
- 英文论文交接给 `academic-paper` 技能（full 模式），中文论文同样使用 academic-paper 并明确指定中文输出与中文期刊规范
- 交接内容：贡献点清单（每条对应假设账本中 supported 的条目）、图表清单、与基线的量化对比、局限性

## 三重决策闸门（Decision Gates）

详细判据见 [references/decision_gates.md](references/decision_gates.md)。摘要：

| 闸门 | 规则 | 动作 |
|---|---|---|
| **算力门槛** | 单次运行预计 > 2 小时，或总 sweep 组合 > 20 组 | 停下向用户报告预算，获批后执行 |
| **方向变更** | 更换方法骨干/研究问题/评估指标/删改基线 | 必须先请示，说明理由与代价 |
| **结果即停** | 主指标达到预设目标，或假设被明确证伪 | 立即停止本轮迭代，禁止"再调调看"，进入分析或交接 |

**反过拟合防线**：连续 3 轮迭代主指标无显著改善 → 强制停下做归因分析（数据/方法/实现三层归因），而不是继续调参。

## 与其他技能的衔接

- **deep-research**：Phase A 的假设来源与文献定位；研究问题尚未成型时先走其 socratic 模式
- **academic-pipeline**：本技能覆盖其 Stage 1→2 之间的实验空档，产物（实验凭证）接入 Stage 2.5 诚信闸门核验
- **academic-paper**：Phase E 论文撰写（英文/中文期刊均可）
- **academic-paper-reviewer**：成稿后投稿前模拟评审

## 快速开始

最小触发（自然语言）：

```
读取 E:\ARS-universal-skills\autonomous-research\SKILL.md，按协议对 <项目> 做自主科研：
我的研究问题是 …，现有代码在 …，目标指标是 …
```

执行顺序：先检查项目现状（数据、代码、算力）→ 建/查假设账本 → 出实验设计 → 过闸门 → 跑。
