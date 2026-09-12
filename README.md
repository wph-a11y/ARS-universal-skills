# ARS 通用技能包（Academic Research Skills — Universal Edition）

由 [academic-research-skills](https://github.com/Imbad0202/academic-research-skills)（Claude Code 版）改装的**通用 LLM 编辑器技能包**，适配 TraeCode、Cursor、Copilot、Windsurf 等任何能读取 Markdown 技能文件的 AI 编辑器。

## 包含的四个技能

| 技能 | 用途 |
|---|---|
| `deep-research/` | 13-agent 研究团队：苏格拉底引导、文献综述、PRISMA 系统综述、事实查核 |
| `academic-paper/` | 12-agent 论文写作团队：章节规划、风格校准、引用格式、LaTeX/APA 输出 |
| `academic-paper-reviewer/` | 模拟同行评审：Journal-Fit 审查 + 3 位审查者 + 魔鬼代言人 |
| `academic-pipeline/` | 10 阶段全流程：研究 → 写作 → 诚信审查 → 评审 → 修订 → 出版 |
| `autonomous-research/` | 自主科研闭环：假设账本 → 实验设计 → 真实跑实验（4090）→ 证伪迭代 → 论文交接，含三重决策闸门 |
| `shared/` | 跨技能共享的协议、schema 与参考资料 |

## 安装方式

**方式 A（支持技能目录的编辑器）**：把四个技能文件夹拷入编辑器的技能/规则目录（如 `.claude/skills/`、`.trae/rules/`、`.cursor/rules/` 等），保证 `SKILL.md` 位于技能文件夹根部。

**方式 B（任意编辑器）**：直接在对话中让 AI 读取对应技能的 `SKILL.md` 并按其执行，例如：

```
请阅读 E:\ARS-universal-skills\deep-research\SKILL.md，按其中的协议引导我做研究
```

## 使用

原 `ars-*` 斜杠命令已改为**路径式触发语**，各技能目录下的 `COMMANDS.md` 给出每个命令的触发语、作用与执行要点。标准触发格式：

```
请读取 E:\ARS-universal-skills\<技能名>\SKILL.md 和 E:\ARS-universal-skills\<技能名>\COMMANDS.md 中的 <命令名> 工作流，按其协议执行：〈你的具体需求〉
```

示例：

```
请读取 E:\ARS-universal-skills\academic-paper-reviewer\SKILL.md 和
E:\ARS-universal-skills\academic-paper-reviewer\COMMANDS.md 中的 ars-reviewer 工作流，
按其协议执行：帮我完整评审这篇论文
```

```
请读取 E:\ARS-universal-skills\autonomous-research\SKILL.md 和
E:\ARS-universal-skills\autonomous-research\COMMANDS.md 中的 ars-auto-run 工作流，
按其协议执行：对 <项目> 做自主科研，研究问题是 …，目标指标是 …
```

## 与原版（Claude Code）的差异

- **斜杠命令 → 路径式触发语**：`/ars-*` 命令全部转为 `COMMANDS.md` 中指定 `E:\ARS-universal-skills\` 路径的触发语
- **hooks 硬闸门 → 提示词级软约束**：原由 hooks 强制阻断的学术诚信检查（引用真实性、Stage 2.5/4.5 闸门、写范围守卫）现在依赖模型自觉执行；建议在关键阶段明确要求模型"执行诚信检查清单并给出结论"
- **子代理编排**：agent team 在支持子代理的编辑器中可并行派发；不支持时顺序执行，协议不变
- `references/`、`templates/`、`shared/` 内容未改动，与原版一致

## 诚信边界（继承原版）

本工具检查论文与所报告的研究过程（引用是否存在、主张与来源是否一致等），**不能**证明程序确实执行、原始数据真实或结果可复现。AI 是副驾驶，不是机长。
