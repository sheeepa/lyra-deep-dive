# Lyra Starter Game（UE 5.8）中文文档总览与参考手册

> 一套针对 UE 5.8 `LyraStarterGame` 的中文技术文档，**不包含** Lyra 工程本体，也不含 Epic 的任何原始源码文件与二进制资产。
> 除引用的 Epic 官方代码片段外，本仓库原创内容统一采用 **[CC BY-SA 4.0](LICENSE)** 授权，详见文末声明。

本仓库汇集了针对 UE 5.8 `LyraStarterGame` 编写并经源码与编辑器实测核验的技术文档。请按你当前的场景选择入口：

- **想从头建立心智模型、系统读懂 Lyra 为什么这样设计** → 直接进入核心教程书 **[《从零读懂 Lyra》](Book/README.md)**（第 0-22 章 + 附录 A1-A3）。
- **已经在改功能、想查某个模块的类表、函数级调用链或速查问答** → 查下方 **[模块参考手册（10-80 号）](Reference/README.md)** 或 **[附录 A3 源码地图](Book/A3-CodeMap.md)**。
- **想亲手从零搭一个完整的新玩法插件** → 直接跟做 **[第 22 章 综合项目：“最后存活者”模式](Book/22-Capstone.md)**（独立工作坊手册见下方，待上传）。
- **遇到静默失效、不知道下什么断点或开哪条日志** → 查 **[附录 A1 调试工具箱](Book/A1-Debugging.md)**。

---

## 一、核心教程书：《从零读懂 Lyra》

入口：**[`Book/README.md`](Book/README.md)**（写作与译名规范见 [`Book/_StyleGuide.md`](Book/_StyleGuide.md)）

| 部分 | 章节范围 | 解决的核心问题 |
|---|---|---|
| **写在前面** | [第 0 章](Book/00-HowToUse.md) | 环境准备、编辑器启动地图 vs 游戏默认地图、PIE 联机与控制台用法 |
| **第一部分：UE 基础** | [第 1 章](Book/01-UObject.md) - [第 4 章](Book/04-DataAndAssets.md) | `UObject`/反射/CDO、Gameplay Framework、服务器权威与复制、主资产/Tag/委托/配置 |
| **第二部分：Lyra 的骨架** | [第 5 章](Book/05-BirdsEye.md) - [第 8 章](Book/08-PawnAssembly.md) | 五层架构与 12 步启动链、Experience 与 GameFeature、`GameFrameworkComponentManager` 组件注入、PawnData 与 `InitState` 状态链 |
| **第三部分：技能系统（GAS）** | [第 9 章](Book/09-GASConcepts.md) - [第 12 章](Book/12-DamageAndDeath.md) | GAS 核心概念、ASC 放在 `PlayerState`、`AbilitySet`、增强输入与 `InputTag`、伤害执行与死亡重生链路 |
| **第四部分：玩法系统** | [第 13 章](Book/13-Inventory.md) - [第 15 章](Book/15-TeamsInteractionMessages.md) | 物品定义/实例与 Fragment、远程武器客户端检测与热度扩散、队伍/交互/全局消息总线 |
| **第五部分：表现层** | [第 16 章](Book/16-Camera.md) - [第 17 章](Book/17-AnimCosmeticsEffects.md) | 相机模式栈与权重混合、Tag 动画属性映射、角色外观部件、表面特效与伤害数字 |
| **第六部分：UI** | [第 18 章](Book/18-UIFramework.md) - [第 19 章](Book/19-HUDAndLoading.md) | CommonUI 四层栈与输入路由、`UIExtension` 槽位注入、世界空间指示器与加载屏 |
| **第七部分：平台与工程** | [第 20 章](Book/20-Settings.md) - [第 21 章](Book/21-OnlineAndBuild.md) | 本地/共享双层设置与动态反射路径、CommonUser/CommonSession 开房进图、构建变体与打包 |
| **第八部分：综合实战** | [第 22 章](Book/22-Capstone.md) | 不改 `ALyraGameMode`，从零搭建完整的“最后存活者（Last Man Standing）”插件模式 |
| **附录** | [A1 调试](Book/A1-Debugging.md) · [A2 术语表](Book/A2-Glossary.md) · [A3 源码地图与已知坑](Book/A3-CodeMap.md) | 按症状排查清单、中英术语对照、“想做什么→去哪找”速查表与 54 条原版已知坑 |

---

