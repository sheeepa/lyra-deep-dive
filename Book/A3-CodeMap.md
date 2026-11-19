# 附录 A3　源码地图与已知坑

## 一个问题：策划说“把近战伤害调高一点”，你该打开哪个文件？

接到“基于 Lyra 改点东西”的需求时，真正花时间的往往不是改，而是找：这件事的代码在 `Source/LyraGame` 的哪个目录，是 C++ 还是蓝图资产，它又被哪一章讲过。本附录就是为这一刻准备的。

第一部分是一张“**想做什么 → 去哪找**”的地图，按问题组织，约 60 条。每条给出文件链接（链接的是源码文件本身，路径已逐个核对过存在）、一句话说明，以及对应的章节。其中蓝图/资产里的内容没有 C++ 路径，会标明“在资产里”并给出资产名与路径。

第二部分是 Lyra 原版里已知的缺陷、未实现的代码和容易误会的怪癖。**每一条都对照随附的 5.8 源码核对过**（带“推断”的条目只是从代码读出来的，没有实际运行复现）；它们大多是官方留下的 `@TODO`，写到教程里的目的是让你别把它们当成“设计如此”，也别在它们上面盖房子。注意区分“缺陷”和“默认选择”：像 OSSv1 这种官方文档也把它当默认值的，放在第三类“不是 bug，但很容易误会”里。

约定：下面的路径省略了前缀 `../../../`（从 `Docs/Lyra/Book/` 回到工程根目录），链接里是完整的相对路径。`LyraGame` 指 `Source/LyraGame`。

## 第一部分　源码地图：想做什么，去哪找

### 玩法装配：Experience、GameFeature、Pawn

- **想改“选哪个 Experience”的优先级或来源**：`Source/LyraGame/GameModes/LyraGameMode.cpp` 的 `HandleMatchAssignmentIfNotExpectingOne`。URL、PIE 开发者设置、命令行、世界设置的顺序都在这一个函数里。[第 6 章](06-Experience.md)。
- **想改 Experience 的加载流程、加载顺序、回调优先级**：`Source/LyraGame/GameModes/LyraExperienceManagerComponent.cpp`（状态机与三档广播）。[第 6 章](06-Experience.md)。
- **想看一个 Experience 里有什么字段**：`Source/LyraGame/GameModes/LyraExperienceDefinition.h` 与 `Source/LyraGame/GameModes/LyraExperienceActionSet.h`。具体玩法的 Experience 是资产：`/ShooterCore/Experiences/B_ShooterGame_Elimination` 等。[第 6 章](06-Experience.md)。
- **想让某张地图默认用某个 Experience**：`Source/LyraGame/GameModes/LyraWorldSettings.h`（`DefaultGameplayExperience`，在世界设置里填）。[第 6 章](06-Experience.md)。
- **想改“什么都没指定时”的兜底 Experience**：同在 `LyraGameMode.cpp`，里面硬编码了 `B_LyraDefaultExperience`，源码里有 `@TODO` 说要改成配置项。[第 6 章](06-Experience.md)。
- **想让新建的 Experience 被 AssetManager 找到**：`Config/DefaultGame.ini` 的 `PrimaryAssetTypesToScan`（核心）或插件 GameFeatureData 里的同名配置；加载逻辑在 `Source/LyraGame/System/LyraAssetManager.cpp`。[第 4 章](04-DataAndAssets.md)、[第 6 章](06-Experience.md)。
- **想写自己的 GameFeature Action**：参考 `Source/LyraGame/GameFeatures/GameFeatureAction_WorldActionBase.cpp`（基类）与 `Source/LyraGame/GameFeatures/GameFeatureAction_AddWidget.cpp`（标准写法）。注意基类 `UGameFeatureAction_WorldActionBase` 没有导出宏，在别的模块里继承它会遇到链接问题（见第二部分“不少类没有导出宏”一条）。[第 7 章](07-ModularGameplay.md)。
- **想让某个 Actor 能被 `Add Components` 注入**：看 `Plugins/ModularGameplayActors` 插件里的 `AModularCharacter` 等基类（`PreInitializeComponents` 注册接收者）。[第 7 章](07-ModularGameplay.md)。
- **想改出生点的选择规则**：`Source/LyraGame/Player/LyraPlayerSpawningManagerComponent.cpp`（重写 `OnChoosePlayerStart`）；按“离敌人最远”选点的现成例子在 `Plugins/GameFeatures/ShooterCore/Source/ShooterCoreRuntime/Private/TDM_PlayerSpawningManagmentComponent.cpp`；地图里的出生点类型是 `Source/LyraGame/Player/LyraPlayerStart.cpp`。[第 8 章](08-PawnAssembly.md)。
- **想改 Pawn 用什么类、什么能力、什么输入**：`Source/LyraGame/Character/LyraPawnData.h`（字段）、`Source/LyraGame/GameModes/LyraGameMode.cpp` 的 `GetDefaultPawnClassForController_Implementation` 与 `SpawnDefaultPawnAtTransform_Implementation`（怎么用它）。实际的 PawnData 是资产，如 `/ShooterCore/Game/HeroData_ShooterGame`。[第 8 章](08-PawnAssembly.md)。
- **想加一个要参与初始化的组件（InitState）**：照着 `Source/LyraGame/Character/LyraHeroComponent.cpp` 与 `Source/LyraGame/Character/LyraPawnExtensionComponent.cpp` 实现 `IGameFrameworkInitStateInterface`；四个状态在 `Source/LyraGame/System/LyraGameInstance.cpp` 的 `Init` 里注册。[第 8 章](08-PawnAssembly.md)。
- **想加机器人或改机器人数量**：`Source/LyraGame/GameModes/LyraBotCreationComponent.cpp`；机器人的控制器基类是 `Source/LyraGame/Player/LyraPlayerBotController.cpp`；AI 本体（行为树、EQS）在资产里，如 `/ShooterCore/Bot/B_AI_Controller_LyraShooter`。[第 6 章](06-Experience.md)、[第 15 章](15-TeamsInteractionMessages.md)。
- **想让某个模式没有 Pawn（比如前端）**：看 `B_LyraFrontEnd_Experience` 的 PawnData 与 Actions；GameMode 侧的拦截点在 `LyraGameMode.cpp`（`HandleStartingNewPlayer_Implementation`、`PlayerCanRestart_Implementation`、`ControllerCanRestart`）。[第 8 章](08-PawnAssembly.md)。

