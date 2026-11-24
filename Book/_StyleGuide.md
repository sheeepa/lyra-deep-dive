# 《从零读懂 Lyra》写作规范（作者必读）

本书目标：一个**懂 C++、懂常见架构设计、用过 UE 编辑器但没读过 UE 源码**的人，按顺序读完本书，就能：
1. 掌握 UE 游戏编程的核心概念（对象系统、Gameplay Framework、网络复制、资产管理、GAS、Enhanced Input、CommonUI……）；
2. 完整理解 Lyra 的架构与每个系统的实现；
3. 接到“基于 Lyra 改一个功能”的需求时，能独立定位代码并动手修改。

这是一本**教程**，不是参考手册。参考手册已经存在（`Reference/10-*.md` - `80-*.md`），它们是你写作的**素材**，不是要复制的格式。

---

## 1. 读者画像（每写一段都要想着他）

- 小林，3 年 C++ 后端/客户端经验，懂多态、RAII、观察者模式、依赖注入、ECS 这些词。
- 用 UE 做过教程项目：会摆 Actor、写过一个 `AActor` 子类、拖过蓝图节点。
- **不知道**：UObject 和普通 C++ 对象有什么区别、CDO 是什么、`UPROPERTY` 背后发生了什么、为什么 PlayerState 存在、复制和 RPC 怎么工作、GAS 是什么、DataAsset 和 AssetManager 有什么用、GameFeature 是什么。
- 他读书的方式：从头读到尾，开着编辑器和 IDE，边读边动手。每章大约花 45~90 分钟。

**判断标准**：小林读到某一句时，如果需要停下来去 Google 一个 UE 术语，这一句就是失败的——要么在这里解释，要么链接到本书前面已解释过的地方。

---

## 2. 每章的学习弧线（不是表格模板，是叙事顺序）

每章按下面的顺序展开。小节标题用你自己的话写，不要照抄这些字。

1. **开场：一个具体的问题。** 用一个真实场景引出本章（“你要给角色加一个冲刺技能，按 Shift 触发，有冷却，网络对战里不能作弊……这需要什么？”）。不要以“本章介绍 XXX 系统”开头。
2. **读完本章你能……**：3~5 条可检验的能力（“能说出按键到技能激活经过的 5 个环节”），不是“了解 XXX”。
3. **需要先知道的 UE 概念**：本章用到、前面还没讲过的 UE 基础概念，在这里讲清楚。讲法：一句话定义 → 为什么需要它（它解决什么问题）→ 最小代码或编辑器例子 → 和读者已知概念的类比（并说明类比在哪里不成立）。已经讲过的概念只链接回去，不重复。
4. **正文：问题驱动地推进。** 每一节遵循：
   - 要解决的问题 →
   - 最直观的做法（读者自己会怎么写）→
   - 这样做在什么场景下会出问题（给出具体的失败场景）→
   - Lyra 的做法 + 一张图 →
   - 代码走读（见第 4 节）→
   - 回头看：这个设计换来了什么，付出了什么代价。
5. **动手实验**：1~3 个在编辑器/IDE 里能做的小实验。格式：目标 → 步骤 → **先写下你的预测** → 预期结果 → 原理解释。实验要真实可行（只引用确实存在的资产、CVar、命令、函数）。
6. **常见误区**：3~6 条“新手最容易做错/想错的地方”，每条说清楚错在哪、正确做法。
7. **本章小结**：一段话 + 一张总结图（mermaid 或 ASCII），把本章的核心模型压缩成可以默画的东西。
8. **自测题**：5~8 题，从“复述”到“应用”到“设计”。答案用 `<details><summary>答案</summary> ... </details>` 折叠。
9. **继续深入**：链接到参考手册对应章节（例如 `[参考手册 20 §8 调用链](../Reference/20-AbilitySystem-Input.md)`），以及本书后续相关章节。

---

## 3. 讲概念的方法

- **一个小节只讲一个核心想法。** 如果一节里出现了三个新名词，就拆开。
- **先讲为什么，再讲是什么，最后讲怎么用。**
- **先图后码。** 复杂流程先给一张图（mermaid sequence/flowchart，或 ASCII），再走代码。图中节点数不超过 ~10 个。
- **术语首次出现**：`中文（English）`，加一句话定义。例如：“能力系统组件（Ability System Component，简称 ASC）：挂在 Actor 上、负责管理这个 Actor 拥有哪些技能、属性和状态标签的组件。”之后统一使用规定译名（见第 6 节）。
- **类比要标边界**：“可以把 GameplayTag 想成层级化的枚举——但和枚举不同，它在运行时由字符串注册，可以在 ini 里加而不用重新编译。”
- **只讲理解所需的 20%。** 不要罗列一个模块的所有类和所有函数。讲透主干，其余用一句“完整列表见参考手册 §2”带过。
- **反复回到主干。** 本书的“脊柱”是第 5 章的 12 步启动时序和五层结构图。后续章节要明确说出“这件事发生在主干的第几步”。
- **网络是一等公民。** 讲任何运行时逻辑，都要交代“这段代码在服务器、拥有者客户端、还是所有客户端上跑”。

