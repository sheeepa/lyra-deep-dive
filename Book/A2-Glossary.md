# 附录 A2　术语表

这份术语表收录全书用到的 UE 与 Lyra 术语，按**英文名的字母顺序**排列。每条给出统一译名（沿用 [写作规范](_StyleGuide.md) 的译名表）、一句话定义，以及“首次出现”的章节链接。“首次出现”指第一次给出一句话定义的章节，包括某些章节开头“先预告几个后面才详讲的名词”的清单；只是在代码或例子里顺带出现、没有给定义的，会在括号里注明。有些术语到后面才讲透，这时括号里会补一句“详解第 N 章”。

使用建议：读到一个眼生的词，先在这里查定义，再点链接回到它被解释的地方。表里不重复讲原理，原理在各章。

> 说明：章节编号与书的目录一致（见 [`_StyleGuide.md` 第 7 节](_StyleGuide.md)）。

## A

- **Ability Set**（技能集，`ULyraAbilitySet`）：一份只读数据资产，一次性授予一组技能、效果和属性集，并返回句柄用于整体撤销。首次出现：[第 1 章](01-UObject.md)（预告），[第 5 章](05-BirdsEye.md)给出定义（详解[第 10 章](10-LyraGAS.md)）。
- **Ability System Component**（能力系统组件，ASC）：挂在 Actor 上、管理这个 Actor 拥有哪些技能、属性和状态标签的组件。Lyra 把它放在 PlayerState 上。首次出现：[第 1 章](01-UObject.md)（预告），[第 5 章](05-BirdsEye.md)给出定义（详解[第 9 章](09-GASConcepts.md)、[第 10 章](10-LyraGAS.md)）。
- **Ability Task**（技能任务）：在技能运行期间异步等待某件事（输入松开、事件、落地……）的小对象。首次出现：[第 9 章](09-GASConcepts.md)。
- **Accolade**（勋章）：ShooterCore 里根据击杀连击、助攻等产生的荣誉提示，由消息处理器产生、HUD 控件显示。首次出现：[第 15 章](15-TeamsInteractionMessages.md)。
- **Action**（动作，`UGameFeatureAction`）：GameFeature 激活时对游戏做点什么、停用时撤销的小对象，例如“加组件”“授予技能”“加控件”。首次出现：[第 1 章](01-UObject.md)（预告），[第 5 章](05-BirdsEye.md)给出定义（详解[第 6 章](06-Experience.md)）。
- **Action Domain**（动作域，CommonUI）：给一组可激活控件指定共同的输入配置的数据资产；控件自己不表态时，输入路由器会先看它所属动作域。首次出现：[第 18 章](18-UIFramework.md)。
- **Action Router**（动作路由器，`UCommonUIActionRouterBase`，引擎类）：CommonUI 里每个 LocalPlayer 一个，维护“当前激活控件”的树，决定哪个控件接收输入、应用什么输入配置。首次出现：[第 18 章](18-UIFramework.md)。
- **ActionSet**（动作集，`ULyraExperienceActionSet`）：可被多个 Experience 共享的一组 Action。首次出现：[第 4 章](04-DataAndAssets.md)（作为一种主资产类型被提到）（详解[第 6 章](06-Experience.md)）。
- **Activatable Widget**（可激活控件）：CommonUI 里能被压入层栈、有“激活/失活”生命周期并决定输入模式的控件。首次出现：[第 18 章](18-UIFramework.md)。
- **Activation Group**（激活组）：Lyra 对技能的一条硬规则，分 Independent、Exclusive_Replaceable、Exclusive_Blocking，决定技能之间能否并存。首次出现：[第 10 章](10-LyraGAS.md)。
- **Activation Policy**（激活策略）：技能“何时尝试激活”的属性，分按下触发、按住持续触发、出生即触发。首次出现：[第 10 章](10-LyraGAS.md)。
- **Actor**：能放进关卡的对象，可以有变换、可以复制、可以挂组件。首次出现：[第 1 章](01-UObject.md)（详解第 [2 章](02-GameplayFramework.md)）。
- **Anim Layer**（动画层）：把一段动画蓝图逻辑按“层”链接到角色的动画实例上，Lyra 用它实现不同武器的动作风格。首次出现：[第 13 章](13-Inventory.md)（详解第 [17 章](17-AnimCosmeticsEffects.md)）。
- **AnimInstance**：驱动骨骼网格动画的运行时对象，Lyra 的版本能把 GameplayTag 映射成动画变量。首次出现：[第 17 章](17-AnimCosmeticsEffects.md)。
- **Asset Bundle**（资产包，Bundle）：给资产的软引用打上的“Client/Server/Equipped”等标记，让不同端只加载自己需要的那部分。首次出现：[第 4 章](04-DataAndAssets.md)。
- **AssetManager**：按“类型 + 名字”维护主资产索引、负责异步加载与资产包切换的引擎对象，Lyra 有自己的子类。首次出现：[第 4 章](04-DataAndAssets.md)。
- **Attribute**（属性）：ASC 上的一个数值，有基础值和当前值两份，如 Health。首次出现：[第 9 章](09-GASConcepts.md)。
- **AttributeSet**（属性集）：一组相关属性的集合，同时负责钳制和对 GE 结果的响应。首次出现：[第 9 章](09-GASConcepts.md)。
- **Authority**（权威端）：对某个 Actor 有最终发言权的那台机器，通常是服务器。首次出现：[第 3 章](03-Networking.md)。
- **Autonomous Proxy**（自主代理）：本地操控的客户端上的那份 Actor 副本。首次出现：[第 3 章](03-Networking.md)。
- **Avatar**（化身）：ASC 的“身体”，即当前被它驱动的那个 Actor；Lyra 里是 Pawn，ASC 的拥有者则是 PlayerState。首次出现：[第 5 章](05-BirdsEye.md)（详解[第 8 章](08-PawnAssembly.md)、[第 9 章](09-GASConcepts.md)、[第 10 章](10-LyraGAS.md)）。