### GAS、输入、伤害与死亡

- **想知道 ASC 在哪创建**：`Source/LyraGame/Player/LyraPlayerState.cpp` 的构造函数（ASC 与两个属性集）；GameState 上还有一个，在 `Source/LyraGame/GameModes/LyraGameState.cpp`。[第 10 章](10-LyraGAS.md)。
- **想看 Avatar 是怎么接上的**：`Source/LyraGame/Character/LyraPawnExtensionComponent.cpp` 的 `InitializeAbilitySystem` / `UninitializeAbilitySystem`。[第 8 章](08-PawnAssembly.md)、[第 10 章](10-LyraGAS.md)。
- **想改技能的激活策略、激活组、失败反馈、额外消耗**：`Source/LyraGame/AbilitySystem/Abilities/LyraGameplayAbility.cpp` 与 `Source/LyraGame/AbilitySystem/LyraAbilitySystemComponent.cpp`。[第 10 章](10-LyraGAS.md)。
- **想成组授予/撤销技能**：`Source/LyraGame/AbilitySystem/LyraAbilitySet.cpp`。授予点见 `LyraPlayerState.cpp` 的 `SetPawnData`、`Source/LyraGame/Equipment/LyraEquipmentManagerComponent.cpp` 与 `Source/LyraGame/GameFeatures/GameFeatureAction_AddAbilities.cpp`。[第 10 章](10-LyraGAS.md)。
- **想改“哪类技能阻止/取消哪类技能”**：`Source/LyraGame/AbilitySystem/LyraAbilityTagRelationshipMapping.cpp`（读取逻辑）；数据是资产，如 `/ShooterCore/Game/TagRelationships_ShooterHero`。[第 10 章](10-LyraGAS.md)。
- **想给全体玩家加技能或效果**：`Source/LyraGame/AbilitySystem/LyraGlobalAbilitySystem.cpp`。[第 10 章](10-LyraGAS.md)。
- **想加游戏阶段（热身、赛后）**：`Source/LyraGame/AbilitySystem/Phases/LyraGamePhaseSubsystem.cpp` 与 `Source/LyraGame/AbilitySystem/Phases/LyraGamePhaseAbility.cpp`；具体阶段是资产，在 `/ShooterCore/Experiences/Phases/`。[第 10 章](10-LyraGAS.md)、[第 22 章](22-Capstone.md)。
- **想改按键如何变成技能**：`Source/LyraGame/Character/LyraHeroComponent.cpp` 的 `InitializePlayerInput`（绑定），`Source/LyraGame/Input/LyraInputComponent.h`（`BindAbilityActions`），`Source/LyraGame/AbilitySystem/LyraAbilitySystemComponent.cpp` 的 `ProcessAbilityInput`（消费）。InputAction 与 InputTag 的配对数据是 `Source/LyraGame/Input/LyraInputConfig.h` 类型的资产，如 `/Game/Input/InputData_Hero`。[第 11 章](11-Input.md)。
- **想改鼠标灵敏度、手柄死区、视角反转**：`Source/LyraGame/Input/LyraInputModifiers.cpp`（读设置的输入修饰器），设置本体在 `Source/LyraGame/Settings/LyraSettingsShared.cpp`。[第 11 章](11-Input.md)、[第 20 章](20-Settings.md)。
- **想改伤害公式（衰减、队伍系数）**：`Source/LyraGame/AbilitySystem/Executions/LyraDamageExecution.cpp`；治疗在 `Source/LyraGame/AbilitySystem/Executions/LyraHealExecution.cpp`。基础伤害数值在各武器的 GE 资产里，如 `GE_Damage_RifleAuto`。[第 12 章](12-DamageAndDeath.md)。
- **想改血量上限、免疫、GodMode、死亡判定**：`Source/LyraGame/AbilitySystem/Attributes/LyraHealthSet.cpp`。[第 12 章](12-DamageAndDeath.md)。
- **想改死亡/重生流程**：`Source/LyraGame/Character/LyraHealthComponent.cpp`（死亡状态机与淘汰消息）、`Source/LyraGame/AbilitySystem/Abilities/LyraGameplayAbility_Death.cpp`、`Source/LyraGame/Character/LyraCharacter.cpp`（`OnDeathStarted` 等）、`LyraGameMode.cpp` 的 `RequestPlayerRestartNextFrame`。谁触发重生是资产里的技能 `GA_AutoRespawn`。[第 12 章](12-DamageAndDeath.md)。
- **想改全局的伤害/治疗 GE**：`Source/LyraGame/System/LyraGameData.h`（指向资产 `/Game/DefaultGameData`）。[第 12 章](12-DamageAndDeath.md)。
- **想扩展 GE 的 Context（带更多信息）**：`Source/LyraGame/AbilitySystem/LyraGameplayEffectContext.cpp`；全局类在 `Config/DefaultGame.ini` 里指定。[第 10 章](10-LyraGAS.md)。
- **想改 GameplayCue 的加载策略**：`Source/LyraGame/AbilitySystem/LyraGameplayCueManager.cpp`。[第 10 章](10-LyraGAS.md)。

### 物品、武器、队伍、交互、消息

