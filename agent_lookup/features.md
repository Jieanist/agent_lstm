# features — feature 范围 vs 实现现状对账

> 目标列 = 交付 feature；状态列 = 代码真实现状。图例：✅ 已实现 ｜ ⚠️ 部分/stub ｜ ❌ 未实现。

## 交付 feature

| feature | 状态 | 关键点 |
|---|---|---|
| 读配置文件 | ❌ | 配置格式 / 解析方式待定 |
| load 动态库 | ❌ | 加载方式 / 路径来源待定 |
| 按配置调接口 | ❌ | 函数签名 / 参数映射待定 |

> 状态 = 代码真实现状（✅ = 已实现，未在真机验证）；feature 边界以 `project.md`【目标】为准。

## 变更记录

- 首次建 `features.md`（对齐 es-pcie-drv-agent_lookup 风格：feature 目标 vs 现状对账）。
