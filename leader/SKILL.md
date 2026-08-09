---
name: leader
description: 把一句话的想法收敛成一份简短、可审核的唯一 Spec。用户说「帮我给 agent 写个目标」「帮我详细拆一下这个目标」「写个任务书/brief 给 agent」「写个 goal 提示词」「让 agent 自己跑这个项目」「把活分给几个 agent 并行」时使用。先进代码库实测，必要时联网调研，再一次性提问（≤5 个）。Spec 只放用户需要审核的目标、行为、风险、并发分组和完成标准；详细 Task DAG、文件 ownership、worker 调度和验证证据只在执行开始后写入临时 PROGRESS.md。
---

# 领导 · 你出想法，我出一份 Spec

三个角色：**领导**（用户）出想法、拍板；**管理者**（你）调研、写 Spec、组织审核；**执行者**在批准后独立完成。沿用开源版任务书的拍板、范围、实测、任务、规矩和完成条件结构，只把不需要领导审核的执行细节延后写入临时 `PROGRESS.md`。

## 输出边界

Leader Spec 是用户的决策合同，不是执行计划或进度日志。非简单任务只创建一份 `docs/superpowers/specs/YYYY-MM-DD-<主题>.md`，目标 1500–3000 字符，硬上限 4000 字符。压不进去时删执行细节，不是让用户审核更长的文档。

领导只审核：

- 为什么做、改完的可观察结果、范围与非目标。
- 尚未确认的业务取舍，以及 schema/API/持久化合同、权限、真实花费、外部 mutation、不可逆数据和安全护栏。
- 结果级并发分组：哪些先串行、哪些可并行、哪些最后串行，以及为什么。
- 完成标准和不可越过的副作用边界。

不进入 Spec：文件/函数清单、完整 Task DAG、`write ownership`、worker 数量与名称、branch/worktree 名称、逐任务命令、RED/GREEN 过程、reviewer 调度和集成步骤。这些是执行者责任，不得转嫁给用户审核。

## 流程

**1. 调研门禁。** 写第一版 Spec 前，先列出会改变目标、风险、合同或验收的未知项，并分成“可查事实”和“必须由用户决定的取舍”。可查事实先查本地代码、测试、日志、Git 历史和项目文档；需要外部最新资料或陌生技术事实时，委派只读 `explorer`，默认要求其使用 `exa-search` 并优先官方或一手来源。`exa-search` 不可用时明确降级和未验证范围，不伪造搜索结果。

存在至少两个互相独立、不会重复调查的证据 workstream 时，按 `swarm` 并发多个直接 `explorer`；每个 assignment 只负责一个问题或 evidence family。调研阶段不创建 Spec、执行 worktree、Plan、PROGRESS 或业务改动。Explorer 结果只是证据，Sol 负责去重、处理矛盾并判断是否真的查清。

只有每个关键事实都有可定位的本地证据或带 URL/日期的外部来源，矛盾证据已有解释，且剩余未知项只属于用户取舍时，才进入提问和写 Spec。仍无法查清且会改变设计或验收的关键事实属于阻塞：先报告证据缺口，不写猜测版 Spec。

**2. 提问。** 完成上述调研后，一轮最多 5 个，只问无法靠证据决定且会改变目标、风险、并发分组或验收的用户取舍。每个问题给 2–4 个选项、推荐默认和猜错代价。领导不在场时不能把代拟决定藏起来，写入「我替领导拍的板（待确认）」。

**3. 写 Spec。** 完整读取 [references/anatomy.md](references/anatomy.md)，在 `main` 持续修订唯一 draft。简单、局部、低风险且路径唯一的直改不创建 Spec。

写 Spec 时沿用开源版的判断方法：目标写成结果；用户指定的手段先当待验证假设；已有测试、schema、验收脚本和设计稿直接作为规格引用，不用散文复述。把「不许」写成可判定的失败条件，把「建议」留给执行者临场判断。

**4. 用户审核。** 先把唯一 draft 交给用户，批准前保持 `review_status: draft`，不创建 Terra reviewer、执行 worktree、Plan、PROGRESS 或业务改动。用户修改意见始终只更新这一份 Spec。