- **想加一种新物品属性**：新写一个 Fragment，照 `Source/LyraGame/Inventory/InventoryFragment_SetStats.cpp`；定义类在 `Source/LyraGame/Inventory/LyraInventoryItemDefinition.h`。[第 13 章](13-Inventory.md)。
- **想改背包的增删与复制**：`Source/LyraGame/Inventory/LyraInventoryManagerComponent.cpp`。[第 13 章](13-Inventory.md)。
- **想改装备/卸下的流程**：`Source/LyraGame/Equipment/LyraEquipmentManagerComponent.cpp` 与 `Source/LyraGame/Equipment/LyraEquipmentInstance.cpp`；快捷栏在 `Source/LyraGame/Equipment/LyraQuickBarComponent.cpp`。[第 13 章](13-Inventory.md)。
- **想改武器射击的检测与服务器校验**：`Source/LyraGame/Weapons/LyraGameplayAbility_RangedWeapon.cpp`（`PerformLocalTargeting`、`OnTargetDataReadyCallback`）。[第 14 章](14-Weapons.md)。
- **想改散布、后坐力、距离衰减**：`Source/LyraGame/Weapons/LyraRangedWeaponInstance.cpp`（参数在武器实例蓝图 `B_WeaponInstance_*` 里调）。[第 14 章](14-Weapons.md)。
- **想改命中标记**：`Source/LyraGame/Weapons/LyraWeaponStateComponent.cpp`。[第 14 章](14-Weapons.md)。
- **想改拾取台**：`Source/LyraGame/Weapons/LyraWeaponSpawner.cpp`，`GiveWeapon` 是蓝图实现的事件（资产 `/ShooterCore/Blueprint/B_WeaponSpawner`）。[第 13 章](13-Inventory.md)、[第 14 章](14-Weapons.md)。
- **想加一个可交互物**：实现 `Source/LyraGame/Interaction/IInteractableTarget.h`；最快的是继承 `Plugins/GameFeatures/ShooterCore/Source/ShooterCoreRuntime/Public/LyraWorldCollectable.h`。交互技能本身是 `Source/LyraGame/Interaction/Abilities/LyraGameplayAbility_Interact.cpp`。[第 15 章](15-TeamsInteractionMessages.md)。
- **想改“谁能伤害谁”（友军伤害、自由混战）**：`Source/LyraGame/Teams/LyraTeamSubsystem.cpp` 的 `CanCauseDamage`。[第 15 章](15-TeamsInteractionMessages.md)、[第 22 章](22-Capstone.md)。
- **想改分队规则、队伍数量**：`Source/LyraGame/Teams/LyraTeamCreationComponent.cpp`（`ServerChooseTeamForPlayer`、`GetLeastPopulatedTeamID`）；队伍数据是资产，如 `/ShooterCore/Game/B_TeamSetup_TwoTeams`。[第 15 章](15-TeamsInteractionMessages.md)。
- **想改队伍颜色**：`Source/LyraGame/Teams/LyraTeamDisplayAsset.cpp`（`ApplyToActor` 等），数据在 `/Game/System/Teams/TeamDA_*`。[第 15 章](15-TeamsInteractionMessages.md)。
- **想广播或监听一条全局消息**：消息结构 `Source/LyraGame/Messages/LyraVerbMessage.h`；子系统在 `Plugins/GameplayMessageRouter/Source/GameplayMessageRuntime/Public/GameFramework/GameplayMessageSubsystem.h`；想做“消息进、消息出”的观察者，继承 `Source/LyraGame/Messages/GameplayMessageProcessor.h`，样板是 `Plugins/GameFeatures/ShooterCore/Source/ShooterCoreRuntime/Private/MessageProcessors/AssistProcessor.cpp`。[第 15 章](15-TeamsInteractionMessages.md)。
- **想让服务器的消息到达客户端**：`Source/LyraGame/GameModes/LyraGameState.cpp` 的 `MulticastMessageToClients`（只在纯客户端广播）、`Source/LyraGame/Player/LyraPlayerState.cpp` 的 `ClientBroadcastMessage`（单个玩家）。[第 15 章](15-TeamsInteractionMessages.md)。

### 表现层

- **想改相机模式、混合时间**：`Source/LyraGame/Camera/LyraCameraMode.cpp` 与 `Source/LyraGame/Camera/LyraCameraComponent.cpp`；具体模式用 `Source/LyraGame/Camera/LyraCameraMode_ThirdPerson.cpp` 派生，资产如 `/Game/Characters/Cameras/CM_ThirdPerson`。[第 16 章](16-Camera.md)。
- **想让动画读取游戏状态**：`Source/LyraGame/Animation/LyraAnimInstance.cpp`（Tag 到属性的映射）。[第 17 章](17-AnimCosmeticsEffects.md)。
- **想换角色外观部件**：`Source/LyraGame/Cosmetics/LyraControllerComponent_CharacterParts.cpp` 与 `Source/LyraGame/Cosmetics/LyraPawnComponent_CharacterParts.cpp`。[第 17 章](17-AnimCosmeticsEffects.md)。
- **想改脚步声/表面特效**：`Source/LyraGame/Feedback/ContextEffects/AnimNotify_LyraContextEffects.cpp` 与 `Source/LyraGame/Feedback/ContextEffects/LyraContextEffectsSubsystem.cpp`。[第 17 章](17-AnimCosmeticsEffects.md)。
- **想改伤害数字**：`Source/LyraGame/Feedback/NumberPops/LyraNumberPopComponent_NiagaraText.cpp`（当前使用的 Niagara 版）与 `Source/LyraGame/Feedback/NumberPops/LyraNumberPopComponent_MeshText.cpp`（网格文字版）。[第 17 章](17-AnimCosmeticsEffects.md)。
- **想改角色移动手感**：`Source/LyraGame/Character/LyraCharacterMovementComponent.cpp`。[第 17 章](17-AnimCosmeticsEffects.md)。

### UI

- **想加一个 HUD 元素**：不用改 C++；在 Experience/ActionSet 的 `Add Widgets` 里填控件与槽位 Tag。槽位容器是插件 `Plugins/UIExtension/Source/Public/UIExtensionSystem.h`，注入逻辑见 `Source/LyraGame/GameFeatures/GameFeatureAction_AddWidget.cpp`。[第 19 章](19-HUDAndLoading.md)。
- **想加一个菜单屏幕/新的 UI 层**：`Source/LyraGame/UI/LyraActivatableWidget.cpp`（屏幕基类）、`Plugins/CommonGame/Source/Private/PrimaryGameLayout.cpp`（层）、`Plugins/CommonGame/Source/Private/CommonUIExtensions.cpp`（推入层）。[第 18 章](18-UIFramework.md)。
- **想改 UI 根布局如何创建**：`Plugins/CommonGame/Source/Private/GameUIPolicy.cpp` 与 `Plugins/CommonGame/Source/Private/CommonPlayerController.cpp`。[第 18 章](18-UIFramework.md)。
- **想让加载屏多一个阻塞条件，或排查它不消失**：`Plugins/CommonLoadingScreen/Source/CommonLoadingScreen/Private/LoadingScreenManager.cpp`；让自己的对象成为阻塞者，实现 `ILoadingProcessInterface`（Experience 管理组件是现成例子）。[第 19 章](19-HUDAndLoading.md)。
- **想加世界空间的指示器（头顶标记、交互提示）**：`Source/LyraGame/UI/IndicatorSystem/LyraIndicatorManagerComponent.cpp` 与 `Source/LyraGame/UI/IndicatorSystem/IndicatorDescriptor.cpp`。[第 19 章](19-HUDAndLoading.md)。
- **想改前端主菜单流程**：`Source/LyraGame/UI/Frontend/LyraFrontendStateComponent.cpp`（流程骨架）；具体界面是资产，在 `/Game/UI/Menu/`。[第 18 章](18-UIFramework.md)、[第 21 章](21-OnlineAndBuild.md)。

