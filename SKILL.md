---
name: paper-code-analyzer
description: Use when the user gives a research paper (PDF/arXiv) together with its code repository and wants to understand the work from both the method and implementation sides — 提取论文核心创新, 梳理公式与算法流程, 定位代码中与论文对应的实现 (paper↔code mapping), 抽取核心模块成可复用/即插即用形式, 标记潜在工程风险, 给出改进方向. INTERACTIVE by design: it analyzes, then discusses paper↔code with the user, and only writes a Markdown report when the user explicitly asks ("生成md"/"出报告"). Static analysis only — never runs or modifies the user's code. Triggers on "分析论文代码", "论文代码映射", "paper code analysis", "读懂这篇论文的代码", "把核心模块抽出来", or when a paper PDF and a code folder are provided together.
version: 2.1.0
---

# Paper-Code Analyzer

把「一篇论文 + 它的代码仓库」从**方法**和**实现**两个层面读懂。
**交互式**：先做分析，然后和用户**就论文↔代码来回探讨**；**只有用户明确说"生成 md / 出报告"时，才落地成一份 Markdown 报告**。

覆盖：① 论文核心创新 ② 算法/公式拆解 ③ 论文↔代码映射 ④ 核心模块即插即用抽取 ⑤ 潜在工程风险 ⑥ 改进方向。
适用：论文复现、代码阅读、科研调研、模型改造、新方法设计。

## 用法

```
/paper-code-analyzer <paper> <repo>
```
- **paper**：论文 PDF 路径 或 arXiv URL/ID　- **repo**：代码文件夹路径 或 GitHub URL

## 工作模式（重要）

对话式工具，不是一键批处理：

- 分析阶段（Phase 2–3）由子代理完成，结果回到主对话，**不写任何文件**。
- 然后**默认停在「探讨」**：主代理依据分析结果与用户交流，回答问题、一起推敲、讨论改造。
- **报告只在用户下令时生成**（"生成 md""出报告""导出"）。没下令就一直保持探讨。

## 三条铁律

1. **静态分析**：只读，**绝不运行、绝不修改**用户代码。
2. **先读论文，再看代码**：避免按实现反推论文。
3. **每条映射带出处**：`[paper §X]` / `[code: file:line]` / `[config: file:key]` / `[guess: 假设]`；拿不准写"待核实"。

## 文件结构

```
paper-code-analyzer/
├── SKILL.md                              # 本文件：主编排 + 交互流程
├── agent-paper-prompt.md                 # Agent P：论文分析
├── agent-code-prompt.md                  # Agent A：代码深读 + 论文映射 + 模块抽取 + 风险
├── agent-quiz-prompt.md                  # Agent B：闭卷出题（可选自检）
├── agent-exam-prompt.md                  # Agent C：闭卷答题（可选自检）
└── references/
    ├── static-analysis.md                # 静态分析方法论（扫描/config/入口/日志/风险）
    ├── paper-code-mapping.md             # 论文↔代码映射方法论（溯源标签+公式核对+round-trip）
    └── output-report-format.md           # 分析报告的章节与格式规范（生成报告时读）
```

子代理返回的是**结构化文本**（回主对话当知识），不写最终文件。

## 流程

### Phase 1 — 准备
确定项目名；repo 是 URL 则 clone（只读，本地路径绝不改源码）；paper 是 arXiv 则抓取（优先 HTML）；`git tag --list` 推荐版本。
**暂停确认**：论文 / 代码 / 跟踪版本对不对。

### Phase 2 — 论文分析（派 Agent P）
传 `{paper-source}`，用 `agent-paper-prompt.md`。P 只读论文，返回：核心创新点、关键公式（编号/符号/作用）、算法流程、论文自报实验与主结果。结果留在主对话。

### Phase 3 — 代码静态分析 + 映射（派 Agent A）
先按 `references/static-analysis.md` 快速扫结构、识别模块、定位 config 与训练/推理/数据入口，把创新点粗映射到候选模块。
**暂停确认**：列出模块 + 粗映射，请用户选要深读的核心模块（或 all）。
对每个选中模块派一个 Agent A（`agent-code-prompt.md`，传 `{source-dir}`/`{module-dir}`/`{module-name}`/`{paper-analysis}`/`{ref}`）。A 依据 `references/paper-code-mapping.md` 与 `references/static-analysis.md`，返回：5 维代码认知、论文↔代码映射（带出处+三态结论）、公式逐项核对、即插即用抽取、工程风险。结果汇回主对话。

### Phase 4 — 交互探讨（默认停留在此）
先给用户一段**关键发现摘要**（创新点 / 映射亮点 / 高优先风险），然后自由探讨：
- 回答用户对论文、代码、二者关系的提问
- 一起推敲某公式在代码里怎么实现、reduction/eps/梯度流对不对
- 讨论核心模块怎么抽出来复用、怎么改造、怎么设计新方法
- 探讨中产生的新结论记在对话里，供最终报告使用

**可选自检**：某块结论需要把握时，派 Agent B（`agent-quiz-prompt.md`，轻量模型，只读代码出题）+ Agent C（`agent-exam-prompt.md`，只读"主代理整理的该模块分析材料"闭卷答），核对 C 是否覆盖 B 的 required_facts；没覆盖说明没吃透，回头补。最多 3 轮，仍有缺口如实标"待核实"。

**不要在此阶段主动写报告文件。** 等用户指令。

### Phase 5 — 生成报告（仅当用户明确要求）
触发词："生成 md""出报告""导出"等。读 `references/output-report-format.md`，把 Phase 2–4 的分析与探讨结论写成**一个** `.md`（默认 `<repo>/paper-code-analysis.md`，可由用户指定）。写完给路径 + 摘要。大项目先问要不要拆分。

## 关键规则
- **默认探讨、按令出报告**：没听到生成指令就别写文件。
- **只读不运行不改源码。** **先论文后代码。** **每条映射带出处。**
- 做自检时 **B 不读分析材料、C 不读代码**，否则闭卷失效。
- 拿不准标"待核实"，不臆断；逐模块跟踪进度。