## B

- **Blueprint**（蓝图）：UE 的可视化脚本和数据资产系统；蓝图类就是继承自某个 C++ 类的资产。首次出现：[第 1 章](01-UObject.md)。
- **BlueprintImplementableEvent**：只声明、不实现的 C++ 函数，实现写在蓝图里；C++ 调用它等于“发事件”。首次出现：[第 1 章](01-UObject.md)。
- **Bot**（机器人）：由 AI 控制器驱动、同样拥有 PlayerState 的“玩家”，由 `ULyraBotCreationComponent` 生成。首次出现：[第 2 章](02-GameplayFramework.md)（介绍 AIController 时）（详解[第 6 章](06-Experience.md)）。
- **Build.cs / Target.cs**：模块的依赖声明文件，以及构建目标（客户端、服务器、编辑器、各在线后端）的声明文件。首次出现：[第 4 章](04-DataAndAssets.md)（详解第 [21 章](21-OnlineAndBuild.md)）。

## C

- **CallOrRegister**（“已有则立即调，没有则登记”）：Lyra 里随处可见的模式；对“只发生一次”的事件，既支持早到的订阅，也支持晚到的订阅。首次出现：[第 4 章](04-DataAndAssets.md)（详解第 [6 章](06-Experience.md)）。
- **Camera Mode**（相机模式）：描述“相机应该怎么看”的一个对象，多个模式组成栈并按权重混合。首次出现：[第 16 章](16-Camera.md)。
- **CDO**（类默认对象）：每个 UClass 的默认实例，蓝图“类默认值”面板里填的数就存在它上面。首次出现：[第 1 章](01-UObject.md)。
- **Character**：自带移动组件和胶囊体的 Pawn 子类。首次出现：[第 2 章](02-GameplayFramework.md)。
- **Character Part**（角色部件）：外观系统里“一个可挂在角色身上的部件”，由 Controller 组件选择、Pawn 组件生成。首次出现：[第 17 章](17-AnimCosmeticsEffects.md)。
- **Cheat Manager**（作弊管理器）：处理 `Exec` 作弊命令的对象，Lyra 的版本把命令转发给服务器执行。首次出现：[第 3 章](03-Networking.md)（详解 [附录 A1](A1-Debugging.md)）。
- **CommonUI**：UE 的一套 UI 框架，提供层栈、可激活控件、输入路由、输入模式管理。首次出现：[第 18 章](18-UIFramework.md)。
- **CommonUser / CommonSession**：Lyra 的登录与会话插件，把在线子系统包装成统一接口，负责登录、开房、搜索和加入。首次出现：[第 0 章](00-HowToUse.md)（插件清单）（详解[第 21 章](21-OnlineAndBuild.md)）。
- **Component**（组件）：挂在 Actor 上的功能单元。首次出现：[第 1 章](01-UObject.md)（详解第 [2 章](02-GameplayFramework.md)）。
- **Component Request**（组件请求）：向 GameFrameworkComponentManager 登记“凡是 X 类的 Actor 都给我加上 Y 组件”，带引用计数。首次出现：[第 7 章](07-ModularGameplay.md)。
- **Context Effect**（表面特效/上下文效果）：由动画通知触发、根据脚下物理表面和动作类型查表选择音效与特效的系统。首次出现：[第 17 章](17-AnimCosmeticsEffects.md)。
- **Control Flow**（控制流，`FControlFlow`，引擎的 ControlFlows 模块）：把一串“做完一步再做下一步”的异步步骤写成线性流程的工具；Lyra 的前端用它串起“等 Experience、等登录、推主菜单”。首次出现：[第 18 章](18-UIFramework.md)。
- **Controller**：决定 Pawn 做什么的对象，可以是玩家控制器或 AI 控制器。首次出现：[第 2 章](02-GameplayFramework.md)。
- **Cosmetics**（外观系统）：负责角色模型、身体风格、动画风格、队伍配色等纯表现的一组系统。首次出现：[第 5 章](05-BirdsEye.md)（详解第 [17 章](17-AnimCosmeticsEffects.md)）。
- **Cost / Cooldown**（消耗 / 冷却）：技能激活的两道门槛，在 GAS 里本质都是 GE。首次出现：[第 9 章](09-GASConcepts.md)。
- **CVar**（控制台变量）：可在运行时通过控制台读写的变量。首次出现：[第 4 章](04-DataAndAssets.md)（只在 `ConsoleVariable` 元数据处顺带提到，书里没有专门讲解；实用清单见 [附录 A1](A1-Debugging.md)）。