### 设置、在线、构建、调试

- **想加一个画质类设置（机器级）**：`Source/LyraGame/Settings/LyraSettingsLocal.cpp`，界面注册在 `Source/LyraGame/Settings/LyraGameSettingRegistry_Video.cpp`。[第 20 章](20-Settings.md)。
- **想加一个玩法/账号类设置（可云存档）**：`Source/LyraGame/Settings/LyraSettingsShared.cpp`，界面注册在 `Source/LyraGame/Settings/LyraGameSettingRegistry_Gameplay.cpp` 等同目录文件。[第 20 章](20-Settings.md)。
- **想加一个前端玩法入口（UserFacingExperience）**：`Source/LyraGame/GameModes/LyraUserFacingExperienceDefinition.h`；数据放进 `System/Playlists` 目录。[第 21 章](21-OnlineAndBuild.md)。
- **想改开房、加入会话、旅行 URL 的拼接**：`Plugins/CommonUser/Source/CommonUser/Private/CommonSessionSubsystem.cpp`（`ConstructTravelURL`），请求对象见 `LyraUserFacingExperienceDefinition.cpp` 的 `CreateHostingRequest`。[第 21 章](21-OnlineAndBuild.md)。
- **想改登录流程**：`Plugins/CommonUser/Source/CommonUser/Private/CommonUserSubsystem.cpp`。[第 21 章](21-OnlineAndBuild.md)。
- **想改专用服务器的启动逻辑**：`LyraGameMode.cpp` 的 `TryDedicatedServerLogin` 与 `HostDedicatedServerMatch`。[第 21 章](21-OnlineAndBuild.md)。
- **想换构建目标（EOS、Steam、专用服务器）**：`Source/LyraGame.Target.cs`、`Source/LyraServer.Target.cs` 及同目录其他 `*.Target.cs`；各平台的配置叠加在 `Config/Custom/`。[第 21 章](21-OnlineAndBuild.md)。
- **想加模块依赖**：`Source/LyraGame/LyraGame.Build.cs`。[第 21 章](21-OnlineAndBuild.md)。
- **想加原生 GameplayTag**：`Source/LyraGame/LyraGameplayTags.cpp` 与 `.h`；不依赖编译的 Tag 写进 `Config/DefaultGameplayTags.ini` 或 GameFeature 的 `Config/Tags/*.ini`。[第 4 章](04-DataAndAssets.md)。
- **想加碰撞通道**：`Source/LyraGame/Physics/LyraCollisionChannels.h` 与 `Config/DefaultEngine.ini` 必须同步。[第 21 章](21-OnlineAndBuild.md)。
- **想改复制策略（复制图）**：`Source/LyraGame/System/LyraReplicationGraph.cpp`，默认关闭。[第 3 章](03-Networking.md)。
- **想加作弊命令或开发者设置**：`Source/LyraGame/Player/LyraCheatManager.cpp`（`Exec` 函数）、`Source/LyraGame/Development/LyraDeveloperSettings.h`。扩展作弊的写法见 `Source/LyraGame/Development/LyraBotCheats.cpp`。[附录 A1](A1-Debugging.md)。
- **想加日志类别**：`Source/LyraGame/LyraLogChannels.h`。[附录 A1](A1-Debugging.md)。
- **想看性能统计 HUD 的数据来源**：`Source/LyraGame/Performance/LyraPerformanceStatSubsystem.cpp`。[第 19 章](19-HUDAndLoading.md)、[第 20 章](20-Settings.md)。
- **想加热修复/运行时选项**：`Source/LyraGame/Hotfix/LyraHotfixManager.cpp`、`Source/LyraGame/Hotfix/LyraRuntimeOptions.h`。[第 21 章](21-OnlineAndBuild.md)。
- **想批量替换静态网格的材质（编辑器工具）**：`Plugins/LyraExtTool/Source/LyraExtTool/Private/BPFunctionLibrary.cpp`（项目特有的编辑器插件，见 CLAUDE.md）。[第 21 章](21-OnlineAndBuild.md)。

## 第二部分　已知坑清单

这一部分汇总 Lyra 原版里的缺陷、未实现代码和怪癖。每一条都附了位置和一个可行的修法；“可行的修法”是建议，不是官方做法，改核心之前请先评估对其他模式的影响。

### 一、会让功能静默失效的

**1. 服务器不校验命中。** `Source/LyraGame/Weapons/LyraGameplayAbility_RangedWeapon.cpp` 的 `OnTargetDataReadyCallback` 里 `bIsTargetDataValid` 是硬编码的 `true`，旁边的 `bProjectileWeapon` 是个永远为 `false` 的局部变量。服务器接受客户端上报的命中，不重新追踪、不检查射程。这是有意的取舍还是示例的简化，官方没有说明（把它写成“Lyra 的设计取舍”只是推断）；Epic 论坛上的社区讨论把 Lyra 描述为客户端权威的命中判定，并转述 Epic 的视频里提到过，本书没有核实原视频。无论动机如何，别把它当成能防作弊的实现。修法：在这里对每个 `FHitResult` 检查起点离 Pawn 的距离、射程不超过 `MaxDamageRange`、必要时在服务器重新追踪，失败时置 `bIsTargetDataValid = false`。（[第 14 章](14-Weapons.md)）

**2. `SpreadRecoveryCooldownDelay` 不生效。** `Source/LyraGame/Weapons/LyraRangedWeaponInstance.cpp` 的冷却用 `TimeSince(LastFireTime)`，但 `LastFireTime` 整个工程里只有声明和读取，从未赋值，永远是 0。修法：在 `AddSpread` 里写 `LastFireTime = GetWorld()->GetTimeSeconds()`。注意 `ULyraWeaponInstance::UpdateFiringTime` 写的是另一个变量。（[第 14 章](14-Weapons.md)）

