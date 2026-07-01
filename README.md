# Paper-Code Analyzer

一个面向「论文 PDF + 对应代码仓库」的 Claude Code skill。从**论文方法**和**代码实现**两个层面把一个工作读懂，**交互式探讨**，并在你下令时产出一份分析报告。

当前版本：**v0.1**。v0.1 的主要变化是默认输出切换为 Obsidian 友好的 reading note，同时保留工程溯源报告作为可选模式。

> 零依赖，单目录即用 —— 不需要安装任何其他 skill / plugin / MCP / pip / npm。

## 能做什么

产出覆盖六个维度：

1. **论文核心创新** —— 提取 3–7 条创新点（解决的问题 / 相对前人的 delta / 为什么 work）
2. **算法与公式拆解** —— 逐式给符号、含义、作用；算法流程伪代码化
3. **论文 ↔ 代码映射** —— 内部保留 evidence ledger，最终报告按模式决定是否展示代码定位
4. **核心模块即插即用抽取** —— 最小依赖、接口契约、解耦点、改造钩子
5. **潜在工程风险** —— 可复现性 / 数值正确性 / 数据泄漏 / 训练稳定性，按高→低排序
6. **改进方向** —— 基于映射 gap 与风险，给出排序后、可操作的改造建议

适用：论文复现、代码阅读、科研调研、模型改造、新方法设计。

## 工作模式（交互式）

不是一键批处理：

1. 分析论文（子代理 P）+ 静态分析代码并做映射（子代理 A）——**只读，绝不运行/修改代码**
2. **默认停在「探讨」**：和你就论文↔代码来回交流，一起推敲公式实现、讨论模块抽取与改造
3. **只有你明确说「生成 md / 出报告」时**，才把结论落成一个 Markdown 报告

可选的 **ABC 闭卷自检**：子代理 B 读代码出题、子代理 C 只读分析材料闭卷作答，答不全说明分析有盲点 —— 用来保证"真的吃透了代码"。

## 报告模式

v0.1 开始，默认报告模式是 **`obsidian-reading-note`**：

- 面向 Obsidian 阅读笔记，优先保证可读性。
- 正文不展示代码路径、`file:line`、commit、本地路径或 `[code: ...]`。
- 默认不用宽 Markdown 表格；mapping / risk / benchmark 等内容会改成纵向 bullet。
- 保留公式、关键判断、算法逻辑、paper-code gap、可复用思路。
- 保留窄的 ASCII 流程图/框架图，因为它们对理解有帮助。

另外还有两种可选模式：

- **`hybrid`**：正文仍是 Obsidian reading note，末尾加 `Evidence Appendix`，保留关键代码证据索引。
- **`engineering-trace`**：完整工程溯源报告，保留 `[code: file:line]`、模块路径、commit、配置项、入口表和风险位置，适合复现、debug、代码审计。

触发方式：

- 「生成 md / 出报告 / 同步 Obsidian」→ 默认 `obsidian-reading-note`
- 「保留代码定位 / trace / 工程审计 / 复现定位」→ `engineering-trace`
- 「正文干净但附代码证据 / 附录保留证据」→ `hybrid`

## 安装

把整个目录拷进你的 Claude Code skills 目录即可：

```bash
git clone <this-repo-url> paper-code-analyzer
cp -r paper-code-analyzer ~/.claude/skills/
```

或直接下载/拷贝 `paper-code-analyzer/` 文件夹到 `~/.claude/skills/`。重开 Claude Code 即生效。

**前提**：仅需 Claude Code 本身（读 PDF、抓 arXiv、子代理均为内置能力）。无其他依赖。

## 用法

```
/paper-code-analyzer <paper> <repo>
```

- `<paper>`：论文 PDF 路径 或 arXiv URL/ID（如 `2508.12345`）
- `<repo>`：对应代码文件夹路径 或 GitHub URL

也可自然语言触发，如「分析这篇论文和它的代码」「读懂这篇论文的代码」「把核心模块抽出来」。

报告默认写到 `<repo>/paper-code-analysis.md`（可指定路径）。

默认生成的是 Obsidian reading note；需要完整代码定位时，在请求里明确说 `engineering-trace`。

## 文件结构

```
paper-code-analyzer/
├── SKILL.md                       # 主编排 + 交互流程
├── agent-paper-prompt.md          # 子代理 P：论文分析
├── agent-code-prompt.md           # 子代理 A：代码深读 + 论文映射 + 模块抽取 + 风险
├── agent-quiz-prompt.md           # 子代理 B：闭卷出题（可选自检）
├── agent-exam-prompt.md           # 子代理 C：闭卷答题（可选自检）
└── references/
    ├── static-analysis.md         # 静态分析方法论（扫描/config/入口/日志/风险）
    ├── paper-code-mapping.md      # 论文↔代码映射（溯源标签 + 公式核对 + round-trip）
    └── output-report-format.md    # 分析报告的章节与格式规范
```

## 设计来源（致谢）

融合并借鉴了两个优秀的开源项目：

- [CiferaTeam/deep-code-reader](https://github.com/CiferaTeam/deep-code-reader) —— 阶段化流程、子代理隔离、ABC 闭卷验证思想
- [fcakyon/phd-skills](https://github.com/fcakyon/phd-skills) —— 溯源标签、公式-代码逐项核对、round-trip 往返校验

在此基础上新增了：静态分析（含潜在工程风险扫描）、核心模块即插即用抽取、改进方向，以及交互式探讨 + 按指令出报告的工作模式。

## Version History

### v0.1

- 默认报告模式改为 `obsidian-reading-note`。
- 新增 `hybrid` 和 `engineering-trace` 两种可选报告模式。
- 将内部 evidence ledger 与最终阅读笔记正文分离：分析仍保留代码证据，默认报告不暴露代码路径。
- 增加 Obsidian 输出规范：不用宽表格，不放 `[code: ...]`、`file:line`、commit、本地路径；保留窄 ASCII 流程图。
- 将报告生成后的检查改为人工自检清单，不引入额外脚本依赖。

### v0.0

- 初始版本：以 paper-code mapping 和工程溯源为主，默认报告更偏代码审计/复现定位。

## License

MIT