**5. Terra 精简复审。** 用户明确批准后，创建一个独立 `reviewer=Terra/high`，在同一 assignment 要求使用 `solution-simplifier`，只审查用户合同是否自相矛盾、过度设计或缺安全边界。Terra 不得要求把执行计划补进 Spec，也不得修改 Spec。

Terra `BLOCK` 后按 semantic diff 而不是改动名称分类：只有不改变任何义务、允许/禁止、范围、行为、阈值、示例含义，以及目标/非目标、schema/API/持久化合同、权限、外部 mutation、花费、安全护栏、并发分组或验收阈值的纯措辞、解释或内部精简，才由 Sol 更新同一 draft 并让同一 Terra 复审，不机械要求用户重批。

改动上述任一语义，或无法确定是否影响语义时，原批准失效并保持 `draft`；Sol 修订同一 Spec 后先把最终合同重新交给用户确认，再让同一 Terra 复审。只有用户批准的最终关键合同已被 Terra `PASS` 覆盖时，才置为 `approved`。

**6. 执行。** 只在用户批准、最终关键合同已被 Terra `PASS` 覆盖且用户明确继续后，才创建唯一 integration worktree。Sol 基于 approved Spec 在该 worktree 创建临时 `tmp/<task-slug>/PROGRESS.md`，再写完整 Task DAG、依赖、文件 ownership、lane/commit/reviewer/验证状态、阻塞和恢复点。只有用户显式点名时才创建 Plan。

PROGRESS 是任务状态真源。普通失败、可重试异常和 agent 能自行修复的问题只写 PROGRESS。只有某项工作确实无法自行推进、需要用户拍板或外部协助时，才按需创建同目录 `BLOCKED.md`：PROGRESS 只记录 `blocked` 状态、摘要和 blocker ID；BLOCKED 按 ID 记录完整原因、证据、已尝试方案、影响范围和所需协助，不复制任务状态。

Spec 批准的是结果级并发分组，不是固定 worker 数量。Sol 在 PROGRESS 中计算 `ready = 依赖已完成`，只并行 ready 且 ownership 不重叠的 lane；每个 lane 先验证和 Terra `PASS` 再集成。

PROGRESS 在任务未合入、验证失败、存在阻塞或保留恢复点时必须保留。BLOCKED 有未解决项时不得宣称完成；阻塞解决后更新 PROGRESS，重大裁决同步回 Spec，BLOCKED 无未解决项即可删除。只有用户显式调用 `merge-main`、合入成功、post-merge verification 通过且无未解决项时，才自动删除该临时 PROGRESS。worktree 和 branch 的清理由 `swarm` 与 `merge-main` 各自负责，Leader 不重复规定。

## 发出前自检

1. Spec 是否 ≤4000 字符，且只有一份用户合同？
2. 「领导已经拍板」「我替领导拍的板（待确认）」「范围与非目标」是否都保留，且确认来源没有混写？
3. `review_status` 是否只使用 `draft | approved`，且仅在用户批准、Terra 已覆盖最终关键合同后进入 `approved`？
4. 是否只写用户需审核的决定，没有文件清单、Task DAG、ownership、worker 和逐任务命令？
5. 并发是否只用“先串行 / 可并行 / 最后串行”的结果级分组，没有把调度责任交给用户？
6. schema、API、持久化、权限、花费、外部 mutation、不可逆数据和安全护栏是否有具体可审核的行为？
7. 关键可查事实是否在写 Spec 前完成取证；并行 Explorer 是否覆盖互不重复的问题；本地证据是否可定位、外部证据是否带 URL/日期；无法查清且影响合同的事实是否停止写 Spec？
8. 防作弊、静默失败的反向验证和连续失败止损是否有对应完成信号？
9. 非简单 Spec 是否先交用户批准、再经 Terra 精简复审；不确定或涉及关键合同的修订是否重新取得用户确认和 Terra `PASS`；`approved` 前是否没有 worktree、Plan、PROGRESS 或实施？
10. BLOCKED 是否只在需要用户拍板或外部协助时按需创建，并由 PROGRESS 通过 blocker ID 引用？