## D

- **Data Asset**（数据资产）：存放结构化数据的资产，不含逻辑；既可以是资产实例，也可以是蓝图类的 CDO。首次出现：[第 4 章](04-DataAndAssets.md)。
- **DataRegistry**（数据注册表）：引擎里按 ID 异步查询数据表行的系统；勋章的图标、音效、时长就是这样按需加载的。首次出现：[第 15 章](15-TeamsInteractionMessages.md)。
- **Dedicated Server**（专用服务器）：没有本地玩家、没有画面的纯服务器进程。首次出现：[第 3 章](03-Networking.md)。
- **Definition / Instance**（定义 / 实例）：Lyra 的通用分层：定义是只读的配置数据（CDO），实例持有运行时状态并负责复制。首次出现：[第 13 章](13-Inventory.md)。
- **Delegate**（委托）：UE 里观察者模式的实现，A 触发、B 绑定，A 不认识 B。首次出现：[第 4 章](04-DataAndAssets.md)。
- **Developer Settings**（开发者设置，`UDeveloperSettings`）：自动出现在“项目设置”或“编辑器偏好设置”窗口里的配置类，由类的 `config=` 决定去哪个窗口。Lyra 的 `ULyraDeveloperSettings` 是 `config=EditorPerProjectUserSettings`，在“编辑器偏好设置”里，只在编辑器里生效。首次出现：[第 4 章](04-DataAndAssets.md)。

## E

- **Elimination Message**（淘汰消息，`Lyra.Elimination.Message`）：血量归零时服务器广播的标准消息，Target 是死者的 PlayerState。首次出现：[第 4 章](04-DataAndAssets.md)（作为消息频道名的例子）（详解[第 12 章](12-DamageAndDeath.md)、[第 15 章](15-TeamsInteractionMessages.md)）。
- **Enhanced Input**（增强输入）：UE 的输入系统，用输入动作、映射上下文、修饰器和触发器组织输入。首次出现：[第 11 章](11-Input.md)。
- **Equipment**（装备）：Lyra 里“手里拿着的东西”的运行时表示，生成外观 Actor、授予技能；挂在 Pawn 上。首次出现：[第 13 章](13-Inventory.md)。
- **Execution**（执行计算，`UGameplayEffectExecutionCalculation`）：当修改器不够用时，GE 里用代码计算结果的钩子；Lyra 的伤害公式就在这里。首次出现：[第 9 章](09-GASConcepts.md)。
- **Experience**（体验）：一份数据资产，描述一种玩法需要哪些插件、哪份 PawnData、哪些 Action。首次出现：[第 1 章](01-UObject.md)（预告了一句定义），[第 5 章](05-BirdsEye.md)放进全局图（详解[第 6 章](06-Experience.md)）。
- **Experience Manager Component**（Experience 管理组件）：挂在 GameState 上的组件，运行 Experience 的加载状态机并广播“已加载”。首次出现：[第 3 章](03-Networking.md)（详解第 [6 章](06-Experience.md)）。
- **Extension Handler**（扩展处理器）：向 GameFrameworkComponentManager 登记的回调，Actor 注册或收到扩展事件时被调用。首次出现：[第 7 章](07-ModularGameplay.md)。
- **Extension Point**（扩展点/槽位，UIExtension）：HUD 布局里一个按 Tag 命名的位置，接收被注册到该 Tag 的控件。首次出现：[第 5 章](05-BirdsEye.md)（详解第 [19 章](19-HUDAndLoading.md)）。

## F

- **FastArraySerializer**：为数组做增量复制的引擎机制，能在客户端回调“新增/变化/移除了哪一项”。首次出现：[第 3 章](03-Networking.md)。
- **Fragment**（片段）：物品定义里按需拼装的小组件，每个片段负责一种能力。首次出现：[第 5 章](05-BirdsEye.md)（作为五种设计模式之一）（详解[第 13 章](13-Inventory.md)）。
- **Frontend**（前端）：游戏启动后的主菜单阶段，在 Lyra 里它本身也是一个 Experience。首次出现：[第 0 章](00-HowToUse.md)（运行 PIE 时提到前端主菜单地图）（详解[第 18 章](18-UIFramework.md)、[第 21 章](21-OnlineAndBuild.md)）。

## G