---

## 4. 代码片段规范

- **必须来自真实源码。** 写之前打开对应文件读一遍。可以简化（删日志、删无关分支、删宏），但必须在代码块上方注明 `// 简化自 Source/LyraGame/.../Xxx.cpp 的 ULyraXxx::Func`。**禁止编造 API、类名、函数名、CVar、资产名。** 不确定就去源码里 grep 确认；仍不确定就不写。
- 长度：引用 Epic / Lyra 官方源码的片段严格不超过 30 行（符合 Unreal Engine EULA 公开讨论条款）；原创教学示例代码超过 40 行时也建议拆成几段分别讲。
- 关键行加中文注释，解释“为什么”而不是复述代码。
- 代码之后用 2~5 句话解释它做了什么、关键点在哪。
- **行号**：教程正文**不写行号**（行号会过时，也打断阅读）。用“文件路径 + 类名::函数名”定位，文件路径写成相对链接，例如 [`LyraGameMode.cpp`](../../../Source/LyraGame/GameModes/LyraGameMode.cpp)。（Book 目录在 `Docs/Lyra/Book/`，所以源码相对路径前缀是 `../../../`。）
- 引擎类（`AActor`、`UGameplayAbility` 等）不在项目里，引用时说明“这是引擎类”，不要给项目内链接。
- 蓝图/资产内容：参考手册里标“推断”的内容，写进教程时要说“在编辑器中打开 `XXX` 可以看到……（请以编辑器为准）”，不要当作确定事实。

---

## 5. 文风

- 中文为主，口语化但准确。用“你”称呼读者。
- 段落短（一般 2~5 句）。少用嵌套列表，多用连贯的段落把因果讲清楚。
- **表格只用于少量项的对比**（比如“三种激活策略的区别”，3~5 行）。禁止用大表格代替讲解。
- 不要堆砌加粗。每段最多一处加粗。
- 不写“众所周知”“显然”“非常强大”这类空话。
- 每章 450-800 行 markdown 为宜（阅读 45-90 分钟）。宁可分成两章，也不要写成一章 1500 行。

---

## 6. 统一译名

| 英文 | 本书用法 |
|---|---|
| Actor / Component | Actor / 组件 |
| Pawn / Character | Pawn / Character（不翻译） |
| Controller / PlayerController / PlayerState / GameMode / GameState / GameInstance / LocalPlayer | 保持英文 |
| Replication / Replicated / RepNotify(OnRep) | 复制 / 被复制的 / OnRep 回调 |
| RPC (Server/Client/NetMulticast) | RPC（Server RPC / Client RPC / 多播 RPC） |
| Authority / Autonomous Proxy / Simulated Proxy | 权威端 / 自主代理（本地操控的客户端）/ 模拟代理（别人看到的副本） |
| Listen Server / Dedicated Server | 主机服务器（Listen Server）/ 专用服务器（Dedicated Server） |
| Reflection / CDO | 反射 / 类默认对象（CDO） |
| Subsystem | 子系统 |
| Delegate | 委托 |
| Soft / Hard reference | 软引用 / 硬引用 |
| Primary Asset / Asset Bundle | 主资产 / 资产包（Bundle） |
| GameplayTag | GameplayTag（简称 Tag） |
| Ability System Component | 能力系统组件（ASC） |
| Gameplay Ability | 技能（Gameplay Ability，GA） |
| Gameplay Effect | 效果（Gameplay Effect，GE） |
| Attribute / AttributeSet | 属性 / 属性集 |
| Gameplay Cue | GameplayCue（表现提示） |
| Prediction | 预测 |
| Avatar / Owner (ASC) | 化身（Avatar）/ 拥有者（Owner） |
| Experience | Experience（体验，不翻译） |
| GameFeature plugin / Action | GameFeature 插件 / Action（动作） |
| InitState | 初始化状态（InitState） |
| Enhanced Input / Input Action / Mapping Context | 增强输入 / 输入动作（IA）/ 映射上下文（IMC） |
| Widget / Activatable Widget | 控件 / 可激活控件 |
| Definition / Instance | 定义 / 实例 |
| Fragment | 片段（Fragment） |

---

## 7. 全书目录（章节编号与文件名固定，交叉引用以此为准）