**3. `FLyraInventoryList::AddEntry(ULyraInventoryItemInstance*)` 是 `unimplemented()`。** `Source/LyraGame/Inventory/LyraInventoryManagerComponent.cpp` 里，`AddItemInstance` 以及 `IPickupable` 里 `FPickupInstance` 那条路径都会走到它，会直接断言或崩溃。现在只有“按物品定义添加”的路径能用。修法：把实例放进 `Entries`、`MarkItemDirty`，并注册复制子对象。（[第 13 章](13-Inventory.md)）

**4. `ULyraAbilityCost_InventoryItem` 是空壳。** `Source/LyraGame/AbilitySystem/Abilities/LyraAbilityCost_InventoryItem.cpp` 的实现被 `#if 0` 注释掉，`CheckCost` 恒返回 false，所以用了它的技能永远无法激活。修法：别用它；要按物品扣费，用 `ULyraAbilityCost_ItemTagStack`。（[第 10 章](10-LyraGAS.md)）

**5. 追加的输入配置不能卸载。** `Source/LyraGame/Character/LyraHeroComponent.cpp` 的 `RemoveAdditionalInputConfig` 只有一句 `@TODO: Implement me!`，所以 `Add Input Binding` 这个 Action 停用后绑定会残留。修法：在 `AddAdditionalInputConfig` 里保存绑定句柄（按配置资产索引），在 Remove 里用 `RemoveBinds`。（[第 11 章](11-Input.md)）

**6. 配置文件的节名拼错。** `Config/DefaultRuntimeOptions.ini` 的节是 `[/Script/LyraGame.LyraRuntimeOptons]`，而类名是 `ULyraRuntimeOptions`，所以里面的配置不会被读取（文件内容目前只是被注释的示例，影响很小）。修法：补上缺的 `i`。（[第 4 章](04-DataAndAssets.md)、[第 20 章](20-Settings.md)、[第 21 章](21-OnlineAndBuild.md)）

**7. 裸 GameplayAbility 不能绑定输入。** `Source/LyraGame/GameFeatures/GameFeatureAction_AddAbilities.h` 里 `FLyraAbilityGrant` 的 InputAction 字段被注释掉了，创建的 Spec 也没有 InputTag；想让技能响应输入必须走 `GrantedAbilitySets`。该 Action 还用同步加载。（[第 10 章](10-LyraGAS.md)）

**8. 只有 `bRegisterWithSettings` 为真的 IMC 才会生效。** `LyraHeroComponent.cpp` 的 `InitializePlayerInput` 里，`AddMappingContext` 写在 `if (Mapping.bRegisterWithSettings)` 里面；勾掉这个选项，该 IMC 既不注册也不添加。并且整个循环还要求 `PawnData->InputConfig` 非空。开头还有一次 `ClearAllMappings()`。（[第 11 章](11-Input.md)）

### 二、会让流程崩溃或卡住的

**9. Experience 加载失败是断言，而不是错误状态。** `Source/LyraGame/GameModes/LyraExperienceManagerComponent.cpp` 的 `SetCurrentExperience` 对蓝图类加载失败直接 `check`，并且 `check(CurrentExperience == nullptr)` 让它不可重复调用；文件头的 `@TODO` 也承认“应进入‘完成但失败’的状态，而不是 check”。修法：改成返回失败并让 GameMode 回退。（[第 6 章](06-Experience.md)）

**10. GameFeature 激活失败被吞掉。** 同文件的 `OnGameFeaturePluginLoadComplete` 只做计数，不看 `Result`；插件激活失败时流程照样进入 `Loaded`，玩家进游戏后缺东西。修法：在这里检查 `Result` 并记录/终止。排查时看 `LogGameFeatures`。（[第 6 章](06-Experience.md)）

**11. `GetCurrentExperienceChecked()` 未加载时崩溃。** 它 `check(LoadState == Loaded)`。依赖 Experience 的逻辑一律用 `CallOrRegister_OnExperienceLoaded*`。（[第 6 章](06-Experience.md)）

**12. Experience 的卸载没做完。** 文件头 `@TODO` 写着“支持停用并卸载”“停用 GameFeature 时目前会‘泄漏’它们仍处于启用状态”（`EndPlay` 里其实已有 `DeactivateGameFeaturePlugin` 的调用，注释可能已过时，以实测为准）；`EndPlay` 里还有“异步停用尚未完全支持”的 Error 日志。每局重新创建 GameState 时影响不大，做“局内切换玩法”时要特别留意。（[第 6 章](06-Experience.md)）

**13. `ULyraTeamCreationComponent` 对非法队伍号 `check`。** `Source/LyraGame/Teams/LyraTeamCreationComponent.cpp` 的 `GetLeastPopulatedTeamID` 里 `check(TeamMemberCounts.Contains(PlayerTeamID))`，把玩家设到不在 `TeamsToCreate` 里的队伍会崩。作弊命令 `SetTeam` 只检查队伍是否存在，不受此限，但自定义分队逻辑要小心。（[第 15 章](15-TeamsInteractionMessages.md)）

**14. 加载屏相关的两个“永远”。** 没有 GameState 或没有本地 PlayerController 时加载屏一直显示；`ULyraFrontendStateComponent` 的主界面类加载异常时同样可能一直显示。排查用 `CommonLoadingScreen.LogLoadingScreenReasonEveryFrame`。（[第 19 章](19-HUDAndLoading.md)）

**15. 交互相关的两处硬失败。** `Source/LyraGame/Interaction/InteractionStatics.cpp` 的 `GetActorFromInteractableTarget` 对既不是 Actor 也不是组件的对象 `unimplemented()`；`AbilityTask_WaitForInteractableTargets` 的 `AimWithPlayerController` 里 `check(PC)`，用在机器人身上会崩（源码注释 `@TODO: Bots?`）。（[第 15 章](15-TeamsInteractionMessages.md)）