- **Game Phase**（游戏阶段）：用 GameState 上的 ASC 里的技能表示的比赛阶段（热身、进行、赛后），用 Tag 层级决定嵌套与互斥。首次出现：[第 4 章](04-DataAndAssets.md)（`IsPhaseActive` 的 Tag 匹配例子），[第 10 章](10-LyraGAS.md)（GameState 上的 ASC）；机制集中在[第 22 章](22-Capstone.md) 2.5 节讲，资产层面的细节见参考手册 70。
- **GameFeature plugin**（GameFeature 插件）：运行时可以切换状态、可热插拔的玩法插件。首次出现：[第 5 章](05-BirdsEye.md)（先给定义）（详解[第 6 章](06-Experience.md)）。
- **GameFeatureData**：每个 GameFeature 插件自带的资产，列出插件激活时执行的 Action 和额外的资产扫描规则。首次出现：[第 5 章](05-BirdsEye.md)（详解[第 6 章](06-Experience.md)）。
- **GameFrameworkComponentManager**（游戏框架组件管理器）：管理接收者、组件请求、扩展处理器和 InitState 的 GameInstance 子系统。首次出现：[第 2 章](02-GameplayFramework.md)（GameInstance 里注册 InitState 的代码），[第 5 章](05-BirdsEye.md)解释用途（详解[第 7 章](07-ModularGameplay.md)）。
- **GameInstance**：一个进程里只有一份、跨地图存在的对象。首次出现：[第 2 章](02-GameplayFramework.md)。
- **GameMode**：只存在于服务器的规则裁判，负责登录、出生、重启；Lyra 只有一个。首次出现：[第 1 章](01-UObject.md)（详解第 [2 章](02-GameplayFramework.md)）。
- **Gameplay Effect Context**（效果上下文，`FLyraGameplayEffectContext`）：随 GE 一起传递“谁打的、用什么打的”的结构体；Lyra 的版本多带了 AbilitySource 和 CartridgeID，这两项不复制。首次出现：[第 10 章](10-LyraGAS.md)（详解[第 12 章](12-DamageAndDeath.md)）。
- **Gameplay Tag Property Map**（Tag 属性映射，`FGameplayTagBlueprintPropertyMap`）：把 ASC 上某个 Tag 的有无或层数自动映射成动画实例里的变量，是 GAS 通往动画的桥。首次出现：[第 17 章](17-AnimCosmeticsEffects.md)。
- **GameState**：存在于所有端、保存整局共享状态的 Actor；Experience 管理组件就挂在它上面。首次出现：[第 2 章](02-GameplayFramework.md)。
- **Gameplay Ability**（技能，GA）：GAS 里“一个可激活的动作”，有激活条件、执行逻辑和结束。首次出现：[第 9 章](09-GASConcepts.md)。
- **Gameplay Cue**（GameplayCue，表现提示）：只负责表现、不影响结果的音效/特效触发机制。首次出现：[第 4 章](04-DataAndAssets.md)（预告了一句定义）（详解[第 9 章](09-GASConcepts.md)）。
- **Gameplay Effect**（效果，GE）：对属性和标签的“修改请求”，分即时、持续、无限三种。首次出现：[第 9 章](09-GASConcepts.md)。
- **Gameplay Event**（游戏事件）：发给 ASC、可以触发技能的事件，如 `GameplayEvent.Death`。首次出现：[第 9 章](09-GASConcepts.md)。
- **Gameplay Message Subsystem**（消息子系统）：以 GameplayTag 为频道、以 USTRUCT 为载荷的广播系统，发送者与监听者互不认识。首次出现：[第 2 章](02-GameplayFramework.md)（作为 GameInstance 子系统的例子），[第 4 章](04-DataAndAssets.md)讲到频道 Tag（详解[第 15 章](15-TeamsInteractionMessages.md)）。
- **GameplayTag**（Tag）：层级化的名字，运行时按字符串注册，Lyra 用它作为系统之间的契约。首次出现：[第 4 章](04-DataAndAssets.md)。
- **Garbage Collection**（垃圾回收，GC）：UE 管理 UObject 生命周期的机制，靠反射知道哪些引用要保活。首次出现：[第 1 章](01-UObject.md)。
- **GAS**（Gameplay Ability System，游戏玩法能力系统）：UE 自带的技能框架（插件名 GameplayAbilities），包含 ASC、技能、效果、属性集、Tag、GameplayCue 和预测。首次出现：[第 0 章](00-HowToUse.md)（只提了缩写），[第 9 章](09-GASConcepts.md)给出定义。
- **Global Ability System**（全局技能系统，`ULyraGlobalAbilitySystem`）：World 子系统，能给所有已注册（包括之后才加入）的 ASC 授予或撤销同一个技能或效果；赛前全员免伤就是靠它。首次出现：[第 10 章](10-LyraGAS.md)。

## H

