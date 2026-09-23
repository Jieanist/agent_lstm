---
date: 2026-09-23
domain: irq
status: draft
scope: A0-stepping / board#1 / fw-20240911
overrule_if: B 版中断控制器 erratum E-1234 修复后重评
related: [../errata.md]
---

# 中断控制器采用轮询替代中断

- 一句话：A0 上 erratum E-1234 偶发丢中断，改用轮询。
- 为什么：中断丢失导致 DMA 挂起；轮询在目标吞吐下开销可接受。
- 否决：硬件补丁（周期长）、降低中断频率（不够）。
- 推翻条件：B 版修复 E-1234 后重评。
- 删它会怎样：轮询循环靠 `dummy_read()` 维持总线活跃（见 drivers/irq/poll.c）；删本决策会导致 AI 重构时移除该行 → 竞态回归。