**16. Niagara 伤害数字的 Style 空指针。** `Source/LyraGame/Feedback/NumberPops/LyraNumberPopComponent_NiagaraText.cpp` 的 `AddNumberPop` 对 `Style` 只在创建组件时判空，之后读取 `Style->NiagaraArrayName` 没有判空，所以必须在蓝图里配置 `Style`。（[第 17 章](17-AnimCosmeticsEffects.md)）

### 三、不是 bug，但很容易误会的设计

**17. “没有队伍就打不掉血”，且没有友军伤害开关。** `Source/LyraGame/Teams/LyraTeamSubsystem.cpp` 的 `CanCauseDamage` 只在 `DifferentTeams` 时放行；攻击者没有队伍则一律拒绝；注释提到“友军伤害设置”，但代码里没有这项设置。（[第 15 章](15-TeamsInteractionMessages.md)、[第 22 章](22-Capstone.md)）

**18. `ControllerCanRestart` 的口子没开。** `ULyraPlayerSpawningManagerComponent::ControllerCanRestart` 是 private、非虚的 TODO，只通过 `friend class ALyraGameMode` 暴露，注释却说“让每个 Experience 自定义重生系统”。想真正拦截出生，要改成 `protected virtual`。（[第 8 章](08-PawnAssembly.md)）

**19. 复活不是 GameMode 做的。** 团队死斗里的复活来自 ShooterCore 里的蓝图被动技能 `GA_AutoRespawn`（放在 `AbilitySet_Elimination` 里），倒计时后调用 `ALyraGameMode::RequestPlayerRestartNextFrame`；C++ 里没有任何地方主动调用该函数，不授予这个技能，死者就不会再出现。（[第 12 章](12-DamageAndDeath.md)、[第 22 章](22-Capstone.md)）

**20. `Controller->Reset()` 不会调用 `ALyraCharacter::Reset`。** `RequestPlayerRestartNextFrame(…, bForceReset=true)` 里调的 `Controller->Reset()`，在引擎里（`APlayerController::Reset`）只做 `PawnPendingDestroy` 和 `UnPossess`，再切到旁观状态，不会去调 Pawn 的 `Reset()`。`ALyraCharacter::Reset` 只由强制重置技能 `ULyraGameplayAbility_Reset`（`GameplayEvent.RequestReset`）触发。参考手册 10 §8.5 里“Controller->Reset 会调 Character::Reset”的写法是错的，以引擎源码为准。（[第 12 章](12-DamageAndDeath.md)）

**21. 默认使用 OSSv1 不是缺陷。** `Plugins/CommonUser/Source/CommonUser/CommonUser.Build.cs` 里的局部变量 `bUseOnlineSubsystemV1 = true` 只是默认选择：它决定编译 OSSv1 还是 OSSv2 的依赖，并生成宏 `COMMONUSER_OSSV1`。Epic 的 EOS 教程就是把它改成 false 来切换到 OSSv2（并配置 `bUseOnlineServicesV2=true`、禁用 OSS EOS，见官方教程）。要提醒的只有一点：`CommonSessionSubsystem.cpp`、`CommonUserSubsystem.cpp` 里的 OSSv2 分支还有 `TODO`（比如“Bind OSSv2 delegates when they are available”），切换后要整套重新测试。（[第 21 章](21-OnlineAndBuild.md)）

**22. 每次 Pawn 初始化都会把血量回满。** `Source/LyraGame/Character/LyraHealthComponent.cpp` 的 `InitializeWithAbilitySystem` 里有一行标着 `TEMP` 的 `SetNumericAttributeBase(Health, MaxHealth)`，所以“ASC 在 PlayerState 上”并不意味着血量跨重生持久。修法：想持久血量就改这一行并另存数据。（[第 12 章](12-DamageAndDeath.md)）

**23. 淘汰消息的 Context 没填。** `HandleOutOfHealth` 和 `LyraHealthSet.cpp` 广播 `Lyra.Elimination.Message` / `Lyra.Damage.Message` 时，`ContextTags` 没有填，“是否敌方击杀、自杀、队友击杀”也没判断（都是 `@TODO`）。想区分要自己比较 Instigator 与 Target 的队伍。（[第 12 章](12-DamageAndDeath.md)）

**24. PawnData 只能设一次，且存两份。** `ALyraPlayerState::SetPawnData` 已有 PawnData 时报错返回；PlayerState 上一份、Pawn 的 PawnExt 上一份，运行时换角色没有现成 API。（[第 8 章](08-PawnAssembly.md)）

**25. `GameState` 的多播消息只在纯客户端广播。** `Source/LyraGame/GameModes/LyraGameState.cpp` 的 `MulticastMessageToClients_Implementation` 只在 `NM_Client` 时广播到本地消息子系统，监听服务器的主机收不到；它还是 Unreliable。需要状态用复制属性。（[第 15 章](15-TeamsInteractionMessages.md)）

**26. 初始装备写在 Pawn 蓝图里。** 所有 Shooter 模式的起手手枪来自 `B_Hero_ShooterMannequin` 的变量，不是 Experience；只改 PawnData 不会换武器。（[第 13 章](13-Inventory.md)）

### 四、武器、物品、交互里的小问题

**27. 子弹从相机出发，不是枪口。** `PerformLocalTargeting` 固定使用 `CameraTowardsFocus`；`GetWeaponTargetingSourceLocation` 只返回 Pawn 位置，有 `@TODO`。贴墙射击会穿过枪管。（[第 14 章](14-Weapons.md)）

**28. 命中标记的 `UniqueId` 类型与取值不匹配。** `Source/LyraGame/Weapons/LyraWeaponStateComponent.h` 里批次的 `UniqueId` 是 `uint8`，RPC 参数是 `uint16`，取值是“未确认批次数量”而不是单调序号。高射速加乱序确认时可能重复，只影响 UI 标记。（[第 14 章](14-Weapons.md)）

**29. 武器 Tick 只作用于第一把远程武器。** `ULyraWeaponStateComponent::TickComponent` 只取 `GetFirstInstanceOfType`；同时装备多把时只有第一把更新散布。（[第 14 章](14-Weapons.md)）

**30. 物品没有真正的堆叠，计数是按条目算。** `CanAddItemDefinition` 恒返回 true（`@TODO`）；`GetTotalItemCountByDefinition` 数的是条目而不是 `StackCount`；`ConsumeItemsByDefinition` 经 `InventoryList.RemoveEntry` 逐条移除，不像 `RemoveItemInstance` 那样注销复制子对象。弹药用的是物品实例上的 TagStack。（[第 13 章](13-Inventory.md)）