- **Health Component**（生命组件，`ULyraHealthComponent`）：挂在 Pawn 上，把 ASC 里的血量变化翻译成死亡状态机和对外事件的组件。首次出现：[第 3 章](03-Networking.md)（详解第 [12 章](12-DamageAndDeath.md)）。
- **Health Set**（`ULyraHealthSet`）：Lyra 的血量属性集，含 Health、MaxHealth 和 Damage、Healing 两个中转用的元属性。首次出现：[第 9 章](09-GASConcepts.md)。
- **Hero Component**（`ULyraHeroComponent`）：玩家 Pawn 上负责输入绑定、相机模式和 ASC 初始化触发的组件。首次出现：[第 5 章](05-BirdsEye.md)（详解[第 8 章](08-PawnAssembly.md)、[第 11 章](11-Input.md)）。
- **Hit Marker**（命中标记）：准星处的命中反馈；客户端先登记“待确认”的标记，等服务器确认后再转成有效命中标记，由 `ULyraWeaponStateComponent` 管理。首次出现：[第 3 章](03-Networking.md)（详解[第 14 章](14-Weapons.md)）。
- **HUD Slot**（HUD 槽位，`HUD.Slot.*`）：一组 Tag，命名 HUD 布局里可以放控件的位置。首次出现：[第 4 章](04-DataAndAssets.md)（详解第 [19 章](19-HUDAndLoading.md)）。
- **Hotfix**（热修复）：靠在线服务下发配置修补运行中的游戏数据与文本。首次出现：[第 21 章](21-OnlineAndBuild.md)。

## I

- **ILoadingProcessInterface**（加载过程接口）：任何对象实现它并返回“还没好”，就能让加载屏继续显示。首次出现：[第 6 章](06-Experience.md)（只说“加载屏接口”，未点名），[第 18 章](18-UIFramework.md)点名（详解[第 19 章](19-HUDAndLoading.md)）。
- **Indicator**（指示器）：显示在世界物体屏幕投影位置上的控件，如交互提示、昵称牌。首次出现：[第 15 章](15-TeamsInteractionMessages.md)（详解第 [19 章](19-HUDAndLoading.md)）。
- **InitState**（初始化状态）：给 Actor 的每个功能模块配一条状态链，数据到齐才推进；Lyra 的四个状态是 Spawned、DataAvailable、DataInitialized、GameplayReady。首次出现：[第 2 章](02-GameplayFramework.md)（GameInstance 注册四个状态），[第 5 章](05-BirdsEye.md)给出定义（详解[第 7 章](07-ModularGameplay.md)、[第 8 章](08-PawnAssembly.md)）。
- **Input Action / Mapping Context**（输入动作 IA / 映射上下文 IMC）：增强输入里“动作是什么”与“按键怎么映射到动作”两层数据。首次出现：[第 11 章](11-Input.md)（此前章节只提到名字）。
- **Input Config**（输入配置，`ULyraInputConfig`）：把输入动作配对到 InputTag 的数据资产，PawnData 通过它决定输入。首次出现：[第 8 章](08-PawnAssembly.md)（详解第 [11 章](11-Input.md)）。
- **Input Mode**（输入模式 / 输入配置，`ELyraWidgetInputMode`、`FUIInputConfig`）：CommonUI 里由控件声明“我要游戏收输入、UI 收输入，还是都要，鼠标是否被捕获”；`Default` 表示继承父级，整条链都不表态时应用默认的 Menu 模式。首次出现：[第 18 章](18-UIFramework.md)。
- **InputTag**（输入标签）：`InputTag.*` 形式的 Tag，是技能与按键之间的间接层。首次出现：[第 4 章](04-DataAndAssets.md)（详解第 [10 章](10-LyraGAS.md)、[第 11 章](11-Input.md)）。
- **Instanced**（内嵌子对象，`UPROPERTY(Instanced)`）：属性上的“所有权”声明：数组或指针里的子对象属于这份资产本身，每个都可以单独编辑；Experience 的 Actions 和物品的 Fragments 都靠它。首次出现：[第 1 章](01-UObject.md)（详解同章“内嵌子对象”一节）。
- **Interaction**（交互）：基于 GAS 的“靠近、指向、按键”流程，目标实现 `IInteractableTarget` 提供选项。首次出现：[第 15 章](15-TeamsInteractionMessages.md)。
- **Iris**：UE 的新一代复制系统，Lyra 的一些结构体为它做了兼容。首次出现：[第 3 章](03-Networking.md)。

## L

- **Listen Server**（主机服务器）：既是服务器又带本地玩家的进程。首次出现：[第 0 章](00-HowToUse.md)（PIE 网络模式选项）（详解[第 3 章](03-Networking.md)）。
- **Loading Screen**（加载屏）：由 CommonLoadingScreen 管理，每帧询问所有“阻塞者”，任一说没好就继续显示。首次出现：[第 6 章](06-Experience.md)（详解第 [19 章](19-HUDAndLoading.md)）。
- **LocalPlayer**：代表“这台机器上的一个人类玩家”的对象，每台机器上有，服务器上对远端玩家没有。首次出现：[第 2 章](02-GameplayFramework.md)。
- **Local Settings / Shared Settings**（本地设置 / 共享设置）：Lyra 把设置分成机器级（画质）和账号级（可云存档的玩法设置）两层。首次出现：[第 20 章](20-Settings.md)。
- **Loose Tag**（松散 Tag）：直接加在 ASC 上、不由 GE 管理、不复制的 Tag。首次出现：[第 9 章](09-GASConcepts.md)。

