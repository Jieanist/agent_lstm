---
date: 2026-09-23
domain: pcie
symptoms: [timeout, ltssm-downgrade]
status: draft
scope: A0-stepping / board#2 / fw-20240911
related: []
---

# PCIe 链路 LTSSM 反复降级导致 timeout

- 芯片/stepping：A0（必填）
- 板卡：board#2 SN xxx（必填）
- 固件/内核/工具链：fw-20240911 / kernel 6.x / gcc xx（必填）

## 1. 现象（可观测 + 发生率）
- 上电后约 1/5 概率 LTSSM 降到 detect→polling，随后 device timeout；发生率约 20%。

## 2. 证据（寄存器快照带位宽 / 波形指针+关键值 / 日志路径+关键行 / 测量条件）
- 寄存器：PCIE_LTSSM_STATE=0x03（32bit 读，注意 W1C）。
- 波形：/data/scope/ltssm-20260923.sr，关键测量 t_pulse=12ns。
- 日志：dmesg | grep pcie 关键行 `link down, ltssm=detect`。

## 3. 排除路径（定量：排除了什么、怎么排、各花多久）
- 排除电源噪声（示波器测 3.3V 纹波 30min）。
- 排除时钟源（换晶振 1h）。
- 定位到复位时序（加 100ms 延时 2h）。

## 4. 根因与归因（HW bug[errata#] / SW bug / HW 的 SW workaround / 测量假象）
- SW bug：复位后过早配置 LTSSM，与 A0 上电时序不匹配。

## 5. 修复与适用范围
- 复位后加 100ms 延时再配置；适用 A0 + fw-20240911。

## 6. 沉淀与复检触发
- 沉淀指针 → errata.md / project.md 接口契约节。
- 复检触发：换 stepping/板卡/固件时复验该延时是否仍必要。

## 7. 演变（选附）
- （空）