**31. 背包的变更消息只在复制回调里广播。** `Source/LyraGame/Inventory/LyraInventoryManagerComponent.cpp` 里 `BroadcastChangeMessage` 只被 `PreReplicatedRemove`、`PostReplicatedAdd`、`PostReplicatedChange` 调用，`AddEntry` / `RemoveEntry` 本身不广播。纯客户端靠复制回调能收到，但监听服务器的主机和单机（权威端，没有复制回调）收不到，订阅这条消息的 UI 会不刷新（社区作者 Garashka 报告，源码核对属实）。修法：在权威端的 `AddEntry` / `RemoveEntry` 里手动补广播。（[第 13 章](13-Inventory.md)）

**32. 装备的 Instigator 晚于 `OnEquipped`。** `EquipItem` 先调 `OnEquipped()`，`ULyraQuickBarComponent` 之后才 `SetInstigator`，所以在 `OnEquipped` 里读物品信息拿到的是空。需要物品信息的逻辑放进技能（`GetAssociatedItem()`）或延后。（[第 13 章](13-Inventory.md)）

**33. 快捷栏移除激活槽不触发 OnRep。** `RemoveItemFromSlot` 把 `ActiveSlotIndex` 置 -1 但不调 `OnRep_ActiveSlotIndex`，UI 可能不刷新。（[第 13 章](13-Inventory.md)）

**34. `DestroyEquipmentActors` 不清数组。** `ULyraEquipmentInstance::DestroyEquipmentActors` 只循环 `Destroy`，`SpawnedActors` 数组保留着已销毁的引用。（[第 13 章](13-Inventory.md)）

**35. 拾取台冷却值没有显式默认。** `ULyraPickupDefinition::SpawnCoolDownSeconds` 是没写初始值的 `int32`（UObject 成员会被引擎清零，所以实际是 0），新建拾取资产忘了填，武器会几乎立刻重生。（[第 13 章](13-Inventory.md)）

**36. 只发不收的交互技能。** `UAbilityTask_GrantNearbyInteraction` 只授予不回收；`IInteractionInstigator` 全工程无调用者，`TriggerInteraction` 固定取 `CurrentOptions[0]`。（[第 15 章](15-TeamsInteractionMessages.md)）

**37. `ALyraTeamPrivateInfo` 并不私有。** 构造函数里只有一句 `@TODO: Actually make private`，队伍信息 Actor 都是 `bAlwaysRelevant`。（[第 15 章](15-TeamsInteractionMessages.md)）

### 五、相机、UI、GAS 细节

**38. 相机 `GetBlendInfo` 读的是栈底。** `Source/LyraGame/Camera/LyraCameraMode.cpp` 里 `GetBlendInfo` 把 `CameraModeStack.Last()` 当成 `TopEntry`，而新模式是插到索引 0 的；变量名和注释都与实现相反。武器散布的瞄准系数（`LyraRangedWeaponInstance::UpdateMultipliers`）用的就是这个函数，所以过渡期间它读到的并不反映正在渐入的新模式。（[第 16 章](16-Camera.md)）

**39. 相机栈只有 HeroComponent 驱动。** 没有 HeroComponent 的 Pawn（简单 AI、旁观者）相机栈为空；旁观系统在原版里不存在。（[第 16 章](16-Camera.md)）

**40. `AddWidgets` 没有去重。** `Source/LyraGame/GameFeatures/GameFeatureAction_AddWidget.cpp` 的 `HandleActorExtension` 对 `ExtensionAdded` 和 `GameActorReady` 两个事件都调用 `AddWidgets`，而 `AddWidgets` 直接往该 HUD 的 `LayoutsAdded` 和 `ExtensionHandles` 里追加，不检查是否已经加过。引擎的 `AddExtensionHandler` 会对已经初始化（`IsActorInitialized()`）的现有接收者立即回调 `ExtensionAdded`，所以同一个处理器理论上可能先后收到两个事件，导致布局被推两次、控件被注册两次。这是读源码得出的推断，没有在 PIE 里复现；正常流程里 HUD 往往在处理器注册之后才初始化，不会重复。自己写的 Action 应让处理器幂等。（[第 7 章](07-ModularGameplay.md)、[第 19 章](19-HUDAndLoading.md)）

**41. `ULyraTaggedWidget` 是半成品。** `Source/LyraGame/UI/LyraTaggedWidget.cpp` 里按 Tag 隐藏的判断全是 `@TODO`（`bHasHiddenTags` 被写死为 false，监听 Tag 变化的几处也只有 `@TODO` 占位），也就是说 `HiddenByTags` 这个属性目前不起作用，别指望用它按 Tag 隐藏控件。（[第 18 章](18-UIFramework.md)）

**42. `ULyraPlayerMappableKeySettings::GetTooltipText()` 只声明未定义。** `Source/LyraGame/Input/LyraInputUserSettings.h` 里声明了，全工程没有实现；C++ 调用会链接失败（目前没有调用者）。（[第 11 章](11-Input.md)）

**43. GE Context 的两个字段不复制。** `Source/LyraGame/AbilitySystem/LyraGameplayEffectContext.cpp` 的 `NetSerialize` 只调基类，`CartridgeID` 与 `AbilitySourceObject` 都不序列化；并且文件里注明，若以后序列化新字段，必须为 Iris 写自定义 NetSerializer。（[第 10 章](10-LyraGAS.md)）

**44. `ProcessAbilityInput` 用静态数组。** 局部 `static TArray<FGameplayAbilitySpecHandle> AbilitiesToActivate` 省分配，但不可重入；不要在技能激活流程里再调它。（[第 11 章](11-Input.md)）

**45. GameplayCue 加载模式不是 CVar，且未加载的 Cue 走异步挂起补播。** 加载模式是文件内的静态变量 `LoadUpfront`（非 CVar）；`ShouldSyncLoadMissingGameplayCues` 返回 false、`ShouldAsyncLoadMissingGameplayCues` 返回 true，因此尚未加载的 Cue 首次触发时会被引擎挂起（Deferred），等异步加载完成后再补播（表现为第一次可能晚半拍；只有挂起期间目标 Actor 已销毁时才会跳过，详见 [附录 A1](A1-Debugging.md) 症状 11）。（[第 10 章](10-LyraGAS.md)）

