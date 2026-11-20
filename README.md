# Lyra Starter Game（UE 5.8）中文文档总览与参考手册

> 一套针对 UE 5.8 `LyraStarterGame` 的中文技术文档，**不包含** Lyra 工程本体，也不含 Epic 的任何原始源码与资产。
> 文字内容采用 CC BY-NC-SA 4.0 授权，详见文末。

本目录汇集了针对 UE 5.8 `LyraStarterGame` 编写并经源码与编辑器实测核验的全套技术文档。请按你当前的场景选择入口：

- **想从头建立心智模型、系统读懂 Lyra 为什么这样设计** → 直接进入核心教程书 **[《从零读懂 Lyra》](Book/README.md)**（第 0-22 章 + 附录 A1-A3）。
- **已经在改功能、想查某个模块的类表、函数级调用链或速查问答** → 查下方**模块参考手册（10~80 号）**或 **[附录 A3 源码地图](Book/A3-CodeMap.md)**。
- **想带团队或自己用半天到一天跑通一次完整玩法开发** → 进入下方**动手工作坊**。
- **遇到静默失效、不知道下什么断点或开哪条日志** → 查 **[附录 A1 调试工具箱](Book/A1-Debugging.md)**。

---

## 一、核心教程书：《从零读懂 Lyra》

入口：**[`Book/README.md`](Book/README.md)**（写作与译名规范见 [`Book/_StyleGuide.md`](Book/_StyleGuide.md)）

| 部分 | 章节范围 | 解决的核心问题 |
|---|---|---|
| **写在前面** | [第 0 章](Book/00-HowToUse.md) | 环境准备、编辑器启动地图 vs 游戏默认地图、PIE 联机与控制台用法 |
| **第一部分：UE 基础** | [第 1 章](Book/01-UObject.md) ~ [第 4 章](Book/04-DataAndAssets.md) | `UObject`/反射/CDO、Gameplay Framework、服务器权威与复制、主资产/Tag/委托/配置 |
| **第二部分：Lyra 的骨架** | [第 5 章](Book/05-BirdsEye.md) ~ [第 8 章](Book/08-PawnAssembly.md) | 五层架构与 12 步启动链、Experience 与 GameFeature、`GameFrameworkComponentManager` 组件注入、PawnData 与 `InitState` 状态链 |
| **第三部分：技能系统（GAS）** | [第 9 章](Book/09-GASConcepts.md) ~ [第 12 章](Book/12-DamageAndDeath.md) | GAS 核心概念、ASC 放在 `PlayerState`、`AbilitySet`、增强输入与 `InputTag`、伤害执行与死亡重生链路 |
| **第四部分：玩法系统** | [第 13 章](Book/13-Inventory.md) ~ [第 15 章](Book/15-TeamsInteractionMessages.md) | 物品定义/实例与 Fragment、远程武器客户端检测与热度扩散、队伍/交互/全局消息总线 |
| **第五部分：表现层** | [第 16 章](Book/16-Camera.md) ~ [第 17 章](Book/17-AnimCosmeticsEffects.md) | 相机模式栈与权重混合、Tag 动画属性映射、角色外观部件、表面特效与伤害数字 |
| **第六部分：UI** | [第 18 章](Book/18-UIFramework.md) ~ [第 19 章](Book/19-HUDAndLoading.md) | CommonUI 四层栈与输入路由、`UIExtension` 槽位注入、世界空间指示器与加载屏 |
| **第七部分：平台与工程** | [第 20 章](Book/20-Settings.md) ~ [第 21 章](Book/21-OnlineAndBuild.md) | 本地/共享双层设置与动态反射路径、CommonUser/CommonSession 开房进图、构建变体与打包 |
| **第八部分：综合实战** | [第 22 章](Book/22-Capstone.md) | 不改 `ALyraGameMode`，从零搭建完整的“最后存活者（Last Man Standing）”插件模式 |
| **附录** | [A1 调试](Book/A1-Debugging.md) · [A2 术语表](Book/A2-Glossary.md) · [A3 源码地图与已知坑](Book/A3-CodeMap.md) | 按症状排查清单、中英术语对照、“想做什么→去哪找”速查表与 54 条原版已知坑 |

---

## 二、模块参考手册（字典式速查）

每份手册统一采用固定结构：**§1 职责 → §2 关键类表 → §3 核心数据流 → §4 跨模块接口 → §5 数据驱动点 → §6 扩展配方 → §7 坑与注意事项 → §8 函数级调用链 → §9/§10 设计意图与速查问答**。

| 编号 | 文档 | 覆盖源码与插件范围 |
|---|---|---|
| **10** | [`10-CoreFramework.md`](Reference/10-CoreFramework.md) | `GameModes`（Experience）、`GameFeatures`、`System`（AssetManager/GameInstance）、`Player`、`Character`（PawnExt/Hero/InitState）、`Development`、`ModularGameplayActors` |
| **20** | [`20-AbilitySystem-Input.md`](Reference/20-AbilitySystem-Input.md) | `AbilitySystem`（ASC、AbilitySet、Phases、GlobalAbilitySystem、Executions、Attributes）、`Input`（EnhancedInput + `InputTag`） |
| **30** | [`30-GameplaySystems.md`](Reference/30-GameplaySystems.md) | `Inventory`、`Equipment`、`Weapons`、`Interaction`、`Teams` |
| **40** | [`40-UI.md`](Reference/40-UI.md) | `UI`、`CommonGame`、`UIExtension`、`CommonLoadingScreen`、`IndicatorSystem`、`Frontend` |
| **50** | [`50-Settings-Online-Platform.md`](Reference/50-Settings-Online-Platform.md) | `Settings`、`GameSettings`、`CommonUser`（登录与会话）、`Hotfix`、`Performance` |
| **60** | [`60-Presentation.md`](Reference/60-Presentation.md) | `Camera`、`Animation`、`Cosmetics`、`Feedback`（ContextEffects/NumberPops）、`Audio`、`Messages`、`Replays` |
| **70** | [`70-GameFeatures-Content.md`](Reference/70-GameFeatures-Content.md) | `ShooterCore`、`ShooterMaps`、`TopDownArena`、`ShooterExplorer`、`ShooterTests` 及项目内自定义插件资产拆解 |
| **80** | [`80-Build-Config.md`](Reference/80-Build-Config.md) | `.uproject`、`*.Build.cs`、`*.Target.cs`、`Config/*.ini` 配置全景与新增模块/通道/主资产菜谱 |
| **95** | [`95-Verification-Report.md`](Reference/95-Verification-Report.md) | 参考手册跨模块源码抽样核验报告与原版未实现代码汇总 |

---

## 三、动手工作坊（实战演练）

待上传

---

## 关于书中的源码引用

正文里所有 `Source/...`、`Config/...`、`Plugins/...` 形式的路径，指的是**你本地 Lyra Starter Game 工程**中的相对位置，不是本仓库里的文件。请在你的工程根目录下对照查找。

本仓库只收录文字文档，不收录引擎和示例工程的任何源码或资产——这部分版权属于 Epic Games。

## 授权

本仓库全部文字内容采用 **[CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)** 授权：可以自由阅读、复制、改写、翻译和非商业分发，条件是保留署名、注明出处，且衍生作品使用相同协议。

Lyra Starter Game 与 Unreal Engine 是 Epic Games 的商标，其源码、资产与截图版权归 Epic Games 所有。本书仅作技术评述与教学用途，与 Epic Games 无隶属关系。
