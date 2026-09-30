# 通用编码风格（项目样板）

> 目的：让代码统一、可审、可维护，合入前能对照本文件逐条自查。
> 基准：Linux 内核编码风格（`Documentation/process/coding-style.rst`）为默认；本文件只写**通用增量 / 强调**项，不与内核风格冲突。
> 项目适配：每个具体项目以本文件为基底，把**项目特有**项（命名前缀、信任边界清单、常量表、状态机、示例）追加到对应小节即可，通用小节保持原文。

## 1. 整体风格 —— 以 checkpatch 为强制门

- 合入门：`scripts/checkpatch.pl --no-tree --strict` 对改动**零 error、零 warning**；`--strict` 的 CHECK 项逐条处置（修，或 commit message 说明豁免）。
- 通用易犯项（内核风格已含，点名强调）：
  - 缩进 **TAB（8 列）**，行宽 ≤ 100。
  - 命名：项目统一前缀（类型 `struct <p>_*`、函数 `<p>_*`、宏 / 常量 `<P>_*`）——前缀在「项目适配」里指定，全项目一致。
  - 错误处理：对外只返回**负 errno**，不返回正数错误码；多步资源获取用 `goto <label>` 单出口 unwind。
  - 内存：`kzalloc` / `kmalloc` 成对 `kfree`，`kzalloc` 优先；`kfree` 后不再用。
  - 日志：优先 `dev_*`（有设备）；`pr_*` 每文件应有 `pr_fmt` 前缀；消息一句英文、无感叹号 / 语气词；高频调用路径不放 info 及以上；可能 bug 短频触发的错误日志限频。

## 2. 入参校验 —— 跨信任边界必查

信任边界（通用；项目在「项目适配」补全自己的清单）：

- 用户态 → 内核：ioctl / sysfs / debugfs / 模块参数。
- 层间可替换边界：vtable / seam 函数指针（同模块也按「可能换实现」对待）。

每条边界必查（命中任一即返负 errno）：

- 指针 NULL；
- 长度 / 偏移溢出：`off + len < off`（防无符号回绕）、`off + len > window_size`；
- 枚举白名单（如 `width ∈ {1,2,4,8}`，其余 `-EINVAL`）；
- ABI magic 不匹配 → 拒绝；
- 状态机前置不满足 → `-EBUSY`；
- `copy_from_user`：返回值是**未拷贝字节数**，非 0 → `-EFAULT`。

返回码语义固定：`-EINVAL`（参数非法）/ `-EFAULT`（拷贝失败）/ `-ENODEV`（设备未就绪）/ `-EBUSY`（状态前置不满足）/ `-E2BIG`（超窗口）。

## 3. 注释 —— 英文、简洁、只写 why、不写需求

- **语言**：一律英文（含专有名词；禁止中文、全角标点、中英混杂）。
- **简洁**：优先一行；注释解释**为什么**（为何是这个魔法值 / 约束），不重复代码已自明的**是什么**。
- **只注释关键处**：非显而易见的魔法值 / 寄存器值、状态机约束、并发 / 锁、溢出 / 边界风险、ABI 布局不变量。
- **禁止**：
  - 中文注释；
  - 冗余注释（`i++; /* increment i */` 式）；
  - 把**需求 / 验收 / 开发过程 / 用户强调**写进注释（`验收 N`、`C1/C3`、`骨架`、`证据面`、`待 RTL 确认`、`某项目先例`、`commit xxx`）——归文档或项目记忆，不进代码；
  - 来源文件路径当注释主体——必要时一句英文 `source: …` 或省略；
  - 把**易变 / 非协议常量**写进注释：注释里的数字只允许是协议 / 寄存器规定的**长期不变量**；会随构建 / 调优变的量（镜像尺寸、超时值、BAR 尺寸）不进注释。

### 改写示例

| 坏                                                        | 好                                            |
| --------------------------------------------------------- | --------------------------------------------- |
| `/* 验收 2：写 GO 前必须校验镜像 */`                       | `/* GO only valid after image verified */`    |
| `/* 134MB 镜像 + 卡 Linux boot 全程 */`                    | `/* covers image transfer + card Linux boot */` |

## 4. 锁 —— 每把锁说清「护什么」、就近声明、一锁一范围

- **每把锁必须有注释说明它保护什么资源**：声明处一行 `/* guards: xxx, yyy */`，禁止裸 `static DEFINE_MUTEX(foo_lock);`。
- **锁与它保护的资源定义相邻**：同一 struct 内锁字段紧跟被保护字段；文件作用域静态量紧跟锁声明。
- **一锁一可见范围**：同一把锁只保护**同一可见范围**的资源；锁自身的声明位置 = 它保护资源的可见范围。
- **锁类型选择**：可睡眠临界区 → `mutex`；原子 / 中断上下文 → `spinlock_t`；简单标志 / 计数器 → `atomic_t` + `READ_ONCE` / `WRITE_ONCE`。
- **加锁规范**：同一函数内成对 lock / unlock；错误路径 `goto unlock` 单出口；持锁不睡眠；多锁注释锁序、全项目固定顺序。
- **可验证**：开 `CONFIG_PROVE_LOCKING`（lockdep）编译通过。

### 改写示例

```c
/* 坏 */
static DEFINE_MUTEX(foo_lock);
static int foo_state;
static struct foo_buf foo_buf;   /* data==NULL = 未加载 */

/* 某注释隔断 */
static u64 foo_done_baseline;

/* 好 */
/* guards: foo state machine — foo_state, foo_buf, foo_done_baseline (module scope) */
static DEFINE_MUTEX(foo_lock);
static int foo_state;            /* guarded by foo_lock */
static struct foo_buf foo_buf;   /* guarded by foo_lock; data==NULL = not loaded */
static u64 foo_done_baseline;    /* guarded by foo_lock */
```

## 整改方法（追溯既有违规）

- 用 `git blame` 定位每一违规行的**引入 commit**；
- 通过 `git rebase -i` 编辑**那个 commit** 就地改正，不新增「整改」commit；
- 重写后 force-push（需人授权）。

## 项目适配（每个项目照此做）

1. 复制本文件到项目 agent_lookup 仓库根（`CODESTYLE.md`）；
2. 复制 `scripts/checkpatch.pl` 到项目 `scripts/`；
3. 在对应小节追加**项目特有**项：命名前缀（`es_` / `ES_`）、信任边界清单（具体 ioctl / vtable）、常量表、状态机、示例。通用小节保持原文，不另立一套。