**46. 输入追加映射里的遗留 TODO。** `Source/LyraGame/GameFeatures/GameFeatureAction_AddInputContextMapping.cpp` 里有一条注释指出 `ControllersAddedTo` 从未被写入，却和 LocalPlayer 混用。（[第 7 章](07-ModularGameplay.md)）

**47. 显著性管理器没接上。** `ALyraCharacter::BeginPlay` 里注册 `ULyraSignificanceManager` 的那行是 `@TODO` 注释，`ULyraSignificanceManager` 本身是空子类。（[第 17 章](17-AnimCosmeticsEffects.md)）

**48. 伤害数字的旁观判断是常量。** `Source/LyraGame/Feedback/NumberPops/LyraNumberPopComponent_MeshText.cpp` 里 `bIsSpectating` 被写死为 false，并有 `@TODO: Determine whether or not we are spectating`。（[第 17 章](17-AnimCosmeticsEffects.md)）

### 六、配置、构建与安全

**49. 不少类没有导出宏，别的模块链接不上。** 下面这几个类在头文件里是裸的 `UCLASS(...)`，既没有 `LYRAGAME_API`/`UE_API`，也没有 `MinimalAPI`：`ULyraQuickBarComponent`（`Equipment/LyraQuickBarComponent.h`）、`ULyraInventoryItemInstance`（`Inventory/LyraInventoryItemInstance.h`）、`UGameFeatureAction_WorldActionBase`（`GameFeatures/GameFeatureAction_WorldActionBase.h`）、`ULyraGamePhaseSubsystem` 与 `ULyraGamePhaseAbility`（`AbilitySystem/Phases/`）。按 UE 的规则，其他模块（比如你的 GameFeature 插件）直接调用它们的非内联函数、或继承它们，会遇到链接错误（未解析的外部符号）；ShooterCore 的相应逻辑是蓝图，所以没有暴露这个问题。这是按链接规则做的推断，没有逐个实际编译验证。全工程这类没有导出的 `UCLASS` 还有很多（`ULyraGlobalAbilitySystem`、`ULyraTeamCreationComponent`、`ULyraWeaponStateComponent`、`ULyraSystemStatics` 等）；做插件时的办法是：优先用蓝图接口（`BlueprintCallable` 的函数在蓝图里照样能调）、或在需要的类上补 `LYRAGAME_API`（改核心，要评估上游合并）。（[第 22 章](22-Capstone.md)）

**50. 复制图默认关闭。** `ULyraReplicationGraphSettings::bDisableReplicationGraph` 默认为 true，`DefaultGame.ini` 里也显式写了 True，`FastSharedReplication` 只在复制图路径驱动。要启用需改配置。（[第 3 章](03-Networking.md)）

**51. 调试加密密钥是硬编码的。** `Source/LyraGame/System/LyraGameInstance.cpp` 的 `Init` 里把 32 字节密钥初始化成 `0..31`，注释写明“不安全，仅作示例”。别用于正式项目。（[第 21 章](21-OnlineAndBuild.md)）

**52. `Cheat` 的 Server RPC 校验恒真。** `ALyraPlayerController::ServerCheat_Validate` 与 `ServerCheatAll_Validate` 都直接返回 true，意味着任何客户端能让服务器执行任意控制台命令；它们被 `USING_CHEAT_MANAGER`（非 Shipping）包住，所以只在开发构建里存在。别把开发构建暴露给不可信玩家。（[附录 A1](A1-Debugging.md)）

**53. 开发者设置只在编辑器里生效。** `ExperienceOverride` 需要 `IsPlayInEditor()`，机器人数量覆盖需要 `GIsEditor`，`CheatsToRun` 需要 `WITH_EDITOR`。这是 PIE 与打包后行为不一致的常见来源。（[附录 A1](A1-Debugging.md)）

**54. `ShooterCoreRuntimeSettings` 的默认值与 ini 不同。** 头文件里 `AimAssistCollisionChannel` 默认 `ECC_EngineTraceChannel5`，`DefaultGame.ini` 里覆盖成 `ECC_GameTraceChannel5`，读代码时别被头文件默认值误导。（[第 14 章](14-Weapons.md)）

### 七、拼写（不要“纠正”）

下面这些拼写都是原样保留在源码、Tag 和资产里的，改了会和现有资产对不上：`UneuippedAnimSet`（`LyraWeaponInstance.h`，应为 Unequipped）；`UTDM_PlayerSpawningManagmentComponent`（缺一个 e）；资产 `EAS_BasicShooterAcolades`（缺一个 c）；Tag `Platform.Trait.Input.PrimarlyController`（Primarily 少一个 i）；配置节 `LyraRuntimeOptons`（见本部分第一类里“配置文件的节名拼错”一条，这个是真的该改的）。

## 小结

这份地图想让你记住的只有一个习惯：**先用问题定位到系统，再用系统定位到文件**。Lyra 里绝大多数“玩法差异”是数据（Experience、PawnData、AbilitySet、GE、Widget 与槽位），文件在资产里；“引擎性”的行为（出生流程、输入路由、伤害管线、复制）才在 `Source/LyraGame`。而第二部分的清单提醒你：这套代码是一个可运行的参考实现，不是一份“已经做完”的产品，凡是 `@TODO` 的地方，都假设它没有做。

## 继续深入

- 每份参考手册的第 7 节是该模块的“坑”清单：[10](../Reference/10-CoreFramework.md)、[20](../Reference/20-AbilitySystem-Input.md)、[30](../Reference/30-GameplaySystems.md)、[40](../Reference/40-UI.md)、[50](../Reference/50-Settings-Online-Platform.md)、[60](../Reference/60-Presentation.md)、[70](../Reference/70-GameFeatures-Content.md)、[80](../Reference/80-Build-Config.md)；
- 核验报告：[参考手册 95](../Reference/95-Verification-Report.md)；
- 排查方法：[附录 A1](A1-Debugging.md)；术语：[附录 A2](A2-Glossary.md)；一个把这些拼起来的综合项目：[第 22 章](22-Capstone.md)。
