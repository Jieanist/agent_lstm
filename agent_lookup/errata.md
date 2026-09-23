# errata — errata 与跨版本 workaround 登记簿（一行一条活条目；不抄原文，只记编号 + 工程结论）

| 编号 | 模块 | 一句话 | scope | workaround 状态 | 复检触发 |
|---|---|---|---|---|---|
| E-1234 | irq | 中断控制器偶发丢中断 | A0 | 采用：轮询替代（见 decisions/2026-09-23-irq-polling-vs-interrupt.md）（验证于 2026-09-23） | B0 修复后移除 |
| E-xxxx | pcie | LTSSM 降级 | A0 | 规避：复位时序加延时 | 复现时重测 |

（每条尾加 `（验证于 YYYY-MM-DD）`；B 版修复确认后删除对应行）
