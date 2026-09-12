# 论文交接协议（Paper Handoff Protocol）

实验闭环结束（结果即停触发）后，进入本协议。目标：产出可直接喂给 academic-paper 技能的高质量交接包，支持**英文**与**中文**高水平期刊论文。

## 交接包内容（`experiments/handoff/` 目录）

1. **实验凭证**（experiment provenance）
   - 按 `shared/handoff_schemas.md` 的 `experiment_provenance[]` 结构整理：方法、配置、环境（GPU/驱动/依赖版本）、结果文件路径、日志路径
   - 运行 `scripts/check_experiment_provenance.py` 校验一致性
   - 论文中每个实验型主张必须能 join 到一条凭证；对不上的一律不得写入论文
2. **贡献点清单**：每条贡献对应假设账本中 supported 的条目 + 量化证据（与基线对比的数值与幅度）
3. **图表清单**：每图注明数据来源（结果文件+字段）、遵循出版级图规范（子图标签 a) 格式、无元素重叠、区分方式明确）
4. **基线对比表**：主指标 + 次指标，含离散度与样本量/种子数（种子数须有设计依据）
5. **局限性自查**：证伪的假设、inconclusive 的结果、环境限制（单卡 4090、数据规模）如实列出——局限性写清楚是高水平期刊的加分项而非减分项

## 语言与目标期刊

- **英文论文**：交接给 `academic-paper`（full 模式），指定目标期刊层级（Q1/顶刊），引用格式按期刊要求（APA/numbered）
- **中文论文**：同样走 `academic-paper`，明确要求中文成稿，遵循中文期刊规范（GB/T 7714 引用格式、中文摘要与关键词）；术语首现给英文对照
- 目标必须是**期刊**（journal），不投会议；选题与贡献表述按期刊论文的完整性要求组织（完整的方法学叙述 + 充分的实验验证）
- 投稿前必须先过 `academic-paper-reviewer` 模拟评审，修订后再定稿

## 诚信红线

- 论文中禁止出现无凭证支撑的实验主张（Stage 2.5/4.5 诚信闸门会核验）
- AI 参与情况按目标期刊与 `shared/` 下披露协议（disclosure/policy anchor）要求如实声明
- 合作者（人类）的知情与署名遵循 `academic-paper/references/credit_authorship_guide.md`
