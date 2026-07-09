# Trivium · 三省

> **三条路，四道门。** Claude Code 的规格驱动开发流水线。

*Trivium*（拉丁语"三岔路口"，亦是西方古典教育的"三艺"）为 Claude Code 提供三个纪律化的入口，对应软件工作的全部三种情形——新建、变更、修复——每条路上设有硬性的人工审批 Gate。中文名取「吾日三省吾身」：三条路，亦是每条路上的审视。

[English README](./README.md)

## 三条路

| 命令 | 场景 | 流水线 |
|---|---|---|
| `/trivium:feature` | 新增需求 | 苏格拉底消歧 → PRD → 架构与冻结 API 合同 → 任务计划 → subagent 实现（TDD + 两阶段 review）→ Playwright 验收 |
| `/trivium:refactor` | 重构 / 需求变更 | 影响面分析 → 行为锁定测试 → 变更方案与 ADR → 计划 → 基线常绿实现 → 回归 |
| `/trivium:bugfix` | 缺陷修复 | 失败复现测试**先行** → 系统化根因定位 → 最小修复 → 红转绿 + 全量回归。支持 Excel/CSV 缺陷清单批量分诊模式。 |

## 四道门

你的角色收敛为在 Gate 处签字，Gate 之间流水线自治运行。

| | Gate 1 | Gate 2 | Gate 3 | Gate 4 |
|---|---|---|---|---|
| feature | 批 PRD | 批架构+合同 | 批计划 | 终审验收 |
| refactor | 确认影响面+行为分类 | 批变更方案 | 批计划 | 终审回归 |
| bugfix（单个） | 确认复现 | — | — | 终审修复 |
| bugfix（批量） | 分诊：范围与顺序 | 每 bug 确认复现 | — | 逐个或批量终审 |

## Trivium 在 superpowers 之上补了什么

Trivium **编排** [superpowers](https://github.com/obra/superpowers) 而非重新实现它。superpowers 提供方法论（brainstorming、writing-plans、subagent 开发、TDD、系统化调试、code review）；Trivium 补齐规格驱动流水线需要而 superpowers 没有的三块：

- **`prd-writing`** — 带优先级、可测 Given/When/Then 验收标准、强制 out-of-scope 清单的 PRD 格式，作为下游一切阶段的单一事实来源。
- **`api-contract`** — 冻结的前后端合同（OpenAPI + 生成的 TypeScript 类型 + mock），使前后端 task 可由互不通信的 subagent 并行开发，合同变更走正式流程。
- **`acceptance`** — 与 PRD 标准一一追溯的 Playwright 端到端用例、追溯矩阵和基于证据的验收报告。

以及流水线机制本身：硬性停等 Gate、全部产物落盘 `docs/pipeline/<slug>/` 并带 `status` frontmatter 支持**跨会话断点续传**、合同变更回 Gate 而非悄悄偏离、开发内返工与交付后 bugfix 的严格边界。

## 依赖

- 已安装 **[superpowers](https://github.com/obra/superpowers)**（Trivium 调用其 brainstorming / writing-plans / using-git-worktrees / subagent-driven-development / systematic-debugging / requesting-code-review / finishing-a-development-branch）
- 目标项目可运行 `@playwright/test`（验收阶段）
- 建议：`openapi-typescript`（类型生成）；MSW 或 `@stoplight/prism-cli`（mock）
- 文件输入：`pandoc`（docx）、`poppler-utils`（pdf）、`libreoffice`（旧 .doc）；Python 兜底：`pip install python-docx pdfplumber pandas openpyxl`

## 安装

官方插件目录（上架后）：

```
/plugin install trivium
```

或直接从本仓库：

```
/plugin marketplace add hellotern/Trivium
/plugin install trivium@trivium
```

## 使用

文字与文件输入可混合：

```
/trivium:feature 用户可以用邮箱注册和登录，支持记住我
/trivium:feature docs/需求/会员体系需求.docx 补充：本期只做邮箱，不做手机号
/trivium:refactor 把订单模块散落的 useState 收敛为状态机
/trivium:bugfix 测试报告/UAT缺陷清单.xlsx
```

文件内容会被提取并原样存档到 `docs/pipeline/<slug>/00-source.md` 作为需求原文依据。Excel/CSV 清单触发 `/trivium:bugfix` 的**批量模式**：解析 → 根因归组 → 分诊 Gate → 逐 bug 复现-定因-修复 → 状态回写 → 批量总报告。

## 什么时候不用 Trivium

这套流水线是 token 密集型设计。单文件小改、快速原型、一次性脚本直接对话即可。三条路留给值得付流程税的工作：要长期维护的功能、跨前后端的需求、有回归风险的重构、已交付代码的缺陷。

## License

MIT
