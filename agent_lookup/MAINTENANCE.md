memory-schema: v2

# agent_lookup — 长期维护纪律（写文件 / 维护时查这个；开工不必整读）

> 本文件 = 「每个文件怎么维护更新」的纪律，是这套系统两年不乱的根基。
> 开工操作见 `INDEX.md`；本文件管「怎么写才不乱、乱了怎么发现、人多久对一次账」。

## 一、每个文件的维护纪律（核心）


| 文件                                   | 谁写               | 何时动                      | 怎么动                         | 硬规则                                                       | 腐化迹象（该查什么）                   |
| -------------------------------------- | ------------------ | --------------------------- | ------------------------------ | ------------------------------------------------------------ | -------------------------------------- |
| `INDEX.md`                             | 人（Agent 可提议） | 改流程 / 加红线             | 直接编辑                       | 只放规则，不放任何条目/目录                                  | 出现具体条目或清单                     |
| `MAINTENANCE.md`                       | 人（Agent 可提议） | 规则变了                    | 直接编辑                       | 每条规则可执行、可被 check 校验                              | 规则与`memory.sh check` 不一致         |
| `project.md`                           | Agent 起草+人审    | 架构/接口/约束真变了        | 整节替换 + 更新该节「验证于」  | 不追加（会成流水账）；可从代码推导的不写；**目标节「交付目标」与「预研现状」分两行，不许把预研平台写成主语** | 出现操作步骤；日期戳久未动             |
| `status.md`                            | Agent（gen 生成【待审】） | 每次开工重写【现在】  | 【现在】覆盖写；【待审】gen 重建 | 不追加；【现在】≤20 行                                      | 【现在】开始记历史流水                 |
| `short_term.md`                        | Agent              | 会话内完成小步              | 只追加一行指针                 | 不复述正文；≤40 条；按「工作线被取代」裁剪                  | 复述正文、只增不剪                     |
| `hardware.md`                          | Agent 起草+人审    | 换板卡/固件/stepping        | 整节替换 + 更新「验证于」      | 旧硬件戳标「已停用」别删（要追溯旧决策的适用域）；不写密钥值 | 硬件戳与实际不符                       |
| `errata.md`                            | Agent 起草+人审    | 确认 errata/workaround/修复 | 加一行；B 版修复确认后删对应行 | 每条带「验证于」+「复检触发」                                | workaround 长期「待验证」              |
| `decisions/`                           | Agent 起草+人审    | 拍板时                      | 一文件一决策；正文只增不删     | ≤10 行；推翻=新决策`superseded_by` 旧（不删旧）             | active 互相矛盾、>10 行                |
| `incidents/`                           | Agent 起草+人审    | bug 修复验证通过时          | 一文件一事故；正文只增不删     | 字段齐全                                                     | 缺字段、症状词不在词表                 |
| `CATALOG.md`                           | 脚本（gen）        | 开工                        | 重建                           | 永不手改                                                     | 被人手改过                             |
| `tools/memory.sh` + `hooks/pre-commit` | 人                 | 规则变 / 发现 bug           | 编辑后重跑 check + 自测        | 改动后在真实提交里验证 hook                                  | check 报错没人管、`--no-verify` 成习惯 |

## 二、命名（decisions/ incidents/）

- `YYYY-MM-DD-<域>-<症状|主题>.md`（ASCII；同日多篇加 `-2`）；域取自词表；**归档后不改名 / 不移动 / 不建年目录**。

## 三、frontmatter（≤9 字段，平铺可 grep，不用 tags）

```
---
date: 2025-06-12
domain: pcie
symptoms: [timeout, 超时, ltssm-downgrade]     # incident 必填
status: draft                                # decision: draft|active|superseded|rejected；incident: draft|open|workaround|resolved|rejected
review: 人审评语（可选，一条一句）
superseded_by: 2025-08-01-irq-polling-revisit.md   # 被取代时填（引文件名）
scope: A0-stepping / board#1 / fw-20240911   # 适用范围，驱动域必填
overrule_if: B 版 erratum E-1234 修复后重评   # decision 必填
related: []                                  # 双链，可空
---
```

