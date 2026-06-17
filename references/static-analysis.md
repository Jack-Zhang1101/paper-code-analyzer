# 静态分析方法论（不运行用户代码）

这是**方法论**，不是脚本。对具体项目按下面思路用 Grep/Glob/Read 或只读 shell **现场生成**命令，结果回到主对话用于探讨、并在用户要求时写进报告。**全程只读，绝不运行训练/推理、绝不改源码。**

六块对应报告里的"代码结构 / 模型配置 / 训练参数 / 实验入口 / 日志 / 潜在工程风险"。

## 1. 仓库扫描与模块边界
- 列结构，忽略 `.git/`、缓存、数据、checkpoint、`node_modules/`、`__pycache__/`。
- 模块边界启发式：顶层 `src/`/`lib/`/`pkg/`/`packages/` 或根下一级目录；Python 包(`__init__.py`)、Go 包、Node 包(`package.json`)；已有的 README 目录说明 / manifest。
- 每个候选模块读入口 + `__init__` 导出，写一句话职责。
- 统计各模块行数：判断是否值得深读、报告里是否单列。

## 2. 配置提取（模型配置 / 训练参数）
定位配置载体：`*.yaml/yml/json/toml`；`argparse`/`click`/`absl.flags`(搜 `add_argument`/`DEFINE`)；Hydra(`@hydra.main`/`conf/`)、OmegaConf；Gin(`@gin.configurable`/`.gin`)；代码内常量/dataclass/`Config` 类。
抽关键超参并归类（便于和论文对照）：
- **optimizer**：名称、peak lr、schedule、warmup、weight decay、betas、eps、grad clip
- **batch**：per-device bs、设备数、global bs(注明是否=乘积)、grad accumulation
- **schedule**：epochs/steps、eval/save 频率
- **augmentation**：增广**顺序**（顺序改变统计量），不只是列表
- **loss**：每项权重/温度/label smoothing，对应论文哪个 eq 的哪个系数
- **model**：层数/宽度/heads/激活/归一化、关键结构开关
- **tricks**：mixup/cutmix、stochastic depth、EMA、grad checkpointing、混合精度、权重初始化

每个值记**出处**(file:line)。注意"默认值陷阱"：argparse default、config base+override 叠加后真正生效的值。

## 3. 入口定位（训练 / 推理 / 数据）
- **训练**：搜 `if __name__ == "__main__"`/`def main(`/`train.py`/`trainer.fit`/`torchrun`/`accelerate launch`/`Makefile`/`scripts/*.sh`/`*.slurm`。记：启动命令、读哪个 config、调用链第一跳。
- **推理/eval**：搜 `eval`/`test`/`inference`/`predict`/`@torch.no_grad`/`model.eval()`/metric 计算。记：加载哪个 ckpt、吃什么输入、出什么指标。
- **数据**：搜 `Dataset`/`DataLoader`/`load_dataset`/`build_dataloader`、数据根路径、split。记：数据集、split 比例、预处理在哪。
- 产出"入口表"：`| 用途 | 启动命令/文件:行 | 读取的config | 关键依赖 |`。

## 4. 日志摘要（只读已有产物，不重跑）
找 `logs/`/`outputs/`/`runs/`/`wandb/`/`*.log`/`events.out.tfevents.*`/`results*.json`/`*.csv`/README 表格。
静态提取（不启服务）：最终/最佳指标、曲线趋势、报告的 epoch/step/bs/设备/耗时、异常痕迹(NaN/Inf/OOM/early stop/反复 warning)。
用途：和论文自报数字交叉核对；作为风险线索。没有日志就写"无"。

## 5. 调用关系 / 依赖图
模块间 import 关系（搜 `import`/`from ... import`）建有向依赖；标出主干 数据→模型→loss→optimizer→eval。

## 6. 潜在工程风险扫描 ⚠️（重点，喂给"改进方向"）
不运行的前提下标记影响 **正确性 / 可复现性 / 复用移植** 的风险。每条给 `位置(file:line) + 现象 + 影响 + 建议`，按 高/中/低 排序。
- **可复现性**：随机种子是否设全(python/numpy/torch/cuda)、`cudnn.deterministic/benchmark`、DataLoader `worker_init_fn`；硬编码/绝对/作者机器路径；依赖版本未固定。
- **数值正确性**：reduction(mean vs sum)、归一化因子、`eps` 缺失致除零/log(0)、softmax 稳定、温度系数位置；该 detach/no_grad 是否漏；in-place 破坏 autograd；混合精度溢出/loss scaling。
- **数据**：train/val/test 泄漏(同源/shuffle 后切/统计量用全集)；split 与论文不符；标签/索引 off-by-one。
- **训练稳定/资源**：无 grad clip 而 lr 偏大；小 batch/分布式下 BN 行为；设备假设(写死 `cuda:0`)；显存峰值；checkpoint 兼容(state_dict vs 整模型、`strict` 加载)。
- **工程卫生**：死代码、与论文不符的注释、`TODO/FIXME/HACK`、被注释掉的关键逻辑、base+override 后"实际生效值"反直觉。

> 风险是"提示"不是"判决"：拿不准标"待核实"，不臆断。

## 输出去向
- 第 1/5 块 → 报告 §3 模块与依赖图。
- 第 2/3 块 → 报告 §3 配置/入口，并参与论文↔代码映射(§4)。
- 第 4 块 → 报告 §3 日志摘要与数值核对。
- 第 6 块 → 报告 §6 工程风险，并直接作为 §7 改进方向的输入。

（交互模式下，这些先回主对话支撑探讨；仅当用户要求时按 `output-report-format.md` 落地。）