## 二、模块参考手册（字典式速查）

入口：**[`Reference/README.md`](Reference/README.md)**。每份手册统一采用固定结构：**§1 职责 → §2 关键类表 → §3 核心数据流 → §4 跨模块接口 → §5 数据驱动点 → §6 扩展配方 → §7 坑与注意事项 → §8 函数级调用链 → §9/§10 设计意图与速查问答**。

| 编号 | 文档 | 覆盖源码与插件范围 |
|---|---|---|
| **10** | [`10-CoreFramework.md`](Reference/10-CoreFramework.md) | `GameModes`（Experience）、`GameFeatures`、`System`（AssetManager/GameInstance）、`Player`、`Character`（PawnExt/Hero/InitState）、`Development`、`ModularGameplayActors` |
| **20** | [`20-AbilitySystem-Input.md`](Reference/20-AbilitySystem-Input.md) | `AbilitySystem`（ASC、AbilitySet、Phases、GlobalAbilitySystem、Executions、Attributes）、`Input`（EnhancedInput + `InputTag`） |
| **30** | [`30-GameplaySystems.md`](Reference/30-GameplaySystems.md) | `Inventory`、`Equipment`、`Weapons`、`Interaction`、`Teams` |
| **40** | [`40-UI.md`](Reference/40-UI.md) | `UI`、`CommonGame`、`UIExtension`、`CommonLoadingScreen`、`IndicatorSystem`、`Frontend` |
| **50** | [`50-Settings-Online-Platform.md`](Reference/50-Settings-Online-Platform.md) | `Settings`、`GameSettings`、`CommonUser`（登录与会话）、`Hotfix`、`Performance` |
| **60** | [`60-Presentation.md`](Reference/60-Presentation.md) | `Camera`、`Animation`、`Cosmetics`、`Feedback`（ContextEffects/NumberPops）、`Audio`、`Messages`、`Replays` |
| **70** | [`70-GameFeatures-Content.md`](Reference/70-GameFeatures-Content.md) | `ShooterCore`、`ShooterMaps`、`TopDownArena`、`ShooterExplorer`、`ShooterTests` 及示例插件资产拆解 |
| **80** | [`80-Build-Config.md`](Reference/80-Build-Config.md) | `.uproject`、`*.Build.cs`、`*.Target.cs`、`Config/*.ini` 配置全景与新增模块/通道/主资产菜谱 |
| **95** | [`95-Verification-Report.md`](Reference/95-Verification-Report.md) | 参考手册跨模块源码抽样核验报告与原版未实现代码汇总 |

---

## 三、动手工作坊（实战演练）

待上传

---

## 关于书中的源码引用与本地跳转

正文里所有 `Source/...`、`Config/...`、`Plugins/...` 形式的路径，指的是**你本地 `LyraStarterGame` 工程**中的相对位置，而非本仓库内的文件。本仓库仅收录文字与图表文档，不分发引擎或示例工程的任何原始源码与资产。

> **本地跳转技巧**：如果你将本仓库克隆到你本地 Lyra 工程的 `<LyraStarterGame>/Docs/lyra-deep-dive/` 目录下，书中形如 `../../../Source/LyraGame/...` 的相对链接在 VS Code、Rider 或 Obsidian 中均可直接点击跳转到本地对应源码文件。

---

## 版权与许可

除下方明确排除的第三方引用内容外，本仓库全部原创内容均采用 **[CC BY-SA 4.0](LICENSE)**（[署名-相同方式共享 4.0 国际许可协议](https://creativecommons.org/licenses/by-sa/4.0/deed.zh-hans)）授权：你可以自由阅读、复制、改写、翻译和分发，条件是保留署名、注明出处并说明修改，且衍生作品使用相同协议。

**第三方权利排除声明（Except where otherwise noted）**：文中出于教学评述与技术讨论目的简短引用的 Unreal Engine 及 Lyra Starter Game 官方源码片段（单段均不超过 30 行，符合 UE EULA 第 5.a.ii 条规定）、`.ini` 配置项与官方资产名称，著作权归 **Epic Games, Inc.** 所有，**明确排除在本仓库 `CC BY-SA 4.0` 授权范围之外**，其使用受 [Unreal Engine EULA](https://www.unrealengine.com/eula) 约束。Unreal® 与 Unreal Engine® 是 Epic Games, Inc. 的商标，本项目仅作技术教学用途，与 Epic Games 无隶属关系。
