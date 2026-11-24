# Lyra 模块参考手册（字典式速查）

本目录存放按模块切分的 Lyra Starter Game（UE 5.8）技术参考手册。若需按认知顺序学习“为什么这样设计”，请前往教程书 **[《从零读懂 Lyra》](../Book/README.md)**；返回总览见 **[文档总览](../README.md)**。

每份手册统一包含：**§1 职责概述 → §2 关键类表 → §3 核心流程与数据流 → §4 跨模块接口 → §5 数据驱动点 → §6 扩展与修改配方 → §7 坑与注意事项 → §8 函数级调用链 → §9/§10 设计意图与速查问答**。

| 编号 | 文档 | 覆盖范围 |
|---|---|---|
| **10** | [`10-CoreFramework.md`](10-CoreFramework.md) | `GameModes`（Experience）、`GameFeatures`、`System`（AssetManager/GameInstance）、`Player`、`Character`（PawnExt/Hero/InitState）、`Development`、`ModularGameplayActors` |
| **20** | [`20-AbilitySystem-Input.md`](20-AbilitySystem-Input.md) | `AbilitySystem`（ASC、AbilitySet、Phases、GlobalAbilitySystem、Executions、Attributes）、`Input`（EnhancedInput + `InputTag`） |
| **30** | [`30-GameplaySystems.md`](30-GameplaySystems.md) | `Inventory`、`Equipment`、`Weapons`、`Interaction`、`Teams` |
| **40** | [`40-UI.md`](40-UI.md) | `UI`、`CommonGame`、`UIExtension`、`CommonLoadingScreen`、`IndicatorSystem`、`Frontend` |
| **50** | [`50-Settings-Online-Platform.md`](50-Settings-Online-Platform.md) | `Settings`、`GameSettings`、`CommonUser`（登录与会话）、`Hotfix`、`Performance` |
| **60** | [`60-Presentation.md`](60-Presentation.md) | `Camera`、`Animation`、`Cosmetics`、`Feedback`（ContextEffects/NumberPops）、`Audio`、`Messages`、`Replays` |
| **70** | [`70-GameFeatures-Content.md`](70-GameFeatures-Content.md) | `ShooterCore`、`ShooterMaps`、`TopDownArena`、`ShooterExplorer`、`ShooterTests` 及项目内自定义插件资产拆解 |
| **80** | [`80-Build-Config.md`](80-Build-Config.md) | `.uproject`、`*.Build.cs`、`*.Target.cs`、`Config/*.ini` 配置全景与新增模块/通道/主资产菜谱 |
| **95** | [`95-Verification-Report.md`](95-Verification-Report.md) | 参考手册跨模块源码抽样核验报告与原版未实现代码汇总 |

---

版权与许可说明见仓库首页 [`README.md`](../README.md) 与 [`LICENSE`](../LICENSE)。
