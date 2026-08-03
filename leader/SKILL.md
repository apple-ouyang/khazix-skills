---
name: leader
description: 把一句话的想法拆成一份可审核、可执行的唯一 Spec。用户说「帮我给 agent 写个目标」「帮我详细拆一下这个目标」「写个任务书/brief 给 agent」「写个 goal 提示词」「让 agent 自己跑这个项目」「把活分给几个 agent 并行」时使用。先进代码库实测，必要时联网调研，再一次性提问（≤5 个），按 references/anatomy.md 产出包含实测数字、白名单、任务 DAG、防作弊验收和断点续跑的 Spec；非简单 Spec 交用户前必须通过 Terra reviewer 的 fail-closed 预审。默认不调用或生成 writing-plans/Plan，只有用户明确点名才允许。
---

# 领导：你出想法，我出一份 Spec

## 入口判断

先只读调研，再用普通文字展示判断和待确认项；讨论阶段不写文件、不建 worktree、不生成 taskbook 或 Plan。若用户明确要求的是简单、局部、路径唯一的直改，直接最小修改和验证，不创建 Spec、Plan、Goal、worktree、PROGRESS.md 或 BLOCKED.md。其它已收敛需求只创建一个 Spec 文件；不输出可复制 taskbook 或独立 Goal，默认不创建 Plan，只有用户明确点名才按流程补 Plan。

三个角色：**领导**（用户）出想法并审核唯一 Spec；**管理者**（父代理 Sol）调研、编写 Spec、收集预审和验收；**执行者**按 Spec 独立完成任务。执行者把 approved Spec 当唯一真源，不在中途等待口头补充。

## 流程

**1 调研。**自己能查的一律不问。有代码库就实测命令、基线数字、文档与实际差异；命令不存在或环境不可达时标明并写入任务 0。行业事实查不到就标「猜的，未验证」。

**2 提问，一轮 ≤5 个。**只问查不到且会改变范围、验收、风险或时间盒的分叉；每个问题给 2–4 个选项并标推荐。领导不在场时采用默认值，明确写入「我替领导拍的板（待确认）」并说明猜错代价。

**3 写 Spec。**读取 [references/anatomy.md](references/anatomy.md)，在 `main` 按模板持续修订唯一文件 `docs/superpowers/specs/YYYY-MM-DD-<主题>.md`。除简单直改外，Spec 本身必须保留可执行的 Task DAG：每个任务写清结果、依赖、`write ownership`、与邻接任务的接口、focused verification、集成顺序和失败后果；不得把任务合同推迟给 Plan。`## Goal` 与 Spec 同文件，第一句直接说明真实样本、目标 stage 和通过信号。

**4 Terra 预审。**非简单 Spec 在交用户前由独立 `reviewer=Terra/high` fail-closed 审核同一 draft：`PASS` 才能交用户；`BLOCK` 必须在原 Spec 修订后复审。预审不是用户的第二份文档，用户始终只审核这一份 Spec；简单直改不触发预审。

**5 确认。**预审通过后交领导审核，批准前保持 `review_status: draft`，不创建执行 worktree、不实施、不启动 `/goal`。只有用户明确点名时才调用 `superpowers:writing-plans` 生成 Plan；Plan 只能补实现细节，不能改写 Spec 的任务、依赖、ownership、接口或验收。

**6 执行与集成。**用户批准并明确继续后，Sol 先调用 `using-git-worktrees` 创建唯一 integration worktree，并从 approved Spec 建立 immutable integration tip；同一 wave 的所有 lane 都从该 tip 创建独立 branch/worktree。Sol 计算 `ready = 依赖已完成`，同时派发所有 `ready` 且 `write ownership` 不重叠的任务，不设固定 worker 数。每个 writer 是独立 branch/worktree 的 `worker=Luna/xhigh`；每个 lane 先完成 commit 与 focused verification，再交 `reviewer=Terra/high`，Terra `PASS` 前 Sol 不得集成。Sol 按 Spec 顺序串行集成，全部集成后运行 Spec 规定的综合验证，不再增加一次 Terra 汇总审查。执行、返工与两阶段清理统一遵循 `swarm`：综合验证通过前保留 lane worktree，方便退回原 Luna 修复；`merge-main` 成功并完成 post-merge verification 后才统一清理 lane branches 与 integration worktree/branch。已有 dirty worktree 只能作为只读基线，不接管、重启或改写。

## 写书规则

Spec 的结构、Schema 例子、实测、护栏、验收与探索型写法统一见 [references/anatomy.md](references/anatomy.md)。Spec 中 JSON 样例必须放在 `json` fenced code block，2 空格缩进，每个 key 单独成行。

先分型：能写出验收命令的是执行型；领导只要答案本身的是探索型，改用 anatomy.md 的学习目标与可复跑来源规则。已有测试、schema、验收脚本和设计稿直接作为规格引用，不用散文复述。把「不许」写成可判定的失败条件，把「建议」留给执行者并要求记录理由。

防作弊与止损必须保留：禁止 skip/todo、放宽断言、mock 被测对象、删测试、改验收阈值或 `|| true`；测试数和覆盖率不得低于基线。任务 0 核验关键前提，数字对不上就停；同一验收连续失败 3 次换项；结果比基线差就如实阻断。每个无法自行发现的静默失败都要做一次反向验证，贴出红→绿证据。执行阶段使用 `PROGRESS.md` 记录任务状态和证据、`BLOCKED.md` 记录未决偏差；Spec 审核阶段不创建这两个文件。

## 发出前自检

1. 只有一个 Spec 文件；没有 taskbook、独立 Goal 或未被点名的 Plan？
2. 任务 0 和按依赖排列的 Task DAG 是否齐全？任务 0 也必须是完整节点（只读 `write ownership`、失败阻断后续 ready）；每项是否有结果、依赖、`write ownership`、接口、focused verification、集成顺序和失败后果？
3. 非简单 Spec 是否已有 Terra `PASS`；批准前是否仍为 `draft`？
4. Goal 是否绑定最高保真、不会越权的真实样本和可判定命令，且没有复制任务级 RED/GREEN？
5. Sol 是否先建唯一 integration worktree 并固定 immutable tip，同 wave lanes 是否从同 tip 创建？是否保留 lane worktree 到 Sol 综合验证通过，并把问题退回原 Luna、重新经过 Terra 审查？失败、未集成、开放 PR 或证据缺失是否保留恢复点？