## M

- **Meta Attribute**（元属性）：不表示状态、只做中转的属性，如 Damage 与 Healing。首次出现：[第 9 章](09-GASConcepts.md)。
- **Mixed**（混合复制模式）：ASC 的一种复制模式：拥有者收到完整的 GE，其他人只收到标签和提示。首次出现：[第 5 章](05-BirdsEye.md)（代码注释里）（详解[第 9 章](09-GASConcepts.md)、[第 10 章](10-LyraGAS.md)）。
- **Modular Actor**（模块化 Actor）：继承自 `AModular*` 基类、在 `PreInitializeComponents` 注册为接收者的 Actor。首次出现：[第 2 章](02-GameplayFramework.md)（详解[第 7 章](07-ModularGameplay.md)）。
- **Module**（模块）：C++ 编译与链接的单位，由 Build.cs 声明。首次出现：[第 4 章](04-DataAndAssets.md)。

## N

- **Net Execution Policy**（网络执行策略）：技能在客户端预测、仅服务器、服务器发起等不同执行位置的选择。首次出现：[第 9 章](09-GASConcepts.md)。
- **NetMode**（网络模式）：同一份代码在不同进程里的“身份”：独立、主机服务器、客户端、专用服务器。首次出现：[第 3 章](03-Networking.md)。
- **Net Role**（网络角色，Role）：这台机器对某个 Actor 有多大发言权，如 Authority、AutonomousProxy、SimulatedProxy。首次出现：[第 3 章](03-Networking.md)。
- **Number Pop**（伤害数字）：命中时弹出数字的反馈系统，有网格文字和 Niagara 两种实现。首次出现：[第 17 章](17-AnimCosmeticsEffects.md)。

## O

- **OnRep**（OnRep 回调，RepNotify）：服务器改了被复制的属性后，客户端收到新值时自动调用的函数。首次出现：[第 3 章](03-Networking.md)。
- **OSSv1 / OSSv2**（在线子系统 / 在线服务）：新旧两代在线接口；Lyra 默认使用 OSSv1，Epic 官方教程给出了切换到 OSSv2 的步骤，这不是缺陷。首次出现：[第 21 章](21-OnlineAndBuild.md)。
- **Outer**：UObject 的“所属对象”，决定其所处的世界和生命周期归属。首次出现：[第 1 章](01-UObject.md)。
- **Owner (ASC)**（拥有者）：ASC 的持有者，Lyra 里是 PlayerState。首次出现：[第 8 章](08-PawnAssembly.md)（详解[第 9 章](09-GASConcepts.md)）。

## P

- **Pawn**：能被 Controller 操控的 Actor。首次出现：[第 1 章](01-UObject.md)（详解第 [2 章](02-GameplayFramework.md)）。
- **PawnData**（`ULyraPawnData`）：描述“这个角色是什么”的数据资产：Pawn 类、技能集、输入配置、相机模式、Tag 关系映射。首次出现：[第 1 章](01-UObject.md)（预告）（详解[第 8 章](08-PawnAssembly.md)）。
- **Pawn Extension Component**（`ULyraPawnExtensionComponent`）：Pawn 上协调 PawnData 与 InitState 链、连接 ASC 的组件。首次出现：[第 5 章](05-BirdsEye.md)（详解[第 8 章](08-PawnAssembly.md)）。
- **PIE**（Play In Editor）：在编辑器里直接运行游戏，支持单机、主机服务器和带专用服务器的客户端等模式。首次出现：[第 0 章](00-HowToUse.md)（详解[第 3 章](03-Networking.md)）。
- **Player Spawning Manager Component**（出生管理组件，`ULyraPlayerSpawningManagerComponent`）：挂在 GameState 上，GameMode 把选出生点、重启判定等调用转交给它，让每个 Experience 自定义自己的出生规则；其中 `ControllerCanRestart` 在原版里是 TODO。首次出现：[第 2 章](02-GameplayFramework.md)（详解[第 22 章](22-Capstone.md) 2.3 节）。
- **Player Start**（出生点，`ALyraPlayerStart`）：地图里的出生位置，Lyra 要求用自己的子类才会被出生管理组件缓存。首次出现：[第 2 章](02-GameplayFramework.md)（出生流程里）（详解[第 8 章](08-PawnAssembly.md)）。
- **PlayerController**：代表一个人类玩家意志的 Controller，服务器和拥有者客户端有。首次出现：[第 2 章](02-GameplayFramework.md)。
- **PlayerState**：存在于所有端、代表一个玩家（含机器人）的信息的 Actor，生命周期长于 Pawn。首次出现：[第 2 章](02-GameplayFramework.md)。
- **Plugin**（插件）：带 `.uplugin` 的文件夹，可含模块和内容。首次出现：[第 4 章](04-DataAndAssets.md)。
- **Prediction**（预测）：客户端先假设成功、等服务器确认后再对账的技术；GAS 用预测键做这件事。首次出现：[第 3 章](03-Networking.md)（详解第 [9 章](09-GASConcepts.md)）。
- **Primary Asset**（主资产）：能被 AssetManager 按“类型 + 名字”（`FPrimaryAssetId`）查找的资产。首次出现：[第 4 章](04-DataAndAssets.md)。
- **Primary Game Layout**（根布局，`UPrimaryGameLayout`）：CommonUI 里每个 LocalPlayer 一份的根控件，里面按 `UI.Layer.*` 放着几个层栈；Lyra 的是 `W_OverallUILayout`。首次出现：[第 18 章](18-UIFramework.md)。
- **Push Model**（推送模型）：属性复制的优化：不再每帧比较，改为修改时手动标脏。首次出现：[第 3 章](03-Networking.md)。

