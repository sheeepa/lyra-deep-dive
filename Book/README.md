# 从零读懂 Lyra

这是一本教程，写给**懂 C++、懂一些架构设计、用过 UE 编辑器，但没读过 UE 源码**的人。按顺序读完，你会：

- 掌握 UE 游戏编程的核心概念：对象系统与反射、Gameplay Framework、网络复制、资产管理、GameplayTag、GAS、增强输入、CommonUI；
- 理解 Lyra 每个系统为什么这样设计、代码怎么走；
- 接到“基于 Lyra 改一个功能”的需求时，知道从哪里下手、改哪些类和资产、会踩什么坑。

## 怎么读

- **按顺序读。** 后面的章节默认你读过前面的章节，每个概念只在第一次出现时详细解释。
- **开着编辑器和 IDE 读。** 每章都有动手实验，先写下预测再验证，这一步最能帮你建立直觉。
- **每章 45~90 分钟。** 读完做章末自测题，答不出来的回去重读对应小节。
- **遇到具体问题时查参考手册。** 本书讲“为什么”和“主干”，细节（完整类列表、函数级调用链、几十个速查问答）在[参考手册](../README.md)里。

## 目录

**写在前面**
- [第 0 章　怎么用这本书 + 环境准备](00-HowToUse.md)

**第一部分　UE 基础：为读懂 Lyra 打地基**
- [第 1 章　UE 的对象世界：UObject、反射、CDO、GC、蓝图与 C++](01-UObject.md)
- [第 2 章　Gameplay Framework：谁负责什么](02-GameplayFramework.md)
- [第 3 章　网络入门：服务器权威、复制、RPC](03-Networking.md)
- [第 4 章　数据驱动：引用与加载、AssetManager、GameplayTag、委托、配置、模块与插件](04-DataAndAssets.md)

**第二部分　Lyra 的骨架**
- [第 5 章　鸟瞰 Lyra：四个需求推出五个决定](05-BirdsEye.md)
- [第 6 章　Experience 与 GameFeature：玩法即数据](06-Experience.md)
- [第 7 章　组件注入：Modular Actor 与 GameFrameworkComponentManager](07-ModularGameplay.md)
- [第 8 章　组装一个角色：PawnData 与初始化状态链](08-PawnAssembly.md)

**第三部分　技能系统（GAS）**
- [第 9 章　GAS 入门：ASC、技能、效果、属性、Tag、Cue 与预测](09-GASConcepts.md)
- [第 10 章　Lyra 怎么用 GAS：ASC 在 PlayerState、AbilitySet、激活策略与 Tag 关系](10-LyraGAS.md)
- [第 11 章　从按键到技能：增强输入与 InputTag](11-Input.md)
- [第 12 章　伤害、生命、死亡与重生](12-DamageAndDeath.md)

**第四部分　玩法系统**
- [第 13 章　物品、装备与快捷栏：定义/实例与片段](13-Inventory.md)
- [第 14 章　武器与射击：客户端检测、服务器确认、扩散与热度](14-Weapons.md)
- [第 15 章　队伍、交互与消息系统](15-TeamsInteractionMessages.md)

**第五部分　表现层**
- [第 16 章　相机：相机模式栈与混合](16-Camera.md)
- [第 17 章　动画、外观部件与表面特效](17-AnimCosmeticsEffects.md)

**第六部分　UI**
- [第 18 章　CommonUI 与 Lyra 的 UI 分层：谁在屏幕上，谁拿到输入](18-UIFramework.md)
- [第 19 章　HUD 注入、指示器与加载屏：内容从哪来，什么时候露脸](19-HUDAndLoading.md)

**第七部分　平台与工程**
- [第 20 章　设置系统：一个界面，两种存储，一百多个选项](20-Settings.md)
- [第 21 章　在线会话、开房到进图、构建变体](21-OnlineAndBuild.md)

**第八部分　实战**
- [第 22 章　综合项目：从零做一个“最后存活者”模式](22-Capstone.md)

**附录**
- [附录 A1　调试 Lyra 的工具箱](A1-Debugging.md)
- [附录 A2　术语表](A2-Glossary.md)
- [附录 A3　源码地图与已知坑](A3-CodeMap.md)

## 学习路线建议

- **只想尽快能改 Lyra**：第 0、2、3 章 → 第 5-8 章 → 第 10-12 章 → 按需求跳到对应章节。第 1、4、9 章遇到不懂的概念时回头查。
- **想系统掌握 UE + Lyra**：从头读到尾，每章做完实验和自测题，最后完成第 22 章的综合项目。

## 版权与许可

除文中为教学评述简短引用的 Unreal Engine 及 Lyra 官方代码片段（版权归 **Epic Games, Inc.** 所有，排除在开源许可之外并受 Unreal Engine EULA 约束）外，本书全部原创内容采用 **[CC BY-SA 4.0](../LICENSE)** 授权，详见仓库首页与 [`LICENSE`](../LICENSE)。
