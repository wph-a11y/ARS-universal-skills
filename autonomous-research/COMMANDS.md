# 通用命令指南（路径式触发语）

在通用 LLM 编辑器（无斜杠命令机制）中，触发本技能命令的标准方式是使用**路径式触发语**。标准格式：

```
请读取 E:\ARS-universal-skills\autonomous-research\SKILL.md 和 E:\ARS-universal-skills\autonomous-research\COMMANDS.md 中的 <命令名> 工作流，按其协议执行：〈你的具体需求〉
```

## ars-auto-run

- **触发语**：「请读取 E:\ARS-universal-skills\autonomous-research\SKILL.md 和 E:\ARS-universal-skills\autonomous-research\COMMANDS.md 中的 ars-auto-run 工作流，按其协议执行：〈研究问题/项目现状/目标指标，中英文均可〉」
- **作用**：启动完整自主科研闭环（Phase A→E）：建假设账本 → 实验设计 → 真实运行（4090）→ 分析证伪 → 迭代 → 结果即停后产出交接包
- **执行要点**：
  - 启动前必须完成项目现状盘点（数据、代码、环境、磁盘/GPU 空闲）
  - 三重闸门（算力门槛/方向变更/结果即停）全程生效，ESCALATE 时停下等用户批复
  - 全程写实验日志，缺失日志的实验不得进入论文

## ars-hypothesis

- **触发语**：「请读取 E:\ARS-universal-skills\autonomous-research\SKILL.md 和 E:\ARS-universal-skills\autonomous-research\COMMANDS.md 中的 ars-hypothesis 工作流，按其协议执行：〈主题或已有实验观察〉」
- **作用**：只做 Phase A：结合文献（可调用 deep-research）生成/更新假设账本，每条假设可证伪、有来源
- **执行要点**：不跑任何实验；输出账本后请用户确认优先级

## ars-exp-design

- **触发语**：「请读取 E:\ARS-universal-skills\autonomous-research\SKILL.md 和 E:\ARS-universal-skills\autonomous-research\COMMANDS.md 中的 ars-exp-design 工作流，按其协议执行：〈针对假设 H<n>〉」
- **作用**：只做 Phase B：产出实验设计文档（变量控制、基线、指标、资源预算、种子依据）
- **执行要点**：设计文档经用户确认后可直接接 ars-auto-run 的 Phase C 执行

## ars-prove

- **触发语**：「请读取 E:\ARS-universal-skills\autonomous-research\SKILL.md 和 E:\ARS-universal-skills\autonomous-research\COMMANDS.md 中的 ars-prove 工作流，按其协议执行：〈交接包位置或实验目录〉」
- **作用**：只做 Phase E：把已有实验产物整理成论文交接包（凭证校验、贡献点、图表清单、局限性），并衔接 academic-paper 开始撰写英文或中文期刊论文
- **执行要点**：先用 `scripts/check_experiment_provenance.py` 校验凭证；无凭证支撑的结果不得进入论文