| 文件 | 标题 | 素材来源 |
|---|---|---|
| `00-HowToUse.md` | 写在前面：怎么用这本书 + 环境准备 | 80 |
| **第一部分：UE 基础——为读懂 Lyra 打地基** | | |
| `01-UObject.md` | UE 的对象世界：UObject、反射、CDO、GC、蓝图与 C++ | 引擎知识 + Lyra 例子 |
| `02-GameplayFramework.md` | Gameplay Framework：谁负责什么（World、Actor/组件、GameMode…Pawn、生命周期、子系统） | 引擎知识 + 10 |
| `03-Networking.md` | 网络入门：服务器权威、复制、RPC、角色（Role）、PIE 多人测试 | 引擎知识 + 10 §3.5 |
| `04-DataAndAssets.md` | 数据驱动：DataAsset、主资产与 AssetManager、软引用与异步加载、GameplayTag、委托、配置 ini、模块与插件 | 引擎知识 + 10/80 |
| **第二部分：Lyra 的骨架** | | |
| `05-BirdsEye.md` | 鸟瞰 Lyra：四个需求推出五个设计、五层结构、12 步主干时序、谁拥有什么 | Learning/L00 + 10 |
| `06-Experience.md` | Experience 与 GameFeature：玩法即数据 | 10, 70 |
| `07-ModularGameplay.md` | 组件注入：Modular Actor 与 GameFrameworkComponentManager | 10 |
| `08-PawnAssembly.md` | 组装一个角色：PawnData 与初始化状态链（InitState） | 10 |
| **第三部分：技能系统（GAS）** | | |
| `09-GASConcepts.md` | GAS 入门：ASC、技能、效果、属性、Tag、Cue、预测 | 20 + 引擎知识 |
| `10-LyraGAS.md` | Lyra 怎么用 GAS：ASC 在 PlayerState、AbilitySet、激活策略与激活组、Tag 关系映射 | 20 |
| `11-Input.md` | 从按键到技能：增强输入与 InputTag | 20 |
| `12-DamageAndDeath.md` | 伤害、生命、死亡与重生 | 20, 30 |
| **第四部分：玩法系统** | | |
| `13-Inventory.md` | 物品、装备与快捷栏：定义/实例与片段 | 30 |
| `14-Weapons.md` | 武器与射击：客户端检测、服务器确认、扩散与热度 | 30 |
| `15-TeamsInteractionMessages.md` | 队伍、交互与消息系统 | 30, 60 |
| **第五部分：表现层** | | |
| `16-Camera.md` | 相机：相机模式栈与混合 | 60 |
| `17-AnimCosmeticsEffects.md` | 动画、外观部件与表面特效 | 60 |
| **第六部分：UI** | | |
| `18-UIFramework.md` | CommonUI 与 Lyra 的 UI 分层 | 40 |
| `19-HUDAndLoading.md` | HUD 注入、指示器与加载屏 | 40 |
| **第七部分：平台与工程** | | |
| `20-Settings.md` | 设置系统 | 50 |
| `21-OnlineAndBuild.md` | 在线会话、开房到进图、构建变体与配置 | 50, 80 |
| **第八部分：实战** | | |
| `22-Capstone.md` | 综合项目：从零做一个“最后存活者”模式 | 全部 |
| `A1-Debugging.md` | 附录：调试 Lyra 的工具箱 | 全部 |
| `A2-Glossary.md` | 附录：术语表 | 全部 |
| `A3-CodeMap.md` | 附录：源码地图与已知坑 | 全部 + 95 |

---

## 8. 外部资料

- 设计意图优先引用官方说法（Epic 的 Lyra 文档、引擎文档、官方演讲），其次是可靠的社区资料（如 X157 的 Lyra 笔记、tranek 的 GASDocumentation、benui 的 CommonUI 文章）。调研汇总见 `_Editorial/_WebResearch_Official.md`、`_Editorial/_WebResearch_Community.md`。
- 源码和文档冲突时以 5.8 源码为准，并在正文里说明“官方文档（5.x 版本）写的是……，当前源码已改为……”。
- 只能从代码推断、官方和社区都没说明的设计意图，正文里要标“（推断）”。
- 每章“继续深入”除了链接参考手册，还要列出对应的官方文档和高质量社区文章。

## 9. 交付检查清单（交稿前自查）

- [ ] 每章都以具体问题开场，而不是“本章介绍”。
- [ ] 本章用到的每个 UE 概念，都已在本章或前面章节解释过（解释过的给出链接）。
- [ ] 每段代码都核对过源码，并标注了出处；没有行号。
- [ ] 至少一张图；至少一个动手实验；有自测题，答案已折叠。
- [ ] 没有大表格堆砌；每节只讲一个核心想法。
- [ ] 交代了每段运行时逻辑在哪一端执行。
- [ ] 交叉引用的章节编号与第 7 节一致。