## Q

- **QuickBar**（快捷栏，`ULyraQuickBarComponent`）：挂在 Controller 上的三个槽位加一个当前激活槽，负责切换时装备/卸下。首次出现：[第 3 章](03-Networking.md)（详解第 [13 章](13-Inventory.md)）。

## R

- **Receiver**（接收者）：向 GameFrameworkComponentManager 举手“我愿意被注入”的 Actor。首次出现：[第 2 章](02-GameplayFramework.md)（`AddGameFrameworkComponentReceiver` 的代码注释）（详解[第 7 章](07-ModularGameplay.md)）。
- **Reflection**（反射）：UE 通过宏和代码生成保存的类型元数据，使序列化、复制、GC、蓝图成为可能。首次出现：[第 1 章](01-UObject.md)。
- **Registry**（`ULyraGameSettingRegistry`）：设置界面的“登记处”，组织设置项的树状结构。首次出现：[第 20 章](20-Settings.md)。
- **Relevancy**（相关性）：服务器判断“这个客户端需要知道这个 Actor 吗”的机制，不相关的 Actor 不会在该客户端创建。首次出现：[第 3 章](03-Networking.md)。
- **Replay**（回放）：把一局游戏录成回放文件再播放的引擎功能，Lyra 有回放用的 PlayerController 等配套。首次出现：[第 21 章](21-OnlineAndBuild.md)。
- **Replication**（复制）：把服务器上 Actor 的状态同步到客户端。首次出现：[第 3 章](03-Networking.md)。
- **Replication Condition**（复制条件，`COND_*`）：控制一个被复制属性发给谁、发几次，如 `COND_InitialOnly`、`COND_OwnerOnly`、`COND_SkipOwner`。首次出现：[第 3 章](03-Networking.md)。
- **Replication Graph**（复制图）：自定义的复制调度策略，Lyra 默认关闭。首次出现：[第 3 章](03-Networking.md)。
- **Respawn**（重生）：Lyra 里不是引擎默认行为，而是内容层的蓝图被动技能 `GA_AutoRespawn`（放在 `AbilitySet_Elimination` 里）监听死亡后调用 `ALyraGameMode::RequestPlayerRestartNextFrame`。首次出现：[第 5 章](05-BirdsEye.md)（需求里提到“角色死了会重生”）（详解[第 12 章](12-DamageAndDeath.md)、[第 22 章](22-Capstone.md)）。
- **RPC**（远程过程调用）：Server RPC、Client RPC、多播 RPC 三种跨机器调用。首次出现：[第 3 章](03-Networking.md)。

## S

- **Seamless Travel**（无缝旅行）：换图时不断开连接、保留一部分对象的旅行方式。Lyra 默认是硬切换（`B_LyraGameMode` 的 `bUseSeamlessTravel` 为 false），PIE 里默认也强制非无缝。首次出现：[第 2 章](02-GameplayFramework.md)（提到“非无缝”切换时旧 World 整个销毁）（详解[第 21 章](21-OnlineAndBuild.md)）。
- **Session**（会话）：一次可被搜索和加入的联机对局，由 CommonSession 子系统管理。首次出现：[第 21 章](21-OnlineAndBuild.md)。
- **SetByCaller**：GE 里由调用方在运行时填入的数值槽位。首次出现：[第 9 章](09-GASConcepts.md)。
- **Simulated Proxy**（模拟代理）：别人那边看到的 Actor 副本。首次出现：[第 3 章](03-Networking.md)。
- **Soft / Hard Reference**（软引用 / 硬引用）：硬引用把被引用资产拖进内存；软引用只记路径，用时再加载。首次出现：[第 4 章](04-DataAndAssets.md)。
- **Spectator**（旁观者）：不控制 Pawn、只看比赛的玩家；Lyra 原版没有现成的旁观系统。首次出现：[第 2 章](02-GameplayFramework.md)（引擎默认出生逻辑里提到旁观者）；怎么处理见[第 22 章](22-Capstone.md) 2.7 节。
- **Spread / Heat**（散布 / 热度）：远程武器的精度模型：每次开火热度上升，散布角随热度变化，随时间冷却。首次出现：[第 14 章](14-Weapons.md)。
- **StatTag / Tag Stack**（统计标签 / 标签层数）：用“Tag + 计数”存数字、能增量复制的容器，弹药、击杀数都放这里。首次出现：[第 3 章](03-Networking.md)（详解第 [13 章](13-Inventory.md)）。
- **Subsystem**（子系统）：由引擎管理生命周期、可按类型取得的单例，分引擎、GameInstance、World、LocalPlayer 四种作用域。首次出现：[第 2 章](02-GameplayFramework.md)（含四种作用域的对照表）。

