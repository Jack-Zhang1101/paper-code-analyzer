# 分析报告格式规范

仅在用户明确要求生成报告时使用。把 Phase 2-4 的分析与探讨结论落成一个 Markdown 文件，默认 `<repo>/paper-code-analysis.md`，用户可指定路径。

## Report Profile

默认 profile：`obsidian-reading-note`。

- `obsidian-reading-note`：默认。面向 Obsidian 阅读笔记，正文读者优先，不展示代码路径、`file:line`、commit、本地路径或 `[code: ...]`。
- `hybrid`：正文同 `obsidian-reading-note`，末尾增加 `Evidence Appendix`，用精简条目列出关键结论的代码证据。
- `engineering-trace`：完整工程溯源报告，可展示模块路径、`[code: file:line]`、commit、配置项、入口表和风险位置。

如果用户只说"生成 md / 出报告 / 同步 Obsidian"，使用 `obsidian-reading-note`。

## 共同质量标准

- 每条关键结论必须来自内部 evidence ledger；最终报告是否展示代码证据由 profile 决定。
- 每条 paper-code 映射保留三态结论：`[matched]` / `[gap]` / `[fundamental disagreement]` / `[partial]`。
- 拿不准写"待核实"，不要为了完整而编。
- 改进方向必须可操作，并能追溯到某条 gap、风险或过强假设。
- 公式使用 Obsidian 兼容写法：行内 `$...$`，独立公式 `$$...$$`。不要把 LaTeX 公式放进代码块。

## Obsidian Reading-Note 规则

默认报告必须遵守：

- 不写绝对路径、本地路径、commit hash、`file:line`、`[code: ...]`、`[config: ...]`。
- 不使用 Markdown 宽表格。任何 mapping / risk / benchmark 表都改成纵向 bullet。
- 可以保留窄的 ASCII 流程图、框架图、算法图；它们必须放在 `text` code fence 中，且单行不要超过约 100 字符。
- 不保留文件树、长代码块、长 API struct、长启动命令。
- 不写"代码路径/代码位置/文件路径"这类面向定位的栏目。
- 删除路径后必须做语义清理，不能留下"对应代码在。"、空编号、空 bullet、"和实际设备"这类断句。
- 章节围绕"读懂论文和方法"组织，而不是围绕"去哪几个文件看"组织。

适合的写法：

```markdown
- **现象**：FastSAC 早期达到 reward 35/38 更快，但高 reward 区域不一定更优。
- **判断**：它是早期 wall-clock 加速项，不是稳定性更强的最终算法。
- **原因**：off-policy replay、critic 估计、熵探索和接触任务 cliff 会放大曲线跳变。
- **建议**：用 fixed deterministic eval 区分训练曲线跳变和策略本体退化。
```

不适合默认报告的写法：

```markdown
| 模块 | 文件:行 | 证据 |
| --- | --- | --- |
| FastSAC | `x/y/z.py:529` | `[code: x/y/z.py:529]` |
```

## Obsidian Reading-Note 模板

```markdown
# <论文标题> - 论文代码阅读笔记

生成时间：<date>
论文：<标题 / arXiv / DOI>

## 0. 概览
- 一句话总结：<这个工作做了什么、关键贡献>
- 核心判断：<最重要的 3-5 条判断>
- 映射可信度：<高/中/低 + 主要不确定点>

## 1. 论文核心创新
### 1.1 <创新点>
<解决什么问题，相对前人的 delta，为什么 work>

## 2. 算法与公式拆解
### 关键公式
- **eq.N**：<符号定义、含义、作用>

### 算法流程
<步骤化说明。可用窄 ASCII 流程图辅助理解。>

## 3. 论文与实现的关系
### 3.1 总体映射
- **<论文元素>**
  - 论文作用：<它在方法里解决什么>
  - 实现情况：<matched/gap/partial>
  - 关键判断：<一句话>

### 3.2 关键 gap
- **<gap 名称>**
  - 现象：<实现和论文哪里不一致>
  - 影响：<会影响复现、性能还是可迁移性>
  - 判断：<严重程度和原因>

## 4. 核心机制详解
按用户关心的模块/创新点展开，不按文件路径展开。

每个机制建议使用：
- 论文设计
- 实现逻辑
- 一致性判断
- 可复用思路
- 风险

## 5. 工程风险与局限
- **高/中/低：<风险名>**
  - 现象：...
  - 影响：...
  - 建议：...

## 6. 改进方向
1. **<建议>**：依据 + 怎么改 + 预期收益。

## 7. 讨论形成的结论
保留和用户交互中新形成的判断，例如算法稳定性、速度提升、是否能迁移、是否该改成 on-policy。
```

## Hybrid 附录格式

`hybrid` 的正文必须仍然符合 `obsidian-reading-note`。末尾加：

```markdown
## Evidence Appendix

- **<关键结论>**
  - Paper：§X / Eq.N
  - Code：<相对路径:行号>，<一句话说明>
  - 状态：matched/gap/partial
```

附录只放关键证据，不要把所有 grep 结果倒进去。

## Engineering-Trace 模板

`engineering-trace` 可以使用表格和路径，但仍要避免无意义堆砌。

```markdown
# <论文标题> - Paper-Code Trace Report

## 0. Overview
- Paper: ...
- Repo: <url/path> @ <commit/tag>
- Scope: <read modules>

## 1. Paper Claims
<创新点、公式、实验主张>

## 2. Code Structure
- `<module>`: <职责>

## 3. Paper-Code Mapping
| Paper element | Paper ref | Code ref | Status | Notes |
| --- | --- | --- | --- | --- |

## 4. Formula Audit
| Formula | Paper ref | Code ref | Variables | Status |
| --- | --- | --- | --- | --- |

## 5. Extractable Modules
- Minimum dependencies
- Interface contract
- Side effects
- Required edits

## 6. Risks
| Severity | Code ref | Symptom | Impact | Recommendation |
| --- | --- | --- | --- | --- |
```

## 输出后检查

生成报告后做人工自检。默认 `obsidian-reading-note` 必须确认：

- 没有 Markdown 宽表格。
- 没有 `[code: ...]` 或 `[config: ...]`。
- 没有本地路径、绝对路径、commit hash、`file:line`。
- 没有"代码路径/文件路径/代码位置"这类定位栏目。
- 没有文件树、长代码块、长 API struct。
- 公式是 Obsidian 兼容的 `$...$` / `$$...$$`。
- 窄 ASCII 流程图可以保留，但不要变成长代码块。
- 删除路径后没有残句、空 bullet、空编号。

若发现问题，先修文档，再交付。