- freshness 用内联 `（验证于 YYYY-MM-DD）`（project/hardware 各节尾、errata 每条尾）；status 缺省一律 `draft`。

## 四、受控词表（封闭集；加词 = 改本文件 + 同步 memory.sh 的 DOMAINS）

- 域(10)：pcie dma irq clk power mem bus boot tool misc
- 症状(16)：timeout / hang / crash / deadlock / leak / oob / garbage / misalign / flaky / regression / bootfail / perf / compat / panic / silent / ltssm-downgrade

## 五、git 纪律

- 消息 `memory(<域>|<文件>): 摘要` 一次一事；**记忆提交**用 `-- agent_lookup/` pathspec（禁 add -A / add . / commit -a）；**根目录工件**（AGENTS.md / README.md / skills/）正常提交。
- **commit 一律英文**：摘要 + 正文全英文（`memory(<域>|<文件>): <English summary>`），禁中文 / 中英混杂。
- **禁止 Agent 自发 push**：`git push`（含 force-push）属**人授权动作**——只有人明确说「push / 推送 / 提交并推送」时才执行；Agent 默认只 commit，不自行 push。
- **提交粒度两层**：①工作期间每落一条 draft 立即 `memory(decision|incident): 摘要`（一条一提交，可单条追溯/回滚）；②每轮批审应用完一次 `memory(review): 批审 N 条`（一轮一提交，作开工前快照）。别把一轮内多条 draft 压成一个提交。
- 禁对 agent_lookup/ reset --hard / rebase / amend；密钥值绝不入记忆（只写「存在哪 / 找谁」）。
- 回滚：`log --follow` → `show` 预览 → `restore --source` → 新提交固化（优先向前修正）。

## 六、对账节奏（人必须守的两个点 + 一个习惯）

- **开工批审（一轮一次；人只判、AI 应用）**：
  - 人：读【待审】各条**全文**，keep/改/drop = 改 frontmatter `status`（draft→active/rejected/resolved）+ 评语写 `review:` 一条一句，然后对 AI 说「审完了」。
  - AI：读 `status` + 各 `review:` → 把指令应用到派生文件（`status.md`【现在】/`short_term.md`/`errata.md`）→ `gen` 重建 → 一次 `memory(review): 批审 N 条` 提交。
  - **勿在正文末尾堆评语、勿手点 status.md 勾勾**（勾勾是 `gen` 生成的）。
- **月度 30s**：`memory.sh check` + 扫上表「腐化迹象」列 + 对 `status` 与 `git log -1` 的时差。
- **引用习惯**：引用任何慢变事实前，先 git log / 现场探测对照（记忆是缓存，repo 是真相）。

## 七、临时文件（scratch/）

- 中间工具性文件、输出临时结果一律放 `scratch/`，**不放外部工作区、不放 /tmp**；内容不入版本库（`scratch/.gitignore`）。
- **超阈值清理**（默认 100MB，见 `scratch/README.md`）：①先清「数据 / 结果类」文件（可直接重建的产物 / 日志 / 测试输出）→ ②再清「日期较久」的文件（按 mtime 旧→新）。

## 八、升级信号（任一命中即评估，先于事故报警）

N≥50 每次 gen ｜ N≥150 脚本查询接口 ｜ 月内 R6≥3 扩词表 ｜ status 与 git log 时差 >14 天连续两轮 ｜ 待审 ≥10 条且 >7 天 ｜ 常驻 >12KB 瘦身 ｜ 硬件组合 >2 维升矩阵 ｜ 多人共享→独立 git ｜ 工具换代重接钩子 ｜ 描述性长查询频发→才评估向量检索。

## 九、风险速查（无法根除，靠纪律 + 升级信号）

静默权威谎言→引用前 git log/现场探测 ｜ 检索失效→R6 ｜ 硬件轴漂移→scope+现场探测 ｜ 写回断裂→开工批审 ｜ 密钥入库→红线 ｜ 钩子失效→换工具重验 ｜ 换岗→交接+健康分。