## T

- **Tag Relationship Mapping**（Tag 关系映射）：一张数据资产，集中描述“哪类技能阻止/取消哪类技能”。首次出现：[第 8 章](08-PawnAssembly.md)（详解第 [10 章](10-LyraGAS.md)）。
- **Target Data**（目标数据）：技能把“打中了谁”传给服务器所用的容器，Lyra 的武器用它上报命中。首次出现：[第 9 章](09-GASConcepts.md)（详解第 [14 章](14-Weapons.md)）。
- **Team**（队伍）：以 `FGenericTeamId` 保存在 PlayerState 上，由队伍子系统统一查询的分组。首次出现：[第 15 章](15-TeamsInteractionMessages.md)。
- **Team Display Asset**（队伍显示资产）：保存队伍颜色等材质参数的数据资产。首次出现：[第 15 章](15-TeamsInteractionMessages.md)。
- **Team Subsystem**（队伍子系统，`ULyraTeamSubsystem`）：统一提供队伍查询、比较与伤害许可判定的 World 子系统。首次出现：[第 15 章](15-TeamsInteractionMessages.md)。
- **Tick**：每帧调用一次的更新函数。首次出现：[第 2 章](02-GameplayFramework.md)。
- **TObjectPtr / TSubclassOf**：UObject 的类型安全引用，以及“某个类的子类”的类引用。首次出现：[第 1 章](01-UObject.md)。
- **Travel URL**（旅行 URL）：地图名加 `?Experience=…` 等选项的字符串，决定进图后选用哪个 Experience。首次出现：[第 6 章](06-Experience.md)（`?Experience=` 选项）（详解[第 21 章](21-OnlineAndBuild.md)）。

## U

- **UCLASS / UPROPERTY / UFUNCTION**：把 C++ 类型、属性、函数暴露给反射系统的宏。首次出现：[第 1 章](01-UObject.md)。
- **UI Layer**（UI 层，`UI.Layer.*`）：根布局里的几个独立栈（Game、GameMenu、Menu、Modal），每层压入的控件各自管理。其中 `GameMenu` 用于局内临时弹出的覆盖面板（如 `GAB_ShowWidget_WhileInputHeld` 派生的记分板技能 `GA_ShowLeaderboard_TDM`）。首次出现：[第 18 章](18-UIFramework.md)。
- **UI Policy**（UI 策略）：决定每个本地玩家根布局类的对象，由 UI 管理子系统创建。首次出现：[第 18 章](18-UIFramework.md)。
- **UserFacingExperience**（前端玩法条目，`ULyraUserFacingExperienceDefinition`）：前端列表里的一张“牌”：地图 + ExperienceID + 标题 + 人数。首次出现：[第 4 章](04-DataAndAssets.md)（详解第 [21 章](21-OnlineAndBuild.md)）。

## V

- **Verb Message**（动词消息，`FLyraVerbMessage`）：“谁对谁做了什么”形式的通用消息载荷，用于淘汰、伤害等。首次出现：[第 3 章](03-Networking.md)（详解第 [15 章](15-TeamsInteractionMessages.md)）。

## W

- **Wait For Experience Ready**（异步节点 `UAsyncAction_ExperienceReady`）：蓝图里等 Experience 加载完成的节点，是 `CallOrRegister` 在蓝图里的对应物。首次出现：[第 6 章](06-Experience.md)。
- **Weapon Instance**（武器实例，`ULyraWeaponInstance`）：装备实例的武器版本，带动画层、手柄属性和（远程武器的）散布参数。首次出现：[第 13 章](13-Inventory.md)（详解第 [14 章](14-Weapons.md)）。
- **Widget**（控件）：UMG 的界面元素基类。首次出现：[第 4 章](04-DataAndAssets.md)（作为资产例子里的控件类）（详解[第 18 章](18-UIFramework.md)）。
- **World**：一张被加载的地图及其中所有 Actor 的容器；PIE 里服务器和客户端可各有一个。首次出现：[第 2 章](02-GameplayFramework.md)。
- **World Settings**（世界设置）：每张地图自带的设置，Lyra 的版本多了“默认 Experience”。首次出现：[第 2 章](02-GameplayFramework.md)（详解[第 6 章](06-Experience.md)）。
